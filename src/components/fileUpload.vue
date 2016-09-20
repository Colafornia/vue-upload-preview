<template>
  <form role="form" class="file-upload-form" enctype="multipart/form-data" encoding="multipart/form-data" autocomplete="off">
    <div id="upload-area" class="upload-area" @dragover.prevent @drop.prevent="onFileChange">
      <input type="file" :id="$parent.id" class="hide" @change="onFileChange" :multiple="$parent.multiple" :store="$parent.store">
    </div>
  </form>
</template>
<script>
export default {
  data() {
    return {
      files: [],
      historyList: [],
      storeList: []
    }
  },
  props: {
    multiple: {
      type: Boolean,
      default: false
    },
    id: {
      type: String,
      default: 'file'
    },
    dropable: {
      type: Boolean,
      default: false
    },
    sizeLimit: {
      type: Number
    },
    store: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    onFileChange (e) {
      const files = e.target.files || e.dataTransfer.files
      for (let i = 0; i < files.length; i++) this.previewFile(files[i])
      /*document.getElementById('upload-pic-input').value = ''*/
      this.$destroy()
    },
    previewFile (file) {
      const fileSize = (file.size / (1024 * 1024)).toFixed(2)
      if (this.sizeLimit && this.sizeLimit < fileSize) return
      const reader = new FileReader()
      reader.addEventListener('load', (e) => {
        const imageSrc = e.target.result
        if (file.type.indexOf('image') !== -1) {
          const image = new Image()
          image.addEventListener('load', () => {
            const imageInfo = {
              file: file,
              src: imageSrc,
              name: file.name,
              original: {
                name: file.name,
                width: image.width,
                height: image.height
              },
              height: image.height,
              width: image.width,
              size: fileSize,
              isCompress: true,
              showPre: false,
              showNameInput: false,
              showDimensionInput: false,
              showSizePopover: false,
              isUploaded: false
            }
            this.addFileItem(imageInfo)
          })
          image.src = reader.result
        } else {
          /*this.alert = {
            show: true,
            type: 'warning',
            msg: '注意：你选择的是非图片文件'
          }*/
          const fileInfo = {
            file: file,
            name: file.name,
            original: {
              name: file.name
            },
            size: fileSize,
            showNameInput: false,
            isUploaded: false
          }
          this.addFileItem(fileInfo)
        }
      })
      reader.readAsDataURL(file)
    },
    addFileItem (file) {
      this.files.push(file)
      console.log(this.files)
    },
    removeFileItem (index) {
      this.fileList.splice(index, 1)
    },
    uploadFileList (fileList) {
      let fileHaveUploadedNum = 0
      let fileToUploadNum = file
      for (let i = 0; i < file; i++) {
        if (!this.fileList[i].isUploaded) {
          this.progressBar.show = true
          const uploadFile = fileList[i].file
          const uploadInfo = Object.assign({}, this.fileList[i])
          if (uploadInfo.src) uploadInfo.src = undefined
          uploadInfo.file = undefined
          uploadInfo.isNameChanged = uploadInfo.name !== uploadInfo.original.name
          const formData = new FormData()
          formData.append('imageInfo', JSON.stringify(uploadInfo))
          formData.append('imageFile', uploadFile)
          $.ajax({
            url: '/god/upload_pic',
            type: 'POST',
            data: formData,
            processData: false,
            contentType: false
          })
            .done((url) => {
              fileHaveUploadedNum = fileHaveUploadedNum + 1
              this.progressBar.number = (fileHaveUploadedNum * 100 / fileToUploadNum).toFixed(2) + '%'
              Vue.set(this.fileList[i], 'url', url)
              this.fileList[i].isUploaded = true
              if (fileHaveUploadedNum === fileToUploadNum) this.progressBar.show = false
              this.storeUploadedHistroy(this.fileList[i])
            })
            .fail((e) => {
              fileToUploadNum = fileToUploadNum - 1
              this.progressBar.show = false
              try {
                // 避免当返回html报错信息时，JSON.parse操作会引发Unexpected token 错误
                const errorObj = JSON.parse(e.responseText)
                this.alert = {
                  show: true,
                  type: 'danger',
                  msg: errorObj.msg
                }
              } catch (e) {
                this.alert = {
                  show: true,
                  type: 'danger',
                  msg: '上传失败！'
                }
              }
            })
        } else {
          fileToUploadNum = fileToUploadNum - 1
        }
      }
    },
    storeUploadedHistroy (file) {
      // localStorage里面不存file，预览用线上链接
      const storeItem = Object.assign({}, file)
      if (file.src) storeItem.src = file.url
      storeItem.file = undefined
      this.storeList.unshift(storeItem)
      if (this.store > 20) this.store = 20
      localStorage.setItem('godUploadHistory', JSON.stringify(this.storeList))
    },
    copyUrl (url, index) {
      this.alert.show = false
      const textArea = document.createElement('textarea')
      textArea.value = url
      document.body.appendChild(textArea)
      textArea.select()
      // 针对不支持 document.execCommand 的旧版浏览器
      try {
        const isCopied = document.execCommand('copy')
        if (isCopied) {
          this.alert = {
            show: true,
            type: 'success',
            msg: '上传文件链接已经复制到剪贴板'
          }
        }
      } catch (e) {
        this.alert = {
          show: true,
          type: 'danger',
          msg: '链接复制失败，请手动复制' + url
        }
      }
    }
  },
  ready () {
    const historyData = localStorage.getItem('godUploadHistory')
    if (!historyData) return
    this.historyList = JSON.parse(historyData)
    this.storeList = this.historyList.slice()
  }
}
</script>
<style scoped>
.upload-area {
  height: 100px;
  width: 100px;
  background-color: #grey;
}
</style>

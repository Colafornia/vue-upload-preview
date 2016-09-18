<template>
  <form role="form" method="POST" class="file-upload-form form-horizontal" action="/god/upload_pic" enctype="multipart/form-data" encoding="multipart/form-data" autocomplete="off">
    <div id="upload-area" class="upload-area" @dragover.prevent @drop.prevent="onFileChange" @dragenter="hovering = true"
         @dragleave="hovering = false" :class="{'hovered': hovering}">
      <input type="file" id="upload-file-input" class="form-control hide" @change="onFileChange" multiple>
      <button type="button" class="btn btn-default upload-img-btn orange-bg" @click.prevent="uploadFileList(fileList)" v-show="fileList.length && toUploadNum">上传已选择的图片</button>
    </div>
  </form>
</template>
<script>
export default {
  el: '#file-upload',
  data: {
    hovering: false,
    fileList: [],
    historyList: [],
    storeList: []
  },
  methods: {
    addFile () {
      document.getElementById('upload-file-input').click()
    },
    onFileChange (e) {
      const files = e.target.files || e.dataTransfer.files
      for (let i = 0; i < files.length; i++) this.previewFile(files[i])
      document.getElementById('upload-pic-input').value = ''
    },
    previewFile (file) {
      const fileSize = (file.size / (1024 * 1024)).toFixed(2)
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
          this.alert = {
            show: true,
            type: 'warning',
            msg: '注意：你选择的是非图片文件'
          }
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
      this.fileList.push(file)
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

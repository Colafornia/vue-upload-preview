<template>
  <form role="form" class="file-upload-form" enctype="multipart/form-data" encoding="multipart/form-data" autocomplete="off">
    <div id="upload-area" class="upload-area" @dragover.prevent @drop.prevent="_onFileChange" @click="_triggerUpload">
      <input id="upload-input" type="file" class="hide" @change="_onFileChange" :multiple="$parent.multiple" :store="$parent.store">
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
    uploadUrl: {
      type: String
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
    _uploadEvents(name, file) {
      this.$dispatch && this.$dispatch(name, file);
    },

    _triggerUpload(){
      document.getElementById('upload-input').click()
    },

    _onFileChange (e) {
      const files = e.target.files || e.dataTransfer.files
      for (let i = 0; i < files.length; i++) this._previewFile(files[i])
      this.$destroy()
    },

    _previewFile (file) {
      const fileSize = (file.size / (1024 * 1024)).toFixed(2)
      if (this.sizeLimit && this.sizeLimit < fileSize) return
      const reader = new FileReader()
      reader.addEventListener('load', (e) => {
        const imageSrc = e.target.result
        if (file.type.indexOf('image') !== -1) {
          // 图片文件
          const image = new Image()
          image.addEventListener('load', () => {
            const imageInfo = {
              file: file,
              src: imageSrc,
              name: file.name,
              height: image.height,
              width: image.width,
              size: fileSize,
              isUploaded: false
            }
            this._addFileItem(imageInfo)
          })
          image.src = reader.result
        } else {
          // 非图片文件
          const fileInfo = {
            file: file,
            name: file.name,
            size: fileSize,
            isUploaded: false
          }
          this._addFileItem(fileInfo)
        }
      })
      reader.readAsDataURL(file)
    },

    _addFileItem (file) {
      this.files.push(file)
      console.log(this.files)
      this._uploadEvents('filesToUpload', this.files);
    },

    _removeFileItem (index) {
      this.fileList.splice(index, 1)
    },

    _formatFileList (fileList) {
      for (let i = 0; i < file; i++) {
        if (!this.fileList[i].isUploaded) {
          const uploadFile = fileList[i].file
          const uploadInfo = Object.assign({}, this.fileList[i])
          if (uploadInfo.src) uploadInfo.src = undefined
          uploadInfo.file = undefined
          const formData = new FormData()
          formData.append('imageInfo', JSON.stringify(uploadInfo))
          formData.append('imageFile', uploadFile)
          this._uploadFileList(formData)
        }
      }
    },

    _uploadFileList (formData) {
      if (!this.uploadUrl) return
      const url = this.uploadUrl
      $.ajax({
        url: this.uploadUrl,
        type: 'POST',
        data: formData,
        processData: false,
        contentType: false
      })
        .done((url) => {
          Vue.set(this.fileList[i], 'url', url)
          this.fileList[i].isUploaded = true
          if (this.store) this.storeUploadedHistroy(this.fileList[i])
        })
        .fail((e) => {
          const errorObj = JSON.parse(e.responseText)
          this._uploadEvents('throwError', errorObj.msg);
        })
    },

    _storeUploadedHistroy (file) {
      const storeItem = Object.assign({}, file)
      if (file.src) storeItem.src = file.url
      storeItem.file = undefined
      this.storeList.unshift(storeItem)
      if (this.store > 20) this.store = 20
      localStorage.setItem('uploadHistory', JSON.stringify(this.storeList))
    }
  },
  ready () {
    const historyData = localStorage.getItem('uploadHistory')
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

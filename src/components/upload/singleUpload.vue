<template>
  <div>
    <el-upload
      :before-upload="beforeUpload"
      :data="dataObj"
      :file-list="fileList"
      :multiple="false" :on-preview="handlePreview"
      :on-remove="handleRemove"
      :on-success="handleUploadSuccess"
      :show-file-list="showFileList"
      action="http://localhost:9100/mall-hello"
      list-type="picture">
      <el-button size="small" type="primary">点击上传</el-button>
      <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过10MB</div>
    </el-upload>
    <el-dialog :visible.sync="dialogVisible">
      <img :src="fileList[0].url" alt="" width="100%">
    </el-dialog>
  </div>
</template>
<script>
import {policy} from './policy'

const minioUrl = 'http://127.0.0.1:9100/'

export default {
  name: 'singleUpload',
  props: {
    value: String
  },
  computed: {
    imageUrl () {
      return this.value
    },
    imageName () {
      if (this.value != null && this.value !== '') {
        return this.value.substr(this.value.lastIndexOf('/') + 1)
      } else {
        return null
      }
    },
    fileList () {
      return [{
        name: this.imageName,
        url: this.imageUrl
      }]
    },
    showFileList: {
      get: function () {
        return this.value !== null && this.value !== '' && this.value !== undefined
      },
      set: function (newValue) {
      }
    }
  },
  data () {
    return {
      dataObj: {
        'x-emz-date': '',
        'x-amz-signature': '',
        'x-amz-algorithm': '',
        'x-amz-credential': '',
        'policy': '',
        'key': '',
        'bucket': ''
        // callback:'',
      },
      dialogVisible: false
    }
  },
  methods: {
    emitInput (val) {
      this.$emit('input', val)
    },
    handleRemove (file, fileList) {
      this.emitInput('')
    },
    handlePreview (file) {
      this.dialogVisible = true
    },
    beforeUpload (file) {
      return new Promise((resolve, reject) => {
        policy(file).then(response => {
          this.dataObj['x-amz-date'] = response.data['x-amz-date']
          this.dataObj['x-amz-signature'] = response.data['x-amz-signature']
          this.dataObj['x-amz-algorithm'] = response.data['x-amz-algorithm']
          this.dataObj['x-amz-credential'] = response.data['x-amz-credential']
          this.dataObj['policy'] = response.data['policy']
          // eslint-disable-next-line no-template-curly-in-string
          // _self.dataObj.key = response.data.dir + getUUID() + '_${filename}'
          this.dataObj['bucket'] = response.data['bucket']
          this.dataObj['key'] = response.data['key']
          resolve(true)
          console.log(this.dataObj)
        }).catch(err => {
          reject(err)
        })
      })
    },
    handleUploadSuccess (res, file) {
      console.log('上传成功...')
      this.showFileList = true
      this.fileList.pop()
      this.fileList.push({
        name: file.name,
        url: minioUrl + this.dataObj.bucket + '/' + this.dataObj.key
      })
      console.log(this.dataObj)
      this.emitInput(this.fileList[0].url)
    }
  }
}
</script>
<style>

</style>



<template>
  <div class="content">
    <h1>{{ msg }}</h1>
    <el-row>
      <el-button class="el-button" type="info" @click="showAllPlain" round>Toggle debug view</el-button>
      <div v-if="plainEnabled" style="margin: 5px">plaintext: {{plaintext}}</div>
      <div v-if="plainEnabled" style="margin: 5px">tableData: {{tableData}}</div>
    </el-row>
    <span v-if="hasFetched">Raw data: {{ fetchedData }}</span>
    <el-row>
      Retrieve your info: <el-input class="el-input" v-model="username" placeholder="Username" type="text"></el-input>
    </el-row>
    <el-row>
      <el-button class="el-button" type="info" @click="fetchUser(username)">Fetch info</el-button>
    </el-row>
    <span v-if="hasFetched && hasFound">Not what you are looking for? Register now!</span>
    <span v-if="hasFetched && !hasFound">Found nothing? Register now!</span>
    <el-row>
      <el-input class="el-input" v-model="localInfo.username_local" placeholder="Username" type="text"></el-input>
    </el-row>
    <el-row>
      <el-input class="el-input" v-model="localInfo.age_local" placeholder="Age" type="text"></el-input>
    </el-row>
    <el-row>
      <el-input class="el-input" v-model="localInfo.gender_local" placeholder="Gender" type="text"></el-input>
    </el-row>
    <span v-if="hasFetched">Welcome, {{ username }}!</span>
    <el-row>
      <el-button class="el-button" type="primary" icon="el-icon-circle-check-outline"
                 @click="queueInfo" round></el-button>
    </el-row>
    <h1 v-if="hasRegistered">Greetings {{ username }}, You are all set!</h1>
    <!--Separator-->
    <el-tag type="primary" style="margin-top: 3%">Registration queue</el-tag>
    <el-table class="el-table-display" highlight-current-row :fit="true"
              :data="queueData" max-height="400" style="width: 70%" tooltip-effect="dark"
              ref="multipleTable" @selection-change="handleSelectionChange"
              @row-click="handleRowSelected">
      <el-table-column fixed type="selection" width="50">
      </el-table-column>
      <el-table-column sortable prop="username_local" label="Username" width="180" show-overflow-tooltip>
      </el-table-column>
      <el-table-column sortable prop="age_local" label="Age" width="180" show-overflow-tooltip>
      </el-table-column>
      <el-table-column sortable prop="gender_local" label="Gender" show-overflow-tooltip>
      </el-table-column>
      <el-table-column fixed="right" label="" width="120" show-overflow-tooltip>
        <template slot-scope="scope">
          <el-button @click.native.prevent="modifyQueueInfo(scope.$index, queueData)"
                     type="text" size="small">Modify</el-button>
          <el-button @click.native.prevent="deleteQueueInfo(scope.$index, queueData)"
                     type="danger" icon="el-icon-delete" size="small" circle></el-button>
        </template>
      </el-table-column>
    </el-table>
    <div style="margin-top: 20px">
      <el-button @click="resetSort" type="info" plain round>重置排序</el-button>
      <el-button class="el-button" type="success" :disabled="this.selectedRows.length === 0"
                 @click="registerMultiple" round>提交注册</el-button>
      <el-button @click="removeSelectedQueues" type="danger"
                 :disabled="this.selectedRows.length === 0" plain round>批量移除</el-button>
    </div>
    <!--Separator-->
    <el-tag type="danger" style="margin-top: 3%">Database entries</el-tag>
    <el-table class="el-table-display" highlight-current-row :fit="true"
              :data="tableData" max-height="400" style="width: 70%" tooltip-effect="dark"
              ref="multipleTable" @selection-change="handleSelectionChange"
              @row-click="handleRowSelected">
      <!--[WHY?]No parameters needed in handleRowSelected ()-->
      <el-table-column fixed type="selection" width="50">
      </el-table-column>
      <el-table-column sortable prop="username" label="Username" width="180" show-overflow-tooltip>
      </el-table-column>
      <el-table-column sortable prop="age" label="Age" width="180" show-overflow-tooltip>
      </el-table-column>
      <el-table-column sortable prop="gender" label="Gender" show-overflow-tooltip>
      </el-table-column>
      <el-table-column fixed="right" label="" width="120" show-overflow-tooltip>
        <template slot-scope="scope">
          <el-button @click.native.prevent="modifyUserInfo(scope.$index, tableData)"
            type="text" size="small">Modify</el-button>
          <el-button @click.native.prevent="deleteUser(scope.$index, tableData)"
            type="danger" icon="el-icon-delete" size="small" circle></el-button>
        </template>
      </el-table-column>
    </el-table>
    <div style="margin-top: 20px">
      <el-button @click="resetSort" type="info" plain round>重置排序</el-button>
      <el-button @click="toggleSelection()" :disabled="this.selectedRows.length === 0" round>取消选择</el-button>
      <!--<el-button @click="removeSelectedQueues" type="danger" plain round>批量移除</el-button>-->
    </div>
    <!--separator-->
    <el-upload ref="upload" class="avatar-uploader" drag action="" :on-change="handleUploadChange"
               :on-preview="handlePreviewFile" :on-remove="handleRemoveFile"
               :on-success = "handleUploadSuccess" :before-upload = "handleBeforeImageUpload"
               list-type="picture" :auto-upload="false" accept="image/*" multiple>
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将图片拖到此处，或<em>点击上传</em></div>
      <div class="el-upload__tip" slot="tip">只能上传图片文件，且不超过5M</div>
    </el-upload>
    <el-row>
      <el-button :disabled="!hasPreviewedUploads" size="small" type="primary" @click="handleSubmitFile">上传</el-button>
      <el-button size="small" type="primary" @click="handleSubmitFile" icon="el-icon-refresh" plain></el-button>
      <el-button :disabled="!hasPreviewedUploads" size="small" @click="handleCancel">取消</el-button>
    </el-row>
    <el-row class="image-display" :gutter="10">
      <el-col :span="24" v-for="o in imageList" :key="o">
        <el-card :body-style="{ padding: '1px' }" style="margin: 5px">
          <!--<div v-if="imageUrl">-->
          <div v-if="o.imageUrl">
            <!--<img :src="imageUrl" class="avatar" width="50%">-->
            <img :src="o.imageUrl" class="static-image">
            <div>
              <el-row class="bottom clearfix">
                <time v-if="uploadTime" class="time">{{ uploadTime }}</time>
                <time v-else class="time">{{ o.imageUrl }}</time>
                <el-button type="danger" icon="el-icon-delete" class="el-button-remove-cloud" @click="removeCloudImage" circle></el-button>
              </el-row>
            </div>
          </div>
          <!--<i v-else class="el-icon-plus avatar-uploader-icon"></i>-->
        </el-card>
      </el-col>
    </el-row>
    <!--Separator-->
    <el-tag>Avatar Section</el-tag>
    <el-card class="el-card-user-info" :body-style="{ padding: '1px' }"
             v-for="(o, index) in sortedUserInfo" :key="o">
      <el-row :gutter="5">
        <el-col :span="4">
          <el-upload class="avatar-uploader" action="" :show-file-list="false"
            :on-success="handleUploadSuccess" :before-upload="handleBeforeAvatarUpload"
          :data="{'index': index}" @click.native="handleAvatarUploadClicked(index)">
            <img v-if="o.userAvatar.avatarUrl" :src="o.userAvatar.avatarUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-col>
        <el-col :span="20">
          {{index}}
          {{o.userAvatar.avatarUrl}}
          <div class="user-desc">
            {{o.userDesc}}
          </div>
        </el-col>
      </el-row>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      plaintext: '',
      plainEnabled: false,
      hasFetched: false,
      hasFetchedAll: false,
      hasFound: false,
      hasRegistered: false,
      fetchedData: [],
      username: '',
      age: '',
      gender: '',
      localInfo: {
        username_local: '',
        age_local: '',
        gender_local: ''
      },
      queueData: [],
      tableData: [],
      selectedRows: [],
      hasPreviewedUploads: false,
      imageList: [],
      imageUrl: '',
      userInfo: {
        userAvatar: '',
        userDesc: ''
      },
      userInfoList: [],
      avatarUploadIndex: 0,
      avatarUrl: '',
      uploadTime: ''
    }
  },
  computed: {
    sortedUserInfo () {
      if (this.userInfoList !== null) {
        var sortedUserInfoList = this.userInfoList
        return sortByKey(sortedUserInfoList, 'userDesc', 'username')
      }
    }
  },
  methods: {
    resetFields () {
      // 如下赋值将也会重置queueData中的字段！(WHY?)
      // this.localInfo.username_local = ''
      // this.localInfo.age_local = ''
      // this.localInfo.gender_local = ''
      this.localInfo = {
        // username_local: '',
        age_local: '',
        gender_local: ''
      }
    },
    getField (data) {
      this.username = data.username
      this.age = data.age
      this.gender = data.gender
    },
    fetchUser (queriedName) {
      let params = {
        username: queriedName
      }
      // Changing from GET method to POST method to retrieve data from server
      this.$http.post('/api/login/get', params)
        .then((res) => {
          console.log('Info successfully retrieved: ' + Date())
          console.log('body: ' + res.body)
          this.hasFetched = true
          this.localInfo.username_local = queriedName
          if (res.body !== '') {
            this.hasFound = true
            this.fetchedData = res.body
            this.getField(this.fetchedData)
            this.$message({
              message: 'Congrats, loading successful!',
              type: 'success'
            })
          }
        }).catch((err) => {
          console.log(err)
        })
    },
    queueInfo () {
      console.log('queueInfo ()...')
      this.queueData.push(this.localInfo)
      this.resetFields()
    },
    modifyQueueInfo (index) {
    //
    },
    deleteQueueInfo (index, rows) {
      // this.queueData.splice(index, 1)
      rows.splice(index, 1)
    },
    handleRowSelected (row) {
      this.$refs.multipleTable.toggleRowSelection(row, true)
    },
    removeSelectedQueues () {
      console.log('removeSelectedQueues ()...')
      this.$confirm('此操作将永久删除选中行, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        for (var i = 0; i < this.selectedRows.length; i++) {
          for (var j = 0; j < this.queueData.length; j++) {
            if (this.selectedRows[i] === this.queueData[j]) {
              this.queueData.splice(j, 1)
            }
          }
        }
      })
    },
    resetSort () {
      this.$refs.multipleTable.clearSort()
    },
    registerCore (url, params) {
      this.$http.post(url, params)
        .then((res) => {
          console.log('registerCore (): ' + JSON.stringify(res))
          this.$message({
            message: 'Congrats, insertion successful!',
            type: 'success'
          })
          this.hasRegistered = true
          // this.resetFields()
          // this.getAllUsers()
        }).catch((err) => {
          console.log('registerCore (): ' + err)
        })
    },
    register () {
      let params = {
        username: this.username,
        age: this.age,
        gender: this.gender
      }
      this.registerCore('/api/login/create', params)
      if (this.hasRegistered) {
        this.hasRegistered = false
        console.log('registerCore (): ok')
      }
    },
    registerMultiple () {
      this.$confirm('您将仅提交指定的注册信息, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'info'
      }).then(() => {
        for (var i = 0; i < this.selectedRows.length; i++) {
          for (var j = 0; j < this.queueData.length; j++) {
            if (this.selectedRows[i] === this.queueData[j]) {
              // 【如何实现上传完成后批量清除本地勾选的数据？回调策略无效】
              // setTimeout(() => {
              //   console.log(this.hasRegistered + ' ' + j)
              //   if (this.hasRegistered) {
              //     this.queueData.splice(j, 1)
              //     this.hasRegistered = false
              //   }
              // }, 1000)
              let params = {
                username: this.queueData[j].username_local,
                age: this.queueData[j].age_local,
                gender: this.queueData[j].gender_local
              }
              this.registerCore('/api/login/create', params)
            }
          }
        }
      }).then(() => {
        this.resetFields()
        this.getAllUsers()
      })
    },
    toggleSelection (rows) {
      console.log('toggleSelection ()...')
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row)
        })
      } else {
        this.$refs.multipleTable.clearSelection()
      }
    },
    handleSelectionChange (val) {
      console.log('handleSelectionChange ()...')
      this.selectedRows = val
    },
    modifyUserInfo (index, rows) {
      // rows.splice(index, 1)
    },
    deleteUser (index, rows) {
      // rows.splice(index, 1)
    },
    getAllUsers () {
      this.$http.get('/api/login/getAll')
        .then((res) => {
          console.log('Login info successfully retrieved: ' + Date())
          console.log('getAllUsers raw: ' + JSON.stringify(res.data))
          if (res.data !== '') {
            this.tableData = res.data
            this.hasFetchedAll = true
            this.$message({
              message: 'Congrats, loading all login info successful!',
              type: 'success'
            })
          }
        }).catch((err) => {
          console.log(err)
        })
    },
    showAllPlain () {
      this.plainEnabled = !this.plainEnabled
      if (this.plainEnabled === true) {
        this.$http.get('/api/login/getAll')
          .then((res) => {
            if (this.hasFetchedAll === true) {
              console.log('showAllPlain: ' + res.data)
              this.plaintext = JSON.stringify(res.data)
              console.log(this.plaintext)
            }
          }).catch((err) => {
            console.log(err)
          })
      }
    },
    handleUploadSuccess (res, file) {
      // this.loadAllImages()
    },
    handleBeforeImageUpload (file) {
      console.log('handleBeforeImageUpload: ')
      console.log(file)
      var formData = new FormData()
      formData.append('file', file)
      this.$http.post('/api/upload/image', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      })
        .then((res) => {
          console.log('handleUploadSuccess: ')
          console.log(res)
          // if (res.status === '0') {
          // eslint-disable-next-line
          if (true) {
            console.log('success msg')
            this.$message({
              message: res.msg,
              type: 'success'
            })
          } else {
            console.log('error msg')
            this.$message.error(res.msg)
          }
        }).catch((err) => {
          console.log(err)
        })
    },
    handlePreviewFile () {
      console.log('handlePreviewFile()')
      this.hasPreviewedUploads = true
    },
    handleRemoveFile () {
      console.log('handleRemoveFile()')
      this.hasPreviewedUploads = false
    },
    handleSubmitFile () {
      console.log('handleSubmitFile ()')
      this.$refs.upload.submit()
      this.$http.get('/api/upload/image/loadAll')
        .then((res) => {
          console.log('Public images successfully retrieved: ' + Date())
          console.log('res.data: ' + JSON.stringify(res.data))
          // upon uploading images, all images are re-obtained
          this.imageList = []
          // this.imageList = JSON.parse(res.data)
          this.imageList = res.data
        }).catch((err) => {
          console.log(err)
        })
    },
    loadUserInfo () {
      console.log('loadUserInfo ()')
      // api needs updating
      this.$http.get('/api/login/getAll')
        .then((resText) => {
          console.log('User desc successfully retrieved: ' + Date())
          console.log('raw resText.data: ' + JSON.stringify(resText.data))
          if (resText.data !== '') {
            this.$message({
              message: 'Congrats, loading user desc successful!',
              type: 'success'
            })
            this.$http.get('/api/upload/avatar/loadAll')
              .then((resImage) => {
                console.log('User avatar successfully retrieved: ' + Date())
                console.log('raw resImage.data: ' + JSON.stringify(resImage.data))
                // [] and null for array[!]
                this.userInfoList = []
                // localStorage -> GET request[!]
                this.userInfoList = JSON.parse(localStorage.getItem('userInfoList'))
                if (this.userInfoList === null) {
                  this.userInfoList = []
                  console.log('userInfoList in localStorage not found, reinitializing...')
                  for (var i = 0; i < resText.data.length; i++) {
                    this.userInfoList.push({
                      userAvatar: resImage.data.length >= i + 1 ? resImage.data[i] : '',
                      userDesc: resText.data[i]
                    })
                    console.log('setting localStorage')
                    localStorage.setItem('userInfoList', JSON.stringify(this.userInfoList))
                    console.log('@userInfoList[' + i + ']: ' + JSON.stringify(this.userInfoList[i]))
                  }
                } else {
                  console.log('userInfoList in localStorage found, loading...')
                }
              }).catch((err) => {
                console.log('error loading all avatar: ' + err)
              })
          }
        }).catch((err) => {
          console.log('error loading all user desc: ' + err)
        })
    },
    handleAvatarUploadClicked (index) {
      this.avatarUploadIndex = index
    },
    handleBeforeAvatarUpload (file) {
      console.log('handleBeforeAvatarUpload: ')
      console.log('file: ' + file)
      // console.log('data: ' + data)
      console.log('avatarUploadIndex: ' + this.avatarUploadIndex)
      var formData = new FormData()
      // variables like 'file' can be used in formidable apis
      formData.append('file', file)
      formData.append('index', this.avatarUploadIndex)
      this.$http.post('/api/upload/avatar', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then((res) => {
        console.log('handleUploadSuccess: ')
        console.log(res)
        // if (res.status === '0') {
        // eslint-disable-next-line
          if (true) {
          console.log('success msg')
          this.$message({
            message: res.msg,
            type: 'success'
          })
          console.log('file.lastModifiedDate: ' + file.lastModifiedDate)
          this.userInfoList[this.avatarUploadIndex].userAvatar = {avatarUrl: res.body.result.data}
          // localStorage -> POST request[!]
          console.log('setting localStorage')
          localStorage.setItem('userInfoList', JSON.stringify(this.userInfoList))
          console.log('handleUploadSuccess: ' + JSON.stringify(this.userInfoList))
          // ---
          // // initialization of temporarily retrieved image files
          // var windowURL = window.URL || window.webkitURL
          // this.imageUrl = windowURL.createObjectURL(file)
          // console.log('this.imageUrl: ' + this.imageUrl)
          // // this.uploadTime = file.lastModifiedDate
          // // this.uploadTime = file.name
          // // windowURL.revokeObjectURL(this.avatarUrl)
        } else {
          console.log('error msg')
          this.$message.error(res.msg)
        }
      }).catch((err) => {
        console.log(err)
      })
    },
    loadAllImages () {
      if (this.imageList.length === 0) {
        this.handleSubmitFile()
      }
      if (this.userInfoList.length === 0) {
        this.loadUserInfo()
      }
    },
    handleUploadChange () {
      console.log('handleUploadChange()')
      this.hasPreviewedUploads = false
    },
    handleCancel () {
      this.$refs.upload.clearFiles()
      this.hasPreviewedUploads = false
    },
    removeCloudImage () {}
  },
  watch: {
    username (newVal, oldVal) {
      if (this.hasFetched === true) {
        this.hasFetched = false
      }
    },
    imageList (newArray, oldArray) {
      if (newArray.length !== oldArray.length && oldArray.length !== 0) {
        console.log('watching imageList...')
        console.log(newArray)
        console.log(oldArray)
        this.handleSubmitFile()
      }
    }
  },
  mounted () {
    this.getAllUsers()
    this.loadAllImages()
  }
}
function sortByKey (array, key1, key2) {
  return array.sort((a, b) => {
    var x = a[key1][key2]
    var y = b[key1][key2]
    return ((x < y) ? -1 : ((x > y) ? 1 : 0))
  })
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.content {
  padding: 15px;
}
.el-input {
  width: 15%;
  margin: 12px 0;
}
.el-button {
  margin: 12px 0;
}
.el-table-display {
  margin: 1rem auto;
}
.image-display {
  margin: 1rem;
}
.el-button-remove-cloud {
  float: right;
  margin: 2%;
}
.el-card-user-info {
  margin: 2%;
}
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #f0c78a;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.static-image {
  margin: 3%;
  border-radius: 8px;
  display: block;
  width: 50%;
}
.avatar {
  margin-top: 5px;
  border-radius: 8px;
  display: block;
  width: 178px;
  height: 178px;
}
.avatar:hover {
  opacity: .75;
}
.user-desc {
  padding: 2%;
}
.time {
  font-size: 13px;
  color: #999;
}
.bottom {
  margin-top: 13px;
  line-height: 12px;
}
.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both
}
</style>

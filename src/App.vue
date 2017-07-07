<template>
  <div id="app">
    <input type="file" @change="ac">
    <el-radio-group v-model="radio">
      <el-radio-button label="基于颜色检索CS"></el-radio-button>
      <el-radio-button label="基于纹理检索VS"></el-radio-button>
      <el-radio-button label="基于形状检索SS"></el-radio-button>
    </el-radio-group>
    <div id="raw" v-show="raw_img">
      <span>原图</span>
      <img :src="raw_img" width="200">
    </div>
    <div id="result" v-loading="load_res">
      <el-row :gutter="20">
        <el-col :span="6" v-for="res in res_imgs" :key="res.path">
          <div class="grid-content bg-purple">
            <img :src="img_dir+res.path.split('C:/Users/zhou/Workspaces/MyEclipse Professional 2014/.metadata/.me_tcat7/webapps/CBIR/')[1]">
            <span v-text="res.name+' 相似度: '"></span>
            <b v-text="res.similarity"></b>
          </div>
        </el-col>
      </el-row>
    </div>
    <el-pagination
      v-if="res_imgs.length"
      layout="total, prev, pager, next, jumper"
      :current-page="current_page"
      :page-size="8"
      @current-change="handleCurrentChange"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>

export default {
  name: 'app',
  watch: {
    radio (newV) {
      if(!document.querySelector("input").files[0]) return
      this.current_page = 1
      this.retry = true
      this.sendFile()
    }
  },
  data () {
    return {
      retry: false, //是否更新数据库相似度字段
      total: null, //数据库总记录条数
      raw_img: null, //显示原图
      load_res: false, //显示加载中
      current_page: 1, //分页显示的当前页数
      res_imgs: [], //搜索结果
      radio: '基于颜色检索CS', //选择检索方式
      url: "http://172.16.122.164:8080/CBIR/servlet/Service", //硬编码后端接口地址
      img_dir: "http://172.16.122.164:8080/CBIR/", //硬编码图片文件夹地址
    }
  },
  methods: {
    ac () {
      let file = document.querySelector("input").files[0]
      if(!file || file.type.slice(0, 5)!="image") {
        this.$notify.warning({
          title: '提示',
          message: '请选择图片文件'
        })
        return
      }
      this.raw_img = window.URL.createObjectURL(document.querySelector("input").files[0])
      this.retry = true
      this.current_page = 1
      this.sendFile()
    },
    handleCurrentChange (page) {
      this.retry = false
      this.current_page = page
      this.sendFile()
    },
    sendFile () {
      let data = new FormData()
      let file = document.querySelector("input").files[0]
      if(!file || file.type.slice(0, 5)!="image") {
        this.$notify.warning({
          title: '提示',
          message: '请选择图片文件'
        })
        return
      }
      data.append("file", file)
      data.append("page", this.current_page)
      data.append("retry", this.retry)
      let table
      if(this.radio == "基于颜色检索CS") table = "color"
      else if(this.radio == "基于纹理检索VS") table = "texture"
      else if(this.radio == "基于形状检索SS") table = "shape"
      data.append("table", table)
      this.load_res = true
      this.$http.post(this.url, data)
      .then(res => {
        this.load_res = false
        this.total = res.body.total
        this.res_imgs = res.body.images
      })
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#result {
  width: 800px;
  margin-left: 350px;
  margin-top: 10px;
}
#raw {
  position: absolute;
  left: 100px;
  top: 50px;
}
#raw span {
  position: absolute;
  top: -2em;
}
.el-radio-group {
  margin: 8px 0;
}
.el-row {
  margin-bottom: 20px;
}
.el-row img {
  width: 100%;
  height: 123px;
}
.el-row:last-child {
  margin-bottom: 0;
}
.el-col {
  border-radius: 4px;
  margin-bottom: 8px;
}
.bg-purple-dark {
  background: #99a9bf;
}
.bg-purple {
  background: #d3dce6;
}
.bg-purple span {
  display: block;
}
.bg-purple-light {
  background: #e5e9f2;
}
.grid-content {
  border-radius: 4px;
  min-height: 36px;
}
.row-bg {
  padding: 10px 0;
  background-color: #f9fafc;
}
</style>

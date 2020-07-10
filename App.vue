<template>
  <div id="app">
    <div class="top box">
      <div class="flex">
        <img src="./img/1.png" alt />
        <div class="title">文件上传</div>
      </div>
      <div>
        <input type="text" class="file" placeholder="文件 csv格式" v-model="file_name" />
        <input class="fileInput" type="file" @change="triggerFile($event)" />
        <button class="select">选择</button>
      </div>
    </div>
    <div class="other box">
      <div class="flex">
        <img src="./img/2.png" alt />
        <div class="title">添加备注</div>
      </div>

      <div class="content">
        <textarea v-model="content" class="textarea" @click="inputContent"></textarea>
        <div class="plac" v-if="show">请输入</div>

        <span :class="disable ? 'over' : 'number'">{{ num }}/100</span>
      </div>
    </div>
    <div class="option box">
      <button :class="['option-btn', iscancel ? 'option-btn' : 'no-cancel']" @click="cancel">取消</button>
      <button :class="['option-btn', !isClick ? 'upload-btn' : 'no-click']" @click="upLoad">开始上传</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  components: {},
  data() {
    return {
      show: true,
      content: "",
      num: 0,
      disable: false,
      isClick: false,
      iscancel: false,
      upLoadclick: false,
      chinese: "",
      english: "",
      file_name: "",
      file_content: "",
      file: "",
      source: null
    };
  },
  watch: {
    content(val) {
      this.computed(val.split(""));
      this.num = this.count(this.chinese, this.english);
      if (this.num > 100) {
        this.disable = true;
        this.isClick = true;
      } else {
        this.isClick = false;
      }
    },
    file_content(val) {
      console.log(val);

      if (!val) {
        this.iscancel = false;
        this.isClick = true;
      } else {
        this.iscancel = true;
      }
    }
  },
  mounted() {
    this.isClick = true;
  },
  methods: {
    cancel() {
      if (this.upLoadclick) {
        this.source.cancel("取消上传");
      } else {
        alert("请上传文件");
      }
    },
    upLoad() {
      this.upLoadclick = true;
      let url = "http://10.22.73.20:8778/upload";
      let data = this.file_content;
      this.source = this.$axios.CancelToken.source(); // 这里初始化source对象

      this.$axios
        .post(url, data, { cancelToken: this.source.token })
        .then(res => {
          alert(res.data);
        })
        .catch(() => {
          alert("取消上传");
        });
    },
    triggerFile(event) {
      let that = this;
      let file = event.target.files[0];

      if (file) {
        const reg = /\.csv$/; //判断上传文件格式
        if (file.name.match(reg)) {
          this.file_name = file.name;
          this.isClick = false;
          let reader = new FileReader();
          reader.readAsText(file, "UTF-8");
          reader.onload = function(e) {
            that.file_content = e.target.result;
          };
        } else {
          alert("仅支持.CSV文件");
          this.isClick = true;
        }
      }
    },
    computed(val) {
      this.chinese = "";
      this.english = "";
      var pattern = new RegExp("[\u4E00-\u9FA5]+");
      val.forEach(str => {
        if (pattern.test(str)) {
          this.chinese += str;
        } else {
          this.english += str;
        }
      });
    },
    count(chinese, english) {
      let num1 = 0,
        num2 = 0,
        num3 = 0;
      if (english) {
        num1 = english.match(/[a-zA-Z]+/g)
          ? english.match(/[a-zA-Z]+/g).length
          : 0;
        num2 = english.match(/[^a-zA-Z]/g)
          ? english.match(/[^a-zA-Z]/g).length
          : 0;
      }
      if (chinese) {
        num3 = chinese.split("") ? chinese.split("").length : 0;
      }
      return num1 + num2 + num3;
    },
    inputContent() {
      this.show = false;
    }
  }
};
</script>

<style>
* {
  box-sizing: border-box;
  font-size: 16px;
}
.flex {
  position: relative;
  left: -20px;
  display: flex;
  height: 30px;
  margin-bottom: 5px;
  align-items: center;
}
.title {
  font-size: 18px;
  font-weight: bold;
}

.box {
  margin-bottom: 18px;
}

button {
  width: 128px;
  height: 43px;
}

.box {
  width: 400px;
  margin-left: 10px;
}

.top {
  position: relative;
}

.top .file {
  border: 1px solid rgb(64, 114, 196);
  height: 43px;
  border-radius: 5px;
  width: 300px;
}

.top .select {
  position: absolute;
  bottom: 0;
  left: 280px;
  background-color: rgb(64, 114, 196);
  border: 1px solid blue;
  color: white;
  border-radius: 5px;
}

.fileInput {
  overflow: hidden;
  position: absolute;
  left: 280px;
  bottom: 0;
  z-index: 999;
  height: 43px;
  /* top: 0; */
  opacity: 0;
  filter: alpha(opacity=0);
  cursor: pointer;
}

.content {
  width: 400px;
  height: 240px;
  position: relative;
}

.content .plac {
  position: absolute;
  top: 0px;
  left: 2px;
  z-index: 999;
  color: #666;
}

.other .textarea {
  display: block;
  border: 1px solid rgb(64, 114, 196);
  width: 100%;
  height: 100%;
  overflow-y: auto;
  resize: none;
}
.number {
  position: absolute;
  right: 0;
  bottom: 0;
  color: #666;
  margin: 4px;
}
.over {
  position: absolute;
  right: 0;
  bottom: 0;
  margin: 4px;
  color: rgba(247, 76, 96, 1);
}
input::-webkit-input-placeholder,
textarea::-webkit-input-placeholder {
  color: #666;
  font-size: 16px;
}

.option {
  width: 400px;
  display: flex;
  justify-content: space-between;
}

.option-btn {
  background-color: white;
  border: 1px solid blue;
  color: rgb(64, 114, 196);
  border-radius: 5px;
  font-weight: 600;
}
.no-cancel {
  background-color: white;
  border: 1px solid blue;
  color: rgb(64, 114, 196);
  border-radius: 5px;
  font-weight: 600;
  cursor: not-allowed;
}

.upload-btn {
  border: none;
  background-color: rgb(245, 255, 255);
}
.no-click {
  border: none;
  background-color: rgb(245, 255, 255);
  cursor: not-allowed;
}
</style>

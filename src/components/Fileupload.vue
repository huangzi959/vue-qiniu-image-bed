<template>
  <section class="wrap">
    <mu-linear-progress v-show="isUpload" />
    <div class="m-upload">
      <div class="dragcover" @dragenter.prevent.stop="onDragenter" @drop.prevent.stop="onDrop" @dragleave.prevent.stop="onDragleave" @dragover.prevent.stop="onDragover"></div>
      <div class="dragbox" :class="{'z-active': isDragOver}">
        <div>
          <mu-icon value="backup" />
          <p>{{tip}}</p>
        </div>
      </div>
    </div>
    <div class="file-wrap">
      <mu-flat-button icon="image" label="选择文件" class="btn-select" primary>
        <input type="file" class="file-button" @change="uploadFile" multiple>
      </mu-flat-button>
      <div class="url-upload">
        <mu-text-field hintText="通过图片URL上传：http://devtools.qiniu.com/qiniu.png" v-model="fetchUrl">
        </mu-text-field>
        <mu-raised-button label="上传" class="btn-url-upload" primary @click.native="uploadFetchUrl" />
      </div>
    </div>
    <div class="card-box" v-show="picList.length">
      <div class="item" v-for="(item,index) in picList" :key="index">
        <mu-card>
          <mu-card-media>
            <div class="image" :style="'background-image:url('+item.src+')'" @click="showImagePop(item.src)"></div>
          </mu-card-media>
          <div class="toolbar">
            <div>
              <mu-text-field type="text" :value="item.src" @mouseenter.native="onInputMouseenter" />
              <mu-flat-button icon="link" class="btn btn-copy" backgroundColor="#4b5c76" color="#FFF" v-clipboard:copy="item.src" v-clipboard:success="onCopy" />
              <mu-flat-button label="markdown" class="btn" primary v-clipboard:copy="'![description]('+item.src+')'" v-clipboard:success="onCopy" />
              <mu-flat-button label="打开" class="" @click="openLink(item.src)" secondary/>
            </div>
          </div>
        </mu-card>
      </div>
    </div>
  </section>
</template>

<script>
  import storage from "../utils/storage";
  import axios from "axios";
  import API from "../api/index";
  import ImagePop from "./ImagePop";
  import Util from "../utils/common";
  export default {
    data() {
      return {
        isDragOver: false,
        picList: [
          /* {
                          key: 'a.jpg',
                          src: "http://qiniu1.huzerui.com//2017-10-23/20709373.png"
                        } */
        ],
        isUpload: false,
        fetchUrl: ""
      };
    },
    computed: {
      qiniuAuth() {
        return this.$store.state.qiniuAuth;
      },
      tip() {
        return this.isDragOver ? "松开鼠标开始上传" : "拖拽图片到这里上传";
      },
      uploadOptions() {
        return this.$store.state.uploadOptions;
      }
    },
    methods: {
      onDragenter() {
        console.log("dragenter");
        this.isDragOver = true;
      },
      onDragover() {
        console.log("dragover");
        this.isDragOver = true;
      },
      onDragleave() {
        console.log("dragLeave");
        this.isDragOver = false;
      },
      onDrop(e) {
        this.isDragOver = false;
        const files = e.dataTransfer.files;
        if (!files.length) return;
        const isAllImage = Array.prototype.every.call(files, file => {
          return file.type.indexOf("image") != -1;
        });
        if (!isAllImage) {
          this.$store.commit("SNACK_BAR_CHANGE", {
            snackbar: true,
            snackMsg: "只支持图片文件上传"
          });
          return;
        }
        this.isUpload = true;
        API.uploadFile(this.qiniuAuth, files, this.uploadOptions)
          .then(res => {
            console.log(res.data);
            if (res.data instanceof Array && res.data.length) {
              res.data.forEach(obj => {
                this.picList.push({
                  hash: obj.hash,
                  key: obj.key,
                  src: `${this.qiniuAuth.domain}/${obj.key}`
                });
              });
              this.$store.commit("SNACK_BAR_CHANGE", {
                snackbar: true,
                snackMsg: "上传成功"
              });
              this.$store.commit("PICLIST_CHANGE", {
                isPicListChange: true
              });
            } else if (res.data.code == 614) {
              this.$store.commit("SNACK_BAR_CHANGE", {
                snackbar: true,
                snackMsg: "文件名重复"
              });
            } else {
              this.$store.commit("SNACK_BAR_CHANGE", {
                snackbar: true,
                snackMsg: "上传失败，请重试"
              });
            }
            this.isUpload = false;
          })
          .catch(err => {
            this.isUpload = false;
            this.$store.commit("SNACK_BAR_CHANGE", {
              snackbar: true,
              snackMsg: "上传失败"
            });
            this.isUpload = false;
          });
      },
      onInputMouseenter(e) {
        const input = e.target.getElementsByClassName("mu-text-field-input")[0];
        input.focus();
        input.select();
      },
      uploadFile(e) {
        console.log(this.uploadOptions)
        const files = e.target.files;
        if (!files.length) return;
        const isAllImage = Array.prototype.every.call(files, file => {
          return file.type.indexOf("image") != -1;
        });
        if (!isAllImage) {
          this.$store.commit("SNACK_BAR_CHANGE", {
            snackbar: true,
            snackMsg: "只支持图片文件上传"
          });
          return;
        }
        this.isUpload = true;
        API.uploadFile(this.qiniuAuth, e.target.files, this.uploadOptions)
          .then(res => {
            console.log(res.data);
            if (res.data instanceof Array && res.data.length) {
              res.data.forEach(obj => {
                this.picList.push({
                  hash: obj.hash,
                  key: res.data.key,
                  src: `${this.qiniuAuth.domain}/${obj.key}`
                });
              });
              this.$store.commit("SNACK_BAR_CHANGE", {
                snackbar: true,
                snackMsg: "上传成功"
              });
              this.$store.commit("PICLIST_CHANGE", {
                isPicListChange: true
              });
            } else if (res.data.code == 614) {
              this.$store.commit("SNACK_BAR_CHANGE", {
                snackbar: true,
                snackMsg: "文件名重复"
              });
            } else {
              this.$store.commit("SNACK_BAR_CHANGE", {
                snackbar: true,
                snackMsg: "上传失败，请重试"
              });
            }
            this.isUpload = false;
          })
          .catch(err => {
            console.log(err);
            this.isUpload = false;
            this.$store.commit("SNACK_BAR_CHANGE", {
              snackbar: true,
              snackMsg: "上传失败"
            });
          });
      },
      onCopy() {
        this.$store.commit("SNACK_BAR_CHANGE", {
          snackbar: true,
          snackMsg: "复制到剪贴板成功"
        });
      },
      openLink(link) {
        window.open(link);
      },
      showImagePop(src) {
        this.$store.commit("IMAGE_POP_CHANGE", {
          isImagePop: true,
          imgview: src
        });
      },
      uploadFetchUrl() {
        if (!this.fetchUrl) {
          this.$store.commit("SNACK_BAR_CHANGE", {
            snackbar: true,
            snackMsg: "在线图片地址不能为空"
          });
          return;
        }
        if (!this.fetchUrl.match(/\.jpg|\.jpeg|\.png|\.gif$/)) {
          this.$store.commit("SNACK_BAR_CHANGE", {
            snackbar: true,
            snackMsg: "只支持图片上传"
          });
          return;
        }
        this.isUpload = true;
        API.uploadFetchUrl({
            fetchUrl: this.fetchUrl,
            uploadOptions: this.uploadOptions,
            ...this.qiniuAuth
          })
          .then(res => {
            console.log(res)
            if (res.data.code == 200) {
              this.picList.push({
                hash: res.data.hash,
                key: res.data.key,
                src: `${this.qiniuAuth.domain}/${res.data.key}`
              });
              this.$store.commit("SNACK_BAR_CHANGE", {
                snackbar: true,
                snackMsg: "上传成功"
              });
            } else if (res.data.code == 204) {
              this.$store.commit("SNACK_BAR_CHANGE", {
                snackbar: true,
                snackMsg: "请检查七牛配置是否正确"
              });
            } else {
              this.$store.commit("SNACK_BAR_CHANGE", {
                snackbar: true,
                snackMsg: "上传失败"
              });
            }
            this.isUpload = false;
          })
          .catch(err => {
            console.log(err);
            this.isUpload = false;
            this.$store.commit("SNACK_BAR_CHANGE", {
              snackbar: true,
              snackMsg: "上传失败"
            });
          });
      }
    }
  };
</script>

<style lang="scss">
  .wrap {
    width: 80%;
    text-align: center;
  }
  .mu-linear-progress {
    position: absolute;
    left: 0;
    top: 0;
  }
  .m-upload {
    position: relative;
    .dragcover {
      position: absolute;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      opacity: 0;
    }
    .dragbox {
      width: 100%;
      height: 300px;
      background-color: #46529d;
      display: flex;
      align-items: center;
      justify-content: center;
      text-align: center;
      margin: 0 auto;
      .mu-icon-button {
        width: auto;
      }
      .mu-icon {
        font-size: 100px;
        color: #fff;
      }
      p {
        color: #fff;
      }
    }
    .dragbox.z-active {
      background-color: #fff;
      border: 1px dashed #666;
      p {
        color: #46529d;
      }
      .mu-icon {
        color: #46529d;
      }
    }
  }
  .file-wrap {
    position: relative;
    text-align: center;
    .file-button {
      position: absolute;
      left: 0;
      right: 0;
      top: 0;
      bottom: 0;
      opacity: 0;
    }
    .btn-select {
      margin: 20px 0;
      input {
        width: 100%;
      }
    }
    .url-upload {
      display: flex;
      align-items: center;
      flex: 1;
      .mu-text-field {
        top: 8px;
        flex: 1;
        text-align: left;
      }
    }
  }
  .card-box {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    .item {
      padding: 0 10px;
      flex: 0 1 50%;
      margin-top: 10px;
    }
    .mu-card {
      .image {
        width: 100%;
        height: 500px;
        background-size: cover;
        background-repeat: no-repeat;
        background-position: center center;
        cursor: pointer;
      }
      .toolbar {
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 10px;
      }
      .mu-text-field,
      .mu-flat-button {
        vertical-align: middle;
      }
      .mu-text-field-content {
        position: relative;
        top: 8px;
      }
      .btn {
        color: #4b5c76;
      }
      .btn-copy {
        min-width: 54px;
      }
    }
  }
</style>

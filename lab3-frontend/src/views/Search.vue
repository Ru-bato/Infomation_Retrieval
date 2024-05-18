<template>
  <div>
    <el-container>
      <el-header style="background-color: purple">
        <div style="color: gold">
          <i class=""></i>
          Image Retrieval
          <el-button v-on:click="showFavorites" style="float: right; margin-top: 12px" type="success" plain>
            我的收藏
          </el-button>
        </div>
      </el-header>
      <el-main>
        <el-card class="box-card">
          <el-upload class="upload-demo" :action="uploadUrl()" :on-preview="handlePreview" :on-remove="handleRemove"
            :on-success="handleSuccess" :file-list="fileList" list-type="picture" :auto-upload="false" ref="upload"
            :before-upload="beforeUpload" :limit=1 accept=".jpg, .png">
            <el-button size="small" type="primary">
              上传图片 <i class="el-icon-upload"></i>
            </el-button>
            <div slot="tip" class="el-upload__tip">
              只能上传jpg/png文件，且不超过1MB
            </div>
          </el-upload>
          <el-input-number v-model="num" :min="1" :max="100" label="显示数量" style="margin-top: 10px;" />
          <el-button type="primary" @click="submitUpload">搜索</el-button>
        </el-card>
        <div v-if="showResult">
          <h4>
            共搜索到 <span>{{ imageList.length }}</span> 张图片
          </h4>
          <div class="block">
            <el-row :gutter="20">
              <el-col :span="8" v-for="item in displayedImageList" :key="item">
                <el-card :body-style="{ padding: '8px' }" shadow="hover">
                  <img :src="item" class="image" @click="viewOriginalImage(item)">
                  <div style="padding: 14px">
                    <span>与 {{ file.name }} 相似的图片</span>
                    <div class="bottom clearfix">
                      <time class="time">{{ item.slice(item.indexOf("im")) }}</time>
                      <el-button type="warning" class="button" v-on:click="handleFavorites($event, item)"
                        icon="el-icon-star-off" circle></el-button>
                    </div>
                  </div>
                </el-card>
              </el-col>
            </el-row>
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
              :current-page="currentPage" :page-sizes="[9, 20, 50, 100]" :page-size="pageSize"
              layout="total, sizes, prev, pager, next, jumper" :total="imageList.length">
            </el-pagination>
          </div>
        </div>
        <div v-if="showFavorite">
          <h4>共收藏有 <span>{{ favorites.length }}</span> 张图片</h4>
          <el-row :gutter="20">
            <el-col :span="8" v-for="item in favorites" :key="item">
              <el-card :body-style="{ padding: '8px' }" shadow="hover">
                <img :src="item" class="image" @click="viewOriginalImage(item)">
                <div style="padding: 14px">
                  <el-input v-model="favoriteNames[item]" placeholder="输入名称"
                    @blur="updateFavoriteName(item)"></el-input>
                  <div class="bottom clearfix">
                    <time class="time">{{ item.slice(item.indexOf("im")) }}</time>
                    <el-button type="warning" @click="handleFavorites($event, item)" class="button"
                      icon="el-icon-delete" circle></el-button>
                  </div>
                </div>
              </el-card>
            </el-col>
          </el-row>
        </div>
      </el-main>
    </el-container>
    <el-dialog :visible.sync="dialogVisible">
      <img :src="originalImage" style="width: 100%;" />
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  mounted() {
    this.loadFavorites();
  },
  data() {
    return {
      showResult: false,
      showFavorite: false,
      firstUpload: true,
      num: 9,
      file: {},
      fileList: [],
      imageList: [],
      favorites: [],
      favoriteNames: {},
      currentPage: 1,
      pageSize: 9,
      dialogVisible: false,
      originalImage: '',
    };
  },
  computed: {
    displayedImageList() {
      const start = (this.currentPage - 1) * this.pageSize;
      const end = start + this.pageSize;
      return this.imageList.slice(start, end);
    },
  },
  methods: {
    uploadUrl() {
      return `http://127.0.0.1:5000/imgUpload?num_images=${this.num}`;
    },
    submitUpload() {
      this.showResult = true; // Resetting showResult before submitting the upload
      this.$refs.upload.submit();
      if (!this.firstUpload) {
        alert("请重新上传图片");
        this.firstUpload = true;
      }
    },
    handleSuccess(response, file, fileList) {
      this.imageList = Object.values(response);
      for (let i = 0; i < this.imageList.length; i++) {
        this.imageList[i] = `http://127.0.0.1:5000${this.imageList[i].replace(
          /\\/g,
          "/"
        )}`;
      }
      this.file = file;
      this.showResult = true;
      this.showFavorite = false;
      fileList = []
    },
    handleRemove(file, fileList) {
      console.log("remove searched pic");
      this.imageList = [];
      this.showResult = false;
    },
    handlePreview(file) {
      //console.log(file);
    },

    handleFavorites(event, url) {
      let key = url.slice(url.indexOf("im"));
      let isFavorite = localStorage.getItem(key) !== null;

      if (!isFavorite) {
        localStorage.setItem(key, url);
        localStorage.setItem(key + "_name", this.favoriteNames[url] || "我的收藏");
      } else {
        localStorage.removeItem(key);
        localStorage.removeItem(key + "_name");
      }

      // 切换收藏状态
      isFavorite = !isFavorite;

      // 更新收藏列表和收藏状态
      this.loadFavorites();

      // 更新星星的样式
      const starButton = event.target;
      starButton.classList.toggle("red-star", isFavorite);
    },
    handleSizeChange(val) {
      this.pageSize = val;
    },
    handleCurrentChange(val) {
      this.currentPage = val;
    },
    showFavorites() {
      this.loadFavorites();
      this.showFavorite = !this.showFavorite;
      this.showResult = !this.showFavorite;
    },

    beforeUpload(file) {
      const isLt500K = file.size / 1024 <= 1000;
      if (!isLt500K) {
        this.$message.error('上传图片大小不能超过 1MB!');
      }
      return isLt500K;
    },
    updateFavoriteName(url) {
      let key = url.slice(url.indexOf("im"));
      localStorage.setItem(key + "_name", this.favoriteNames[url]);
      this.loadFavorites()
    },
    loadFavorites() {
      this.favorites = [];
      this.favoriteNames = {};
      for (let i = 0; i < localStorage.length; i++) {
        let key = localStorage.key(i);
        let url = localStorage.getItem(key);
        if (url.startsWith("http")) {
          this.favorites.push(url);
          this.favoriteNames[url] = localStorage.getItem(key + "_name") || "我的收藏";
        }
      }
    },
    viewOriginalImage(url) {
      this.originalImage = url;
      this.dialogVisible = true;
    },
  },
};
</script>

<style scoped>
.box-card {
  margin-bottom: 5vh;
}

.time {
  font-size: 13px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
}

.button {
  padding: 0;
  float: right;
}

.image {
  width: 100%;
  height: 200px;
  /* 固定高度 */
  object-fit: cover;
  /* 保持比例，裁剪 */
  cursor: pointer;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both;
}

.el-header {
  background-color: #143761;
  color: #f1ecec;
  text-align: center;
  line-height: 60px;
}

.button.red-star {
  color: red;
}
</style>

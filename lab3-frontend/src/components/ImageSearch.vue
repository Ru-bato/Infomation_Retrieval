<template>
    <div>
      <nav class="navbar">
        <div class="navbar-inner">
          <h1 class="text-center" style="color: black;">
            <b>IMAGE SEARCH DEMO</b>&nbsp;&nbsp;&nbsp;
            <!-- <a class="navbar-brand" href="#">
              <img src="/images/logo.svg" style="height: 40px;" alt="Logo" />
            </a> -->
          </h1>
        </div>
      </nav>
      <div><br /><br /></div>
      <div class="container">
        <table
          class="table"
          style="
            background: white;
            border: 1px solid beige;
            box-shadow: 3px 5px 15px 0px rgba(0, 0, 0, 0.2),
              3px 5px 15px 0 rgba(0, 0, 0, 0.19);
          "
        >
          <tr style="background: lightgrey;">
            <td><b>Choose your file to upload</b></td>
            <td><b></b></td>
          </tr>
          <tr>
            <td>
              <form @submit.prevent="submitForm" enctype="multipart/form-data">
                <input type="file" name="file" required @change="onFileChange" />
                <input type="submit" value="Search!" />
              </form>
            </td>
            <td><b></b></td>
          </tr>
          <tr v-if="showClearButton">
            <td>
              <button @click="clearResults">Clear</button>
            </td>
            <td></td>
          </tr>
        </table>
      </div>
      <center>
        <!-- <img
          v-if="loading"
          id="load"
          src="/images/ajax-loader.gif"
          style="height: 100px; width: 100px;"
          alt="Loading..."
        /> -->
      </center>
      <div class="container" v-if="results.length > 0">
        <table class="table" style="background: white; border: 1px;">
          <tbody>
            <tr v-for="(row, index) in resultRows" :key="index">
              <td v-for="(image, i) in row" :key="i" style="box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2); transition: 0.3s; width: 200px; height: 200px; background: white;">
                <img :src="image" alt="Result" style="width: 200px; height: 200px; box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2); transition: 0.3s;" />
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    name: 'ImageSearch',
    data() {
      return {
        loading: false,
        showClearButton: false,
        results: [],
        file: null,
      };
    },
    computed: {
      resultRows() {
        const rows = [];
        for (let i = 0; i < this.results.length; i += 3) {
          rows.push(this.results.slice(i, i + 3));
        }
        return rows;
      },
    },
    methods: {
      onFileChange(event) {
        this.file = event.target.files[0];
      },
      async submitForm() {
        if (!this.file) return;
        
        this.loading = true;
        this.results = [];
        
        const formData = new FormData();
        formData.append('file', this.file);
  
        try {
          const response = await axios.post('/imgUpload', formData, {
            headers: {
              'Content-Type': 'multipart/form-data',
            },
          });
  
          this.results = [
            response.data.image0,
            response.data.image1,
            response.data.image2,
            response.data.image3,
            response.data.image4,
            response.data.image5,
            response.data.image6,
            response.data.image7,
            response.data.image8,
          ];
  
          this.showClearButton = true;
        } catch (error) {
          console.error('Error uploading image:', error);
        } finally {
          this.loading = false;
        }
      },
      clearResults() {
        this.results = [];
        this.showClearButton = false;
        this.file = null;
      },
    },
  };
  </script>
  
  <style scoped>
  /* 组件内样式 */
  </style>
  
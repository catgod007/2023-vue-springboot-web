<template>
<div>
  <span class="demonstration">默认 Hover 指示器触发</span>
  <el-carousel height="300px">
    <el-carousel-item v-for="item in 4" :key="item">
     <h3 class="small">{{ item }}</h3>
    </el-carousel-item>
  </el-carousel>
  <br>

  <el-image v-for="url in urls" :key="url" :src="url" lazy></el-image>
  <el-row>
    <el-button >默认按钮</el-button>
    <el-button type="primary">主要按钮</el-button>
    <el-button type="success">成功按钮</el-button>
    <el-button type="info">信息按钮</el-button>
    <el-button type="warning">警告按钮</el-button>
    <el-button type="danger">危险按钮</el-button>
  </el-row>

  <el-upload
      class="avatar-uploader"
      action="https://jsonplaceholder.typicode.com/posts/"
      :show-file-list="false"
      :on-success="handleAvatarSuccess"
      :before-upload="beforeAvatarUpload">
    <img v-if="imageUrl" :src="imageUrl" class="avatar">
    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
  </el-upload>
  <br>
  <br>
  <el-calendar v-model="value">
  </el-calendar>
</div>
</template>

<script>
export default {
  name: "name",
  data() {
    return {
      imageUrl: '',
      urls: [
        'https://fuss10.elemecdn.com/a/3f/3302e58f9a181d2509f3dc0fa68b0jpeg.jpeg',
        'https://fuss10.elemecdn.com/1/34/19aa98b1fcb2781c4fba33d850549jpeg.jpeg',
        'https://fuss10.elemecdn.com/0/6f/e35ff375812e6b0020b6b4e8f9583jpeg.jpeg',
        'https://fuss10.elemecdn.com/9/bb/e27858e973f5d7d3904835f46abbdjpeg.jpeg',
        'https://fuss10.elemecdn.com/d/e6/c4d93a3805b3ce3f323f7974e6f78jpeg.jpeg',
        'https://fuss10.elemecdn.com/3/28/bbf893f792f03a54408b3b7a7ebf0jpeg.jpeg',
        'https://fuss10.elemecdn.com/2/11/6535bcfb26e4c79b48ddde44f4b6fjpeg.jpeg'
      ],
      value: new Date()
    };
  },
  methods: {
    handleAvatarSuccess(res, file) {
      this.imageUrl = URL.createObjectURL(file.raw);
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg';
      const isLt2M = file.size / 1024 / 1024 < 20;

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 JPG 格式!');
      }
      if (!isLt20M) {
        this.$message.error('上传头像图片大小不能超过 20MB!');
      }
      return isJPG && isLt20M;
    }
  }
}
</script>

<style scoped>
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
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}

.el-carousel__item h3 {
  color: #475669;
  font-size: 150px;
  opacity: 0.75;
  line-height: 150px;
  margin: 0;
}

</style>
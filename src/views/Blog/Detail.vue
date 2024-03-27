<template>
  <div>
    <el-row class="warp">
      <!--
        新增博文
            Form 组件提供了表单验证的功能，只需要通过 rule 属性传入约定的验证规则，并 Form-Item 的 prop 属性设置为需校验的字段名即可。具体可以参考官网：http://element.eleme.io/#/zh-CN/component/form
      -->
      <el-col :span="24" class="warp-main">
        <el-form
          ref="infoForm"
          :model="infoForm"
          :rules="rules"
          label-width="120px"
        >
          <el-form-item label="标题" prop="btitle">
            <el-input v-model="infoForm.btitle"></el-input>
          </el-form-item>

          <!-- <el-form-item label="分类" prop="bcategory">
            <el-input v-model="infoForm.bcategory"></el-input>
          </el-form-item> -->

          <el-form-item label="分类" prop="bcategory">
            <el-select v-model="thisbcategory" filterable clearable>
              <el-option
                v-for="item in blogNodeinfoForm"
                :key="item.bID"
                :label="item.btitle"
                :value="item.btitle"
              >
              </el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="作者" prop="bsubmitter">
            <el-input v-model="infoForm.bsubmitter"></el-input>
          </el-form-item>

          <!-- 这里开始新建 -->
          <el-form-item label="父导航" prop="bparentId">
            <el-select
              v-model="boxFather.btitle"
              filterable
              clearable
              @visible-change="handleSelectVisibleChange"
              @change="handleSelectChange"
            >
              <el-option
                v-for="item in blogFatherinfoForm"
                :key="item.bID"
                :label="item.btitle"
                :value="item.btitle"
              >
              </el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="节点等级" prop="bnodeLevel">
            <el-input v-model="infoForm.bnodeLevel"></el-input>
          </el-form-item>

          <el-form-item label="推荐等级" prop="bstarLevel">
            <el-input v-model="infoForm.bstarLevel"></el-input>
          </el-form-item>

          <p>{{ newstarlist }}</p>
          <el-form-item label="相关推荐" prop="StarList">
            <el-select
              v-model="newstarlist"
              multiple
              filterable
              @visible-change="handleSelectStart"
            >
              <el-option
                v-for="item in blogFatherinfoForm"
                :key="item.bID"
                :label="item.btitle"
                :value="item.bID"
              >
              </el-option>
            </el-select>
          </el-form-item>

          <!-- 展示已上传图片 -->
          <el-form-item label="展示图" prop="bimage">
            <el-upload
              class="upload-demo"
              action="/images/Upload/Pic"
              :show-file-list="false"
              :on-success="handleUploadSuccess"
              :before-upload="beforeUpload"
              multiple
            >
              <div class="upload-list">
                <!-- 循环展示已上传的图片 -->
                <div
                  v-for="(image, index) in images"
                  :key="index"
                  class="image-item"
                >
                  <img :src="image.ImagePath" class="avatar" />
                  <p>{{ images.ImagePath }}</p>
                  <span class="delete-icon" @click.stop="deleteImage(index)"
                    >X</span
                  >
                  <!-- 点击删除图片 -->
                </div>
              </div>
              <i class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
        
      



          <!--使用编辑器
          -->
          <el-form-item label="详细" prop="bcontent">
            <div class="edit_container">
              <quill-editor
                v-model="infoForm.bcontent"
                ref="myQuillEditor"
                class="editer"
                :options="editorOption"
                @ready="onEditorReady($event)"
              ></quill-editor>
            </div>
          </el-form-item>

          <el-form-item>
            <el-button type="primary" :disabled="submitting" @click="onSubmit">确认提交</el-button>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { getBlogListPage, quillEditor } from "vue-quill-editor"; //调用编辑器
// require styles
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";
import {
  getBlogDeatil,
  editBlog,
  DelBlogImages,
  GetBlogNode,
} from "../../api/api";

export default {
  data() {
    return {
      submitting:false,
      id: this.$route.params.id,
      infoForm: {
        btitle: "",
        bsubmitter: "",
        bcategory: "技术博文",
        bcontent: "",
        bstarLevel: null,
        StarList: [],
        bstarList: "",
        bparentId: 0,
        Father: {
          bID: 0,
          btitle: "",
        },
        imageUrlList: [],
        bstarList: "",
        bnodeLevel: 0,
      },
      blogNodeinfoForm: [],
      thisbcategory: {},
      blogFatherinfoForm: [],
      boxFather: {},
      dataform: [],
      newstarlist: [],
      bimage: "",
      images: [],
      editorOption: {},
      //表单验证
      rules: {
        btitle: [{ required: true, message: "请输入标题", trigger: "blur" }],
        bcontent: [
          { required: true, message: "请输入详细内容", trigger: "blur" },
        ],
      },
    };
  },

  computed: {
    editor() {
      return this.$refs.myQuillEditor.quill;
    },
  },
  mounted() {
    //初始化
    var para = { id: this.id };
    getBlogDeatil(para).then((res) => {
      this.infoForm = res.data.response;
      this.images = this.infoForm.DisplayImageData;
      this.thisbcategory = this.infoForm.bcategory;
      this.boxFather = this.infoForm.Father;
      this.newstarlist = this.infoForm.bstarList.split(",");
    });

    //一级节点（类型）
    var bcategorynull = { bcategory: null };
    GetBlogNode(bcategorynull).then((res) => {
      this.blogNodeinfoForm = res.data.response;
    });
  },

  methods: {
    //初始化數據
    handleSelectStart(visible) {
      if (visible) {
        if (true) {
          var bcategorythis = { bcategory: "ALL" };
          GetBlogNode(bcategorythis).then((res) => {
            this.blogFatherinfoForm = res.data.response;
          });
        }
      }
    },

    handleSelectVisibleChange(visible) {
      if (visible) {
        var bcategorythis = { bcategory: this.thisbcategory };
        GetBlogNode(bcategorythis).then((res) => {
          this.blogFatherinfoForm = res.data.response;
        });
      }
      //  else{
      //   this.infoForm.bnodeLevel=boxFather

      // }
    },

    handleSelectChange(value) {
      // 在下拉框选项改变时更新boxFather为选中的对象
      this.blogFatherinfoForm.forEach((item) => {
        if (item.btitle === value) {
          this.boxFather = item;
          //节点等级比父级第一级
          this.infoForm.bnodeLevel = item.bnodeLevel + 1;
        }
      });
    },

    deleteImage(index) {
      if (this.images[index].Id > 0) {
        let para = {
          id: this.images[index].Id,
        };
        DelBlogImages(para)
          .then((res) => {
            console.log(res);
            this.images.splice(index, 1); // 在异步操作完成后再移除数组元素
          })
          .catch((error) => {
            console.error("删除失败:", error);
          });
      } else {
        this.images.splice(index, 1);
      }
    },

    //图片处理
    handleUploadSuccess(response, file, fileList) {
      const imagesString = response.response.replace(/,/g, ""); // 去掉逗号
      const imageUrls = imagesString.split(" ");
      imageUrls.forEach((url) => {
        const ImagePath = "http://localhost:9291/" + url;
        this.images.push({ ImagePath });
      });
      // this.bimage = "http://localhost:9291/" + imageUrls; // 假设服务器返回的数据中包含图片的 URL
    },
    beforeUpload(file) {
      // 在上传之前的处理函数，可以做一些限制或者处理文件的操作
      // 返回 false 可以阻止上传
      // 返回 true 或者不返回任何值都将继续上传
      const isJPG = file.type === "image/jpeg" || file.type === "image/png";
      if (!isJPG) {
        this.$message.error("上传图片只能是 JPG/PNG 格式!");
        return false;
      }
      const isLt2M = file.size / 1024 / 1024 < 2;
      if (!isLt2M) {
        this.$message.error("上传图片大小不能超过 2MB!");
        return false;
      }
      return true;
    },

    onEditorReady(editor) {},

    onSubmit() {
      //提交
      //this.$refs.infoForm.validate，这是表单验证
      this.submitting = true;
      //按钮锁定
      this.$refs.infoForm.validate((valid) => {
        if (valid) {
          this.infoForm.Father = this.boxFather;
          this.infoForm.bparentId = this.infoForm.Father.bID;
          if (this.infoForm.StarList.length == 0) {
            this.infoForm.StarList = null;
          }
          this.infoForm.imageUrlList = this.images
            .filter((image) => image.Id ==null)
            .map((image) => image.ImagePath);
          this.infoForm.bstarList = this.newstarlist
            .map((item) => item)
            .join(",");

          var postPara = this.infoForm;
          editBlog(postPara).then((res) => {
            if (res.data.success) {
              this.$notify({
                type: "success",
                message: "修改成功!",
                duration: 3000,
              });
              this.$router.replace(`/Blog/Blogs`);
            } else {
              var errorMsg = res.data.msg;
              this.$message({
                type: "error",
                message: errorMsg,
                showClose: true,
              });
            }
          });
        }
      });
      this.submitting = true;
    },
  },
  components: {
    //使用编辑器
    quillEditor,
  },
};
</script>

<style>
.warp {
  background: #fff;
  padding-top: 20px;
  width: 90%;
  margin: 0 auto;
}
.ql-container.ql-snow,
.ql-editor.ql-blank {
  min-height: 500px !important;
}
.upload-list {
  display: flex;
  flex-wrap: wrap;
}
.image-item {
  position: relative;
  margin-right: 10px;
  margin-bottom: 10px;
}
.avatar {
  width: 100px;
  height: 100px;
}
.delete-icon {
  position: absolute;
  top: 5px;
  right: 5px;
  font-size: 20px;
  color: red;
  cursor: pointer;
}
</style>

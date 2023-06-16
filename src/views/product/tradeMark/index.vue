<template>
  <div>
    <el-button
      type="primary"
      icon="el-icon-plus"
      style="margin: 10px 0px"
      @click="showDialog"
      >添加</el-button
    >
    <!-- 
      表格组件 
      :data 要展示的数据
      el-table-column 表示一列一列数据
      border 边框
      label 显示的标题
      width 列的宽度
      align 对齐方式
      prop 对应列内容字段名 要展示数据的字段名
      tips1:tabel数据是列展示
    -->
    <el-table style="width: 100%" border :data="list">
      <el-table-column type="index" label="序号" width="80px" align="center">
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称" width="width">
      </el-table-column>
      <el-table-column prop="logoUrl" label="Logo" width="width">
        <!-- 
          作用域插槽 父组件:el-table 子组件:el-table-column
          父组件里面使用了子组件 父组件里面写template
          子组件里面本身又slot 父组件传结构和数据 
          把img放在子组件隐身的slot里面展示
        -->
        <!-- 数据是父组件给的或者数据是在子组件里面 父组件给子组件传递结构 -->
        <!-- 子组件再将数据回传给插槽的使用者 -->
        <!-- prop应该就是把list的每一项的具体名称传过去,传什么就显示什么,
        url本来就是一个字符串,当然就显示字符串
        要利用插槽加img标签来展示图片 -->
        <template slot-scope="{ row, $index }">
          <!-- { row, $index }是解构出来的 -->
          <img :src="row.logoUrl" alt="" style="width: 50px; height: 50px" />
        </template>
      </el-table-column>
      <el-table-column prop="prop" label="操作" width="width">
        <!-- row 是子组件table的数据 这一行的品牌信息 -->
        <template slot-scope="{ row, $index }">
          <el-button
            type="warning"
            size="mini"
            icon="el-icon-edit"
            @click="updateTradeMark(row)"
            >修改</el-button
          >
          <el-button
            type="danger"
            size="mini"
            icon="el-icon-delete"
            @click="deleteTradeMark(row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页器：total pageNo pageSize  
    current-page 当前页
    page-sizes 设置每一页展示几条数据
    page-count 显示可点击页码的数量  
    @size-change="handleSizeChange"
    @current-change="handleCurrentChange"
    点击当前页进行改变 省略了一个函数
    -->
    <el-pagination
      style="margin-top: 20px; textalign: center"
      :current-page="9"
      :total="49"
      :page-size="3"
      :page-count="7"
      :page-sizes="[3, 5, 10]"
      @size-change="handleSizeChange"
      @current-change="getPageList"
      layout=" prev, pager, next, jumper,->,sizes, total"
    >
      <!-- 上一页 中间连续页 下一页 跳转到哪一页 选择每一页展示几个 总页数 -->
    </el-pagination>
    <!-- Form 组件提供了表单验证的功能，只需要通过 rules 属性传入约定的验证规则，
    并将 Form-Item 的 prop 属性设置为需校验的字段名即可 -->
    <el-dialog
      :title="tmForm.id ? '修改品牌' : '添加品牌'"
      :visible.sync="dialogFormVisible"
    >
      <!-- :model="form" 将表单数据收集到某个对象上 -->
      <el-form ref="ruleForm" style="width: 90%" :model="tmForm" :rules="rules">
        <!-- input框 -->
        <el-form-item label="品牌名称" label-width="100px" prop="tmName">
          <!-- v-model="form.name"  -->
          <el-input autocomplete="off" v-model="tmForm.tmName"></el-input>
          <!-- 这个input框 里面v-model绑定了tmForm.tmName -->
        </el-form-item>
        <!-- 上传品牌图片 -->
        <el-form-item label="品牌Logo" label-width="100px" prop="logoUrl">
          <!-- 
            :on-success="handleAvatarSuccess" 文件上传成功时的钩子
            :before-upload="beforeAvatarUpload"  上传文件之前的钩子
            -->
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <!-- <img v-if="tmForm.logoUrl" :src="tmForm.logoUrl" class="avatar" /> -->
            <img v-if="imageUrl" :src="imageUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div class="el-upload__tip" slot="tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTradeMark"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { call } from "body-parser";

export default {
  name: "tradeMark",
  data() {
    // let validateTmName = (rule,value,callback)=>{

    // }
    return {
      //自定义校验规则
      //当前页
      page: 1,
      //展示数据的条数
      limit: 3,
      //总条数
      total: 0,
      //列表展示数据
      list: [],
      //对话框的显示与否
      dialogFormVisible: false,
      //上传图片使用的属性
      imageUrl: "",
      //弹出框收集品牌的信息
      tmForm: {
        logoUrl: "",
        tmName: "",
      },
      rules: {
        //品牌名称的验证规则
        tmName: [
          { required: true, message: "请输入品牌名称", trigger: "blur" },
          {
            min: 2,
            max: 9,
            message: "长度在 2 到 9 个字符",
            trigger: "chenge",
          },
          //长度2-9
        ],
        //品牌logo的验证规则
        logoUrl: [{ required: true, message: "请选择活动区域" }],
      },
    };
  },
  mounted() {
    // 不止一次的点击页数 请求封装函数
    // console.log(this.$API,'1111');
    this.getPageList();
  },
  /*
    mounted 调函数
    methods 写函数 发请求(带参数)
    捞到请求数据赋值给data 展示
  */
  methods: {
    //获取品牌列表的数据 默认展示第一页
    async getPageList(pager = 1) {
      // 改变当前页 传回来的回调参数 是点击的页面
      this.page = pager;
      //解构出数据
      const { page, limit } = this;
      //请求代参数 page,limit 两个参数
      let result = await this.$API.trademark.reqTardeMarkList(page, limit);
      // console.log(result, "trademark");
      // result就是请求回来的品牌信息和改页面的展示信息
      if (result.code == 200) {
        this.total = result.data.total;
        this.list = result.data.records;
      }
    },
    // 分页器的展示条数改变时 触发函数
    handleSizeChange(limit) {
      // 回调参数是每一页的条数
      this.limit = limit;
      this.getPageList();
    },
    // handleCurrentChange(pager){
    //   // 有默认的回调参数
    //   // console.log(pager);
    //   this.page = pager
    //   this.getPageList()
    // }

    //点击添加提示对话框
    showDialog() {
      //显示对话框
      this.dialogFormVisible = true;
      //清理上一次取消的数据
      this.tmForm = { tmName: "", logoUrl: "" };
    },
    //点击修改
    updateTradeMark(row) {
      //选中当前品牌的这一行的信息
      console.log(row, "row------");
      // 点击修改 弹出dailog
      this.dialogFormVisible = true;
      // tmForm 存储的也是服务器返回的信息
      this.tmForm = { ...row };
      // this.tmForm = row;
      // 将服务器请求的信息直接赋值 tmForm 但是也用tmForm收集数据
      // tmForm
      // 点击修改时 将点击的对应的已有信息进行展示 再进行修改 但是直接绑定不行
      // 浅拷贝 让tmFrom接收的并非是服务器返回的那个数据
      // tmForm直接操作了表单数据
    },
    // 点击删除
    deleteTradeMark(row) {
      this.$confirm(`确定要删除${row.tmName}品牌?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        // 点击确定时 会触发这个
        .then(async () => {
          let result = await this.$API.trademark.reqDeleteTradeMark(row.id);
          console.log(result);
          if (result.code == 200) {
            this.$message({
              type: "success",
              message: "删除成功!",
            });
            //再次获取品牌的数据
            //某一页删除了一个 如果还有数据 就荏苒停留再这页
            //如果这一页删没了 那就是 返回上一页
            this.getPageList(this.list.length>1?this.page:this.page-1);
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },

    //图片上传成功
    handleAvatarSuccess(res, file) {
      // res/file是上传成功之后返回前端的数组
      console.log(res, "1------");
      // console.log(file,'2------');
      // 收集图片
      this.imageUrl = URL.createObjectURL(file.raw);
      this.tmForm.logoUrl = res.data;
    },
    //图片上传之前
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    // 添加品牌
    addOrUpdateTradeMark() {
      this.$refs.ruleForm.validate(async (success) => {
        // console.log(success);
        // 所有验证都通过了
        if (success) {
          this.dialogFormVisible = false;
          //1.发请求 添加/修改
          let res = await this.$API.trademark.reqAddOrUpdateTradeMark(
            this.tmForm
          );
          console.log(res, "------");
          if (res.code == 200) {
            //是添加/修改 需要判断一下
            this.$message({
              type: "success",
              message: this.tmForm.id ? "修改成功" : "添加成功",
            });
            // 再次渲染
            // 如果添加品牌 就去第一页
            // 修改品牌就留在修改的那一页
            this.getPageList(this.tmForm.id ? this.page : 1);
          }
        }
      });
    },
  },
};
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
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
</style>

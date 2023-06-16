<template>
  <div>
    <!-- inline 行内表单 一行多个表单元素 -->
    <!-- 三级分类 全局组件 -->
    <el-form :inline="true" :model="cForm" class="demo-form-inline">
      <!-- :model="cForm" 中cForm用于收集form表单数据 -->
      <el-form-item label="一级分类">
        <el-select
          value=""
          placeholder="请选择"
          v-model="cForm.category1Id"
          @change="handler1"
          :disabled="show"
        >
          <!-- v-model收集数据  @change	options框 选中值发生变化时触发handler1函数 -->
          <el-option
            v-for="(c1, index) in list1"
            :key="c1.id"
            :label="c1.name"
            :value="c1.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="二级分类">
        <el-select
          placeholder="请选择"
          v-model="cForm.category2Id"
          @change="handler2"
          :disabled="show"
        >
          <el-option
            v-for="(c2, index) in list2"
            :key="c2.id"
            :label="c2.name"
            :value="c2.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="三级分类">
        <el-select
          placeholder="请选择"
          v-model="cForm.category3Id"
          @change="handler3"
          :disabled="show"
        >
        <!-- 三级分类的change options框内改变需要发请求  -->
          <el-option
            :label="c3.name"
            :value="c3.id"
            v-for="(c3, index) in list3"
            :key="c3.id"
          ></el-option>
        </el-select>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "CategorySelect",
  data() {
    return {
      list1: [],
      //收集三级分类的id
      list2: [],
      list3: [],
      cForm: {
        category1Id: "",
        category2Id: "",
        category3Id: "",
      },
    };
  },
  props:['show'],
  //组件挂载完毕 发请求 获取一级数据的方法
  mounted() {
    this.getCategory1List();
  },
  methods: {
    //获取一级
    async getCategory1List() {
      let res = await this.$API.attr.reqCategory1List();
      // console.log(res,'List');
      if (res.code == 200) {
        this.list1 = res.data;
        //请求的下拉框的数据
      }
    },
    //一级分类的change自定义事件的回调
    //一级分类的option变化时触发
    async handler1() {
      // console.log('一级分类的option变化');
      // 解构出一级分类的id
      //一级分类改变时 二三级分类清空
      this.list2 = []
      this.list3 = []
      this.cForm.category2Id = "";
      this.cForm.category3Id = "";
      const { category1Id } = this.cForm;
      // 自定义事件
      this.$emit("getCategoryId", { categoryId: category1Id, level: 1 });
      //通过一级分类的id，获取二级分类的数据
      let res = await this.$API.attr.reqCategory2List(category1Id);
      // console.log(res, "22222222");
      if (res.code == 200) {
        this.list2 = res.data;
      }
    },
    async handler2() {
      //二级分类改变 三级分类清空
      this.list3 = []
      this.cForm.category3Id = "";
      const { category2Id } = this.cForm;
      this.$emit("getCategoryId", { categoryId: category2Id, level: 2 });
      let res = await this.$API.attr.reqCategory3List(category2Id);
      // console.log(res, "22222222");
      if (res.code == 200) {
        this.list3 = res.data;
      }
    },
    handler3() {
      //获取三级分类的id
      //三级分类确定的时候 再发请求
      const { category3Id } = this.cForm;
      this.$emit("getCategoryId", { categoryId: category3Id, level: 3 });
    },
  },
};
</script>

<style></style>

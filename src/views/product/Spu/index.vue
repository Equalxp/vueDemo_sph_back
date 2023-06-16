<template>
  <div>
    <!-- 三级联动 -->
    <el-card style="margin: 20px 0px">
      <CategorySelect @getCategoryId="getCategoryId"></CategorySelect>
      <!-- 子给父传数据 自定义事件 和 回调 -->
    </el-card>
    <!-- 底部 三部分切换 -->
    <el-card>
      <!-- spu列表结构 -->
      <div v-show="scene == 0">
        <el-button
          type="primary"
          @click="addSpu"
          icon="el-icon-plus"
          :disabled="!category3Id"
        >
          添加SPU</el-button
        >
        <el-table style="width: 100%" border :data="records">
          <el-table-column
            align="center"
            type="index"
            label="序号"
            width="width"
          >
          </el-table-column>
          <el-table-column prop="spuName" label="SPU名称" width="width">
          </el-table-column>
          <el-table-column prop="description" label="SPU描述" width="width">
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <!-- row表示一行的属性数据 -->
              <hint-button
                type="success"
                icon="el-icon-plus"
                size="mini"
                title="添加sku"
                @click="addSku(row)"
              >
              </hint-button>
              <hint-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                title="修改spu"
                @click="updateSpu(row)"
              ></hint-button>
              <hint-button
                type="info"
                icon="el-icon-info"
                size="mini"
                title="查看当前spu全部sku列表"
                @click="handler(row)"
              ></hint-button>
              <el-popconfirm
                @onConfirm="deleteSpu(row)"
                title="这是一段内容确定删除吗？"
              >
                <hint-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  title="删除spu"
                  slot="reference"
                ></hint-button>
              </el-popconfirm>
              <!-- 删除spu -->
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          style="textalign: center"
          :current-page="9"
          :page-sizes="[3, 5, 9]"
          :page-size="limit"
          layout=" prev, pager, next, jumper,->,sizes, total"
          :total="total"
        >
        </el-pagination>
      </div>
      <!--  -->
      <spu-form v-show="scene == 1" @changeScene="changeScene" ref="spu"
        >添加spu</spu-form
      >
      <!-- spuform 是用show操作显示隐藏 不能再子组件mounted发请求 因为只会发一次 该子组件并没有卸载 使用mounted就挂载了一次 -->
      <SkuForm v-show="scene == 2" ref="sku" @changeScene="changeScene"
        >添加sku</SkuForm
      >
      <!-- 点击 -->
      <el-dialog
      :title="`${spu.spuName}的sku列表`"
      :visible.sync="dialogTableVisible"
      :before-close="close"
      >
      <!-- dialog 关闭之前的回调 before-close -->
      <!-- table展示sku列表-->
      <el-table :data="skuList" style="width: 100%" border v-loading="loading">
        <el-table-column prop="skuName" label="名称" width="width">
        </el-table-column>
        <el-table-column prop="price" label="价格" width="width">
        </el-table-column>
        <el-table-column prop="weight" label="重量" width="width">
        </el-table-column>
        <el-table-column label="默认图片" width="width">
           <template slot-scope="{row,$index}">
              <img :src="row.skuDefaultImg" alt="" style="width:100px;height:100px;">
           </template>
        </el-table-column>
      </el-table>
    </el-dialog>
    </el-card>
  </div>
</template>

<script>
import SpuForm from "./SpuForm";
import SkuForm from "./SkuForm";
export default {
  name: "Spu",
  data() {
    return {
      category1Id: "",
      category2Id: "",
      category3Id: "",
      // isShowTable:true,
      //三级联动的可操作性
      show: true,
      page: 1,
      limit: 3,
      //存储sku列表数据
      records: [],
      //分页器一共数据
      total: 0,
      //0 展示spu 1 添加/修改spu 2添加sku
      scene: 0,
      //查看的对话框的显示/隐藏
      dialogTableVisible: false,
      //
      spu:{},
      skuList:[],
      loading:true
    };
  },
  methods: {
    // 自定义事件 子组件对应的id穿递给父组件
    // this.$emit("getCategoryId", { categoryId: category1Id, level: 1 });
    getCategoryId({ categoryId, level }) {
      if (level == 1) {
        this.category1Id = categoryId;
        //改变一级id之后 以前的二三级都需要清空
        this.category2Id = "";
        this.category3Id = "";
      } else if (level == 2) {
        this.category2Id = categoryId;
        this.category3Id = "";
      } else {
        this.category3Id = categoryId;
        // 得到三级id的时候 发请求 请求数据
        this.getSpuList();
      }
    },
    //获取spu数据的请求
    async getSpuList() {
      const { page, limit, category3Id } = this;
      let res = await this.$API.spu.reqSpuList(page, limit, category3Id);
      console.log(res);
      if (res.code == 200) {
        this.records = res.data.records;
        this.total = res.data.total;
      }
    },
    //点击分页器 跳转到相应页面
    //其实可以节省一个函数
    handleCurrentChange(page) {
      //参数page就是点击的页数
      this.page = page;
      // 再次获取最新的页面
      this.getSpuList();
    },
    //分页器的展示条数变化
    handleSizeChange(limit) {
      this.limit = limit;
      this.getSpuList();
    },
    changeScene(scene) {
      //自定义事件的回调 取消使scene为0
      this.scene = scene;
      //修改之后服务器那边的数据变化了 还是要重新刷新请求 页面要在修改的这一页
      this.getSpuList(this.page);
    },
    //添加spu的回调
    addSpu() {
      this.scene = 1;
      //通知子组件发请求
      this.$refs.spu.addSpuData(this.category3Id);
      //把3id送过去
    },
    //点击修改按钮 修改spu的回调
    updateSpu(row) {
      this.scene = 1;
      // 在父组件里面获取spuform子组件
      // console.log(this.$refs.spu,'----------');
      //当前spu的id->row
      this.$refs.spu.initSpuData(row);
    },
    //删除spu的接口
    async deleteSpu(row) {
      //点击确定要发请求
      let res = await this.$API.spu.reqDeleteSpu(row.id);
      console.log(res, "33333333333");
      if (res.code == 200) {
        this.$message({ type: "success", message: "删除成功" });
        // 删除了 去哪一页
        //这一页删没了 就去上一页
        this.getSpuList(this.records.length > 1 ? this.page : this.page - 1);
      }
    },
    //添加sku的回调
    addSku(row) {
      //1.换场景
      this.scene = 2;
      //2.点击这个就直接发请求
      this.$refs.sku.getData(this.category1Id, this.category2Id, row);
      console.log(row, "1111113213");
    },
    //点击查看的回调
    async handler(spu){
      //row就是spu数据
      this.dialogTableVisible = true
      this.spu = spu
      // console.log(spu,'spuuuuuuuuuuuuuu');
      // 发请求捞数据
      let result = await this.$API.spu.reqSkuList(spu.id)
      if(result.code == 200){
        this.skuList = result.data
        //数据请求回来以后就不转圈圈了
        this.loading = false
      }
    },
    //关闭对话框的回调
    close(done){
      //loading属性再次变为真 下次点击查看就可以显示loading
      this.loading = true;
      //清除sku列表的数据 点击x的时候吧对话框数据清除了
      this.skuList = [];
      //管理对话框
      done();
    }
  },
  components: {
    SpuForm,
    SkuForm,
  },
};
</script>

<style></style>

<template>
  <div>
    <el-form ref="form" label-width="80px">
      <el-form-item label="SPU名称">{{ spu.spuName }}</el-form-item>
      <el-form-item label="SKU名称">
        <el-input placeholder="SKU名称" v-model="skuInfo.skuName"></el-input>
      </el-form-item>
      <el-form-item label="价格（¥）">
        <el-input
          placeholder="价格（¥）"
          type="number"
          v-model="skuInfo.price"
        ></el-input>
      </el-form-item>
      <el-form-item label="重量（kg）">
        <el-input placeholder="重量（kg）" v-model="skuInfo.weight"></el-input>
      </el-form-item>
      <el-form-item label="规格描述">
        <el-input type="textarea" rows="4" v-model="skuInfo.skuDesc"></el-input>
      </el-form-item>
      <el-form-item label="平台属性">
        <el-form :inline="true" ref="form" label-width="80px">
          <el-form-item
            :label="attr.attrName"
            v-for="(attr, index) in attrInfoList"
            :key="attr.id"
          >
            <!-- attrInfoList是请求回来的 将展示的数据变成动态的 -->
            <el-select placeholder="请选择" v-model="attr.attrIdAndValueId">
              <!-- v-model的值为当前被选中的el-option的 value 属性值 -->
              <!-- 收集到平台属性对象上 -->
              <el-option
                :label="attrValue.valueName"
                :value="`${attr.id}:${attrValue.id}`"
                v-for="(attrValue, index) in attr.attrValueList"
                :key="attrValue.id"
              ></el-option>
              <!-- label是分组的组名  value / v-model绑定值-->
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-form :inline="true" ref="form" label-width="80px">
          <el-form-item
            :label="saleAttr.saleAttrName"
            v-for="(saleAttr, index) in spuSaleAttrList"
            :key="saleAttr.id"
          >
          <!-- 遍历不同的销售属性 为不同的select框 -->
            <el-select placeholder="请选择" v-model="saleAttr.attrIdAndValueId">
              <!-- 将属性id和属性值的id 保存在saleAttr.attrIdAndValueId里面 -->
              <el-option
                :label="saleAttrValue.saleAttrValueName"
                :value="`${saleAttr.id}:${saleAttrValue.id}`"
                v-for="(saleAttrValue, index) in saleAttr.spuSaleAttrValueList"
                :key="saleAttrValue.id"
              ></el-option>
              <!-- 遍历销售属性中 的 不同销售属性值 -->
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="图片列表">
        <el-table
          style="width: 100%"
          border
          :data="spuImageList"
          @selection-change="handleSelectionChange"
        >
          <!-- 当选择项改变时 触发 handleSelectionChange -->
          <el-table-column type="selection" prop="prop" width="80">
          </el-table-column>
          <el-table-column prop="prop" label="图片" width="width">
            <template slot-scope="{ row, $index }">
              <img :src="row.imgUrl" style="width: 100px; height: 100px" />
            </template>
          </el-table-column>
          <el-table-column prop="imgName" label="名称" width="width">
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <!-- 两个按钮进行切换 -->
            <template slot-scope="{ row, $index }">
              <el-button
                type="primary"
                @click="changeDefault(row, spuImageList)"
                v-if="row.isDefault == 0"
                >设置默认</el-button
              >
              <el-button v-else>默认</el-button>
              <!-- 排他思想 当前值 和 所有值 -->
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="save">保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "SkuForm",
  data() {
    return {
      //请求回来的图片数组
      spuImageList: [],
      //存储销售属性
      spuSaleAttrList: [],
      //存储平台属性的数据
      attrInfoList: [],
      spu: {},
      //收集sku字段
      skuInfo: {
        //第一类收集的数据：父组件给的数据
        category3Id: 0,
        spuId: 0,
        tmId: 0,
        //第二类：需要通过form表单数据双向绑定v-model收集
        skuName: "",
        price: 0,
        weight: "",
        skuDesc: "",
        //第三类：需要自己书写代码
        //默认图片
        skuDefaultImg: "",
        //收集图片的字段
        skuImageList: [
          // {
          //   id: 0,
          //   imgName: "string",
          //   imgUrl: "string",
          //   isDefault: "string",
          //   skuId: 0,
          //   spuImgId: 0,
          // },
        ],
        //平台属性
        skuAttrValueList: [
          // {
          //   attrId: 0,//平台属性的id
          //   valueId: 0,//平台属性值的id
          // },
        ],
        //销售属性
        skuSaleAttrValueList: [
          // {
          //   id: 0,
          //   saleAttrId: 0,
          //   saleAttrName: "string",
          //   saleAttrValueId: 0,
          //   saleAttrValueName: "string",
          //   skuId: 0,
          //   spuId: 0,
          // },
        ],
      },
      //用户选中图片收集确实isDefault字段
      imageList: [],
    };
  },
  methods: {
    //点击添加sku时发请求
    async getData(category1Id, category2Id, spu) {
      //收集父组件给的数据
      this.skuInfo.category3Id = spu.ategory3Id;
      this.skuInfo.spuId = spu.id;
      this.skuInfo.tmId = spu.tmId;
      this.spu = spu;
      //获取图片的数据
      let res1 = await this.$API.spu.reqSpuImageList(spu.id);
      // console.log(res1,'9099999999999');
      if (res1.code == 200) {
        //请求回来的数据是没有 isDefault 这个属性的
        let list = res1.data;
        list.forEach((item) => {
          item.isDefault = 0;
        });
        this.spuImageList = list;
      }
      //获取销售属性的数据
      let res2 = await this.$API.spu.reqSpuSaleAttrList(spu.id);
      // console.log(res2, "res2222222222222");
      if (res2.code == 200) {
        this.spuSaleAttrList = res2.data;
      }
      //获取平台属性的数据
      let res3 = await this.$API.spu.reqAttrInfoList(
        category1Id,
        category2Id,
        spu.category3Id
      );
      // console.log(res3, "res33333");
      if (res3.code == 200) {
        this.attrInfoList = res3.data;
      }
    },
    // 表格选择变化时触发函数
    handleSelectionChange(params) {
      // 获取用户选中图片的信息
      this.imageList = params;
    },
    // 排它操作
    changeDefault(row, spuImageList) {
      //全部变成0 自己点击的那个变成1
      spuImageList.forEach((item) => {
        item.isDefault = 0;
      });
      row.isDefault = 1;
      // console.log(row,'rowwwwwwwwwwwwww');
      //收集设为默认的图片
      this.skuInfo.skuDefaultImg = row.imgUrl;
    },
    // 取消按钮
    cancel() {
      //1.改变场景
      this.$emit("changeScene", 0);
      //清除数据
      Object.assign(this._data,this.$options.data());
    },
    // 保存按钮
    async save() {
      //整理参数
      //整理平台属性
      const { attrInfoList, skuInfo, spuSaleAttrList, imageList } = this;
      //整理平台属的数据
      skuInfo.skuAttrValueList = attrInfoList.reduce((prev, item) => {
        if (item.attrIdAndValueId) {
          const [attrId, valueId] = item.attrIdAndValueId.split(":");
          prev.push({ attrId, valueId });
        }
        return prev;
      }, []);
      //整理销售属性
      skuInfo.skuSaleAttrValueList = spuSaleAttrList.reduce((prev, item) => {
        if (item.attrIdAndValueId) {
          const [saleAttrId, saleAttrValueId] =
            item.attrIdAndValueId.split(":");
          prev.push({ saleAttrId, saleAttrValueId });
        }
        return prev;
      }, []);
      //整理图片的数据
      skuInfo.skuImageList = imageList.map((item) => {
        return {
          imgName: item.imgName,
          imgUrl: item.imgUrl,
          isDefault: item.isDefault,
          spuImgId: item.id,
        };
      });
      //发请求
      let result = await this.$API.spu.reqAddSku(skuInfo);
      if (result.code == 200) {
        this.$message({ type: "success", message: "添加SKU成功" });
        this.$emit("changeScenes", 0);
      }
      this.$emit("changeScene", 0);
    },
  },
};
</script>

<style></style>

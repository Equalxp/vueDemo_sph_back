<template>
  <div>
    <el-form ref="form" label-width="80px" :model="spu">
      <!-- 将数据收集到spu -->
      <el-form-item label="spu名称">
        <el-input placeholder="spu名称" v-model="spu.spuName"></el-input>
        <!-- v-model 1.会显示spu.spuName的数据 2.input框输入什么就会改变data.spu.spuName的数据 -->
      </el-form-item>
      <el-form-item label="品牌">
        <el-select placeholder="请选择品牌" v-model="spu.tmId" value="">
          <el-option
            v-for="(tm, index) in tradeMarkList"
            :key="tm.id"
            :value="tm.id"
            :label="tm.tmName"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="SPU描述">
        <el-input
          type="textare"
          v-model="spu.description"
          placeholder="描述"
          rows="4"
        ></el-input>
      </el-form-item>
      <el-form-item label="SPU图片">
        <!-- 照片上传服务器的业务 
          action是地址
          on-preview 照片预览触发
          on-remove 删除触发
          file-list 要展示的数组 数据里面务必要有name/url
        -->
        <el-upload
          action="/dev-api/admin/product/fileUpload"
          list-type="picture-card"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :file-list="spuImageList"
          :on-success="handlerSuccess"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="" />
        </el-dialog>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-select
          :placeholder="`还有${unSelectSaleAttr.length}未选择`"
          v-model="attrId"
        >
        <!-- unSelectSaleAttr是计算属性 算出来没有选择的 -->
          <el-option
            :label="unselect.name"
            v-for="(unselect, index) in unSelectSaleAttr"
            :value="`${unselect.id}:${unselect.name}`"
            :key="unselect.id"
          ></el-option>
          <!-- :value收集数据 -->
        </el-select>
        <el-button type="primary" icon="el-icon-plus" @click="addSaleAttr" :disabled="!attrId">添加销售属性</el-button>
      </el-form-item>
      <!-- 展示當前spu数据 -->
      <el-table style="width: 100%" border :data="spu.spuSaleAttrList">
        <el-table-column type="index" label="序号" width="80px" align="center">
        </el-table-column>
        <el-table-column prop="saleAttrName" label="属性名" width="width">
        </el-table-column>
        <el-table-column prop="prop" label="属性值名称列表" width="width">
          <template slot-scope="{ row, $index }">
            <!-- el-tag:用户展示已有属性值列表的数据的 -->
            <!-- {{ row }} -->
            <!-- //row是销售属性/】 -->
            <el-tag
              :key="tag.id"
              v-for="(tag, index) in row.spuSaleAttrValueList"
              closable
              :disable-transitions="false"
              @close=" row.spuSaleAttrValueList.splice(index,1)"
              >
              <!-- close要删除这个数组里面的数据 所以要row.spuSaleAttrValueList写全 -->
              {{ tag.saleAttrValueName }}
            </el-tag>
            <!-- @close="row.spuSaleAttrValueList.splice(index, 1)" -->
            <!-- 底下的解构可以当中咱们当年的span与input切换 -->
            <!--  @keyup.enter.native="handleInputConfirm"  -->
            <el-input
              class="input-new-tag"
              v-if="row.inputVisible"
              v-model="row.inputValue"
              ref="saveTagInput"
              size="small"
              @blur="handeInputConfirm(row)"
            >
              <!-- @blur="handleInputConfirm(row)" -->
            </el-input>
            <!-- @click="showInput" -->
            <el-button
              v-else
              class="button-new-tag"
              size="small"
              @click="addSaleAttrValue(row)"
              >添加</el-button
            >
          </template>
        </el-table-column>
        <el-table-column prop="prop" label="操作" width="width">
          <template slot-scope="{ row, $index }">
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="spu.spuSaleAttrList.splice($index, 1)"
            ><!-- 删除一行的属性 -->
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-button type="primary" @click="addOrUpdateAttr()">保存</el-button>
      <el-button @click="cancel()">取消</el-button>
      <!-- 点击取消让父组件里面scene为0 -->
    </el-form>
  </div>
</template>

<script>
export default {
  name: "SpuForm",
  data() {
    return {
      dialogImageUrl: "",
      dialogVisible: false,
      //存储spu信息
      spu: {
        //三级分类的id
        category3Id: 0,
        //描述
        description: "",
        //spu名称
        spuName: "",
        //平台的id
        tmId: "",
        //收集SPU图片的信息
        spuImageList: [
          // {
          //   id: 0,
          //   imgName: "string",
          //   imgUrl: "string",
          //   spuId: 0,
          // },
        ],
        //平台属性与属性值收集
        spuSaleAttrList: [
          // {
          //   baseSaleAttrId: 0,
          //   id: 0,
          //   saleAttrName: "string",
          //   spuId: 0,
          //   spuSaleAttrValueList: [
          //     {
          //       baseSaleAttrId: 0,
          //       id: 0,
          //       isChecked: "string",
          //       saleAttrName: "string",
          //       saleAttrValueName: "string",
          //       spuId: 0,
          //     },
          //   ],
          // },
        ],
      },
      // 初始是空 然后发请求获取数据 但是收集的数据也是放在这里
      // spu的添加 没向服务器发请求 数据收集到这里
      spuImageList: [],//服务器请求回来spu数据用于展示的
      tradeMarkList: [],//服务器请求回来spu数据用于展示的
      saleAttrList: [], //销售属性
      attrId: "", //收集未选择的销售属性的id
    };
  },
  computed: {
    //计算未选择的销售属性
    unSelectSaleAttr() {
      //一共三个销售属性  data.saleAttrList 是发请求返回的
      //当前spu的属性 data.spu.spuSaleAttrList
      return this.saleAttrList.filter((item) => {
        return this.spu.spuSaleAttrList.every((item1) => {
          // 返回真 给filter 就说明这个元素要返回给filter的新数组
          return item.name != item1.saleAttrName;
        });
      });
    },
  },
  methods: {
    //照片墙删除某一个图片的时候会触发
    handleRemove(file, fileList) {
      //file参数:代表的是删除的那个张图片
      //fileList:照片墙删除某一张图片以后，剩余的其他的图片
      // console.log(file, fileList,22222);
      //收集照片墙图片的数据
      //对于已有的图片【照片钱中显示的图片：有name、url字段的】，因为照片墙显示数据务必要有这两个属性
      //对于服务器而言，不需要name、url字段，将来对于有的图片的数据在提交给服务器的时候，需要处理的
      this.spuImageList = fileList;
    },
    //照片墙预览的回调 不需要数据 就是将照片放大而已
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      //对话框显示
      this.dialogVisible = true;
    },
    //照片墙图片上传成功的回调
    handlerSuccess(response, file, fileList) {
      //收集图片的信息
      this.spuImageList = fileList;
    },
   //添加新的销售属性
    addSaleAttr() {
      //已经收集需要添加的销售属性的信息
      //把收集到的销售属性数据进行分割
      const [baseSaleAttrId, saleAttrName] = this.attrId.split(":");
      //向SPU对象的spuSaleAttrList属性里面添加新的销售属性
      //添加新的销售属性 有属性id 属性名 属性的值..组成的一个对象
      let newSaleAttr = {
        baseSaleAttrId,
        saleAttrName,
        spuSaleAttrValueList: [],
      };
      //添加新的销售属性
      this.spu.spuSaleAttrList.push(newSaleAttr);
      //清空数据
      this.attrId = "";
    },
    //新增销售属性值 添加按钮
    addSaleAttrValue(row){
      //1.显示button->input
      // row.inputVisible = true //不好使 不是响应式属性
      // inputVisible控制button和input的切换
      this.$set(row,'inputVisible',true)
      // 通过响应式数据收集新增的销售属性值
      this.$set(row,'inputValue','')

    },
    //blur 
    handeInputConfirm(row){
      //1.修改字段为fasle 就显示 这个控制切换的放在了row里面 不能放在data用一个属性去控制
      // 不然会都显示button和input切换
      row.inputVisible = false
      //2.收集数据 新增的销售属性值
      //整理数据
      //3.不为空
      const {baseSaleAttrId,inputValue} = row
      if(inputValue.trim()==''){
        this.$message('输入值不能为空')
        return  
      }
      //4.不能重复添加 用some更加合理 重复为false
      let flag = row.spuSaleAttrValueList.every(item=>{
        return item.saleAttrValueName!=inputValue
        // List里面的每一项都要满足return的判断 才返回真
      })
      // console.log(flag,'0000000');
      if(flag == false) return
      let newSaleAttrValue = {baseSaleAttrId,saleAttrValueName:inputValue}
      row.spuSaleAttrValueList.push(newSaleAttrValue)
    },
    //修改和添加的取消按钮
    cancel(){
      this.$emit('changeScene', 0)
      //回显数据的清除
      Object.assign(this._data,this.$options.data())
      //合并对象 this._data可以操作data中的响应式数据
      // console.log(this.$options.data(),'6666666');
      //this.$options.data() 返回的是 data里面的值 都是空
    },
    //保存按钮的回调
    async addOrUpdateAttr(){
      // alert(111)
      //1.整理参数-照片墙 要imgName,imgUrl 放到spu里面带给服务器
      //新增的照片处理
      this.spu.spuImageList = this.spuImageList.map(item=>{
        return {
          imgName:item.name,
          imgUrl:(item.response&&item.response.data)||item.imgUrl
        }
      })
      //2.发请求
      let res = await this.$API.spu.reqAddOrUpdateSpu(this.spu)
      // console.log(res,'999999999');
      if(res.code==200){
        //提示
        this.$message({type:'success',message:'保存成功'})
        //跳转页面 通知父组件到scene=1
        this.$emit('changeScene', 0)
        // 要判断修改时要跳到自己 添加回到第一页
      }
      //也要清除数据
      Object.assign(this._data,this.$options.data())
    },
    //点击添加spu按钮 发请求
    async addSpuData(id){
      // console.log('添加');
      // 添加也要提供一些按钮供选择
      let tradeMarkResult = await this.$API.spu.reqTradeMarkList();
      if (tradeMarkResult.code == 200) {
        this.tradeMarkList = tradeMarkResult.data;
      }
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      if (saleResult.code == 200) {
        this.saleAttrList = saleResult.data;
      }
      //sku里面没有对应的id 要父组件点击添加时传过来
      this.spu.category3Id = id
    },
    // 1.初始化spu的数据 拿到数据进行展示
    // 父组件点击修改按钮时 传对应点击的那一条spu的信息作为参数-> 发请求-> 展示信息
    async initSpuData(spu) {
      // console.log('发请求',spu);
      // 获取SPU信息的数据
      let res1 = await this.$API.spu.reqSpu(spu.id);
      console.log(res1,'spu的数据');
      if (res1.code == 200) {
        this.spu = res1.data;
      }
      // 获取品牌的信息
      let tradeMarkResult = await this.$API.spu.reqTradeMarkList();
      if (tradeMarkResult.code == 200) {
        this.tradeMarkList = tradeMarkResult.data;
      }
      //获取spu图片的数据
      let spuImageResult = await this.$API.spu.reqSpuImageList(spu.id);
      if (spuImageResult.code == 200) {
        let listArr = spuImageResult.data;
        //由于照片墙显示图片的数据需要数组，数组里面的元素需要有name与url字段
        //需要把服务器返回的数据进行修改
        listArr.forEach((item) => {
          //foreach添加属性
          item.name = item.imgName;
          item.url = item.imgUrl;
        });
        //把整理好的数据赋值给
        this.spuImageList = listArr;
      }
      //获取平台全部的销售属性
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      if (saleResult.code == 200) {
        this.saleAttrList = saleResult.data;
      }
    },
  },
  mounted() {},
};
</script>

<style>
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>

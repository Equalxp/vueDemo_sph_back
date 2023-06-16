<template>
  <div>
    <el-card style="margin: 20px 0px">
      <CategorySelect @getCategoryId="getCategoryId" :show="!isShowTable">
        <!-- 子给父传数据 自定义事件 和 回调 -->
      </CategorySelect>
    </el-card>
    <el-card>
      <div v-show="isShowTable">
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          @click="addAttr"
          >添加属性</el-button
        >
        <el-table style="width: 100%" border :data="attrList">
          <el-table-column prop="index" label="序号" width="80" align="center">
          </el-table-column>
          <el-table-column prop="attrName" label="属性名称" width="150">
          </el-table-column>
          <el-table-column prop="prop" label="属性值列表" width="width">
            <template slot-scope="{ row, $index }">
              <!-- row是数组中的每一个元素 -->
              <el-tag
                type="success"
                v-for="(attrValue, index) in row.attrValueList"
                :key="attrValue.id"
                >{{ attrValue.valueName }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="150">
            <template slot-scope="{ row, $index }">
              <el-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                @click="updateAttr(row)"
              ></el-button>
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <!-- 修改属性的结构 -->
      <div v-show="!isShowTable">
        <el-form :inline="true" ref="form" :model="attrInfo" label-width="80px">
          <!-- 
            :model="attrInfo"就是收集表单数据 form表单里面的输入框 
            输入的数据v-model都双向绑定到了attrInfo
           -->
          <el-form-item label="属性名">
            <el-input placeholder="请输入属性名" v-model="attrInfo.attrName">
              <!-- v-model="attrInfo.attrName" 将input输入的东西都放到了"attrInfo.attrName"里面 -->
            </el-input>
          </el-form-item>
        </el-form>
        <el-button
          type="primary"
          @click="addAttrValue"
          icon="el-icon-plus"
          :disabled="!attrInfo.attrName"
          >添加属性值</el-button
        >
        <!-- :disabled="!attrInfo.attrName" -->
        <!-- 要先数入属性名才能添加属性值  -->
        <el-button @click="isShowTable = true">取消</el-button>
        <el-table
          style="width: 100%; margin: 20px 0px"
          border
          :data="attrInfo.attrValueList"
        >
          <el-table-column align="center" type="index" label="序号" width="80">
          </el-table-column>
          <el-table-column prop="prop" label="属性值名称" width="width">
            <template slot-scope="{ row, $index }">
              <el-input
                v-model="row.valueName"
                placeholder="请输入属性值名称"
                size="mini"
                v-if="row.flag"
                @blur="toLook(row)"
                @keyup.native.enter="toLook(row)"
                :ref="$index"
              ></el-input>
              <!-- input和span的来回显示 -->
              <span
                v-else
                @click="toEdit(row, $index)"
                style="display: block"
                >{{ row.valueName }}</span
              >
              <!-- 这里span是个块元素 没有单独占一行 点击空白的地方不会触发click -->
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <el-popconfirm
                :title="`确定删除${row.valueName}?`"
                @onConfirm="deleteAttrValue($index)"
              >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  slot="reference"
                ></el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <!-- :disabled="attrInfo.attrValueList.length<1" -->
        <el-button type="primary" @click="addOrUpdateAttr" 
        :disabled="attrInfo.attrValueList.length<1">
          <!-- 没有数据的时候不能保存 -->
          保存
        </el-button>
        <el-button @click="isShowTable = true">取消</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
import cloneDeep from "lodash/cloneDeep";
export default {
  name: "Attr",
  data() {
    return {
      // input和span的轮流展示标志
      // flag:true, //放在这里就不能区分单个显示模式
      category1Id: "",
      category2Id: "",
      category3Id: "",
      //接收平台属性的字段
      attrList: [],
      //table的显示与隐藏 //三级联动是否可操作
      isShowTable: true,
      //el-form 收集 新增数据/修改数据使用
      attrInfo: {
        attrName: "", //属性名
        attrValueList: [
          //属性值，因为属性值可以有多个因此用数组，每一个属性值都是一个对象需要attrId，valueName
        ],
        categoryId: 0, //三级分类的id
        categoryLevel: 3, //因为服务器也需要区分几级id
      },
    };
  },
  methods: {
    // 自定义事件回调 子给父传数据 让父组件区分是几级id
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
        this.getAttrList();
      }
    },
    //获取平台属性 一行的数据
    async getAttrList() {
      //获取分类的id
      const { category1Id, category2Id, category3Id } = this;
      let res = await this.$API.attr.reqAttrList(
        category1Id,
        category2Id,
        category3Id
      );
      console.log(res, "idididid");
      if (res.code == 200) {
        this.attrList = res.data;
        // data是一行的数据 属性名 属性值 id level categoryid
      }
    },
    //新增属性值
    addAttrValue() {
      this.attrInfo.attrValueList.push({
        // attrId：是你相应的属性的id，目前而言我们是添加属性的操作，还没有相应的属性的id，目前而言带给服务器的id为undefined
        // 新增时也要带上attrid 就是attrInfo.id
        attrId: this.attrInfo.id,
        // 对于修改某一个属性的时候，可以在已有的属性值基础之上新增新的属性值
        //（新增属性值的时候，需要把已有的属性的id带上）
        // 相应属性值的名称
        valueName: "",
        // 加一个flag属性 来回切换span和input
        flag: true,
      });
      this.$nextTick(() => {
        this.$refs[this.attrInfo.attrValueList.length - 1].focus();
      });
    },
    //添加属性的回调
    addAttr() {
      this.isShowTable = false;
      //取消按钮 数据回显
      //清除上一次操作的数据 (因为点击的是取消要 清除回写)
      this.attrInfo = {
        attrName: "", //属性名
        attrValueList: [
          //属性值，因为属性值可以有多个因此用数组，每一个属性值都是一个对象需要attrId，valueName
        ],
        // 收集id
        categoryId: this.category3Id, //三级分类的id
        categoryLevel: 3, //因为服务器也需要区分几级id
      };
    },
    //修改某个属性
    updateAttr(row) {
      // console.log(row, "0000000");
      this.isShowTable = false;
      //将选中的属性 赋值给attrInfo
      //展示要修改的数据
      //不能直接赋值操作 不能用浅拷贝 因为对象里面还有数组
      // this.attrInfo = row
      this.attrInfo = cloneDeep(row);
      // attr.info 是拿来展示的数据 是row给的 row就是attrList，1id，2id，3id给的
      //attr.info <---赋值 attrList 指向同一个对象
      //修改属性时添加flag这个属性
      this.attrInfo.attrValueList.forEach((item) => {
        // 添加了字段 但是没有效果 因为flag不是响应式的数组
        // item.flag = false
        // Vue.set用于给一个响应式对象添加一个新的响应式属性
        // Vue无法探测出新增的普通属性 这种.追加的方法不能检测 数组方法可以
        this.$set(item, "flag", false);
        // 对象 属性 属性值
      });
    },
    toLook(row) {
      if (row.valueName.trim() == "") {
        this.$message("不准只输入空格");
        return;
      }
      //新增的属性值不能与已有的属性值重复
      let isRepeat = this.attrInfo.attrValueList.some((item) => {
        //row是新增的 刚刚已经添加有的是attrValueList
        //将row去除
        if (row !== item) {
          return row.valueName == item.valueName;
          // return是判断条件 条件是两个valueName相等
          // 只要有一个item符合valueName相等 就退出 返回一个true
        }
      });
      if (isRepeat) return;
      row.flag = false;
    },
    toEdit(row, index) {
      //点击span的时候换成input同时让input聚焦
      row.flag = true;
      //点击完span input才出来 是不能立马就获取到this.$refs[index]的值的
      //需要注意：点击span的时候，切换为input变为编辑模式，但是需要注意，
      //对于浏览器而言，页面重绘与重拍耗时间的
      //点击span的时候，重绘重拍一个input它是需要耗费事件
      //因此我们不可能一点击span立马获取到input
      //$nextTick,当节点渲染完毕了，会执行一次
      // console.log(this.$refs[index],'-------')
      this.$nextTick(() => {
        this.$refs[index].focus();
      });
    },
    //删除要添加的属性和属性值 气泡确认框的回调
    deleteAttrValue(index) {
      //从数组中删除选中的index对应的属性值
      this.attrInfo.attrValueList.splice(index, 1);
    },
    async addOrUpdateAttr() {
      //整理参数时 空的不能提交上去
      //自己添加的flag也要去除
      this.attrInfo.attrValueList = this.attrInfo.attrValueList.filter(
        (item) => {
          //过滤不为空的
          if (item.valueName != "") {
            //删除flag属性
            delete item.flag;
            return true;
          }
        }
      );
      try {
        //发请求
        let res = await this.$API.attr.reqAddOrUpdateAttr(this.attrInfo);
        console.log(res);
        this.isShowTable = true
        //显示展示列表
        this.$message({type:'success',message:'保存成功s'})
        this.getAttrList()
        //再次获取数据
      } catch (error) {
        alert(error.message);
      }
    },
  },
};
</script>

<style></style>
 
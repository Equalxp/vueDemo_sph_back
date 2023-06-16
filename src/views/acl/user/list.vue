<template>
  <div class="app-container">
    <el-form inline>
      <!-- 表单元素 -->
      <el-form-item>
         <el-input v-model="tempSearchObj.username" placeholder="用户名" />
      </el-form-item>
      <!-- 查询与情况的按钮 -->
      <el-button type="primary" icon="el-icon-search" @click="search">查询</el-button>
      <el-button type="default" @click="resetSearch">清空</el-button>
    </el-form>

    <div style="margin-bottom: 20px">
      <!-- 添加与批量添加按钮 -->
      <el-button type="primary" @click="showAddUser">添 加</el-button>
      <el-button type="danger" @click="revomveUsers" :disabled="selectedIds.length===0"
        >批量删除</el-button>
    </div>

    <!-- table表格：展示用户信息的地方 -->
    <el-table
      border
      stripe
      v-loading="listLoading"
      :data="users"
      @selection-change="handleSelectionChange">
      <!-- 当选择项发生改变时 触发selection-change -->
      <el-table-column
        type="selection"
        width="55" />

      <el-table-column
        type="index"
        label="序号"
        width="80"
        align="center"
      />

      <el-table-column prop="username" label="用户名" width="150" />
      <el-table-column prop="nickName" label="用户昵称" />
      <el-table-column prop="roleName" label="权限列表" />
      
      <el-table-column prop="gmtCreate" label="创建时间" width="180"/>
      <el-table-column prop="gmtModified" label="更新时间" width="180"/>

      <el-table-column label="操作" width="230" align="center">
        <template slot-scope="{row}">
          <HintButton type="info" size="mini" icon="el-icon-user-solid" title="分配角色"
            @click="showAssignRole(row)"/>
            <!-- row是角色的属性和属性值 -->
          <HintButton type="primary" size="mini" icon="el-icon-edit" title="修改用户"
            @click="showUpdateUser(row)"/>
            <!-- 修改逻辑1  -->
          <el-popconfirm :title="`确定删除 ${row.username} 吗?`" @onConfirm="removeUser(row.id)">
            <HintButton style="margin-left:10px" slot="reference" type="danger" size="mini" icon="el-icon-delete" 
              title="删除用户"/>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页器 -->
    <el-pagination
      :current-page="page"
      :total="total"
      :page-size="limit"
      :page-sizes="[3, 10, 20, 30, 40, 50, 100]"
      style="padding: 20px 0;"
      layout="prev, pager, next, jumper, ->, sizes, total"
      @current-change="getUsers"
      @size-change="handleSizeChange"
    />
    <!-- 对话框的结构 -->
    <el-dialog :title="user.id ? '修改用户' : '添加用户'" :visible.sync="dialogUserVisible">
      <!-- 修改逻辑2对话框显示 -->
      <!-- 对话框的表单结构 user有id就是修改 没有就是添加用户 -->
      <el-form ref="userForm" :model="user" :rules="userRules" label-width="120px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="user.username"/>
        </el-form-item>
        <el-form-item label="用户昵称">
          <el-input v-model="user.nickName"/>
        </el-form-item>
        <!-- 修改逻辑2 v-model双向绑定数据 -->
        <!-- 修改只弹出用户名和用户昵称 添加就多显示一个用户密码 -->
        <el-form-item v-if="!user.id" label="用户密码" prop="password">
          <el-input v-model="user.password"/>
        </el-form-item>
      </el-form>
      <!-- 对话框的取消确定按钮 -->
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <!-- 点击确定发请求更新或者添加 -->
        <el-button :loading="loading" type="primary" @click="addOrUpdate">确 定</el-button>
        <!-- 修改逻辑3 点击确定 执行回调 -->
      </div>
    </el-dialog>

    <el-dialog title="设置角色" :visible.sync="dialogRoleVisible" :before-close="resetRoleData">
      <!-- 表单结构 用户名和角色列表（选择） -->
      <el-form label-width="80px">
        <el-form-item label="用户名">
          <el-input disabled :value="user.username"></el-input>
        </el-form-item>

        <el-form-item label="角色列表">
          <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">全选</el-checkbox>
          <!-- indeterminate 全选时为true 否则为false -->
          <div style="margin: 15px 0;"></div>
          <el-checkbox-group v-model="userRoleIds" @change="handleCheckedChange">
            <!-- handleCheckedChange 普通选择  -->
            <el-checkbox v-for="role in allRoles" :key="role.id" :label="role.id">{{role.roleName}}</el-checkbox>
          </el-checkbox-group>
        </el-form-item>
      </el-form>

      <div slot="footer">
        <el-button :loading="loading" type="primary" @click="assignRole">保存</el-button>
        <el-button @click="resetRoleData">取消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import cloneDeep from 'lodash/cloneDeep'

export default {
  name: 'AclUserList',

  data () {
    return {
      listLoading: false, // 是否显示列表加载的提示
      searchObj: { // 包含请求搜索条件数据的对象
        username: ''
      },
      tempSearchObj: { // 收集搜索条件输入的对象
        username: ''
      },
      selectedIds: [], // 所有选择的user的id的数组
      users: [], // 当前页的用户列表
      page: 1, // 当前页码
      limit: 3, // 每页数量
      total: 0, // 总数量
      user: {}, // 当前要操作的user
      dialogUserVisible: false, // 是否显示用户添加/修改的dialog
      // :rules 表单验证规则
      userRules: { // 用户添加/修改表单的校验规则
        username: [
          { required: true, message: '用户名必须输入' },
          { min: 4, message: '用户名不能小于4位' }
        ],
        password: [
          { required: true, validator: this.validatePassword }
        ]
      },
      loading: false, // 是否正在提交请求中
      dialogRoleVisible: false, // 是否显示设置角色的对话框
      allRoles: [], // 所有角色列表
      userRoleIds: [], // 用户的角色ID的列表
      isIndeterminate: false, // 是否是不确定的
      checkAll: false, // 是否全选
    }
  },

  //发请求一般情况下，我们都是在mounted去发，但是也可以在created内部去发
  created () {
    this.getUsers()
  },

  methods: {
    /* 
    5.显示指定角色的界面
    */
    showAssignRole (user) {
      // console.log(user,'row-------------');
      // 传过来的就是row
      this.user = user
      // this.user 被赋值为当前选中的角色
      this.dialogRoleVisible = true
      // 角色对话框弹出 后才发请求
      this.getRoles()
      //调函数 发请求
    },

    /* 
    7.全选勾选状态发生改变的监听
    */
    handleCheckAllChange (value) {
      // chenge事件 value为更新后的值3
      // value 当前勾选状态true/false
      // 如果当前全选, userRoleIds就是所有角色id的数组, 否则是空数组
      this.userRoleIds = value ? this.allRoles.map(item => item.id) : []
      // 如果当前不是全选但是也选择了, 指定为false
      this.isIndeterminate = false
    },

    /* 
    6.根据id 异步获取用户的角色列表
    */
    async getRoles () {
      // getRoles是通过点击事件再发请求的 
      // 根据当前角色的id 请求到该用户的角色身份
      const result = await this.$API.user.getRoles(this.user.id)
      // console.log(result,'result-----------');
      // 全部的角色列表 该用户有的角色列表
      const {allRolesList, assignRoles} = result.data
      this.allRoles = allRolesList
      // 获得角色身份的ids userRoleIds=id
      this.userRoleIds = assignRoles.map(item => item.id)
      // check选择 全选按钮的判断 
      // 这只是按钮的显示 没有业务（点击进行选择....）
      this.checkAll = allRolesList.length===assignRoles.length
      // 不确定按钮的判断 选择了但是没有全选
      this.isIndeterminate = assignRoles.length>0 && assignRoles.length<allRolesList.length
    },

    /* 
    8.角色列表选中项发生改变的监听
    */
    handleCheckedChange (value) {
      const {userRoleIds, allRoles} = this
      // 普通选择框 change改变时 先判断checkAll 再判断isIndeterminate
      this.checkAll = userRoleIds.length === allRoles.length && allRoles.length>0
      this.isIndeterminate = userRoleIds.length>0 && userRoleIds.length<allRoles.length
    },

    /* 
    请求给用户进行角色授权
    */
    async assignRole () {
      const userId = this.user.id
      const roleIds = this.userRoleIds.join(',')
      this.loading = true
      const result = await this.$API.user.assignRoles(userId, roleIds)
      this.loading = false
      this.$message.success(result.message || '分配角色成功')
      this.resetRoleData()

      // console.log(this.$store.getters.name, this.user)
      if (this.$store.getters.name===this.user.username) {
        window.location.reload()
      }
    },

    /* 
    重置用户角色的数据
    */
    resetRoleData () {
      this.dialogRoleVisible = false
      this.allRoles = []
      this.userRoleIds = []
      this.isIndeterminate = false
      this.checkAll = false
    },

    /* 
    4.自定义密码校验函数
    */
    validatePassword (rule, value, callback) {
      if (!value) {
        callback('密码必须输入')
      } else if (!value || value.length < 6) {
        callback('密码不能小于6位')
      } else {
        callback()
      }
    },
    /* 
    2.根据输入进行搜索
    */
    search () {
      // tempSearchObj是v-model绑定的获取的数据
      // searchObj是发请求携带的参数 
      this.searchObj = {...this.tempSearchObj}
      // 携带搜索参数再次发请求 再次更新页面 
      this.getUsers()
    },

    /* 
    3.重置输入后搜索
    */
    resetSearch () {
      // 将-model中的username数据和发请求的params参数username置为空
      this.searchObj = {
        username: ''
      }
      this.tempSearchObj = {
        username: ''
      }
      // 空的username再次发请求回到原来的页面
      this.getUsers()
    },

    /* 
   13.显示添加用户的界面
    */
    showAddUser () {
      // 添加用户 是没有id的 user为当前的用户 添加是新增一个没有当前user 所以置为空
      this.user = {}
      this.dialogUserVisible = true
      // ？？？
      this.$nextTick(() => this.$refs.userForm.clearValidate())
    },

    /* 
    11.删除所有选中的用户
    */
    revomveUsers () {
      //删除全部
      this.$confirm('确定删除吗?').then(async () => {
        await this.$API.user.removeUsers(this.selectedIds)
        this.$message.success('删除成功')
        this.getUsers()
      }).catch(() => {
        this.$message.info('取消删除')
      })
    },

    /* 
    12.列表选中状态发生改变的监听回调
    */
    handleSelectionChange (selection) {
      // 选择项发生改变时 调用函数
      this.selectedIds = selection.map(item => item.id)
    },

    /* 
    10.显示更新用户的界面
    */
    showUpdateUser (user) {
      //修改用户界面弹出 点击确定之后才发请求修改 所以业务逻辑不在这里
      //见 addOrUpdate 14
      //深拷贝？？
      this.user = cloneDeep(user)
      this.dialogUserVisible = true
    },

    /* 
    9.删除某个用户 
    */
    async removeUser (id) {
      // 删除某一个用户 id为row.id
      await this.$API.user.removeById(id)
      this.$message.success('删除成功')
      // 删除后也要刷新页面 判断展示页面 this.users是该页面要展示的user
      this.getUsers(this.users.length===1 ? this.page-1 : this.page)
    },

    /* 
    1.获取分页列表 展示数据
    */
    async getUsers (page=1) {
      // 点击分页器切换page
      this.page = page
      const {limit, searchObj} = this
      this.listLoading = true
      // 根据page, limit, searchObj发请求 获得数据
      const result = await this.$API.user.getPageList(page, limit, searchObj)
      console.log(result,'page------------------');
      // 请求成功后 loading为false 加载成功
      this.listLoading = false
      // 解构出数据
      const {items, total} = result.data
      this.users = items.filter(item => item.username!=='admin')
      this.total = total-1
      // 被选中的id数组？
      this.selectedIds = []
    },

    /* 
    处理pageSize发生改变的监听回调
    */
    handleSizeChange (pageSize) {
      this.limit = pageSize
      this.getUsers()
    },

    /* 
    取消用户的保存或更新
    */
    cancel () {
      this.dialogUserVisible = false
      this.user = {}
    },

    /* 
    14.保存或者更新用户
    */
    addOrUpdate () {
      //修改逻辑4 执行函数
      this.$refs.userForm.validate(valid => {
        // 先验证
        if (valid) {
          // 点击的是修改的话this里面就有user不为空 点击添加user为{}
          const {user} = this
          this.loading = true
          // 判断id是否存在
          this.$API.user[user.id ? 'update' : 'add'](user).then((result) => {
            // 成功的回调
            this.loading = false
            this.$message.success('保存成功!')
            //页面展示情况
            this.getUsers(user.id ? this.page : 1)
            //？再次为空？修改或者添加
            this.user = {}
            this.dialogUserVisible = false
          })
        }
      })
    }
  }
}
</script>

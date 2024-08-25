<template>
  <div class="dashboard-container">
    <!-- 搜索表单 -->
    <el-form :model="tableData" label-width="80px" :inline="true" size="small">
      <el-form-item label="活动名称">
        <el-input
          v-model="tableData.userName"
          placeholder="请输入用户名称"
          clearable
        />
      </el-form-item>
      <el-form-item label="创建时间">
        <el-date-picker
          v-model="tableData.minCreateTime"
          type="datetime"
          placeholder="起始时间"
          class="data-picker"
        />
        <el-date-picker
          v-model="tableData.maxCreateTime"
          type="datetime"
          placeholder="截止时间"
          class="data-picker"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="getUserList">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <!-- 用户操作按钮 -->
    <div>
      <el-button type="primary" plain icon="el-icon-plus" size="mini" @click="handleCreateUser">新增</el-button>
      <el-button type="danger" plain icon="el-icon-delete" size="mini" @click="handleBatchDelete">删除</el-button>
      <!-- <el-button type="info" plain icon="el-icon-upload2" size="mini" @click="handleImportUser">导入</el-button> -->
    </div>

    <!-- 用户列表 -->
    <el-table
      :data="tableData.list"
      @selection-change="val => tableData.selection = val"
      @sort-change="hanleSortChange"
    >
      <el-table-column type="index" width="60" />
      <el-table-column type="selection" width="50" />
      <el-table-column width="50">
        <template slot-scope="scope">
          <!-- <img  class="table-avatar"> :src="scope.row.id" -->
          <!-- 直接使用src绑定会导致刷新失败 -->
          <img :id="'avater-'+scope.row.id" class="table-avatar">
        </template>
      </el-table-column>
      <el-table-column prop="userName" label="用户名" sortable="custom" />
      <el-table-column prop="trueName" label="真实姓名" sortable="custom" />
      <el-table-column prop="roleList" label="角色" sortable="custom" />
      <el-table-column prop="createTime" label="创建时间" sortable="custom" />
      <el-table-column prop="status" label="是否激活" sortable="custom" width="100">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.status" :active-value="1" :inactive-value="0" @change="handleSwitch(scope.row)" />
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="text" icon="el-icon-edit" size="small" @click="handleEdit(scope.row)">编辑</el-button>
          <el-button type="text" icon="el-icon-delete" style="color: red;" size="small" @click="handleDelete([scope.row.id])">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <el-pagination
      class="pagination"
      :current-page.sync="tableData.pageNum"
      :page-sizes="[10, 20, 30, 40]"
      :page-size.sync="tableData.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="tableData.total"
      @size-change="getUserList"
      @current-change="getUserList"
    />

    <!-- 用户编辑/创建窗口 -->
    <el-dialog class="user-edit-dialog" :title="userEditForm.id ? '用户编辑' : '新增用户'" :visible.sync="userEditDialogVisible" width="50%" top="8vh">
      <el-form
        ref="userEditForm"
        status-icon
        :model="userEditForm"
        label-width="80px"
        :rules="userEditForm.id ? userUpdateRules : userCreateRules"
      >
        <el-form-item label="用户名" prop="userName">
          <el-input v-model="userEditForm.userName" />
        </el-form-item>
        <el-form-item label="真实姓名">
          <el-input v-model="userEditForm.trueName" />
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="userEditForm.password" />
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="userEditForm.email" />
        </el-form-item>
        <el-form-item label="性别">
          <el-radio-group v-model="userEditForm.gender">
            <el-radio :label="0">男</el-radio>
            <el-radio :label="1">女</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="地址">
          <el-input v-model="userEditForm.address" />
        </el-form-item>
        <el-form-item label="简介">
          <el-input v-model="userEditForm.introduction" />
        </el-form-item>
        <el-form-item label="电话">
          <el-input v-model="userEditForm.phone" />
        </el-form-item>
        <el-form-item label="角色" prop="roleIds">
          <el-select v-model="userEditForm.roleIds" multiple placeholder="请选择角色">
            <el-option v-for="role in allRoles" :key="role.id" :label="role.name" :value="role.id" />
          </el-select>
        </el-form-item>
        <el-form-item label="头像">
          <el-upload
            class="avatar-uploader"
            action=""
            :auto-upload="false"
            :show-file-list="false"
            :on-change="file => handleAvatarChange(file)"
          >
            <!-- <img v-if="avatarUploadData.url" :src="avatarUploadData.url" class="avatar"> -->
            <!-- <i v-else class="el-icon-plus avatar-uploader-icon" /> -->
          </el-upload>
          <!-- <el-button v-if="avatarUploadData.row" size="mini" @click="resetUploadData(false)">重置</el-button> -->
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="userEditDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 用户导入弹窗 -->
    <!-- <user-import-dialog ref="userImportDialog" @import-success="getUserList" /> -->
  </div>
</template>

<script>
import axios from '@/axios'
import LoadingUtils from '@/utils/loading-utils'
// import * as USerApi from '@/api/user'

// 不复制引用，复制变量值
const copyObject = obj => JSON.parse(JSON.stringify(obj))
export default {
  name: 'User',
  data() {
    return {
      tableData: {
        userName: '',
        minCreateTime: '',
        maxCreateTime: '',
        list: [{
          id: 1,
          userName: '张三',
          trueName: '张三',
          roleList: [],
          createTime: '',
          status: true
        }],
        selection: '',
        pageNum: 1,
        pageSize: 10,
        total: 1
      },
      userEditForm: {
        id: '',
        userName: '',
        trueName: '',
        password: '',
        email: '',
        gender: '',
        address: '',
        introduction: '',
        phone: '',
        roleIds: []
      },
      userEditDialogVisible: false,
      allRoles: [], // 系统内角色列表
      userCreateRules: {
        userName: [{ required: true, trigger: 'blur', validator: this.userNameValidator }],
        password: [{ required: true, trigger: 'change', validator: this.passwordValidator }],
        roleIds: [{ required: true, trigger: 'change', validator: this.roleValidator }]
      },
      userUpdateRules: {
        userName: [{ required: true, trigger: 'blur', validator: this.userNameValidator }],
        password: [{ trigger: 'change', validator: this.passwordValidator }],
        roleIds: [{ required: true, trigger: 'change', validator: this.roleValidator }]
      },
      currentEditRow: null // 当前编辑行
    }
  },
  // 默认显示
  mounted() {
    this.getUserList()
    this.getAllRoles()
  },
  methods: {
    /**
     * 创建用户
     */
    handleCreateUser() {
      this.resetUserEditForm()
      // this.resetUploadData()
      this.openUserEditForm()
    },
    /**
     * 重置用户编辑表单
     */
    resetUserEditForm() {
      for (const key in this.userEditForm) {
        this.userEditForm[key] = ''
      }
      this.userEditForm.roleIds = []
    },
    /**
     * 获取所有角色
     */
    getAllRoles() {
      this.getRoles().then(res => {
        this.allRoles = res.data.data
        // this.$refs.userImportDialog.setRoles(this.allRoles)
      })
    },
    // 用户名验证函数
    userNameValidator(rule, value, callback) {
      if (!value) {
        callback(new Error('请输入用户名'))
      } else if (this.userEditForm.id && value === this.currentEditRow.userName) {
        callback()
      } else {
        // callback()
        // 使用接口判断用户名是否重名
        this.checkUserName(value).then(res => {
          if (res.data.data) {
            callback(new Error('用户名已存在'))
          } else {
            callback(undefined)
          }
          // callback(res.data.data ? res.Error('用户名已存在') : undefined)
        }).catch(error => {
          this.$message.error('检查用户名时出错')
          console.error(error)
        })
      }
    },
    checkUserName(userName) {
      return axios.post(`/users/user-name/exist`, userName, {
        headers: {
          'Content-Type': 'application/json'
        }
      })
    },
    // 密码验证函数
    passwordValidator(rule, value, callback) {
      if (!value && this.userEditForm.id) {
        callback()
      } else if (!value || value.length < 6) {
        callback(new Error('密码长度不能小于6位'))
      } else {
        callback()
      }
    },
    // 角色验证函数
    roleValidator(rule, value, callback) {
      if (!value || value.length === 0) {
        callback(new Error('角色不能为空'))
      } else {
        callback()
      }
    },
    /**
     * 获取用户列表
     * @param {Object} data {userName,minCreateTime,maxCreateTime,orderBy,orderMethod,pageNum,pageSize}
     */
    getUsers(tableData) {
      const params = new URLSearchParams(tableData)
      // orderBy由驼峰转下划线
      if (params.has('orederBy')) {
        params.set('orderBy', params.get('orderBy').replace(/([A-Z])/g, '_$1').toLowerCase())
      }

      const url = axios.get(`/users/getUserList?${params.toString()}`)
      return url
    },
    getUserList() {
      this.getUsers(this.tableData).then(res => {
        this.tableData.list = res.data.data.records
        this.tableData.total = res.data.data.total
        // 更新头像
        // this.$nextTick(() => {
        //   this.tableData.list.forEach(row => {
        //     this.getAvatar(row.id,row)
        //   })
        // })
      })
    },
    /**
     * 导入 批量创建用户
     */
    handleImportUser() {

    },
    /**
     * 切换用户账号激活状态
     * @param {Object} row 行数据
     */
    handleSwitch(row) {
      this.changeUserStatus(row.id, row.status).then(() => {
        this.$message.success('操作成功')
      })
    },
    /**
     * 修改用户状态
     */
    changeUserStatus(userId, status) {
      axios.put(`/users/${userId}/status?status=${status}`)
    },
    getRoles(searchContent) {
      return axios.get(`/roles/getRoleList`, { params: { searchContent }})
    },
    /**
     * 编辑用户信息
     */
    handleEdit(row) {
      this.currentEditRow = row
      for (const key in this.userEditForm) {
        this.userEditForm[key] = row[key]
      }
      // 对角色权限数据进行处理
      this.userEditForm.roleIds = row.roleList ? row.roleList.map(item => {
        const role = this.allRoles.find(role => role.name === item)
        return role && role.id
      }) : []
      this.userEditForm.roleIds.filter(id => id)
      this.openUserEditForm()
      // this.userEditDialogVisible = true
    },
    /**
     * 打开用户编辑窗口
     */
    openUserEditForm() {
      this.userEditDialogVisible = true
      this.$nextTick(() => {
        this.$refs.userEditForm.clearValidate()
      })
    },
    // 批量删除用户
    handleBatchDelete() {
      // 将tableData.selection中的id提取出来，传递给handleDelete
      if (this.tableData.selection.length === 0) {
        this.$message.warning('请选择要删除的用户')
        return
      }
      const userIds = this.tableData.selection.map(item => item.id)
      this.handleDelete(userIds)
    },
    /**
     * 删除用户
     * @param {id} userIds id
     */
    handleDelete(userIds) {
      this.$confirm('此操作将永久删除该用户，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 对接删除用户接口
        LoadingUtils.createFullScreenLoading('正在删除...')
        this.deleteUsers(userIds).then(() => {
          this.$message.success('删除成功')
          this.getUserList()
        }).finally(() => {
          LoadingUtils.closeFullScreenLoading()
        })
      })
    },
    /**
     * 删除用户
     * @param {list} userIds
     */
    deleteUsers(userIds) {
      return axios.delete(`/users/deleteUsers?ids=` + userIds.join(','))
    },
    /**
     * 重置查询条件
     */
    resetQuery() {
      this.tableData.userName = ''
      this.tableData.minCreateTime = ''
      this.tableData.maxCreateTime = ''
      // this.tableData.pageNum = 1
      // this.getUserList()
    },
    /**
     * 处理排序变化
     * @param {object} column 列对象
     * @param {string} prop 列属性
     * @param {string} order 排序方式
     */
    hanleSortChange({ column, prop, order }) {
      this.tableData.orderBy = prop
    },
    /**
     * 添加或更新用户
     */
    addOrUpdateUser() {
      this.$refs.userEditForm.validate(valid => {
        if (valid) {
          // 如果验证通过就调用添加或者更新用户的接口
          const params = copyObject(this.userEditForm)
          // if (!params.password) {
          //   delete params.password
          // }
          LoadingUtils.createFullScreenLoading('正在保存')
          const tempApi = this.userEditForm.id ? this.updateUser : this.addUser
          tempApi(params).then(res => {
            this.$message.success('操作成功')
            if (!this.userEditForm.id) {
              this.userEditForm.id = res.data.data.id
            }
            // this.updateAvatar()
            this.getUserList()
          }).finally(() => {
            this.userEditDialogVisible = false
            LoadingUtils.closeFullScreenLoading()
          })
        }
      })
    },
    /**
     * 更新用户信息
     * @param {Object} userEditForm {userName,trueName,password,email,gender,address,introduction,phone,roleId}
     */
    updateUser(userEditForm) {
      return axios.put(`/users/updateUserInfo`, userEditForm)
    },
    /**
     * 添加用户
     * @param {Object} userEditForm {userName,trueName,password,email,gender,address,introduction,phone,roleId}
     */
    addUser(userEditForm) {
      return axios.post(`/users/addOneUser`, userEditForm)
    },
    getInfo() {
      return axios.get(`/users/me`)
    },
    getAvatar() {
      return new Promise((reslove, reject) => {
        axios({
          method: 'get',
          url: `/users/me/avatar`,
          responseType: 'blob'
        }).then(response => {
          // reslove(generateAvatarUrl(response))
        }).catch(error => {
          reject(error)
        })
      })
    }
  }
}
</script>
<style scoped>
.data-picker{
  margin-right: 10px;
  width: 160px;
}
</style>

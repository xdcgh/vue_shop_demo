<template>
  <div>
    <!--    面包屑导航区-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--  卡片视图  -->
    <el-card>
      <!--   搜索与添加区域   -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容"
                    v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>

        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>

      <!--   数据表格区   -->
      <el-table :data="userList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>

        <el-table-column label="状态">
          <template v-slot="scope">
            <el-switch v-model="scope.row['mg_state']" @change="userStateChanged(scope.row)">
            </el-switch>
          </template>
        </el-table-column>

        <el-table-column label="操作" width="180px">
          <template v-slot="scope">
            <el-button type="primary" icon="el-icon-edit"
                       size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            <el-button type="danger" icon="el-icon-delete"
                       size="mini" @click="removeUserById(scope.row.id)"></el-button>
            <!--     分配角色按钮       -->
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!--   分页区域   -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[2, 5, 10, 20, 50]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <!--  添加用户的对话框  -->
    <el-dialog
      title="添加用户" width="50%"
      :visible.sync="addDialogVisible"
      @close="addDialogClosed">

      <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!--  修改用户的对话框  -->
    <el-dialog
      title="修改用户" width="50%"
      :visible.sync="editDialogVisible"
      @close="editDialogClosed">

      <el-form ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data () {
      // 验证规则
      let checkEmail = (rule, value, callback) => {
        const regEmail = /^([A-Za-z0-9_\-.])+@([A-Za-z0-9_\-.])+\.([A-Za-z]{2,4})$/

        if (regEmail.test(value)) {
          // 校验通过
          callback()
        }

        callback(new Error('请输入合法的邮箱'))
      }

      let checkMobile = (rule, value, callback) => {
        const regMobile = /^1[3-9]\d{9}$/

        if (regMobile.test(value)) {
          // 校验通过
         callback()
        }

        callback(new Error('请输入合法的手机号'))
      }

      return {
        // 获取用户列表的参数对象
        queryInfo: {
          query: '',
          // 当前页码
          pagenum: 1,
          pagesize: 2
        },
        userList: [],
        total: 0,

        // 用户对话框的显示值
        addDialogVisible: false,
        editDialogVisible: false,
        editForm: {},

        // 添加的用户表达数据
        addForm: {
          username: '',
          password: '',
          email: '',
          mobile: ''
        },
        addFormRules: {
          username: [
            {
              required: true,
              message: '请输入用户名',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 10,
              message: '用户名长度在3~10个字符之间',
              trigger: 'blur'
            }
          ],
          password: [
            {
              required: true,
              message: '请输入密码',
              trigger: 'blur'
            },
            {
              min: 6,
              max: 15,
              message: '用户名长度在6~15个字符之间',
              trigger: 'blur'
            }
          ],
          email: [
            {
              required: true,
              message: '请输入邮箱',
              trigger: 'blur'
            },
            {
              validator: checkEmail,
              trigger: 'blur'
            }
          ],
          mobile: [
            {
              required: true,
              message: '请输入手机号',
              trigger: 'blur'
            },
            {
              validator: checkMobile,
              trigger: 'blur'
            }
          ]
        },
        editFormRules: {
          email: [
            {
              required: true,
              message: '请输入邮箱',
              trigger: 'blur'
            },
            {
              validator: checkEmail,
              trigger: 'blur'
            }
          ],
          mobile: [
            {
              required: true,
              message: '请输入手机号',
              trigger: 'blur'
            },
            {
              validator: checkMobile,
              trigger: 'blur'
            }
          ]
        }
      }
    },
    created () {
      this.getUserList()
    },
    methods: {
      async getUserList () {
        const { data: res } = await this.$http.get('users', {
          params: this.queryInfo
        })

        if (res.meta.status !== 200) {
          return this.$message.error('获取用户列表失败')
        }

        this.userList = res.data.users
        this.total = res.data.total
      },
      // pagesize 改变事件
      handleSizeChange (newSize) {
        this.queryInfo.pagesize = newSize

        this.getUserList()
      },
      // 页码值的改变事件
      handleCurrentChange (newPage) {
        this.queryInfo.pagenum = newPage

        this.getUserList()
      },
      // switch 状态改变事件
      async userStateChanged (userInfo) {
        const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)

        if (res.meta.status !== 200) {
          userInfo.mg_state = !userInfo.mg_state
          return this.$message.error('更新状态失败')
        }

        this.$message.success('更新用户状态成功')
      },
      // 添加用户 对话框的关闭事件
      addDialogClosed () {
        this.$refs.addFormRef.resetFields()
      },
      addUser () {
        this.$refs.addFormRef.validate(async valid => {

          if (!valid) {
            return
          }

          const { data: res } = await this.$http.post('users', this.addForm)

          if (res.meta.status !== 201) {
            return this.$message.error('添加用户失败！')
          }

          this.$message.success('添加用户成功！')

          this.addDialogVisible = false

          await this.getUserList()
        })
      },
      async showEditDialog (id) {
        const { data: res } = await this.$http.get(`users/${id}`)

        if (res.meta.status !== 200) {
          return this.$message.error('查询用户失败')
        }

        this.editForm = res.data

        this.editDialogVisible = true
      },
      editDialogClosed () {
        this.$refs.editFormRef.resetFields()
      },
      editUserInfo () {
        this.$refs.editFormRef.validate(async valid => {
          if (!valid) {
            return
          }

          const { data:res } = await this.$http.put(`users/${this.editForm.id}`,{
            email: this.editForm.email,
            mobile: this.editForm.mobile
          })

          if (res.meta.status !== 200) {
            return this.$message.error('更新失败')
          }

          this.$message.success('更新成功')

          this.editDialogVisible = false

          await this.getUserList()
        })
      },
      async removeUserById (id) {
        const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)

        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除')
        }

        const { data:res } = await this.$http.delete(`users/${id}`)

        if (res.meta.status !== 200) {
          return this.$message.error('删除用户失败')
        }

        this.$message.success('删除用户成功')

        await this.getUserList()
      }
    }
  }
</script>

<style lang="less" scoped>

</style>

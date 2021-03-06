<template>
  <div class="container">
    <el-row>
      <el-col :lg="16"
              :md="20"
              :sm="24"
              :xs="24">
        <el-form :model="form"
                 status-icon
                 :rules="rules"
                 :label-position="labelPosition"
                 ref="form"
                 label-width="100px">
          <el-form-item label="用户名"
                        prop="nickname">
            <el-input clearable
                      v-model="form.nickname"
                      :disabled="isEdited"></el-input>
          </el-form-item>
          <el-form-item label="邮箱"
                        prop="email">
            <el-input clearable
                      v-model="form.email"></el-input>
          </el-form-item>
          <el-form-item v-if="pageType === 'add'"
                        label="密码"
                        prop="password">
            <el-input clearable
                      type="password"
                      v-model="form.password"
                      autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item v-if="pageType === 'add'"
                        label="确认密码"
                        prop="confirm_password"
                        label-position="top">
            <el-input clearable
                      type="password"
                      v-model="form.confirm_password"
                      autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item v-if="pageType !== 'password'"
                        label="选择分组">
            <el-radio-group v-model="form.group_id"
                            label-position="top">
              <el-radio :label="item.id"
                        v-for="(item, index) in groups"
                        :key=index>{{item.name}}</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item v-show="submit"
                        class="submit">
            <el-button type="primary"
                       @click="submitForm('form')">保 存</el-button>
            <el-button @click="resetForm('form')">重 置</el-button>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import Admin from '@/lin/models/admin'
import User from '@/lin/models/user'

export default {
  props: {
    submit: {
      type: Boolean, // 表单是否显示功能按钮
      default: true,
    },
    id: { // 用户id
      type: Number,
      default: undefined,
    },
    groups: { // 所有分组
      type: Array,
      default: () => { },
    },
    labelPosition: { // 表单label位置
      type: String,
      default: 'right',
    },
    info: { // 用户信息
      type: Object,
      default: () => { },
    },
    pageType: {
      type: String,
      default: 'add', // 区分编辑页面/添加页面
    },
  },
  inject: ['eventBus'],
  data() {
    // 验证回调函数
    const checkUserName = (rule, value, callback) => { // eslint-disable-line
      if (!value) {
        return callback(new Error('用户名不能为空'))
      }
      callback()
    }
    const validatePassword = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入密码'))
      } else if (value.length < 6) {
        callback(new Error('密码长度不能少于6位数'))
      } else {
        if (this.form.checkPassword !== '') {
          this.$refs.form.validateField('confirm_password')
        }
        callback()
      }
    }
    const validatePassword2 = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.form.password) {
        callback(new Error('两次输入密码不一致!'))
      } else {
        callback()
      }
    }
    return {
      isEdited: false, // 能否编辑
      form: {
        nickname: '',
        password: '',
        confirm_password: '',
        email: '',
        group_id: 0,
      },
      // 验证规则
      rules: {
        nickname: [
          { validator: checkUserName, trigger: ['blur', 'change'], required: true },
        ],
        password: [
          { validator: validatePassword, trigger: 'blur', required: true },
        ],
        confirm_password: [
          { validator: validatePassword2, trigger: 'blur', required: true },
        ],
        email: [
          { type: 'email', message: '请输入正确的邮箱地址或者不填', trigger: ['blur', 'change'] },
        ],
      },
    }
  },
  methods: {
    // 提交表单
    async submitForm(formName) {
      this.$refs[formName].validate(async (valid) => { // eslint-disable-line
        if (valid) {
          // 新增用户
          if (this.pageType === 'add') {
            const res = await User.register(this.form)
            if (res.error_code === 0) {
              this.$message.success(`${res.msg}`)
              this.eventBus.$emit('addUser', true)
              this.resetForm(formName)
            } else {
              this.$message.error(`${res.msg}`)
            }
          } else {
            // 更新用户信息
            if (this.form.email === this.info.email && this.form.group_id === this.info.group_id) {
              return
            }
            const res = await Admin.updateOneUser(this.form.email, this.form.group_id, this.id)
            if (res.error_code === 0) {
              this.$message.success(`${res.msg}`)
              this.$emit('handleInfoResult', true)
            }
          }
        } else {
          console.log('error submit!!')
          this.$message.error('请填写正确的信息')
        }
      })
    },
    // 重置表单
    resetForm(formName) {
      if (this.pageType === 'edit') {
        this.setInfo()
      } else {
        this.form.group_id = this.groups[0].id
        this.$refs[formName].resetFields()
      }
    },
    setInfo() {
      this.form.nickname = this.info.nickname
      this.form.email = this.info.email
      this.form.group_id = this.info.group_id
    },
  },
  watch: {
    groups: {
      // 默认选中管理员组
      handler() {
        if (this.groups && this.groups[0] && this.groups[0].id) {
          this.form.group_id = this.groups[0].id
        }
      },
      immediate: true,
    },
  },
  created() {
    // 通过是否接收到数据来判断当前页面是添加数据还是编辑数据
    if (this.pageType === 'edit') {
      this.setInfo()
      this.isEdited = true
    }
  },
}
</script>

<style scoped lang="scss">
.container {
  margin-top: 20px;
  margin-left: -5px;
  .submit {
    float: left;
  }
}
</style>

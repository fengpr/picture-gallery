<template>
  <div id="userRegisterPage" class="glass-card animate-fade-in">
    <h2 class="glow-title title">智能云图库 - 用户注册</h2>
    <div class="desc">企业级智能协同云图库</div>
    <a-form :model="formState" name="basic" autocomplete="off" @finish="handleSubmit">
      <a-form-item name="userAccount" :rules="[{ required: true, message: '请输入账号' }]">
        <a-input v-model:value="formState.userAccount" placeholder="请输入账号" class="tech-input" />
      </a-form-item>
      <a-form-item
        name="userPassword"
        :rules="[
          { required: true, message: '请输入密码' },
          { min: 8, message: '密码长度不能小于 8 位' },
        ]"
      >
        <a-input-password v-model:value="formState.userPassword" placeholder="请输入密码" class="tech-input" />
      </a-form-item>
      <a-form-item
        name="checkPassword"
        :rules="[
          { required: true, message: '请输入确认密码' },
          { min: 8, message: '确认密码长度不能小于 8 位' },
        ]"
      >
        <a-input-password v-model:value="formState.checkPassword" placeholder="请输入确认密码" class="tech-input" />
      </a-form-item>
      <div class="tips">
        已有账号？
        <RouterLink to="/user/login" class="tech-link">去登录</RouterLink>
      </div>
      <a-form-item>
        <a-button type="primary" html-type="submit" class="flow-button tech-submit-btn">
          注册
        </a-button>
      </a-form-item>
    </a-form>
  </div>
</template>
<script lang="ts" setup>
import { reactive } from 'vue'
import { userRegisterUsingPost } from '@/api/userController.ts'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { message } from 'ant-design-vue'
import router from '@/router' // 用于接受表单输入的值

// 用于接受表单输入的值
const formState = reactive<API.UserRegisterRequest>({
  userAccount: '',
  userPassword: '',
  checkPassword: '',
})

const loginUserStore = useLoginUserStore()

/**
 * 提交表单
 * @param values
 */
const handleSubmit = async (values: any) => {
  // 校验两次输入的密码是否一致
  if (values.userPassword !== values.checkPassword) {
    message.error('两次输入的密码不一致')
    return
  }
  const res = await userRegisterUsingPost(values)
  // 注册成功，跳转到登录页面
  if (res.data.code === 0 && res.data.data) {
    message.success('注册成功')
    router.push({
      path: '/user/login',
      replace: true,
    })
  } else {
    message.error('注册失败，' + res.data.message)
  }
}
</script>

<style scoped>
#userRegisterPage {
  max-width: 420px;
  margin: 60px auto;
  padding: 40px;
}

.title {
  text-align: center;
  margin-bottom: 8px;
  font-size: 24px;
}

.desc {
  text-align: center;
  color: rgba(255, 255, 255, 0.45);
  margin-bottom: 32px;
  font-size: 14px;
}

.tech-input :deep(.ant-input),
.tech-input :deep(.ant-input-password) {
  background: rgba(255, 255, 255, 0.05) !important;
  border-color: rgba(255, 255, 255, 0.1) !important;
  color: #fff !important;
}

.tips {
  color: rgba(255, 255, 255, 0.45);
  text-align: right;
  font-size: 13px;
  margin-bottom: 24px;
}

.tech-link {
  color: #165DFF;
  margin-left: 4px;
}

.tech-link:hover {
  text-shadow: 0 0 8px rgba(22, 93, 255, 0.5);
}

.tech-submit-btn {
  width: 100%;
  height: 40px;
  font-size: 16px;
  font-weight: 600;
  box-shadow: 0 4px 15px rgba(22, 93, 255, 0.3);
}
</style>
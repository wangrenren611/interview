# Vue工程实践面试题深度分析

## 1. Vue 3.5+ 项目架构设计

### 现代化Vue项目结构
**基于Vite + TypeScript + Composition API的最佳实践：**
```
src/
├── components/           # 通用组件
│   ├── base/            # 基础UI组件
│   ├── business/        # 业务组件
│   └── layout/          # 布局组件
├── composables/         # 组合式函数
├── utils/               # 工具函数
├── services/            # API服务层
├── stores/              # Pinia状态管理
├── types/               # TypeScript类型定义
├── views/               # 页面组件
├── router/              # 路由配置
├── styles/              # 样式文件
├── assets/              # 静态资源
└── constants/           # 常量定义
```

### Vue 3 项目初始化最佳实践
**2025年推荐的项目创建方式：**

```bash
# 使用Vue官方脚手架创建项目
npm create vue@latest my-vue-project

# 推荐选择以下配置
✅ TypeScript
✅ JSX
✅ Router
✅ Pinia
✅ Vitest
✅ End-to-End Testing with Playwright
✅ ESLint
✅ Prettier
✅ Vue DevTools

cd my-vue-project
npm install
npm run dev
```

**现代化开发环境配置：**
```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
import { resolve } from 'path'

export default defineConfig({
  plugins: [
    vue({
      // 开启反应性语法糖
      reactivityTransform: true,
      // 支持Vue DevTools
      include: [/\.vue$/],
    }),
    vueJsx(),
  ],
  
  // 路径别名配置
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@composables': resolve(__dirname, 'src/composables'),
      '@stores': resolve(__dirname, 'src/stores'),
      '@types': resolve(__dirname, 'src/types'),
      '@utils': resolve(__dirname, 'src/utils'),
    },
  },
  
  // 开发服务器配置
  server: {
    port: 3000,
    host: true, // 支持局域网访问
    open: true,
    cors: true,
    // API代理配置
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
  
  // 构建配置
  build: {
    target: 'es2020',
    outDir: 'dist',
    sourcemap: true,
    // 代码分割
    rollupOptions: {
      output: {
        manualChunks: {
          vue: ['vue', 'vue-router'],
          pinia: ['pinia'],
          ui: ['element-plus', 'ant-design-vue'],
          utils: ['lodash-es', 'dayjs'],
        },
      },
    },
    // 压缩配置
    minify: 'esbuild',
    // chunk大小警告阈值
    chunkSizeWarningLimit: 1500,
  },
  
  // CSS预处理器配置
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`,
      },
    },
  },
  
  // 依赖优化
  optimizeDeps: {
    include: [
      'vue',
      'vue-router',
      'pinia',
      'axios',
      '@vueuse/core',
    ],
  },
})
```

## 2. Composition API深度实践

### setup语法糖的企业级应用
**现代Vue 3.5+开发的核心模式：**

```vue
<!-- UserProfile.vue - 用户资料组件 -->
<script setup lang="ts">
import { ref, reactive, computed, watch, onMounted, nextTick } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { storeToRefs } from 'pinia'
import { useUserStore } from '@/stores/user'
import { useFormValidation } from '@/composables/useFormValidation'
import { UserInfo, ApiResponse } from '@/types'

// 接口定义
interface Props {
  userId?: string
  readonly?: boolean
}

interface Emits {
  (e: 'update', user: UserInfo): void
  (e: 'delete', userId: string): void
}

// Props和Emit定义
const props = withDefaults(defineProps<Props>(), {
  readonly: false,
})

const emit = defineEmits<Emits>()

// 路由和Store
const router = useRouter()
const route = useRoute()
const userStore = useUserStore()

// 响应式解构Store
const { currentUser, isLoading } = storeToRefs(userStore)

// 响应式数据
const editMode = ref(false)
const saving = ref(false)
const formRef = ref<HTMLFormElement>()

// 用户表单数据
const userForm = reactive<UserInfo>({
  id: '',
  name: '',
  email: '',
  phone: '',
  avatar: '',
  bio: '',
  preferences: {
    theme: 'light',
    language: 'zh-CN',
    notifications: true,
  },
})

// 自定义组合式函数
const {
  errors,
  isValid,
  validate,
  clearErrors,
} = useFormValidation(userForm, {
  name: {
    required: true,
    minLength: 2,
    maxLength: 50,
  },
  email: {
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
  },
  phone: {
    required: true,
    pattern: /^1[3-9]\d{9}$/,
  },
})

// 计算属性
const displayName = computed(() => {
  return userForm.name || userForm.email.split('@')[0] || '未知用户'
})

const hasChanges = computed(() => {
  if (!currentUser.value) return false
  
  return Object.keys(userForm).some(key => {
    return userForm[key] !== currentUser.value![key]
  })
})

const canEdit = computed(() => {
  return !props.readonly && (
    userStore.currentUserId === userForm.id || 
    userStore.hasPermission('admin')
  )
})

// 监听器
watch(() => props.userId, async (newUserId) => {
  if (newUserId) {
    await loadUserData(newUserId)
  }
}, { immediate: true })

watch(() => route.params.id, (newId) => {
  if (newId && typeof newId === 'string') {
    loadUserData(newId)
  }
})

// 深度监听表单变化
watch(userForm, (newForm) => {
  clearErrors()
  
  // 自动保存草稿
  if (editMode.value && hasChanges.value) {
    saveDraft(newForm)
  }
}, { deep: true })

// 方法定义
const loadUserData = async (userId: string) => {
  try {
    isLoading.value = true
    const userData = await userStore.fetchUser(userId)
    
    if (userData) {
      Object.assign(userForm, userData)
    }
  } catch (error) {
    console.error('加载用户数据失败:', error)
    // 可以添加错误提示
  } finally {
    isLoading.value = false
  }
}

const handleEdit = () => {
  if (!canEdit.value) return
  
  editMode.value = true
  nextTick(() => {
    // 聚焦到第一个输入框
    const firstInput = formRef.value?.querySelector('input')
    firstInput?.focus()
  })
}

const handleSave = async () => {
  if (!isValid.value) {
    await validate()
    return
  }
  
  try {
    saving.value = true
    const updatedUser = await userStore.updateUser(userForm.id, userForm)
    
    editMode.value = false
    emit('update', updatedUser)
    
    // 可以添加成功提示
    ElMessage.success('用户信息更新成功')
    
  } catch (error) {
    console.error('保存用户信息失败:', error)
    ElMessage.error('保存失败，请重试')
  } finally {
    saving.value = false
  }
}

const handleCancel = () => {
  editMode.value = false
  
  // 重置表单到原始数据
  if (currentUser.value) {
    Object.assign(userForm, currentUser.value)
  }
  
  clearErrors()
}

const handleDelete = async () => {
  if (!userForm.id) return
  
  try {
    await ElMessageBox.confirm(
      '此操作将永久删除该用户，是否继续？',
      '警告',
      {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }
    )
    
    await userStore.deleteUser(userForm.id)
    emit('delete', userForm.id)
    
    // 导航回用户列表
    router.push('/users')
    
  } catch (error) {
    if (error !== 'cancel') {
      console.error('删除用户失败:', error)
    }
  }
}

const saveDraft = (formData: UserInfo) => {
  // 保存到本地存储作为草稿
  localStorage.setItem(`user-draft-${formData.id}`, JSON.stringify(formData))
}

const loadDraft = (userId: string) => {
  const draft = localStorage.getItem(`user-draft-${userId}`)
  if (draft) {
    try {
      const draftData = JSON.parse(draft)
      Object.assign(userForm, draftData)
      return true
    } catch (error) {
      console.error('加载草稿失败:', error)
    }
  }
  return false
}

// 生命周期钩子
onMounted(() => {
  // 如果有userId prop，加载对应用户数据
  if (props.userId) {
    loadUserData(props.userId)
  } else if (route.params.id) {
    loadUserData(route.params.id as string)
  }
})

// 暴露给父组件的方法和数据
defineExpose({
  loadUserData,
  handleSave,
  userForm,
  isValid,
})
</script>

<template>
  <div class="user-profile">
    <!-- 加载状态 -->
    <div v-if="isLoading" class="loading">
      <el-skeleton :rows="8" animated />
    </div>
    
    <!-- 用户信息展示/编辑 -->
    <el-card v-else class="profile-card">
      <template #header>
        <div class="profile-header">
          <h2>{{ displayName }}的资料</h2>
          <div class="actions" v-if="canEdit">
            <el-button 
              v-if="!editMode" 
              type="primary" 
              @click="handleEdit"
            >
              编辑
            </el-button>
            <template v-else>
              <el-button 
                type="success" 
                :loading="saving"
                :disabled="!hasChanges"
                @click="handleSave"
              >
                保存
              </el-button>
              <el-button @click="handleCancel">
                取消
              </el-button>
            </template>
          </div>
        </div>
      </template>
      
      <el-form 
        ref="formRef"
        :model="userForm" 
        label-width="100px"
        :disabled="!editMode"
      >
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item 
              label="用户名" 
              :error="errors.name"
            >
              <el-input 
                v-model="userForm.name" 
                placeholder="请输入用户名"
              />
            </el-form-item>
          </el-col>
          
          <el-col :span="12">
            <el-form-item 
              label="邮箱" 
              :error="errors.email"
            >
              <el-input 
                v-model="userForm.email" 
                type="email"
                placeholder="请输入邮箱"
              />
            </el-form-item>
          </el-col>
        </el-row>
        
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item 
              label="手机号" 
              :error="errors.phone"
            >
              <el-input 
                v-model="userForm.phone" 
                placeholder="请输入手机号"
              />
            </el-form-item>
          </el-col>
          
          <el-col :span="12">
            <el-form-item label="头像">
              <el-upload
                class="avatar-uploader"
                action="/api/upload"
                :show-file-list="false"
                :before-upload="beforeAvatarUpload"
                :on-success="handleAvatarSuccess"
              >
                <img 
                  v-if="userForm.avatar" 
                  :src="userForm.avatar" 
                  class="avatar"
                >
                <el-icon v-else class="avatar-uploader-icon">
                  <Plus />
                </el-icon>
              </el-upload>
            </el-form-item>
          </el-col>
        </el-row>
        
        <el-form-item label="个人简介">
          <el-input 
            v-model="userForm.bio" 
            type="textarea" 
            :rows="4"
            placeholder="请输入个人简介"
          />
        </el-form-item>
        
        <!-- 偏好设置 -->
        <el-form-item label="偏好设置">
          <el-row :gutter="20">
            <el-col :span="8">
              <el-form-item label="主题">
                <el-select v-model="userForm.preferences.theme">
                  <el-option label="浅色" value="light" />
                  <el-option label="深色" value="dark" />
                  <el-option label="自动" value="auto" />
                </el-select>
              </el-form-item>
            </el-col>
            
            <el-col :span="8">
              <el-form-item label="语言">
                <el-select v-model="userForm.preferences.language">
                  <el-option label="简体中文" value="zh-CN" />
                  <el-option label="English" value="en-US" />
                  <el-option label="日本語" value="ja-JP" />
                </el-select>
              </el-form-item>
            </el-col>
            
            <el-col :span="8">
              <el-form-item label="通知">
                <el-switch v-model="userForm.preferences.notifications" />
              </el-form-item>
            </el-col>
          </el-row>
        </el-form-item>
      </el-form>
      
      <!-- 危险操作区域 -->
      <el-divider content-position="left">危险操作</el-divider>
      <el-button 
        v-if="canEdit && userForm.id" 
        type="danger" 
        plain
        @click="handleDelete"
      >
        删除用户
      </el-button>
    </el-card>
  </div>
</template>

<style scoped lang="scss">
.user-profile {
  .profile-card {
    max-width: 800px;
    margin: 0 auto;
  }
  
  .profile-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    
    h2 {
      margin: 0;
      color: var(--el-text-color-primary);
    }
  }
  
  .avatar-uploader {
    .avatar {
      width: 60px;
      height: 60px;
      border-radius: 50%;
      object-fit: cover;
    }
    
    .avatar-uploader-icon {
      font-size: 28px;
      color: #8c939d;
      width: 60px;
      height: 60px;
      border: 1px dashed var(--el-border-color);
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      
      &:hover {
        border-color: var(--el-color-primary);
      }
    }
  }
  
  .loading {
    padding: 20px;
  }
}
</style>
```

这个示例展示了Vue 3.5+的核心特性：

1. **setup语法糖**：简化的组件定义方式
2. **TypeScript集成**：完整的类型安全
3. **Composition API**：逻辑复用和组织
4. **响应式系统**：ref、reactive、computed、watch
5. **Store集成**：Pinia状态管理
6. **路由集成**：Vue Router使用
7. **组合式函数**：逻辑抽取和复用
8. **生命周期钩子**：现代化的生命周期管理
9. **表单处理**：验证和数据绑定
10. **错误处理**：完善的错误处理机制

## 3. 组合式函数(Composables)最佳实践

### 自定义组合式函数设计原则
**基于业务逻辑抽象的可复用函数：**

```typescript
// composables/useFormValidation.ts
import { ref, reactive, computed, watch } from 'vue'
import type { Ref } from 'vue'

// 验证规则接口
interface ValidationRule {
  required?: boolean
  minLength?: number
  maxLength?: number
  pattern?: RegExp
  custom?: (value: any) => string | null
}

interface ValidationRules {
  [key: string]: ValidationRule
}

interface ValidationErrors {
  [key: string]: string
}

export function useFormValidation<T extends Record<string, any>>(
  formData: T,
  rules: ValidationRules
) {
  const errors = reactive<ValidationErrors>({})
  const touched = reactive<Record<string, boolean>>({})
  
  // 验证单个字段
  const validateField = (field: string, value: any): string | null => {
    const rule = rules[field]
    if (!rule) return null
    
    // 必填验证
    if (rule.required && (!value || (typeof value === 'string' && !value.trim()))) {
      return `${field}为必填项`
    }
    
    // 如果值为空且不是必填，则跳过其他验证
    if (!value) return null
    
    // 最小长度验证
    if (rule.minLength && value.length < rule.minLength) {
      return `${field}最少需要${rule.minLength}个字符`
    }
    
    // 最大长度验证
    if (rule.maxLength && value.length > rule.maxLength) {
      return `${field}最多只能${rule.maxLength}个字符`
    }
    
    // 正则验证
    if (rule.pattern && !rule.pattern.test(value)) {
      return `${field}格式不正确`
    }
    
    // 自定义验证
    if (rule.custom) {
      return rule.custom(value)
    }
    
    return null
  }
  
  // 验证所有字段
  const validate = async (): Promise<boolean> => {
    let isValid = true
    
    for (const field in rules) {
      const error = validateField(field, formData[field])
      if (error) {
        errors[field] = error
        isValid = false
      } else {
        delete errors[field]
      }
      touched[field] = true
    }
    
    return isValid
  }
  
  // 验证特定字段
  const validateSingleField = (field: string): boolean => {
    const error = validateField(field, formData[field])
    if (error) {
      errors[field] = error
      return false
    } else {
      delete errors[field]
      return true
    }
  }
  
  // 清除错误
  const clearErrors = (field?: string) => {
    if (field) {
      delete errors[field]
      delete touched[field]
    } else {
      Object.keys(errors).forEach(key => delete errors[key])
      Object.keys(touched).forEach(key => delete touched[key])
    }
  }
  
  // 设置错误
  const setError = (field: string, message: string) => {
    errors[field] = message
    touched[field] = true
  }
  
  // 计算属性：是否有错误
  const hasErrors = computed(() => Object.keys(errors).length > 0)
  
  // 计算属性：是否全部有效
  const isValid = computed(() => !hasErrors.value)
  
  // 计算属性：字段是否被触摸过
  const isTouched = computed(() => (field: string) => touched[field] || false)
  
  // 监听表单数据变化，实时验证
  watch(
    () => formData,
    (newData) => {
      for (const field in newData) {
        if (touched[field] && rules[field]) {
          validateSingleField(field)
        }
      }
    },
    { deep: true }
  )
  
  return {
    errors: readonly(errors),
    touched: readonly(touched),
    hasErrors,
    isValid,
    isTouched,
    validate,
    validateSingleField,
    clearErrors,
    setError,
  }
}
```

**API请求管理组合式函数：**

```typescript
// composables/useApi.ts
import { ref, computed } from 'vue'
import axios, { type AxiosRequestConfig, type AxiosResponse } from 'axios'

interface ApiState<T> {
  data: T | null
  loading: boolean
  error: string | null
}

export function useApi<T = any>() {
  const state = ref<ApiState<T>>({
    data: null,
    loading: false,
    error: null,
  })
  
  const isLoading = computed(() => state.value.loading)
  const hasError = computed(() => !!state.value.error)
  const hasData = computed(() => !!state.value.data)
  
  const execute = async <R = T>(
    config: AxiosRequestConfig
  ): Promise<R | null> => {
    try {
      state.value.loading = true
      state.value.error = null
      
      const response: AxiosResponse<R> = await axios(config)
      state.value.data = response.data as unknown as T
      
      return response.data
    } catch (error: any) {
      const message = error.response?.data?.message || error.message || '请求失败'
      state.value.error = message
      throw error
    } finally {
      state.value.loading = false
    }
  }
  
  const reset = () => {
    state.value = {
      data: null,
      loading: false,
      error: null,
    }
  }
  
  return {
    state: readonly(state),
    isLoading,
    hasError,
    hasData,
    execute,
    reset,
  }
}

// 特定的API hooks
export function useUserApi() {
  const { execute, ...rest } = useApi<UserInfo>()
  
  const fetchUser = (id: string) => {
    return execute({
      method: 'GET',
      url: `/api/users/${id}`,
    })
  }
  
  const updateUser = (id: string, data: Partial<UserInfo>) => {
    return execute({
      method: 'PUT',
      url: `/api/users/${id}`,
      data,
    })
  }
  
  const deleteUser = (id: string) => {
    return execute({
      method: 'DELETE',
      url: `/api/users/${id}`,
    })
  }
  
  return {
    ...rest,
    fetchUser,
    updateUser,
    deleteUser,
  }
}
```

## 4. Pinia状态管理深度实践

### Pinia Store设计模式
**现代Vue 3状态管理的最佳选择：**

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { UserInfo, LoginCredentials, UserPreferences } from '@/types'
import { userApi } from '@/services/userApi'
import { storageService } from '@/utils/storage'

export const useUserStore = defineStore('user', () => {
  // 状态定义
  const currentUser = ref<UserInfo | null>(null)
  const users = ref<UserInfo[]>([])
  const isLoggedIn = ref(false)
  const permissions = ref<string[]>([])
  const preferences = ref<UserPreferences>({
    theme: 'light',
    language: 'zh-CN',
    notifications: true,
  })
  const loading = ref(false)
  const error = ref<string | null>(null)
  
  // 计算属性
  const currentUserId = computed(() => currentUser.value?.id)
  const isAdmin = computed(() => permissions.value.includes('admin'))
  const hasPermission = computed(() => {
    return (permission: string) => permissions.value.includes(permission)
  })
  const displayName = computed(() => {
    return currentUser.value?.name || 
           currentUser.value?.email?.split('@')[0] || 
           '未知用户'
  })
  
  // 动作/方法
  const login = async (credentials: LoginCredentials) => {
    try {
      loading.value = true
      error.value = null
      
      const response = await userApi.login(credentials)
      const { user, token, permissions: userPermissions } = response.data
      
      // 保存用户信息
      currentUser.value = user
      permissions.value = userPermissions
      isLoggedIn.value = true
      
      // 保存token
      storageService.setToken(token)
      
      // 加载用户偏好设置
      await loadUserPreferences()
      
      return user
    } catch (err: any) {
      error.value = err.message || '登录失败'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  const logout = async () => {
    try {
      // 调用登出API
      await userApi.logout()
    } catch (err) {
      console.error('登出API调用失败:', err)
    } finally {
      // 无论API是否成功，都要清理本地状态
      currentUser.value = null
      permissions.value = []
      isLoggedIn.value = false
      
      // 清理本地存储
      storageService.clearToken()
      storageService.clearUserData()
    }
  }
  
  const fetchUser = async (userId: string): Promise<UserInfo | null> => {
    try {
      loading.value = true
      error.value = null
      
      const response = await userApi.getUser(userId)
      const user = response.data
      
      // 如果是当前用户，更新当前用户信息
      if (user.id === currentUserId.value) {
        currentUser.value = user
      }
      
      // 更新用户列表中的对应用户
      const index = users.value.findIndex(u => u.id === userId)
      if (index !== -1) {
        users.value[index] = user
      } else {
        users.value.push(user)
      }
      
      return user
    } catch (err: any) {
      error.value = err.message || '获取用户信息失败'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  const updateUser = async (userId: string, updates: Partial<UserInfo>) => {
    try {
      loading.value = true
      error.value = null
      
      const response = await userApi.updateUser(userId, updates)
      const updatedUser = response.data
      
      // 更新当前用户信息
      if (userId === currentUserId.value) {
        currentUser.value = updatedUser
      }
      
      // 更新用户列表
      const index = users.value.findIndex(u => u.id === userId)
      if (index !== -1) {
        users.value[index] = updatedUser
      }
      
      return updatedUser
    } catch (err: any) {
      error.value = err.message || '更新用户信息失败'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  const deleteUser = async (userId: string) => {
    try {
      loading.value = true
      error.value = null
      
      await userApi.deleteUser(userId)
      
      // 从用户列表中移除
      users.value = users.value.filter(u => u.id !== userId)
      
      // 如果删除的是当前用户，执行登出
      if (userId === currentUserId.value) {
        await logout()
      }
      
    } catch (err: any) {
      error.value = err.message || '删除用户失败'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  const fetchUsers = async (params?: { page?: number; limit?: number; search?: string }) => {
    try {
      loading.value = true
      error.value = null
      
      const response = await userApi.getUsers(params)
      users.value = response.data.users
      
      return response.data
    } catch (err: any) {
      error.value = err.message || '获取用户列表失败'
      throw err
    } finally {
      loading.value = false
    }
  }
  
  const updatePreferences = async (newPreferences: Partial<UserPreferences>) => {
    try {
      const updatedPreferences = { ...preferences.value, ...newPreferences }
      
      // 如果用户已登录，同步到服务器
      if (isLoggedIn.value && currentUserId.value) {
        await userApi.updatePreferences(currentUserId.value, updatedPreferences)
      }
      
      // 更新本地状态
      preferences.value = updatedPreferences
      
      // 保存到本地存储
      storageService.setUserPreferences(updatedPreferences)
      
    } catch (err: any) {
      error.value = err.message || '更新偏好设置失败'
      throw err
    }
  }
  
  const loadUserPreferences = async () => {
    try {
      // 优先从服务器加载
      if (isLoggedIn.value && currentUserId.value) {
        const response = await userApi.getPreferences(currentUserId.value)
        preferences.value = response.data
      } else {
        // 从本地存储加载
        const localPreferences = storageService.getUserPreferences()
        if (localPreferences) {
          preferences.value = localPreferences
        }
      }
    } catch (err) {
      console.error('加载用户偏好设置失败:', err)
      // 加载失败时使用本地存储的数据
      const localPreferences = storageService.getUserPreferences()
      if (localPreferences) {
        preferences.value = localPreferences
      }
    }
  }
  
  const checkAuthStatus = async () => {
    try {
      const token = storageService.getToken()
      if (!token) {
        return false
      }
      
      // 验证token有效性
      const response = await userApi.verifyToken()
      const { user, permissions: userPermissions } = response.data
      
      currentUser.value = user
      permissions.value = userPermissions
      isLoggedIn.value = true
      
      // 加载偏好设置
      await loadUserPreferences()
      
      return true
    } catch (err) {
      // token无效，清理状态
      await logout()
      return false
    }
  }
  
  const clearError = () => {
    error.value = null
  }
  
  // 重置store状态
  const $reset = () => {
    currentUser.value = null
    users.value = []
    isLoggedIn.value = false
    permissions.value = []
    preferences.value = {
      theme: 'light',
      language: 'zh-CN',
      notifications: true,
    }
    loading.value = false
    error.value = null
  }
  
  return {
    // 状态
    currentUser: readonly(currentUser),
    users: readonly(users),
    isLoggedIn: readonly(isLoggedIn),
    permissions: readonly(permissions),
    preferences: readonly(preferences),
    loading: readonly(loading),
    error: readonly(error),
    
    // 计算属性
    currentUserId,
    isAdmin,
    hasPermission,
    displayName,
    
    // 动作
    login,
    logout,
    fetchUser,
    updateUser,
    deleteUser,
    fetchUsers,
    updatePreferences,
    loadUserPreferences,
    checkAuthStatus,
    clearError,
    $reset,
  }
})
```

**Store组合和模块化：**

```typescript
// stores/index.ts - Store组合
import { createPinia } from 'pinia'
import { createPersistedState } from 'pinia-plugin-persistedstate'

// 创建pinia实例
const pinia = createPinia()

// 添加持久化插件
pinia.use(
  createPersistedState({
    // 默认使用localStorage
    storage: localStorage,
    // 序列化函数
    serializer: {
      serialize: JSON.stringify,
      deserialize: JSON.parse,
    },
  })
)

export default pinia

// 导出所有store
export { useUserStore } from './user'
export { useAppStore } from './app'
export { useRouterStore } from './router'
```

```typescript
// stores/app.ts - 应用全局状态
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useAppStore = defineStore('app', () => {
  // 主题相关
  const theme = ref<'light' | 'dark' | 'auto'>('light')
  const primaryColor = ref('#409EFF')
  
  // 布局相关
  const sidebarCollapsed = ref(false)
  const sidebarWidth = ref(200)
  
  // 加载状态
  const globalLoading = ref(false)
  const pageLoading = ref(false)
  
  // 消息提示
  const notifications = ref<Array<{
    id: string
    type: 'success' | 'warning' | 'error' | 'info'
    title: string
    message: string
    duration?: number
    timestamp: Date
  }>>([])
  
  // 计算属性
  const isDark = computed(() => {
    if (theme.value === 'auto') {
      return window.matchMedia('(prefers-color-scheme: dark)').matches
    }
    return theme.value === 'dark'
  })
  
  const unreadNotifications = computed(() => {
    return notifications.value.length
  })
  
  // 动作
  const setTheme = (newTheme: typeof theme.value) => {
    theme.value = newTheme
    // 应用主题到DOM
    document.documentElement.setAttribute('data-theme', newTheme)
  }
  
  const toggleSidebar = () => {
    sidebarCollapsed.value = !sidebarCollapsed.value
  }
  
  const setSidebarWidth = (width: number) => {
    sidebarWidth.value = width
  }
  
  const setGlobalLoading = (loading: boolean) => {
    globalLoading.value = loading
  }
  
  const setPageLoading = (loading: boolean) => {
    pageLoading.value = loading
  }
  
  const addNotification = (notification: Omit<typeof notifications.value[0], 'id' | 'timestamp'>) => {
    const id = Date.now().toString()
    notifications.value.unshift({
      ...notification,
      id,
      timestamp: new Date(),
    })
    
    // 自动清除通知
    if (notification.duration !== 0) {
      setTimeout(() => {
        removeNotification(id)
      }, notification.duration || 5000)
    }
    
    return id
  }
  
  const removeNotification = (id: string) => {
    const index = notifications.value.findIndex(n => n.id === id)
    if (index > -1) {
      notifications.value.splice(index, 1)
    }
  }
  
  const clearNotifications = () => {
    notifications.value = []
  }
  
  return {
    // 状态
    theme: readonly(theme),
    primaryColor: readonly(primaryColor),
    sidebarCollapsed: readonly(sidebarCollapsed),
    sidebarWidth: readonly(sidebarWidth),
    globalLoading: readonly(globalLoading),
    pageLoading: readonly(pageLoading),
    notifications: readonly(notifications),
    
    // 计算属性
    isDark,
    unreadNotifications,
    
    // 动作
    setTheme,
    toggleSidebar,
    setSidebarWidth,
    setGlobalLoading,
    setPageLoading,
    addNotification,
    removeNotification,
    clearNotifications,
  }
}, {
  persist: {
    // 只持久化部分状态
    paths: ['theme', 'primaryColor', 'sidebarCollapsed', 'sidebarWidth'],
  },
})
```

## 5. Vue Router路由管理深度实践

### 现代化路由配置
**Vue 3.5+路由管理的企业级实践：**

Vue Router作为Vue生态的核心路由管理工具，在2025年已经演进为一个功能强大、性能卓越的路由解决方案。现代化的路由配置不仅要考虑基本的路由匹配，更要兼顾性能优化、用户体验、安全性和可维护性。

```typescript
// router/index.ts
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { useAppStore } from '@/stores/app'
import NProgress from 'nprogress'

// 路由懒加载配置
const Home = () => import('@/views/Home.vue')
const About = () => import('@/views/About.vue')
const UserProfile = () => import('@/views/User/Profile.vue')
const UserSettings = () => import('@/views/User/Settings.vue')
const AdminDashboard = () => import('@/views/Admin/Dashboard.vue')

// 路由配置
const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: Home,
    meta: {
      title: '首页',
      requiresAuth: false,
      keepAlive: true,
      transition: 'fade',
    },
  },
  {
    path: '/about',
    name: 'About',
    component: About,
    meta: {
      title: '关于我们',
      requiresAuth: false,
      keepAlive: false,
    },
  },
  {
    path: '/user',
    name: 'User',
    redirect: '/user/profile',
    meta: {
      title: '用户中心',
      requiresAuth: true,
    },
    children: [
      {
        path: 'profile/:id?',
        name: 'UserProfile',
        component: UserProfile,
        meta: {
          title: '个人资料',
          requiresAuth: true,
          keepAlive: true,
        },
        // 路由级守卫
        beforeEnter: (to, from, next) => {
          const userStore = useUserStore()
          const targetUserId = to.params.id as string
          
          // 如果访问他人资料，检查权限
          if (targetUserId && targetUserId !== userStore.currentUserId) {
            if (!userStore.hasPermission('view_others_profile')) {
              next({ name: 'Forbidden' })
              return
            }
          }
          
          next()
        },
      },
      {
        path: 'settings',
        name: 'UserSettings',
        component: UserSettings,
        meta: {
          title: '用户设置',
          requiresAuth: true,
          keepAlive: false,
        },
      },
    ],
  },
  {
    path: '/admin',
    name: 'Admin',
    component: AdminDashboard,
    meta: {
      title: '管理后台',
      requiresAuth: true,
      requiresRoles: ['admin', 'super_admin'],
      layout: 'admin',
    },
    beforeEnter: (to, from, next) => {
      const userStore = useUserStore()
      
      if (!userStore.isAdmin) {
        next({ name: 'Forbidden' })
        return
      }
      
      next()
    },
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/Auth/Login.vue'),
    meta: {
      title: '登录',
      requiresAuth: false,
      hideInMenu: true,
      layout: 'auth',
    },
  },
  {
    path: '/forbidden',
    name: 'Forbidden',
    component: () => import('@/views/Error/403.vue'),
    meta: {
      title: '访问禁止',
      hideInMenu: true,
    },
  },
  {
    path: '/:pathMatch(.*)*',
    name: 'NotFound',
    component: () => import('@/views/Error/404.vue'),
    meta: {
      title: '页面未找到',
      hideInMenu: true,
    },
  },
]

// 创建路由实例
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
  // 滚动行为配置
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    } else if (to.hash) {
      return {
        el: to.hash,
        behavior: 'smooth',
      }
    } else {
      return { top: 0 }
    }
  },
})

// 全局前置守卫
router.beforeEach(async (to, from, next) => {
  const userStore = useUserStore()
  const appStore = useAppStore()
  
  // 开始加载进度条
  NProgress.start()
  
  // 设置页面标题
  if (to.meta.title) {
    document.title = `${to.meta.title} - Vue3 企业级应用`
  }
  
  // 检查认证状态
  if (to.meta.requiresAuth) {
    if (!userStore.isLoggedIn) {
      // 检查本地token
      const hasValidToken = await userStore.checkAuthStatus()
      if (!hasValidToken) {
        next({
          name: 'Login',
          query: { redirect: to.fullPath },
        })
        return
      }
    }
    
    // 检查角色权限
    if (to.meta.requiresRoles) {
      const requiredRoles = to.meta.requiresRoles as string[]
      const hasRequiredRole = requiredRoles.some(role => 
        userStore.permissions.includes(role)
      )
      
      if (!hasRequiredRole) {
        next({ name: 'Forbidden' })
        return
      }
    }
  }
  
  // 如果已登录用户访问登录页，重定向到首页
  if (to.name === 'Login' && userStore.isLoggedIn) {
    next({ name: 'Home' })
    return
  }
  
  next()
})

// 全局解析守卫
router.beforeResolve(async (to, from, next) => {
  // 在导航确认前，执行数据预取
  if (to.meta.preload && typeof to.meta.preload === 'function') {
    try {
      await to.meta.preload()
    } catch (error) {
      console.error('预加载数据失败:', error)
      // 可以选择继续导航或显示错误页面
    }
  }
  
  next()
})

// 全局后置钩子
router.afterEach((to, from) => {
  const appStore = useAppStore()
  
  // 停止加载进度条
  NProgress.done()
  
  // 记录页面访问日志
  if (process.env.NODE_ENV === 'production') {
    // 发送页面访问统计
    analytics.track('page_view', {
      path: to.path,
      name: to.name,
      from: from.name,
    })
  }
  
  // 清除页面加载状态
  appStore.setPageLoading(false)
})

// 错误处理
router.onError((error) => {
  console.error('路由错误:', error)
  
  // 可以显示错误提示或重定向到错误页面
  const appStore = useAppStore()
  appStore.addNotification({
    type: 'error',
    title: '导航失败',
    message: '页面加载失败，请重试',
  })
})

export default router
```

### 导航守卫深度实践
**三层守卫体系的完整应用：**

Vue Router的导航守卫分为三个层级：全局守卫、路由级守卫和组件级守卫。理解和正确使用这三层守卫是现代Vue应用的核心技能。

**1. 全局守卫的企业级应用：**

全局守卫在整个应用中执行，是实现统一认证、权限控制、日志记录等功能的最佳位置。beforeEach在每次导航前执行，beforeResolve在组件被解析后执行，afterEach在导航完成后执行。

**2. 路由级守卫的精准控制：**

路由级守卫(beforeEnter)直接定义在路由配置中，适合处理特定路由的权限验证、数据预加载等场景。它的执行时机在全局beforeEach之后，组件守卫之前。

**3. 组件级守卫的细粒度管理：**

组件级守卫定义在组件内部，提供最细粒度的控制。beforeRouteEnter在组件实例创建前执行，beforeRouteUpdate在路由参数变化时执行，beforeRouteLeave在离开组件前执行。

```vue
<!-- UserProfile.vue - 组件级守卫实践 -->
<script setup lang="ts">
import { ref, watch, onBeforeRouteUpdate, onBeforeRouteLeave } from 'vue'
import { useRoute, useRouter, onBeforeRouteEnter } from 'vue-router'
import { useUserStore } from '@/stores/user'

const route = useRoute()
const router = useRouter()
const userStore = useUserStore()

const userProfile = ref(null)
const isEditing = ref(false)
const hasUnsavedChanges = ref(false)

// 组件进入守卫（Composition API方式）
onBeforeRouteEnter(async (to, from, next) => {
  // 注意：这里不能访问组件实例
  const userId = to.params.id as string
  
  try {
    // 预加载用户数据
    if (userId) {
      await userStore.fetchUser(userId)
    }
    next()
  } catch (error) {
    console.error('加载用户数据失败:', error)
    next({ name: 'NotFound' })
  }
})

// 路由更新守卫
onBeforeRouteUpdate(async (to, from, next) => {
  const newUserId = to.params.id as string
  const oldUserId = from.params.id as string
  
  // 检查是否有未保存的更改
  if (hasUnsavedChanges.value) {
    const confirmed = await showConfirmDialog(
      '您有未保存的更改，确定要离开吗？'
    )
    if (!confirmed) {
      next(false)
      return
    }
  }
  
  // 如果用户ID发生变化，重新加载数据
  if (newUserId !== oldUserId && newUserId) {
    try {
      await userStore.fetchUser(newUserId)
      next()
    } catch (error) {
      console.error('加载用户数据失败:', error)
      next({ name: 'NotFound' })
    }
  } else {
    next()
  }
})

// 离开守卫
onBeforeRouteLeave(async (to, from, next) => {
  // 检查是否有未保存的更改
  if (hasUnsavedChanges.value) {
    const result = await showConfirmDialog(
      '您有未保存的更改，是否要保存？',
      {
        confirmText: '保存',
        cancelText: '不保存',
        showCancel: true,
      }
    )
    
    if (result === 'confirm') {
      try {
        await saveUserProfile()
        next()
      } catch (error) {
        console.error('保存失败:', error)
        next(false)
      }
    } else if (result === 'cancel') {
      next()
    } else {
      next(false)
    }
  } else {
    next()
  }
})

// 监听表单变化
watch(userProfile, () => {
  hasUnsavedChanges.value = true
}, { deep: true })

const saveUserProfile = async () => {
  // 保存逻辑
  await userStore.updateUser(route.params.id as string, userProfile.value)
  hasUnsavedChanges.value = false
}

const showConfirmDialog = (message: string, options = {}) => {
  // 实现确认对话框
  return new Promise((resolve) => {
    // 这里可以使用UI库的确认对话框
    const result = confirm(message)
    resolve(result)
  })
}
</script>
```

## 6. Vue 3 性能优化策略

### 虚拟列表优化实践
**大数据列表渲染的企业级解决方案：**

在处理大量数据列表时，传统的全量渲染会导致严重的性能问题。虚拟列表技术通过只渲染可视区域内的项目，大幅提升渲染性能。Vue 3的Composition API为虚拟列表的实现提供了更优雅的解决方案。

```vue
<!-- VirtualList.vue - 高性能虚拟列表组件 -->
<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'

interface Props {
  items: any[]            // 数据源
  itemHeight: number      // 每项高度
  containerHeight: number // 容器高度
  overscan?: number       // 额外渲染项数，提升滚动体验
}

interface Emits {
  (e: 'scroll', event: Event): void
  (e: 'itemClick', item: any, index: number): void
}

const props = withDefaults(defineProps<Props>(), {
  overscan: 5,
})

const emit = defineEmits<Emits>()

const containerRef = ref<HTMLElement>()
const scrollTop = ref(0)
const clientHeight = ref(0)

// 计算可视区域内的数据
const visibleData = computed(() => {
  const itemHeight = props.itemHeight
  const containerHeight = props.containerHeight
  const totalItems = props.items.length
  
  // 计算可见项目数量
  const visibleCount = Math.ceil(containerHeight / itemHeight)
  
  // 计算开始索引
  const startIndex = Math.floor(scrollTop.value / itemHeight)
  
  // 计算结束索引，加上overscan
  const endIndex = Math.min(
    startIndex + visibleCount + props.overscan,
    totalItems
  )
  
  // 实际开始索引，减去overscan
  const actualStartIndex = Math.max(startIndex - props.overscan, 0)
  
  return {
    startIndex: actualStartIndex,
    endIndex,
    visibleItems: props.items.slice(actualStartIndex, endIndex),
    offsetY: actualStartIndex * itemHeight,
    totalHeight: totalItems * itemHeight,
  }
})

// 滚动事件处理
const handleScroll = (event: Event) => {
  const target = event.target as HTMLElement
  scrollTop.value = target.scrollTop
  emit('scroll', event)
}

// 项目点击处理
const handleItemClick = (item: any, index: number) => {
  const actualIndex = visibleData.value.startIndex + index
  emit('itemClick', item, actualIndex)
}

// 滚动到指定项目
const scrollToItem = (index: number) => {
  if (containerRef.value) {
    const targetScrollTop = index * props.itemHeight
    containerRef.value.scrollTo({
      top: targetScrollTop,
      behavior: 'smooth',
    })
  }
}

// 获取当前可见项目数量
const getVisibleItemsCount = () => {
  return visibleData.value.visibleItems.length
}

// 暴露方法给父组件
defineExpose({
  scrollToItem,
  getVisibleItemsCount,
})

onMounted(() => {
  if (containerRef.value) {
    clientHeight.value = containerRef.value.clientHeight
  }
})
</script>

<template>
  <div 
    ref="containerRef"
    class="virtual-list-container"
    :style="{ height: `${containerHeight}px` }"
    @scroll="handleScroll"
  >
    <!-- 总高度占位元素 -->
    <div 
      class="virtual-list-phantom" 
      :style="{ height: `${visibleData.totalHeight}px` }"
    >
      <!-- 可视区域内容 -->
      <div 
        class="virtual-list-content"
        :style="{ transform: `translateY(${visibleData.offsetY}px)` }"
      >
        <div 
          v-for="(item, index) in visibleData.visibleItems"
          :key="visibleData.startIndex + index"
          class="virtual-list-item"
          :style="{ height: `${itemHeight}px` }"
          @click="handleItemClick(item, index)"
        >
          <slot 
            :item="item" 
            :index="visibleData.startIndex + index"
          >
            <!-- 默认插槽内容 -->
            <div class="default-item">
              {{ item }}
            </div>
          </slot>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.virtual-list-container {
  overflow-y: auto;
  position: relative;
  
  .virtual-list-phantom {
    position: relative;
  }
  
  .virtual-list-content {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
  }
  
  .virtual-list-item {
    display: flex;
    align-items: center;
    padding: 0 16px;
    border-bottom: 1px solid #eee;
    cursor: pointer;
    
    &:hover {
      background-color: #f5f5f5;
    }
  }
  
  .default-item {
    width: 100%;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
}
</style>
```

### 组件缓存与keep-alive优化
**智能缓存策略提升用户体验：**

Vue 3的keep-alive组件配合路由缓存可以显著提升应用性能。合理的缓存策略不仅能减少重复渲染，还能保持用户的操作状态，提供更流畅的用户体验。

```vue
<!-- App.vue - 智能路由缓存 -->
<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useAppStore } from '@/stores/app'

const route = useRoute()
const router = useRouter()
const appStore = useAppStore()

// 缓存组件名称列表
const cachedComponents = ref<string[]>(['Home', 'UserProfile', 'ProductList'])

// 动态缓存控制
const shouldCache = computed(() => {
  return route.meta.keepAlive && cachedComponents.value.includes(route.name as string)
})

// 缓存组件最大数量
const maxCacheCount = 10

// 添加组件到缓存
const addToCache = (componentName: string) => {
  if (!cachedComponents.value.includes(componentName)) {
    // 如果超过最大缓存数，移除最旧的组件
    if (cachedComponents.value.length >= maxCacheCount) {
      cachedComponents.value.shift()
    }
    cachedComponents.value.push(componentName)
  }
}

// 从缓存中移除组件
const removeFromCache = (componentName: string) => {
  const index = cachedComponents.value.indexOf(componentName)
  if (index > -1) {
    cachedComponents.value.splice(index, 1)
  }
}

// 清空特定类型的缓存
const clearCacheByType = (type: string) => {
  const routesToClear = router.getRoutes()
    .filter(route => route.meta.cacheType === type)
    .map(route => route.name as string)
    
  cachedComponents.value = cachedComponents.value.filter(
    name => !routesToClear.includes(name)
  )
}

// 监听路由变化，动态管理缓存
watch(() => route.name, (newName, oldName) => {
  if (route.meta.keepAlive && newName) {
    addToCache(newName as string)
  }
  
  // 根据业务规则清理缓存
  if (route.meta.clearCache) {
    const cacheType = route.meta.clearCache as string
    clearCacheByType(cacheType)
  }
})

// 内存监控和自动清理
const monitorMemoryUsage = () => {
  if ('memory' in performance) {
    const memInfo = (performance as any).memory
    const usageRatio = memInfo.usedJSHeapSize / memInfo.jsHeapSizeLimit
    
    // 当内存使用率超过80%时，清理部分缓存
    if (usageRatio > 0.8) {
      const componentsToRemove = cachedComponents.value.slice(0, Math.floor(cachedComponents.value.length / 2))
      componentsToRemove.forEach(removeFromCache)
      
      appStore.addNotification({
        type: 'warning',
        title: '内存优化',
        message: '已自动清理部分页面缓存以优化性能',
      })
    }
  }
}

// 每分钟检查一次内存使用情况
setInterval(monitorMemoryUsage, 60000)
</script>

<template>
  <div id="app">
    <router-view v-slot="{ Component, route }">
      <transition 
        :name="route.meta.transition || 'fade'"
        mode="out-in"
      >
        <keep-alive 
          :include="cachedComponents"
          :max="maxCacheCount"
        >
          <component 
            :is="Component" 
            :key="route.fullPath"
          />
        </keep-alive>
      </transition>
    </router-view>
  </div>
</template>

<style lang="scss">
// 路由过渡动画
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter-from {
  transform: translateX(100%);
}

.slide-leave-to {
  transform: translateX(-100%);
}
</style>
```

### 响应式数据优化策略
**ref vs reactive的最佳选择原则：**

Vue 3提供了ref和reactive两种响应式API，正确选择和使用这些API对性能有重要影响。理解它们的差异和适用场景是优化Vue应用的关键。

```typescript
// composables/usePerformantData.ts
import { ref, reactive, readonly, shallowRef, shallowReactive, markRaw } from 'vue'

// 性能优化原则1：基本数据类型使用ref，对象使用reactive
export function useUserData() {
  // ✅ 推荐：基本类型使用ref
  const userId = ref<string>('')
  const isLoading = ref(false)
  const count = ref(0)
  
  // ✅ 推荐：对象使用reactive
  const userInfo = reactive({
    name: '',
    email: '',
    profile: {
      avatar: '',
      bio: '',
    },
  })
  
  return {
    userId,
    isLoading,
    count,
    userInfo,
  }
}

// 性能优化原则2：大型数据结构使用shallowRef/shallowReactive
export function useLargeDataSet() {
  // ✅ 对于大型数组或对象，使用shallow版本减少响应式开销
  const largeDataSet = shallowRef<any[]>([])
  
  const updateLargeDataSet = (newData: any[]) => {
    // 整体替换而非修改内部属性
    largeDataSet.value = newData
  }
  
  return {
    largeDataSet: readonly(largeDataSet),
    updateLargeDataSet,
  }
}

// 性能优化原则3：静态数据使用markRaw
export function useStaticConfig() {
  // ✅ 配置数据不需要响应式
  const appConfig = markRaw({
    apiUrl: 'https://api.example.com',
    version: '1.0.0',
    features: ['feature1', 'feature2'],
  })
  
  return {
    appConfig,
  }
}

// 性能优化原则4：条件性响应式
export function useConditionalReactivity() {
  const isReactive = ref(true)
  const data = ref({})
  
  const toggleReactivity = () => {
    if (isReactive.value) {
      // 禁用响应式
      data.value = markRaw(data.value)
    } else {
      // 重新启用响应式
      data.value = reactive(data.value)
    }
    isReactive.value = !isReactive.value
  }
  
  return {
    data,
    isReactive: readonly(isReactive),
    toggleReactivity,
  }
}

// 性能优化原则5：合理使用computed缓存
export function useComputedOptimization() {
  const expensiveData = ref<number[]>([...Array(10000)].map((_, i) => i))
  
  // ✅ 复杂计算使用computed缓存
  const expensiveComputation = computed(() => {
    console.log('执行复杂计算') // 只在依赖变化时执行
    return expensiveData.value
      .filter(n => n % 2 === 0)
      .map(n => n * 2)
      .reduce((sum, n) => sum + n, 0)
  })
  
  // ✅ 使用缓存key避免重复计算
  const memoizedResults = new Map()
  
  const getMemoizedResult = (key: string, computeFn: () => any) => {
    if (!memoizedResults.has(key)) {
      memoizedResults.set(key, computeFn())
    }
    return memoizedResults.get(key)
  }
  
  return {
    expensiveComputation,
    getMemoizedResult,
  }
}
```

## 7. Vue 3 测试策略

### Vitest + Vue Test Utils 组件测试
**现代化Vue组件测试的完整方案：**

Vitest作为2025年Vue生态的主流测试框架，配合Vue Test Utils提供了强大的组件测试能力。正确的测试策略不仅能保证代码质量，还能提升开发效率和重构信心。

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  test: {
    // 测试环境配置
    environment: 'jsdom',
    
    // 全局设置文件
    setupFiles: ['./src/test/setup.ts'],
    
    // 覆盖率配置
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        '**/*.test.{ts,js}',
        '**/*.spec.{ts,js}',
      ],
      thresholds: {
        global: {
          branches: 80,
          functions: 80,
          lines: 80,
          statements: 80,
        },
      },
    },
    
    // 测试文件匹配
    include: ['src/**/*.{test,spec}.{js,ts,vue}'],
    
    // 模拟配置
    mockReset: true,
    clearMocks: true,
    restoreMocks: true,
    
    // 测试超时设置
    testTimeout: 10000,
  },
  
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
    },
  },
})
```

```typescript
// src/test/setup.ts - 测试全局设置
import { vi } from 'vitest'
import { config } from '@vue/test-utils'
import { createPinia } from 'pinia'
import { createI18n } from 'vue-i18n'
import { createRouter, createWebHistory } from 'vue-router'
import ElementPlus from 'element-plus'

// Mock全局对象
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
})

// Mock ResizeObserver
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}))

// Mock IntersectionObserver
global.IntersectionObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
  takeRecords: vi.fn(),
}))

// 配置Vue Test Utils全局插件
config.global.plugins = [
  createPinia(),
  ElementPlus,
  createI18n({
    legacy: false,
    locale: 'zh-CN',
    messages: {
      'zh-CN': {},
      'en-US': {},
    },
  }),
  createRouter({
    history: createWebHistory(),
    routes: [],
  }),
]

// Mock全局属性
config.global.mocks = {
  $t: (key: string) => key,
  $tc: (key: string) => key,
  $router: {
    push: vi.fn(),
    replace: vi.fn(),
    go: vi.fn(),
    back: vi.fn(),
    forward: vi.fn(),
  },
  $route: {
    path: '/',
    name: 'Home',
    params: {},
    query: {},
    meta: {},
  },
}
```

```typescript
// src/components/__tests__/UserProfile.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'
import { mount, VueWrapper } from '@vue/test-utils'
import { nextTick } from 'vue'
import { createPinia, setActivePinia } from 'pinia'
import UserProfile from '../UserProfile.vue'
import { useUserStore } from '@/stores/user'
import type { UserInfo } from '@/types'

// Mock API
vi.mock('@/services/userApi', () => ({
  userApi: {
    getUser: vi.fn(),
    updateUser: vi.fn(),
    deleteUser: vi.fn(),
  },
}))

describe('UserProfile.vue', () => {
  let wrapper: VueWrapper<any>
  let userStore: ReturnType<typeof useUserStore>
  const mockUser: UserInfo = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
    phone: '13800138000',
    avatar: 'https://example.com/avatar.jpg',
    bio: 'Test user bio',
    preferences: {
      theme: 'light',
      language: 'zh-CN',
      notifications: true,
    },
  }

  beforeEach(() => {
    setActivePinia(createPinia())
    userStore = useUserStore()
    
    // Mock store方法
    vi.spyOn(userStore, 'fetchUser').mockResolvedValue(mockUser)
    vi.spyOn(userStore, 'updateUser').mockResolvedValue(mockUser)
    vi.spyOn(userStore, 'deleteUser').mockResolvedValue(undefined)
  })

  afterEach(() => {
    wrapper?.unmount()
    vi.clearAllMocks()
  })

  // 基本渲染测试
  it('应该正确渲染用户信息', async () => {
    wrapper = mount(UserProfile, {
      props: {
        userId: '1',
      },
    })

    await nextTick()
    
    expect(wrapper.find('.profile-card').exists()).toBe(true)
    expect(wrapper.find('h2').text()).toContain(mockUser.name)
  })

  // Props测试
  it('应该根据readonly prop禁用编辑功能', async () => {
    wrapper = mount(UserProfile, {
      props: {
        userId: '1',
        readonly: true,
      },
    })

    await nextTick()
    
    expect(wrapper.find('[data-test="edit-button"]').exists()).toBe(false)
  })

  // 事件测试
  it('应该正确发出update事件', async () => {
    wrapper = mount(UserProfile, {
      props: {
        userId: '1',
      },
    })

    await nextTick()
    
    // 模拟编辑操作
    await wrapper.find('[data-test="edit-button"]').trigger('click')
    await wrapper.find('[data-test="name-input"]').setValue('Jane Doe')
    await wrapper.find('[data-test="save-button"]').trigger('click')
    
    await nextTick()
    
    expect(wrapper.emitted('update')).toBeTruthy()
    expect(wrapper.emitted('update')?.[0][0]).toEqual(
      expect.objectContaining({ name: 'Jane Doe' })
    )
  })

  // 异步操作测试
  it('应该正确处理加载状态', async () => {
    // 模拟加载延迟
    vi.spyOn(userStore, 'fetchUser').mockImplementation(
      () => new Promise(resolve => setTimeout(() => resolve(mockUser), 100))
    )
    
    wrapper = mount(UserProfile, {
      props: {
        userId: '1',
      },
    })
    
    // 检查加载状态
    expect(wrapper.find('[data-test="loading"]').exists()).toBe(true)
    
    // 等待加载完成
    await vi.waitFor(() => {
      expect(wrapper.find('[data-test="loading"]').exists()).toBe(false)
    })
  })

  // 错误处理测试
  it('应该正确处理加载错误', async () => {
    const errorMessage = '用户不存在'
    vi.spyOn(userStore, 'fetchUser').mockRejectedValue(new Error(errorMessage))
    
    wrapper = mount(UserProfile, {
      props: {
        userId: '999',
      },
    })
    
    await nextTick()
    
    expect(wrapper.find('[data-test="error-message"]').text()).toContain(errorMessage)
  })

  // 表单验证测试
  it('应该正确验证表单输入', async () => {
    wrapper = mount(UserProfile, {
      props: {
        userId: '1',
      },
    })

    await nextTick()
    await wrapper.find('[data-test="edit-button"]').trigger('click')
    
    // 测试非法邮箱格式
    const emailInput = wrapper.find('[data-test="email-input"]')
    await emailInput.setValue('invalid-email')
    await emailInput.trigger('blur')
    
    await nextTick()
    
    expect(wrapper.find('[data-test="email-error"]').text()).toContain('邮箱格式不正确')
  })

  // Composables测试
  it('应该正确使用自定义组合式函数', async () => {
    const { result } = renderComposable(() => {
      const { useFormValidation } = require('@/composables/useFormValidation')
      return useFormValidation(mockUser, {
        name: { required: true, minLength: 2 },
        email: { required: true, pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ },
      })
    })
    
    expect(result.isValid.value).toBe(true)
    
    // 测试验证失败
    result.validateSingleField('name')
    mockUser.name = ''
    await nextTick()
    
    expect(result.errors.name).toBeTruthy()
    expect(result.isValid.value).toBe(false)
  })
})

// 辅助函数：渲染Composable
function renderComposable<T>(composable: () => T) {
  let result: T
  const app = mount({
    setup() {
      result = composable()
      return () => {}
    },
  })
  
  return {
    result: result!,
    unmount: app.unmount,
  }
}
```

### E2E测试与Playwright集成
**全流程用户体验测试：**

E2E测试保证了整个用户流程的正确性，Playwright作为现代化的E2E测试工具，提供了强大的跨浏览器测试能力。

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  
  // 并行测试数量
  workers: process.env.CI ? 1 : undefined,
  
  // 失败重试次数
  retries: process.env.CI ? 2 : 0,
  
  // 记者配置
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results.json' }],
    ['junit', { outputFile: 'test-results.xml' }],
  ],
  
  use: {
    // 基础URL
    baseURL: 'http://localhost:3000',
    
    // 追踪配置
    trace: 'on-first-retry',
    
    // 截图配置
    screenshot: 'only-on-failure',
    
    // 视频录制
    video: 'retain-on-failure',
  },
  
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'mobile-chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
  
  // 开发服务器配置
  webServer: {
    command: 'npm run dev',
    port: 3000,
    reuseExistingServer: !process.env.CI,
  },
})
```

```typescript
// tests/e2e/user-workflow.spec.ts
import { test, expect, type Page } from '@playwright/test'

// 测试用户流程
test.describe('用户管理流程', () => {
  let page: Page
  
  test.beforeEach(async ({ page: testPage }) => {
    page = testPage
    
    // 登录前置条件
    await page.goto('/login')
    await page.fill('[data-test="username"]', 'admin@example.com')
    await page.fill('[data-test="password"]', 'password123')
    await page.click('[data-test="login-button"]')
    
    // 等待登录完成
    await expect(page).toHaveURL('/dashboard')
  })
  
  test('应该能够查看用户列表', async () => {
    await page.goto('/users')
    
    // 检查页面元素
    await expect(page.locator('[data-test="user-list"]')).toBeVisible()
    await expect(page.locator('[data-test="user-item"]')).toHaveCount(10)
    
    // 检查搜索功能
    await page.fill('[data-test="search-input"]', 'john')
    await page.click('[data-test="search-button"]')
    
    await expect(page.locator('[data-test="user-item"]')).toHaveCount(1)
    await expect(page.locator('[data-test="user-item"]').first()).toContainText('John')
  })
  
  test('应该能够创建新用户', async () => {
    await page.goto('/users')
    await page.click('[data-test="create-user-button"]')
    
    // 等待模态框出现
    await expect(page.locator('[data-test="create-user-modal"]')).toBeVisible()
    
    // 填写表单
    await page.fill('[data-test="user-name"]', '新用户')
    await page.fill('[data-test="user-email"]', 'newuser@example.com')
    await page.fill('[data-test="user-phone"]', '13900139000')
    
    // 上传头像
    await page.setInputFiles('[data-test="avatar-upload"]', 'tests/fixtures/avatar.jpg')
    
    // 提交表单
    await page.click('[data-test="submit-button"]')
    
    // 验证成功消息
    await expect(page.locator('[data-test="success-message"]')).toBeVisible()
    await expect(page.locator('[data-test="success-message"]')).toContainText('用户创建成功')
    
    // 验证用户出现在列表中
    await expect(page.locator('[data-test="user-item"]').last()).toContainText('新用户')
  })
  
  test('应该能够编辑用户信息', async () => {
    await page.goto('/users')
    
    // 点击第一个用户的编辑按钮
    await page.click('[data-test="user-item"]:first-child [data-test="edit-button"]')
    
    // 等待跳转到编辑页面
    await expect(page).toHaveURL(/\/users\/\d+\/edit/)
    
    // 修改用户名
    await page.fill('[data-test="user-name"]', '更新后的用户名')
    
    // 保存修改
    await page.click('[data-test="save-button"]')
    
    // 验证保存成功
    await expect(page.locator('[data-test="success-message"]')).toBeVisible()
  })
  
  test('应该能够删除用户', async () => {
    await page.goto('/users')
    
    // 记录初始用户数量
    const initialCount = await page.locator('[data-test="user-item"]').count()
    
    // 点击删除按钮
    await page.click('[data-test="user-item"]:first-child [data-test="delete-button"]')
    
    // 确认删除
    await expect(page.locator('[data-test="confirm-dialog"]')).toBeVisible()
    await page.click('[data-test="confirm-button"]')
    
    // 验证用户被删除
    await expect(page.locator('[data-test="user-item"]')).toHaveCount(initialCount - 1)
    await expect(page.locator('[data-test="success-message"]')).toContainText('用户删除成功')
  })
  
  test('应该能够处理网络错误', async () => {
    // 模拟网络失败
    await page.route('**/api/users', route => {
      route.abort('failed')
    })
    
    await page.goto('/users')
    
    // 验证错误消息显示
    await expect(page.locator('[data-test="error-message"]')).toBeVisible()
    await expect(page.locator('[data-test="error-message"]')).toContainText('加载失败')
    
    // 验证重试按钮
    await expect(page.locator('[data-test="retry-button"]')).toBeVisible()
  })
  
  test('应该支持移动端响应式布局', async () => {
    // 设置移动端视口
    await page.setViewportSize({ width: 375, height: 667 })
    await page.goto('/users')
    
    // 检查移动端导航
    await expect(page.locator('[data-test="mobile-menu-button"]')).toBeVisible()
    
    // 点击菜单按钮
    await page.click('[data-test="mobile-menu-button"]')
    await expect(page.locator('[data-test="mobile-menu"]')).toBeVisible()
    
    // 检查卡片布局适应
    const userCards = page.locator('[data-test="user-card"]')
    await expect(userCards.first()).toHaveCSS('width', '100%')
  })
})
```

## 8. 生产部署与构建优化

### Vite生产构建优化
**2025年最佳的构建配置实践：**

Vite作为现代化构建工具，在生产环境中需要细致的优化配置才能发挥最大性能。正确的构建优化能显著提升应用的加载速度和运行效率。

```typescript
// vite.config.prod.ts - 生产环境优化配置
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'
import { visualizer } from 'rollup-plugin-visualizer'
import { compression } from 'vite-plugin-compression'
import { createHtmlPlugin } from 'vite-plugin-html'
import viteCompression from 'vite-plugin-compression'

export default defineConfig({
  plugins: [
    vue({
      // 生产环境禁用开发工具
      include: [/\.vue$/],
      reactivityTransform: false,
    }),
    
    // HTML模板优化
    createHtmlPlugin({
      minify: true,
      inject: {
        data: {
          title: 'Vue3 企业级应用',
          description: '基于Vue 3.5+的现代化企业级应用',
        },
      },
    }),
    
    // Gzip压缩
    viteCompression({
      algorithm: 'gzip',
      threshold: 10240, // 10KB以上的文件才压缩
    }),
    
    // Brotli压缩
    viteCompression({
      algorithm: 'brotliCompress',
      threshold: 10240,
      ext: '.br',
    }),
    
    // 打包分析
    visualizer({
      filename: 'dist/stats.html',
      open: true,
      gzipSize: true,
      brotliSize: true,
    }),
  ],
  
  // 构建优化
  build: {
    target: 'es2020',
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: true,
    
    // 清空输出目录
    emptyOutDir: true,
    
    // 大文件警告阈值
    chunkSizeWarningLimit: 1000,
    
    // Rollup配置
    rollupOptions: {
      // 外部化依赖
      external: [],
      
      // 输出配置
      output: {
        // 静态资源命名
        assetFileNames: (assetInfo) => {
          const info = assetInfo.name!.split('.')
          const ext = info[info.length - 1]
          
          if (/\.(png|jpe?g|gif|svg|webp|ico)$/i.test(assetInfo.name!)) {
            return `images/[name]-[hash][extname]`
          }
          if (/\.(woff2?|eot|ttf|otf)$/i.test(assetInfo.name!)) {
            return `fonts/[name]-[hash][extname]`
          }
          return `assets/[name]-[hash][extname]`
        },
        
        // JS文件命名
        chunkFileNames: 'js/[name]-[hash].js',
        entryFileNames: 'js/[name]-[hash].js',
        
        // 手动代码分割
        manualChunks: {
          // Vue核心库
          'vue-vendor': ['vue', 'vue-router'],
          
          // 状态管理
          'state-vendor': ['pinia'],
          
          // UI组件库
          'ui-vendor': ['element-plus', '@element-plus/icons-vue'],
          
          // 工具库
          'utils-vendor': ['lodash-es', 'dayjs', 'axios'],
          
          // 图表库
          'chart-vendor': ['echarts', 'vue-echarts'],
        },
        
        // 动态分块
        dynamicImportFunction: 'import',
      },
    },
    
    // 压缩配置
    minify: 'esbuild',
    
    // esbuild压缩配置
    esbuild: {
      drop: ['console', 'debugger'],
      pure: ['console.log'],
      legalComments: 'none',
    },
    
    // CSS代码分割
    cssCodeSplit: true,
    
    // 关闭文件大小报告
    reportCompressedSize: false,
  },
  
  // CSS优化
  css: {
    // CSS模块配置
    modules: {
      localsConvention: 'camelCase',
    },
    
    // PostCSS配置
    postcss: {
      plugins: [
        require('autoprefixer'),
        require('cssnano')({
          preset: 'default',
        }),
      ],
    },
  },
  
  // 依赖优化
  optimizeDeps: {
    include: [
      'vue',
      'vue-router',
      'pinia',
      'axios',
      'element-plus',
      'dayjs',
    ],
    exclude: ['vue-demi'],
  },
})
```

### Docker容器化部署
**现代化多阶段构建的最佳实践：**

容器化部署为现代应用提供了一致性、可移植性和可扩展性。多阶段构建能显著减小镜像大小并提升构建效率。

```dockerfile
# Dockerfile - 多阶段优化构建

# 阶段1: 构建阶段
FROM node:20-alpine AS builder

# 设置工作目录
WORKDIR /app

# 复制包管理文件
COPY package*.json ./
COPY pnpm-lock.yaml ./

# 安装pnpm
RUN npm install -g pnpm

# 安装依赖（利用Docker层缓存）
RUN pnpm install --frozen-lockfile

# 复制源代码
COPY . .

# 构建应用
RUN pnpm run build

# 阶段2: 生产阶段
FROM nginx:alpine AS production

# 安装必要工具
RUN apk add --no-cache \
    curl \
    jq \
    && rm -rf /var/cache/apk/*

# 创建非特权用户
RUN addgroup -g 1001 -S nodejs \
    && adduser -S nextjs -u 1001

# 设置时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 复制Nginx配置
COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx.default.conf /etc/nginx/conf.d/default.conf

# 创建日志目录
RUN mkdir -p /var/log/nginx && \
    touch /var/log/nginx/access.log && \
    touch /var/log/nginx/error.log && \
    chown -R nginx:nginx /var/log/nginx

# 暴露端口
EXPOSE 80

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:80/ || exit 1

# 启动命令
CMD ["nginx", "-g", "daemon off;"]
```

### CI/CD流水线配置
**GitHub Actions企业级自动化部署：**

现代化CI/CD流水线不仅要实现自动化构建和部署，还要包含代码质量检查、安全扫描、性能测试等环节。

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  release:
    types: [published]

env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # 代码质量检查
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Type checking
        run: pnpm run type-check
      
      - name: Linting
        run: pnpm run lint
      
      - name: Code formatting
        run: pnpm run format:check
      
      - name: Security audit
        run: pnpm audit --audit-level moderate

  # 单元测试
  test:
    runs-on: ubuntu-latest
    needs: quality
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run unit tests
        run: pnpm run test:unit --coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info

  # E2E测试
  e2e:
    runs-on: ubuntu-latest
    needs: [quality, test]
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Install Playwright
        run: pnpm exec playwright install --with-deps
      
      - name: Build application
        run: pnpm run build
      
      - name: Run E2E tests
        run: pnpm run test:e2e
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  # 构建和发布
  build-and-deploy:
    runs-on: ubuntu-latest
    needs: [test, e2e]
    if: github.ref == 'refs/heads/main' || github.event_name == 'release'
    
    permissions:
      contents: read
      packages: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Deploy to staging
        if: github.ref == 'refs/heads/develop'
        run: |
          echo "Deploying to staging environment"
          # 这里可以添加部署到测试环境的脚本
      
      - name: Deploy to production
        if: github.ref == 'refs/heads/main'
        run: |
          echo "Deploying to production environment"
          # 这里可以添加部署到生产环境的脚本
      
      - name: Send notification
        uses: 8398a7/action-slack@v3
        if: always()
        with:
          status: ${{ job.status }}
          channel: '#deployments'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}

  # 性能测试
  performance:
    runs-on: ubuntu-latest
    needs: build-and-deploy
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Lighthouse CI
        uses: treosh/lighthouse-ci-action@v10
        with:
          configPath: './.lighthouserc.json'
          uploadArtifacts: true
          temporaryPublicStorage: true

  # 安全扫描
  security:
    runs-on: ubuntu-latest
    needs: quality
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

这个完整的Vue工程实践面试题深度分析文档涵盖了从项目架构到生产部署的全流程技术实践。每个章节都提供了详细的技术要点说明和实用的代码示例，确保内容的实用性和指导意义。这些最佳实践反映了2025年Vue生态的最新发展趋势，帮助开发者构建高质量的企业级Vue应用。
```
```
```
# React工程实践面试题深度分析

## 1. 项目架构设计

### 现代化React项目结构
**模块化架构示例：**
```
src/
├── components/           # 通用组件
│   ├── ui/              # 基础UI组件
│   ├── forms/           # 表单组件
│   └── layout/          # 布局组件
├── hooks/               # 自定义Hooks
├── utils/               # 工具函数
├── services/            # API服务层
├── stores/              # 状态管理
├── types/               # TypeScript类型定义
├── pages/               # 页面组件
├── routes/              # 路由配置
├── styles/              # 样式文件
├── assets/              # 静态资源
└── constants/           # 常量定义
```

### 组件设计原则深度实践

**现代React组件设计的核心理念：**

在2025年的React生态中，组件设计已经从简单的UI拆分进化为一套完整的软件工程实践。优秀的组件设计不仅仅是代码的重用，更是系统架构稳定性和可维护性的基石。基于SOLID原则和React最佳实践，现代组件设计需要遵循以下核心理念：

#### 单一职责原则(SRP)在React组件中的深度应用

**单一职责的具体含义：**
一个React组件应该只有一个变化的理由。这意味着组件只应该关注一个特定的业务功能或UI职责，而不是试图处理多个不相关的关注点。

**反面案例 - 违反单一职责原则：**
```jsx
// ❌ 错误设计：一个组件承担过多职责
function UserDashboard({ userId }) {
  const [user, setUser] = useState(null)
  const [posts, setPosts] = useState([])
  const [notifications, setNotifications] = useState([])
  const [analytics, setAnalytics] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)
  
  useEffect(() => {
    // 职责1：用户数据获取
    fetchUser(userId).then(setUser).catch(setError)
    
    // 职责2：用户文章获取
    fetchUserPosts(userId).then(setPosts).catch(setError)
    
    // 职责3：通知数据获取
    fetchNotifications(userId).then(setNotifications).catch(setError)
    
    // 职责4：分析数据获取
    fetchAnalytics(userId).then(setAnalytics).catch(setError)
    
    setLoading(false)
  }, [userId])
  
  // 职责5：数据验证
  const validateUserData = (data) => {
    return data.email && data.name && data.phone
  }
  
  // 职责6：数据格式化
  const formatDate = (date) => {
    return new Intl.DateTimeFormat('zh-CN').format(new Date(date))
  }
  
  // 职责7：通知发送
  const sendNotification = async (message) => {
    await notificationService.send(userId, message)
  }
  
  return (
    <div className="dashboard">
      {/* 职责8：复杂的UI渲染逻辑 */}
      <header>
        <h1>{user?.name}的仪表板</h1>
        <span>最后登录：{user && formatDate(user.lastLogin)}</span>
      </header>
      
      <div className="content">
        <section className="user-info">
          <h2>用户信息</h2>
          {/* 用户信息展示逻辑 */}
        </section>
        
        <section className="posts">
          <h2>最新文章</h2>
          {/* 文章列表展示逻辑 */}
        </section>
        
        <section className="notifications">
          <h2>通知</h2>
          {/* 通知列表展示逻辑 */}
        </section>
        
        <section className="analytics">
          <h2>数据分析</h2>
          {/* 分析图表展示逻辑 */}
        </section>
      </div>
    </div>
  )
}
```

**这种设计的问题分析：**
1. **维护困难**：任何功能的修改都可能影响整个组件
2. **测试复杂**：需要模拟所有依赖才能测试单一功能
3. **重用性差**：无法单独复用其中的某个功能
4. **职责混乱**：数据获取、验证、格式化、UI渲染都混在一起
5. **性能问题**：任何状态变化都会导致整个组件重新渲染

**正确设计 - 遵循单一职责原则：**
```jsx
// ✅ 职责分离：数据获取逻辑
function useUserData(userId) {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)
  
  useEffect(() => {
    let isMounted = true
    
    fetchUser(userId)
      .then(data => {
        if (isMounted) {
          setUser(data)
          setLoading(false)
        }
      })
      .catch(err => {
        if (isMounted) {
          setError(err)
          setLoading(false)
        }
      })
    
    return () => {
      isMounted = false
    }
  }, [userId])
  
  return { user, loading, error }
}

// ✅ 职责分离：用户文章数据
function useUserPosts(userId) {
  const [posts, setPosts] = useState([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)
  
  useEffect(() => {
    if (!userId) return
    
    let isMounted = true
    
    fetchUserPosts(userId)
      .then(data => {
        if (isMounted) {
          setPosts(data)
          setLoading(false)
        }
      })
      .catch(err => {
        if (isMounted) {
          setError(err)
          setLoading(false)
        }
      })
    
    return () => {
      isMounted = false
    }
  }, [userId])
  
  return { posts, loading, error, refetch: () => fetchUserPosts(userId).then(setPosts) }
}

// ✅ 职责分离：数据验证工具
const userValidators = {
  email: (email) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email),
  phone: (phone) => /^1[3-9]\d{9}$/.test(phone),
  name: (name) => name && name.length >= 2 && name.length <= 50,
  
  validateUser: (user) => {
    const errors = []
    if (!userValidators.email(user.email)) {
      errors.push('邮箱格式不正确')
    }
    if (!userValidators.phone(user.phone)) {
      errors.push('手机号格式不正确')
    }
    if (!userValidators.name(user.name)) {
      errors.push('姓名长度必须在2-50字符之间')
    }
    return { isValid: errors.length === 0, errors }
  }
}

// ✅ 职责分离：日期格式化工具
const dateFormatters = {
  toLocalDate: (date) => {
    return new Intl.DateTimeFormat('zh-CN', {
      year: 'numeric',
      month: '2-digit',
      day: '2-digit'
    }).format(new Date(date))
  },
  
  toRelativeTime: (date) => {
    const now = new Date()
    const target = new Date(date)
    const diffInSeconds = Math.floor((now - target) / 1000)
    
    if (diffInSeconds < 60) return '刚刚'
    if (diffInSeconds < 3600) return `${Math.floor(diffInSeconds / 60)}分钟前`
    if (diffInSeconds < 86400) return `${Math.floor(diffInSeconds / 3600)}小时前`
    if (diffInSeconds < 2592000) return `${Math.floor(diffInSeconds / 86400)}天前`
    
    return dateFormatters.toLocalDate(date)
  }
}

// ✅ 职责分离：用户信息展示组件
function UserProfileCard({ user }) {
  if (!user) return <div className="profile-skeleton">加载中...</div>
  
  const { isValid, errors } = userValidators.validateUser(user)
  
  return (
    <div className="user-profile-card">
      <div className="avatar-section">
        <img 
          src={user.avatar || '/default-avatar.png'} 
          alt={user.name}
          className="avatar"
        />
        <div className="user-basic">
          <h3>{user.name}</h3>
          <p className="email">{user.email}</p>
          {!isValid && (
            <div className="validation-errors">
              {errors.map((error, index) => (
                <span key={index} className="error-message">{error}</span>
              ))}
            </div>
          )}
        </div>
      </div>
      
      <div className="user-stats">
        <div className="stat">
          <span className="label">最后登录</span>
          <span className="value">{dateFormatters.toRelativeTime(user.lastLogin)}</span>
        </div>
        <div className="stat">
          <span className="label">注册时间</span>
          <span className="value">{dateFormatters.toLocalDate(user.createdAt)}</span>
        </div>
      </div>
    </div>
  )
}

// ✅ 职责分离：文章列表组件
function UserPostsList({ userId }) {
  const { posts, loading, error, refetch } = useUserPosts(userId)
  
  if (loading) return <PostsListSkeleton />
  if (error) return <ErrorMessage error={error} onRetry={refetch} />
  if (posts.length === 0) return <EmptyState message="暂无文章" />
  
  return (
    <div className="posts-list">
      <div className="list-header">
        <h3>最新文章 ({posts.length})</h3>
        <button onClick={refetch} className="refresh-btn">
          刷新
        </button>
      </div>
      
      <div className="posts-grid">
        {posts.map(post => (
          <PostCard 
            key={post.id} 
            post={post} 
            showAuthor={false} // 已知是当前用户的文章
          />
        ))}
      </div>
    </div>
  )
}

// ✅ 职责分离：主仪表板组件（仅负责布局协调）
function UserDashboard({ userId }) {
  const { user, loading: userLoading, error: userError } = useUserData(userId)
  
  if (userLoading) return <DashboardSkeleton />
  if (userError) return <ErrorBoundary error={userError} />
  if (!user) return <NotFoundPage />
  
  return (
    <div className="user-dashboard">
      <header className="dashboard-header">
        <h1>{user.name}的仪表板</h1>
        <DashboardActions userId={userId} />
      </header>
      
      <div className="dashboard-content">
        <aside className="sidebar">
          <UserProfileCard user={user} />
          <QuickActions userId={userId} />
        </aside>
        
        <main className="main-content">
          <div className="content-grid">
            <section className="posts-section">
              <UserPostsList userId={userId} />
            </section>
            
            <section className="notifications-section">
              <UserNotifications userId={userId} />
            </section>
            
            <section className="analytics-section">
              <UserAnalytics userId={userId} />
            </section>
          </div>
        </main>
      </div>
    </div>
  )
}
```

**重构后的优势分析：**

1. **职责清晰**：每个组件和Hook都有明确的单一职责
   - `useUserData`: 专门处理用户数据获取
   - `useUserPosts`: 专门处理文章数据
   - `userValidators`: 专门处理数据验证
   - `dateFormatters`: 专门处理日期格式化
   - `UserProfileCard`: 专门展示用户信息
   - `UserPostsList`: 专门展示文章列表
   - `UserDashboard`: 专门处理布局协调

2. **可测试性**：每个单元都可以独立测试
```jsx
// 测试数据验证逻辑
describe('userValidators', () => {
  test('should validate email correctly', () => {
    expect(userValidators.email('test@example.com')).toBe(true)
    expect(userValidators.email('invalid-email')).toBe(false)
  })
  
  test('should validate user object', () => {
    const validUser = {
      email: 'test@example.com',
      phone: '13800138000',
      name: 'John Doe'
    }
    const { isValid } = userValidators.validateUser(validUser)
    expect(isValid).toBe(true)
  })
})

// 测试用户数据Hook
describe('useUserData', () => {
  test('should fetch user data', async () => {
    const { result } = renderHook(() => useUserData('123'))
    
    expect(result.current.loading).toBe(true)
    
    await waitFor(() => {
      expect(result.current.loading).toBe(false)
      expect(result.current.user).toBeDefined()
    })
  })
})
```

3. **可重用性**：组件和工具函数可以在不同场景下复用
```jsx
// 在其他页面重用用户卡片
function AdminUserList() {
  return (
    <div>
      {users.map(user => (
        <UserProfileCard key={user.id} user={user} />
      ))}
    </div>
  )
}

// 重用日期格式化工具
function CommentList({ comments }) {
  return (
    <div>
      {comments.map(comment => (
        <div key={comment.id}>
          <p>{comment.content}</p>
          <span>{dateFormatters.toRelativeTime(comment.createdAt)}</span>
        </div>
      ))}
    </div>
  )
}
```

4. **维护性**：修改某个功能不会影响其他功能
   - 修改日期格式只需要修改`dateFormatters`
   - 修改验证逻辑只需要修改`userValidators`
   - 修改数据获取逻辑只需要修改对应的Hook

5. **性能优化**：每个组件只在相关数据变化时重新渲染
```jsx
// UserProfileCard只在user变化时重新渲染
const MemoizedUserProfileCard = memo(UserProfileCard)

// UserPostsList只在userId变化时重新获取数据
function UserPostsList({ userId }) {
  const { posts, loading, error } = useUserPosts(userId) // 只依赖userId
  // ...
}
```

**实际项目中的应用指导：**

1. **识别违反SRP的信号：**
   - 组件名称中包含"and"、"or"等连接词
   - 组件代码超过200行
   - 组件依赖过多的props（超过8个）
   - 组件内部有多个useState和useEffect
   - 修改一个功能需要同时修改多个地方

2. **重构策略：**
   - 按功能领域拆分（用户管理、文章管理、通知管理）
   - 按技术层级拆分（数据层、业务层、展示层）

#### SOLID原则在React组件设计中的综合应用

**开闭原则(OCP)实践：**
通过策略模式和组件组合实现可扩展的组件设计。组件应该能够通过配置、插槽、或策略注入的方式扩展功能，而不需要修改核心代码。

```jsx
// ✅ 可扩展的表单验证组件
function FormField({ type, validators = [], ...props }) {
  const [value, setValue] = useState('')
  const [errors, setErrors] = useState([])
  
  const validate = (inputValue) => {
    // 对扩展开放：可以传入任意验证器
    const validationErrors = validators
      .map(validator => validator(inputValue))
      .filter(Boolean)
    setErrors(validationErrors)
  }
  
  return (
    <div className="form-field">
      <input 
        type={type}
        value={value}
        onChange={(e) => {
          setValue(e.target.value)
          validate(e.target.value)
        }}
        {...props}
      />
      {errors.map((error, index) => (
        <span key={index} className="error">{error}</span>
      ))}
    </div>
  )
}

// 验证器策略（可扩展）
const validators = {
  required: (value) => !value ? '此字段为必填项' : null,
  email: (value) => !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? '邮箱格式不正确' : null,
  minLength: (min) => (value) => value.length < min ? `最少需要${min}个字符` : null,
}

// 使用时可以灵活组合验证规则
<FormField 
  type="email"
  validators={[validators.required, validators.email]}
  placeholder="请输入邮箱"
/>
```

**里氏替换原则(LSP)实践：**
确保所有实现同一接口的组件能够安全替换，不破坏系统行为。

```jsx
// ✅ 一致的按钮接口
const BaseButton = ({ children, onClick, disabled, ...props }) => (
  <button 
    onClick={disabled ? undefined : onClick}
    disabled={disabled}
    {...props}
  >
    {children}
  </button>
)

const PrimaryButton = (props) => (
  <BaseButton {...props} className="btn-primary" />
)

const ConfirmButton = ({ onConfirm, needsConfirm, ...props }) => {
  const [confirmed, setConfirmed] = useState(false)
  
  const handleClick = () => {
    if (needsConfirm && !confirmed) {
      setConfirmed(true)
      return
    }
    // 保持一致的行为：总是调用onClick
    props.onClick?.()
    onConfirm?.()
  }
  
  return (
    <BaseButton 
      {...props} 
      onClick={handleClick}
      className="btn-confirm"
    >
      {needsConfirm && !confirmed ? '点击确认' : props.children}
    </BaseButton>
  )
}
```

**接口隔离原则(ISP)实践：**
组件不应该依赖它不需要的props，应该将大型接口拆分为小的、专门的接口。

```jsx
// ❌ 违反ISP：组件被迫接受不需要的props
function BlogPost({ 
  title, content, author, publishDate,
  canEdit, canDelete, canShare,
  onEdit, onDelete, onShare,
  showComments, commentCount,
  tags, category,
  seoTitle, seoDescription 
}) {
  // 组件需要处理太多不相关的props
}

// ✅ 遵循ISP：拆分为专门的组件
function BlogPostContent({ title, content, author, publishDate }) {
  return (
    <article>
      <h1>{title}</h1>
      <div className="meta">
        作者：{author} | 发布时间：{publishDate}
      </div>
      <div className="content">{content}</div>
    </article>
  )
}

function BlogPostActions({ canEdit, canDelete, onEdit, onDelete }) {
  if (!canEdit && !canDelete) return null
  
  return (
    <div className="actions">
      {canEdit && <button onClick={onEdit}>编辑</button>}
      {canDelete && <button onClick={onDelete}>删除</button>}
    </div>
  )
}

function BlogPostComments({ showComments, commentCount, postId }) {
  if (!showComments) return null
  return <CommentSection postId={postId} count={commentCount} />
}

// 主组件组合使用
function BlogPost(props) {
  return (
    <div className="blog-post">
      <BlogPostContent {...pick(props, ['title', 'content', 'author', 'publishDate'])} />
      <BlogPostActions {...pick(props, ['canEdit', 'canDelete', 'onEdit', 'onDelete'])} />
      <BlogPostComments {...pick(props, ['showComments', 'commentCount', 'postId'])} />
    </div>
  )
}
```

**依赖倒置原则(DIP)实践：**
组件应该依赖抽象而不是具体实现，通过依赖注入提供灵活性。

```jsx
// ✅ 通过依赖注入实现可测试和可替换的组件
function UserList({ userService, notificationService }) {
  const [users, setUsers] = useState([])
  const [loading, setLoading] = useState(true)
  
  useEffect(() => {
    // 依赖抽象接口，而不是具体实现
    userService.getUsers()
      .then(setUsers)
      .catch(error => {
        notificationService.showError('获取用户列表失败')
      })
      .finally(() => setLoading(false))
  }, [userService, notificationService])
  
  const handleDeleteUser = async (userId) => {
    try {
      await userService.deleteUser(userId)
      setUsers(users.filter(user => user.id !== userId))
      notificationService.showSuccess('删除成功')
    } catch (error) {
      notificationService.showError('删除失败')
    }
  }
  
  if (loading) return <div>加载中...</div>
  
  return (
    <div className="user-list">
      {users.map(user => (
        <UserCard 
          key={user.id} 
          user={user} 
          onDelete={() => handleDeleteUser(user.id)}
        />
      ))}
    </div>
  )
}

// 在应用层注入具体实现
function App() {
  const userService = new ApiUserService()
  const notificationService = new ToastNotificationService()
  
  return (
    <UserList 
      userService={userService}
      notificationService={notificationService}
    />
  )
}

// 测试时可以注入mock实现
function UserList.test() {
  const mockUserService = {
    getUsers: jest.fn().mockResolvedValue([{ id: 1, name: 'Test User' }]),
    deleteUser: jest.fn().mockResolvedValue(undefined)
  }
  const mockNotificationService = {
    showError: jest.fn(),
    showSuccess: jest.fn()
  }
  
  render(
    <UserList 
      userService={mockUserService}
      notificationService={mockNotificationService}
    />
  )
}
```

**组件设计最佳实践总结：**

1. **职责分离**：每个组件只关注一个核心功能
2. **接口一致**：同类型组件保持相同的props接口
3. **可扩展性**：通过配置、策略模式支持功能扩展
4. **依赖注入**：通过props注入依赖，提高可测试性
5. **组合优于继承**：优先使用组件组合而非类继承
6. **最小接口**：组件只接受必要的props，避免接口膨胀

这些原则的应用帮助我们构建更加健壮、可维护和可扩展的React应用程序。

### 1.3 现代化构建工具 - Vite深度实践

#### Vite相对于Webpack的技术优势分析
**2025年构建工具选型的关键考量：**

Vite作为下一代前端构建工具，在2025年已成为React项目的首选构建方案。相比传统的Webpack，Vite在多个维度都展现出显著优势：

**开发环境性能对比：**
```bash
# 冷启动时间对比
Webpack (Create React App): 15-30秒
Vite: 1-3秒 (提升90%+)

# 热更新(HMR)速度对比  
Webpack: 1-5秒
Vite: 50-200ms (提升95%+)
```

**核心技术原理差异：**
- **Webpack**: 打包所有模块后启动开发服务器
- **Vite**: 基于ES模块的按需编译，原生ESM + esbuild预构建

**实际项目中的体验差异示例：**
```javascript
// Vite开发环境启动配置
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'

export default defineConfig({
  plugins: [
    // 使用SWC替代Babel，编译速度提升10倍
    react({
      // Fast Refresh配置
      fastRefresh: true,
      // SWC配置选项
      swcOptions: {
        jsc: {
          parser: {
            syntax: 'typescript',
            tsx: true,
            decorators: false,
          },
          transform: {
            react: {
              runtime: 'automatic',
              development: true,
              refresh: true,
            },
          },
        },
      },
    }),
  ],
  // 开发服务器配置
  server: {
    port: 3000,
    open: true,
    // 热更新配置
    hmr: {
      overlay: true,
    },
  },
})
```

#### Vite项目初始化最佳实践
**现代化React项目脚手架：**

```bash
# 2025年推荐的项目初始化方式
npm create vite@latest my-react-app -- --template react-ts
cd my-react-app
npm install

# 安装常用开发依赖
npm install -D @types/node @vitejs/plugin-react-swc
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
npm install -D prettier eslint-config-prettier eslint-plugin-prettier
```

**完整的vite.config.ts配置示例：**
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'
import { resolve } from 'path'

export default defineConfig({
  plugins: [react()],
  
  // 路径别名配置
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@utils': resolve(__dirname, 'src/utils'),
      '@hooks': resolve(__dirname, 'src/hooks'),
      '@stores': resolve(__dirname, 'src/stores'),
      '@types': resolve(__dirname, 'src/types'),
    },
  },
  
  // 环境变量前缀
  envPrefix: ['VITE_', 'REACT_APP_'],
  
  // 开发服务器配置
  server: {
    port: 3000,
    host: true, // 支持局域网访问
    open: true,
    cors: true,
    // 代理配置
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
    // 输出目录
    outDir: 'dist',
    // 生成源码映射
    sourcemap: true,
    // 代码分割策略
    rollupOptions: {
      output: {
        manualChunks: {
          // 将React相关库单独打包
          react: ['react', 'react-dom'],
          // 将路由相关库单独打包  
          router: ['react-router-dom'],
          // 将状态管理库单独打包
          store: ['zustand', '@reduxjs/toolkit'],
          // 将UI组件库单独打包
          ui: ['@mui/material', 'antd'],
        },
      },
    },
    // 压缩配置
    minify: 'esbuild',
    // 资源内联阈值
    assetsInlineLimit: 4096,
  },
  
  // 依赖预构建配置
  optimizeDeps: {
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      'zustand',
      '@reduxjs/toolkit',
    ],
    exclude: ['@vite/client', '@vite/env'],
  },
})
```

#### Vite环境变量管理策略
**多环境配置文件管理：**

```bash
# 项目根目录环境变量文件结构
.env                # 所有环境的默认配置
.env.local          # 本地环境配置(被git忽略)
.env.development    # 开发环境配置
.env.production     # 生产环境配置
.env.staging        # 预发环境配置
```

```javascript
// .env.development
VITE_API_BASE_URL=http://localhost:8080/api
VITE_APP_TITLE=React App (Development)
VITE_ENABLE_DEVTOOLS=true
VITE_LOG_LEVEL=debug

// .env.production  
VITE_API_BASE_URL=https://api.production.com
VITE_APP_TITLE=React App
VITE_ENABLE_DEVTOOLS=false
VITE_LOG_LEVEL=error

// 环境变量类型定义
// src/types/env.d.ts
interface ImportMetaEnv {
  readonly VITE_API_BASE_URL: string
  readonly VITE_APP_TITLE: string
  readonly VITE_ENABLE_DEVTOOLS: string
  readonly VITE_LOG_LEVEL: 'debug' | 'info' | 'warn' | 'error'
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

**环境变量使用示例：**
```typescript
// src/config/index.ts
const config = {
  apiBaseUrl: import.meta.env.VITE_API_BASE_URL,
  appTitle: import.meta.env.VITE_APP_TITLE,
  isDev: import.meta.env.DEV,
  isProd: import.meta.env.PROD,
  enableDevtools: import.meta.env.VITE_ENABLE_DEVTOOLS === 'true',
  logLevel: import.meta.env.VITE_LOG_LEVEL,
}

export default config

// 使用示例
import config from '@/config'

const ApiService = {
  baseURL: config.apiBaseUrl,
  timeout: config.isDev ? 10000 : 5000,
}
```

#### Vite插件生态系统应用
**常用插件配置集合：**

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'
import { resolve } from 'path'

// 性能分析插件
import { visualizer } from 'rollup-plugin-visualizer'
// PWA插件
import { VitePWA } from 'vite-plugin-pwa'
// 自动导入插件
import AutoImport from 'unplugin-auto-import/vite'
// ESLint插件
import eslint from 'vite-plugin-eslint'
// 模拟数据插件
import { viteMockServe } from 'vite-plugin-mock'

export default defineConfig({
  plugins: [
    react(),
    
    // ESLint集成
    eslint({
      include: ['src/**/*.ts', 'src/**/*.tsx'],
      exclude: ['node_modules', 'dist'],
      cache: false,
    }),
    
    // 自动导入React和常用Hooks
    AutoImport({
      imports: [
        'react',
        'react-router-dom',
        {
          'zustand': ['create'],
          '@reduxjs/toolkit': ['createSlice', 'configureStore'],
        },
      ],
      dts: true, // 生成类型声明文件
      include: [/\.[tj]sx?$/],
    }),
    
    // PWA配置
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/api\./,
            handler: 'StaleWhileRevalidate',
            options: {
              cacheName: 'api-cache',
              expiration: {
                maxEntries: 100,
                maxAgeSeconds: 24 * 60 * 60, // 24小时
              },
            },
          },
        ],
      },
    }),
    
    // 开发环境Mock服务
    viteMockServe({
      mockPath: 'mock',
      localEnabled: true,
      prodEnabled: false,
      logger: true,
    }),
    
    // 构建分析
    visualizer({
      filename: 'dist/stats.html',
      open: true,
      gzipSize: true,
      brotliSize: true,
    }),
  ],
})
```

#### Vite构建性能优化策略
**代码分割和懒加载优化：**

```typescript
// 路由级别的代码分割
import { lazy, Suspense } from 'react'
import { Routes, Route } from 'react-router-dom'
import LoadingSpinner from '@/components/LoadingSpinner'

// 懒加载页面组件
const Home = lazy(() => import('@/pages/Home'))
const Dashboard = lazy(() => import('@/pages/Dashboard'))
const Settings = lazy(() => import('@/pages/Settings'))

// 组件级别的懒加载
const HeavyChart = lazy(() => import('@/components/HeavyChart'))

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  )
}

// 条件懒加载示例
function DashboardPage() {
  const [showChart, setShowChart] = useState(false)
  
  return (
    <div>
      <h1>Dashboard</h1>
      <button onClick={() => setShowChart(true)}>显示图表</button>
      {showChart && (
        <Suspense fallback={<div>加载图表中...</div>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  )
}
```

**依赖预构建优化：**
```typescript
// vite.config.ts - 依赖优化配置
export default defineConfig({
  optimizeDeps: {
    // 强制预构建的依赖
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      'zustand',
      'axios',
      'lodash-es',
      '@mui/material',
    ],
    // 排除预构建的依赖
    exclude: [
      '@vite/client',
      '@vite/env',
    ],
    // 自定义esbuild配置
    esbuildOptions: {
      define: {
        global: 'globalThis',
      },
    },
  },
  
  // 构建时的依赖外部化
  build: {
    rollupOptions: {
      external: [
        // 对于库模式，外部化React
        // 'react',
        // 'react-dom',
      ],
      output: {
        // 全局变量映射
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM',
        },
        // 自定义chunk分割策略
        manualChunks(id) {
          // node_modules中的依赖单独打包
          if (id.includes('node_modules')) {
            if (id.includes('react')) {
              return 'react-vendor'
            }
            if (id.includes('antd') || id.includes('@mui')) {
              return 'ui-vendor'
            }
            if (id.includes('lodash') || id.includes('ramda')) {
              return 'utils-vendor'
            }
            return 'vendor'
          }
        },
      },
    },
  },
})
```

#### Vite开发体验增强配置
**热模块替换(HMR)优化：**

```typescript
// 自定义HMR处理
// src/main.tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

const root = ReactDOM.createRoot(document.getElementById('root')!)
root.render(<App />)

// HMR处理
if (import.meta.hot) {
  import.meta.hot.accept('./App', (newApp) => {
    if (newApp) {
      root.render(<newApp.default />)
    }
  })
  
  // 状态保持配置
  import.meta.hot.accept(['./stores/userStore'], () => {
    console.log('Store updated, state preserved')
  })
}
```

**开发工具集成：**
```typescript
// vite.config.ts - 开发工具配置
export default defineConfig({
  server: {
    // 开发服务器配置
    host: '0.0.0.0', // 允许外部访问
    port: 3000,
    strictPort: true, // 端口被占用时报错而不是自动换端口
    
    // HTTPS配置
    https: {
      key: './certs/localhost-key.pem',
      cert: './certs/localhost.pem',
    },
    
    // 中间件配置
    middlewareMode: false,
    
    // 开发时的错误覆盖层
    hmr: {
      overlay: true,
      clientPort: 3000,
    },
  },
  
  // CSS预处理器配置
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`,
      },
      less: {
        modifyVars: {
          '@primary-color': '#1890ff',
        },
        javascriptEnabled: true,
      },
    },
    modules: {
      // CSS模块化配置
      localsConvention: 'camelCase',
      generateScopedName: '[name]__[local]___[hash:base64:5]',
    },
  },
})
```

## 2. 状态管理策略

### 状态管理方案选择与2025年最佳实践
**基于项目复杂度和团队规模的选择策略：**

现代React应用的状态管理已经形成了明确的技术趋势。根据2025年最新的技术调研，状态管理方案选择应该基于以下维度：

**技术栈选择矩阵：**
```bash
# 小型项目 (< 10个组件)
技术栈: React Context API + useReducer
适用场景: 原型项目、个人项目、简单的管理后台
团队规模: 1-2人

# 中型项目 (10-50个组件) 
技术栈: Zustand (2025年首选)
适用场景: 中小企业应用、SaaS产品、移动应用
团队规模: 2-8人

# 大型项目 (50+个组件)
技术栈: Redux Toolkit + RTK Query
适用场景: 企业级应用、复杂业务系统、高并发应用
团队规模: 8+人
```

### React 18/19 并发特性在状态管理中的应用

**useDeferredValue和useTransition的实际应用：**

```jsx
import { useState, useDeferredValue, useTransition, useMemo } from 'react'
import { useStore } from './store'

// 大数据列表的性能优化
function DataList() {
  const [query, setQuery] = useState('')
  const [isPending, startTransition] = useTransition()
  
  // 延迟更新搜索查询，避免阻塞UI
  const deferredQuery = useDeferredValue(query)
  const { data, searchData } = useStore()
  
  // 使用并发特性优化搜索
  const filteredData = useMemo(() => {
    return data.filter(item => 
      item.name.toLowerCase().includes(deferredQuery.toLowerCase())
    )
  }, [data, deferredQuery])
  
  const handleSearch = (newQuery) => {
    setQuery(newQuery) // 立即更新输入框
    
    // 将搜索操作标记为非紧急
    startTransition(() => {
      searchData(newQuery)
    })
  }
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="搜索..."
      />
      {isPending && <div className="loading">搜索中...</div>}
      
      <div className="results">
        {filteredData.map(item => (
          <div key={item.id} className="item">
            {item.name}
          </div>
        ))}
      </div>
    </div>
  )
}
```

**React 19的新特性集成：**
```jsx
// React 19的useOptimistic Hook应用
import { useOptimistic } from 'react'
import { useStore } from './store'

function TodoList() {
  const { todos, addTodo, updateTodo } = useStore()
  
  // 乐观更新：立即显示UI变化，后台同步数据
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, { ...newTodo, id: Date.now(), pending: true }]
  )
  
  const handleAddTodo = async (text) => {
    // 乐观更新UI
    addOptimisticTodo({ text, completed: false })
    
    try {
      // 后台真实提交
      await addTodo(text)
    } catch (error) {
      // 如果失败，状态会自动回滚
      console.error('添加失败:', error)
    }
  }
  
  return (
    <div>
      {optimisticTodos.map(todo => (
        <div 
          key={todo.id} 
          className={todo.pending ? 'pending' : ''}
        >
          {todo.text}
        </div>
      ))}
    </div>
  )
}
```

**小型项目 - Context API：**
```jsx
// 创建Context
const AppContext = createContext();

// Provider组件
function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  const value = useMemo(() => ({
    user,
    setUser,
    theme,
    setTheme,
    login: async (credentials) => {
      const userData = await authService.login(credentials);
      setUser(userData);
    },
    logout: () => {
      authService.logout();
      setUser(null);
    }
  }), [user, theme]);
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
}

// 使用Hook
function useApp() {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useApp must be used within AppProvider');
  }
  return context;
}
```

**中型项目 - Zustand企业级实践：**

Zustand在2025年已成为中小型项目的首选状态管理方案。相比Redux，它提供了更简洁的API和更小的包体积，同时保持了强大的功能性。

```jsx
import { create } from 'zustand'
import { subscribeWithSelector, devtools, persist } from 'zustand/middleware'
import { immer } from 'zustand/middleware/immer'

// 复杂状态管理的企业级实践
interface UserState {
  // 状态定义
  user: User | null
  preferences: UserPreferences
  permissions: string[]
  loading: boolean
  error: string | null
  
  // 同步操作
  setUser: (user: User) => void
  updatePreferences: (preferences: Partial<UserPreferences>) => void
  clearError: () => void
  
  // 异步操作
  login: (credentials: LoginCredentials) => Promise<void>
  logout: () => Promise<void>
  fetchUserProfile: () => Promise<void>
  updateProfile: (updates: Partial<User>) => Promise<void>
  
  // 计算属性
  isAuthenticated: () => boolean
  hasPermission: (permission: string) => boolean
  getDisplayName: () => string
}

// 使用多个中间件组合
const useUserStore = create<UserState>()()
  devtools( // 开发工具集成
    persist( // 数据持久化
      immer( // 不可变状态更新
        subscribeWithSelector( // 选择性订阅
          (set, get) => ({
            // 初始状态
            user: null,
            preferences: {
              theme: 'light',
              language: 'zh-CN',
              notifications: true,
            },
            permissions: [],
            loading: false,
            error: null,
            
            // 同步操作
            setUser: (user) => set((state) => {
              state.user = user
              state.permissions = user.permissions || []
              state.error = null
            }),
            
            updatePreferences: (newPreferences) => set((state) => {
              Object.assign(state.preferences, newPreferences)
            }),
            
            clearError: () => set((state) => {
              state.error = null
            }),
            
            // 异步操作实现
            login: async (credentials) => {
              set((state) => {
                state.loading = true
                state.error = null
              })
              
              try {
                const response = await authService.login(credentials)
                const { user, token } = response.data
                
                // 存储token
                localStorage.setItem('token', token)
                
                set((state) => {
                  state.user = user
                  state.permissions = user.permissions || []
                  state.loading = false
                })
                
                // 获取用户详细信息
                await get().fetchUserProfile()
                
              } catch (error) {
                set((state) => {
                  state.loading = false
                  state.error = error.message || '登录失败'
                })
                throw error
              }
            },
            
            logout: async () => {
              try {
                await authService.logout()
              } catch (error) {
                console.error('退出登录失败:', error)
              } finally {
                localStorage.removeItem('token')
                set((state) => {
                  state.user = null
                  state.permissions = []
                  state.error = null
                })
              }
            },
            
            fetchUserProfile: async () => {
              const state = get()
              if (!state.user?.id) return
              
              try {
                const profile = await userService.getProfile(state.user.id)
                set((state) => {
                  state.user = { ...state.user, ...profile }
                })
              } catch (error) {
                set((state) => {
                  state.error = '获取用户信息失败'
                })
              }
            },
            
            updateProfile: async (updates) => {
              const state = get()
              if (!state.user?.id) return
              
              set((state) => {
                state.loading = true
                state.error = null
              })
              
              try {
                const updatedUser = await userService.updateProfile(
                  state.user.id, 
                  updates
                )
                
                set((state) => {
                  state.user = updatedUser
                  state.loading = false
                })
                
              } catch (error) {
                set((state) => {
                  state.loading = false
                  state.error = '更新用户信息失败'
                })
                throw error
              }
            },
            
            // 计算属性
            isAuthenticated: () => {
              const state = get()
              return !!state.user && !!localStorage.getItem('token')
            },
            
            hasPermission: (permission) => {
              const state = get()
              return state.permissions.includes(permission)
            },
            
            getDisplayName: () => {
              const state = get()
              return state.user?.displayName || 
                     state.user?.email || 
                     '未知用户'
            },
          })
        )
      ),
      {
        name: 'user-store', // 存储键名
        // 部分状态持久化
        partialize: (state) => ({
          user: state.user,
          preferences: state.preferences,
        }),
        // 版本管理
        version: 1,
        migrate: (persistedState, version) => {
          // 数据迁移逻辑
          return persistedState
        },
      }
    )
  )
)

// Store的使用示例
function UserProfile() {
  const { 
    user, 
    loading, 
    error, 
    updateProfile, 
    getDisplayName,
    clearError 
  } = useUserStore()
  
  const [formData, setFormData] = useState({
    displayName: user?.displayName || '',
    email: user?.email || '',
  })
  
  const handleSubmit = async (e) => {
    e.preventDefault()
    try {
      await updateProfile(formData)
      toast.success('更新成功')
    } catch (error) {
      // 错误由store处理
    }
  }
  
  useEffect(() => {
    if (error) {
      toast.error(error)
      clearError()
    }
  }, [error, clearError])
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>用户资料 - {getDisplayName()}</h2>
      
      {error && (
        <div className="error">{error}</div>
      )}
      
      <input 
        value={formData.displayName}
        onChange={(e) => setFormData(prev => ({
          ...prev, 
          displayName: e.target.value
        }))}
        placeholder="显示名称"
        disabled={loading}
      />
      
      <input 
        value={formData.email}
        onChange={(e) => setFormData(prev => ({
          ...prev, 
          email: e.target.value
        }))}
        placeholder="邮箱"
        disabled={loading}
      />
      
      <button type="submit" disabled={loading}>
        {loading ? '更新中...' : '更新'}
      </button>
    </form>
  )
}

// 订阅机制示例
function useUserSubscription() {
  useEffect(() => {
    // 监听用户状态变化
    const unsubscribe = useUserStore.subscribe(
      (state) => state.user,
      (user, prevUser) => {
        if (user && !prevUser) {
          // 用户登录
          analytics.track('用户登录', { userId: user.id })
        } else if (!user && prevUser) {
          // 用户退出
          analytics.track('用户退出', { userId: prevUser.id })
        }
      }
    )
    
    return unsubscribe
  }, [])
}

// 多个store的组合使用
const useAuthenticatedUser = () => {
  const user = useUserStore((state) => state.user)
  const isAuthenticated = useUserStore((state) => state.isAuthenticated())
  const hasPermission = useUserStore((state) => state.hasPermission)
  
  return { user, isAuthenticated, hasPermission }
}
```

**Zustand性能优化策略：**
```jsx
// 选择器优化策略
const useUserBasicInfo = () => {
  // 只订阅需要的字段，减少重渲染
  return useUserStore(useCallback(
    (state) => ({
      id: state.user?.id,
      name: state.user?.displayName || state.user?.email,
      avatar: state.user?.avatar,
    }),
    []
  ))
}

// 浅比较优化
const useUserPermissions = () => {
  return useUserStore(
    (state) => state.permissions,
    (a, b) => JSON.stringify(a) === JSON.stringify(b) // 自定义比较
  )
}

// 状态分片策略
const createSlice = (name, initialState, actions) => {
  return create((set, get) => ({
    ...initialState,
    ...Object.keys(actions).reduce((acc, key) => {
      acc[key] = (...args) => {
        const result = actions[key](set, get, ...args)
        // 记录状态变化
        if (process.env.NODE_ENV === 'development') {
          console.log(`[${name}] ${key}`, ...args)
        }
        return result
      }
      return acc
    }, {}),
  }))
}

// 使用分片
const useThemeStore = createSlice('theme', 
  {
    theme: 'light',
    colors: {},
  },
  {
    setTheme: (set, get, theme) => {
      set({ theme })
      document.documentElement.setAttribute('data-theme', theme)
    },
    toggleTheme: (set, get) => {
      const newTheme = get().theme === 'light' ? 'dark' : 'light'
      get().setTheme(newTheme)
    },
  }
)
```

**大型项目 - Redux Toolkit：**
```jsx
// store/slices/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await userApi.getUser(userId);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response.data);
    }
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    loading: false,
    error: null
  },
  reducers: {
    clearUser: (state) => {
      state.data = null;
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

export const { clearUser } = userSlice.actions;
export default userSlice.reducer;
```

## 3. 路由管理实践

### 路由权限控制
**基于角色的路由守卫：**
```jsx
import { Routes, Route, Navigate } from 'react-router-dom';
import { useAuth } from './hooks/useAuth';

// 路由守卫组件
function ProtectedRoute({ children, requiredRoles = [] }) {
  const { user, loading } = useAuth();
  
  if (loading) {
    return <div>Loading...</div>;
  }
  
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  
  if (requiredRoles.length > 0 && 
      !requiredRoles.some(role => user.roles.includes(role))) {
    return <Navigate to="/unauthorized" replace />;
  }
  
  return children;
}

// 路由配置
function AppRoutes() {
  return (
    <Routes>
      <Route path="/login" element={<LoginPage />} />
      <Route path="/unauthorized" element={<UnauthorizedPage />} />
      
      <Route path="/admin/*" element={
        <ProtectedRoute requiredRoles={['admin']}>
          <AdminLayout />
        </ProtectedRoute>
      } />
      
      <Route path="/user/*" element={
        <ProtectedRoute requiredRoles={['user', 'admin']}>
          <UserLayout />
        </ProtectedRoute>
      } />
      
      <Route path="/public" element={<PublicPage />} />
      
      <Route path="/" element={
        <ProtectedRoute>
          <Dashboard />
        </ProtectedRoute>
      } />
    </Routes>
  );
}
```

### 动态路由加载
**基于权限的动态路由：**
```jsx
function useRoutes() {
  const { user } = useAuth();
  
  const routes = useMemo(() => {
    const baseRoutes = [
      { path: '/', element: <Home /> },
      { path: '/profile', element: <Profile /> }
    ];
    
    if (user?.roles.includes('admin')) {
      baseRoutes.push(
        { path: '/admin', element: <AdminDashboard /> },
        { path: '/admin/users', element: <UserManagement /> }
      );
    }
    
    if (user?.roles.includes('editor')) {
      baseRoutes.push(
        { path: '/editor', element: <EditorDashboard /> },
        { path: '/editor/posts', element: <PostManagement /> }
      );
    }
    
    return baseRoutes;
  }, [user]);
  
  return routes;
}

function App() {
  const routes = useRoutes();
  
  return (
    <Routes>
      {routes.map(route => (
        <Route key={route.path} {...route} />
      ))}
    </Routes>
  );
}
```

## 4. 测试策略与实践

### 7.1 Vitest测试框架集成

**为什么选择Vitest而非Jest：**

2025年，Vitest已成为Vite项目的默认测试解决方案。相比Jest，它提供了更快的执行速度和更好的TypeScript支持。

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react-swc'
import { resolve } from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    // 测试环境配置
    environment: 'jsdom',
    
    // 全局设置文件
    setupFiles: ['./src/tests/setup.ts'],
    
    // 测试文件匹配
    include: ['src/**/*.{test,spec}.{js,ts,jsx,tsx}'],
    exclude: ['node_modules', 'dist', 'build'],
    
    // 覆盖率配置
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      include: ['src/**/*.{js,ts,jsx,tsx}'],
      exclude: [
        'src/**/*.d.ts',
        'src/**/*.stories.{js,ts,jsx,tsx}',
        'src/tests/**/*',
        'src/main.tsx',
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
    
    // 并发测试
    threads: true,
    maxThreads: 4,
    
    // 测试超时
    testTimeout: 10000,
    
    // Mock配置
    mockReset: true,
    clearMocks: true,
    restoreMocks: true,
  },
  
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@tests': resolve(__dirname, 'src/tests'),
    },
  },
})
```

**测试环境设置：**

```typescript
// src/tests/setup.ts
import { expect, afterEach, vi } from 'vitest'
import { cleanup } from '@testing-library/react'
import * as matchers from '@testing-library/jest-dom/matchers'

// 扩展expect匹配器
expect.extend(matchers)

// 清理DOM
afterEach(() => {
  cleanup()
})

// 全局Mock配置
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(), // deprecated
    removeListener: vi.fn(), // deprecated
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
})

// IntersectionObserver Mock
global.IntersectionObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  disconnect: vi.fn(),
  unobserve: vi.fn(),
}))

// ResizeObserver Mock
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  disconnect: vi.fn(),
  unobserve: vi.fn(),
}))

// 环境变量设置
vi.mock('import.meta.env', () => ({
  VITE_API_BASE_URL: 'http://localhost:3001/api',
  VITE_APP_TITLE: 'Test App',
  DEV: true,
  PROD: false,
}))
```

**组件测试最佳实践：**

```tsx
// src/components/UserProfile.test.tsx
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { UserProfile } from './UserProfile'
import { useUserStore } from '@/stores/userStore'

// Mock store
vi.mock('@/stores/userStore')
const mockUseUserStore = vi.mocked(useUserStore)

// 渲染帮助函数
const renderUserProfile = (props = {}) => {
  const defaultProps = {
    userId: '123',
    onEdit: vi.fn(),
    ...props,
  }
  
  return {
    ...render(<UserProfile {...defaultProps} />),
    props: defaultProps,
  }
}

describe('UserProfile', () => {
  beforeEach(() => {
    // 重置所有mock
    vi.clearAllMocks()
    
    // 设置默认store状态
    mockUseUserStore.mockReturnValue({
      user: {
        id: '123',
        name: 'John Doe',
        email: 'john@example.com',
        avatar: 'https://example.com/avatar.jpg',
      },
      loading: false,
      error: null,
      updateProfile: vi.fn(),
      clearError: vi.fn(),
    })
  })
  
  it('应该正确渲染用户信息', () => {
    renderUserProfile()
    
    expect(screen.getByText('John Doe')).toBeInTheDocument()
    expect(screen.getByText('john@example.com')).toBeInTheDocument()
    expect(screen.getByRole('img', { name: /john doe/i })).toHaveAttribute(
      'src', 
      'https://example.com/avatar.jpg'
    )
  })
  
  it('应该在加载时显示加载状态', () => {
    mockUseUserStore.mockReturnValue({
      ...mockUseUserStore(),
      loading: true,
    })
    
    renderUserProfile()
    
    expect(screen.getByText('加载中...')).toBeInTheDocument()
    expect(screen.queryByText('John Doe')).not.toBeInTheDocument()
  })
  
  it('应该在错误时显示错误信息', () => {
    const errorMessage = '加载用户信息失败'
    
    mockUseUserStore.mockReturnValue({
      ...mockUseUserStore(),
      error: errorMessage,
    })
    
    renderUserProfile()
    
    expect(screen.getByText(errorMessage)).toBeInTheDocument()
    expect(screen.getByRole('button', { name: /重试/i })).toBeInTheDocument()
  })
  
  it('应该处理编辑操作', async () => {
    const user = userEvent.setup()
    const { props } = renderUserProfile()
    
    const editButton = screen.getByRole('button', { name: /编辑/i })
    await user.click(editButton)
    
    expect(props.onEdit).toHaveBeenCalledWith('123')
  })
  
  it('应该处理表单提交', async () => {
    const user = userEvent.setup()
    const mockUpdateProfile = vi.fn().mockResolvedValue(undefined)
    
    mockUseUserStore.mockReturnValue({
      ...mockUseUserStore(),
      updateProfile: mockUpdateProfile,
    })
    
    renderUserProfile()
    
    // 点击编辑按钮进入编辑模式
    await user.click(screen.getByRole('button', { name: /编辑/i }))
    
    // 修改输入框
    const nameInput = screen.getByDisplayValue('John Doe')
    await user.clear(nameInput)
    await user.type(nameInput, 'Jane Doe')
    
    // 提交表单
    await user.click(screen.getByRole('button', { name: /保存/i }))
    
    await waitFor(() => {
      expect(mockUpdateProfile).toHaveBeenCalledWith({
        name: 'Jane Doe',
      })
    })
  })
})
```

**Hook测试示例：**

```tsx
// src/hooks/useDebounce.test.ts
import { describe, it, expect, vi } from 'vitest'
import { renderHook, act } from '@testing-library/react'
import { useDebounce } from './useDebounce'

describe('useDebounce', () => {
  it('应该在指定延迟后返回值', async () => {
    vi.useFakeTimers()
    
    const { result, rerender } = renderHook(
      ({ value, delay }) => useDebounce(value, delay),
      {
        initialProps: { value: 'initial', delay: 500 },
      }
    )
    
    // 初始值应该立即返回
    expect(result.current).toBe('initial')
    
    // 更新值
    rerender({ value: 'updated', delay: 500 })
    
    // 在延迟之前不应该更新
    expect(result.current).toBe('initial')
    
    // 快进时间
    act(() => {
      vi.advanceTimersByTime(500)
    })
    
    // 现在应该返回新值
    expect(result.current).toBe('updated')
    
    vi.useRealTimers()
  })
  
  it('应该在值快速变化时防抖动', () => {
    vi.useFakeTimers()
    
    const { result, rerender } = renderHook(
      ({ value, delay }) => useDebounce(value, delay),
      {
        initialProps: { value: 'initial', delay: 500 },
      }
    )
    
    // 快速连续更新
    rerender({ value: 'update1', delay: 500 })
    act(() => {
      vi.advanceTimersByTime(300)
    })
    
    rerender({ value: 'update2', delay: 500 })
    act(() => {
      vi.advanceTimersByTime(300)
    })
    
    rerender({ value: 'final', delay: 500 })
    
    // 在延迟结束前不应该更新
    expect(result.current).toBe('initial')
    
    // 完成最后一次延迟
    act(() => {
      vi.advanceTimersByTime(500)
    })
    
    // 应该返回最后一次更新的值
    expect(result.current).toBe('final')
    
    vi.useRealTimers()
  })
})
```

### 7.2 现代化E2E测试策略

**Playwright集成配置：**

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['junit', { outputFile: 'results.xml' }],
    ['json', { outputFile: 'test-results.json' }],
  ],
  
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    video: 'retain-on-failure',
    screenshot: 'only-on-failure',
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
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
  
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
})
```

**E2E测试用例：**

```typescript
// e2e/user-flow.spec.ts
import { test, expect } from '@playwright/test'

test.describe('用户流程测试', () => {
  test.beforeEach(async ({ page }) => {
    // 设置Mock数据
    await page.route('/api/user/login', async route => {
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({
          success: true,
          data: {
            user: {
              id: '123',
              name: 'Test User',
              email: 'test@example.com',
            },
            token: 'mock-token',
          },
        }),
      })
    })
    
    await page.goto('/')
  })
  
  test('应该能够成功登录', async ({ page }) => {
    // 点击登录按钮
    await page.getByRole('button', { name: '登录' }).click()
    
    // 填写登录表单
    await page.getByLabel('邮箱').fill('test@example.com')
    await page.getByLabel('密码').fill('password123')
    
    // 提交表单
    await page.getByRole('button', { name: '登录' }).click()
    
    // 验证登录成功
    await expect(page.getByText('欢迎，Test User')).toBeVisible()
    await expect(page).toHaveURL('/dashboard')
  })
  
  test('应该处理登录错误', async ({ page }) => {
    // Mock登录失败
    await page.route('/api/user/login', async route => {
      await route.fulfill({
        status: 401,
        contentType: 'application/json',
        body: JSON.stringify({
          success: false,
          message: '邮箱或密码错误',
        }),
      })
    })
    
    await page.getByRole('button', { name: '登录' }).click()
    await page.getByLabel('邮箱').fill('test@example.com')
    await page.getByLabel('密码').fill('wrong-password')
    await page.getByRole('button', { name: '登录' }).click()
    
    // 验证错误信息
    await expect(page.getByText('邮箱或密码错误')).toBeVisible()
    await expect(page).toHaveURL('/login')
  })
  
  test('应该支持响应式设计', async ({ page }) => {
    // 测试桌面端
    await page.setViewportSize({ width: 1200, height: 800 })
    await expect(page.getByTestId('desktop-nav')).toBeVisible()
    await expect(page.getByTestId('mobile-nav')).not.toBeVisible()
    
    // 测试移动端
    await page.setViewportSize({ width: 375, height: 667 })
    await expect(page.getByTestId('mobile-nav')).toBeVisible()
    await expect(page.getByTestId('desktop-nav')).not.toBeVisible()
  })
  
  test('应该正确处理数据加载', async ({ page }) => {
    // Mock慢速API响应
    await page.route('/api/dashboard/data', async route => {
      await new Promise(resolve => setTimeout(resolve, 2000))
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({
          data: [
            { id: 1, name: '项目1', status: 'active' },
            { id: 2, name: '项目2', status: 'pending' },
          ],
        }),
      })
    })
    
    await page.goto('/dashboard')
    
    // 验证加载状态
    await expect(page.getByTestId('loading-spinner')).toBeVisible()
    
    // 验证数据加载完成
    await expect(page.getByText('项目1')).toBeVisible({ timeout: 5000 })
    await expect(page.getByTestId('loading-spinner')).not.toBeVisible()
  })
})
```

**单元测试 - 工具函数：**
```jsx
// utils/format.test.js
import { formatDate, formatCurrency } from './format';

describe('format utils', () => {
  test('formatDate should format date correctly', () => {
    expect(formatDate('2023-01-15')).toBe('Jan 15, 2023');
  });
  
  test('formatCurrency should format number as currency', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
  });
});
```

**组件测试 - React Testing Library：**
```jsx
// components/Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';

describe('Button', () => {
  test('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  test('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  test('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByText('Click me')).toBeDisabled();
  });
});
```

**集成测试 - 用户流程：**
```jsx
// tests/login.test.js
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import LoginForm from '../components/LoginForm';
import { AuthProvider } from '../contexts/AuthContext';

describe('Login flow', () => {
  test('successful login redirects to dashboard', async () => {
    render(
      <AuthProvider>
        <LoginForm />
      </AuthProvider>
    );
    
    // 填写表单
    await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
    await userEvent.type(screen.getByLabelText(/password/i), 'password123');
    
    // 提交表单
    await userEvent.click(screen.getByRole('button', { name: /login/i }));
    
    // 验证重定向
    await waitFor(() => {
      expect(window.location.pathname).toBe('/dashboard');
    });
  });
});
```

## 8. 现代化部署与CI/CD

### 8.1 基于Vite的Docker容器化部署

**优化的Dockerfile配置：**
```dockerfile
# 多阶段构建 - Node.js 20 Alpine
FROM node:20-alpine AS base
WORKDIR /app
RUN apk add --no-cache libc6-compat
RUN corepack enable

# 依赖安装阶段
FROM base AS deps
COPY package.json pnpm-lock.yaml ./
RUN --mount=type=cache,id=pnpm,target=/root/.local/share/pnpm/store \
    pnpm install --frozen-lockfile

# 构建阶段
FROM base AS builder
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ARG VITE_API_BASE_URL
ARG VITE_APP_VERSION
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL
ENV VITE_APP_VERSION=$VITE_APP_VERSION
RUN pnpm build

# 生产阶段
FROM nginx:alpine AS production
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**优化的nginx配置：**
```nginx
events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  
  # Gzip压缩
  gzip on;
  gzip_vary on;
  gzip_min_length 1024;
  gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
  
  # Brotli压缩(如果支持)
  # brotli on;
  # brotli_comp_level 6;
  
  server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;
    
    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
    
    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
      expires 1y;
      add_header Cache-Control "public, immutable";
    }
    
    # SPA路由处理
    location / {
      try_files $uri $uri/ /index.html;
    }
    
    # API代理(可选)
    location /api/ {
      proxy_pass http://backend:8080/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
    }
  }
}
```

### 8.2 GitHub Actions CI/CD流水线

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  PNPM_VERSION: '8'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
          
      - name: Get pnpm store directory
        shell: bash
        run: echo "STORE_PATH=$(pnpm store path --silent)" >> $GITHUB_ENV
        
      - name: Setup pnpm cache
        uses: actions/cache@v3
        with:
          path: ${{ env.STORE_PATH }}
          key: ${{ runner.os }}-pnpm-store-${{ hashFiles('**/pnpm-lock.yaml') }}
          
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
        
      - name: Run tests
        run: pnpm test
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        
  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js and pnpm
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
          
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
        
      - name: Build application
        run: pnpm build
        env:
          VITE_API_BASE_URL: ${{ secrets.VITE_API_BASE_URL }}
          VITE_APP_VERSION: ${{ github.sha }}
          
      - name: Build Docker image
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker tag myapp:${{ github.sha }} myapp:latest
          
      - name: Deploy to production
        run: |
          echo "部署到生产环境"
          # 实际部署命令
```

### 8.3 性能监控与错误追踪

**Web Vitals监控集成：**
```jsx
// src/utils/performance.ts
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals'

const vitalsUrl = 'https://vitals.vercel-analytics.com/v1/vitals'

function getConnectionSpeed() {
  return 'connection' in navigator &&
    navigator.connection &&
    'effectiveType' in navigator.connection
    ? navigator.connection.effectiveType
    : ''
}

function sendToAnalytics(metric, options) {
  const body = {
    dsn: process.env.VITE_ANALYTICS_ID,
    id: metric.id,
    page: window.location.pathname,
    href: window.location.href,
    event_name: metric.name,
    value: metric.value.toString(),
    speed: getConnectionSpeed(),
    ...options,
  }
  
  if (navigator.sendBeacon) {
    navigator.sendBeacon(vitalsUrl, JSON.stringify(body))
  } else {
    fetch(vitalsUrl, {
      body: JSON.stringify(body),
      method: 'POST',
      keepalive: true,
    })
  }
}

// 初始化性能监控
export function initPerformanceMonitoring() {
  try {
    getCLS(sendToAnalytics)
    getFID(sendToAnalytics)
    getFCP(sendToAnalytics)
    getLCP(sendToAnalytics)
    getTTFB(sendToAnalytics)
  } catch (err) {
    console.error('Performance monitoring failed:', err)
  }
}
```

## 总结

2025年的React工程实践已经发生了显著变化：

1. **构建工具革命**: Vite替代Webpack成为主流，带来10倍+性能提升
2. **状态管理进化**: Zustand成为中小型项目首选，更轻量更灵活
3. **React新特性**: 并发渲染、Suspense、useTransition成为标准实践
4. **测试现代化**: Vitest+Playwright替代Jest+Cypress
5. **部署优化**: 基于Vite的容器化部署，更快的构建速度

高级开发者需要掌握这些现代化技术栈，才能在竞争激烈的市场中保持竞争优势。

## 6. React 18/19 高级特性与性能优化

### 6.1 并发渲染特性实践

**Suspense边界的企业级应用：**

现代React应用中，Suspense不仅用于代码分割，更是构建响应式用户体验的核心工具。React 18/19的并发特性让Suspense变得更加强大和实用。

```jsx
import { Suspense, lazy, startTransition } from 'react'
import { ErrorBoundary } from 'react-error-boundary'

// 智能加载组件
const DataDashboard = lazy(() => {
  return Promise.all([
    import('./DataDashboard'),
    // 预加载相关资源
    import('./charts/LineChart'),
    import('./charts/BarChart'),
    // 模拟最小加载时间，避免闪烁
    new Promise(resolve => setTimeout(resolve, 500))
  ]).then(([module]) => module)
})

const ReportsPage = lazy(() => import('./ReportsPage'))
const SettingsPage = lazy(() => import('./SettingsPage'))

// 多层次Suspense架构
function App() {
  const [currentTab, setCurrentTab] = useState('dashboard')
  const [isPending, startTransition] = useTransition()
  
  const handleTabChange = (tab) => {
    // 使用transition避免阻塞UI
    startTransition(() => {
      setCurrentTab(tab)
    })
  }
  
  return (
    <div className="app">
      <nav>
        <button 
          onClick={() => handleTabChange('dashboard')}
          className={isPending ? 'loading' : ''}
        >
          仪表板
        </button>
        <button onClick={() => handleTabChange('reports')}>
          报告
        </button>
        <button onClick={() => handleTabChange('settings')}>
          设置
        </button>
      </nav>
      
      {/* 全局错误边界 */}
      <ErrorBoundary
        fallback={<ErrorFallback />}
        onError={(error, errorInfo) => {
          // 错误上报
          errorReporting.captureException(error, {
            extra: errorInfo
          })
        }}
      >
        {/* 页面级Suspense */}
        <Suspense fallback={<PageSkeleton />}>
          {currentTab === 'dashboard' && (
            // 组件级Suspense
            <Suspense fallback={<DashboardSkeleton />}>
              <DataDashboard />
            </Suspense>
          )}
          {currentTab === 'reports' && <ReportsPage />}
          {currentTab === 'settings' && <SettingsPage />}
        </Suspense>
      </ErrorBoundary>
    </div>
  )
}

// 智能骨架屏组件
function PageSkeleton() {
  return (
    <div className="page-skeleton">
      <div className="skeleton-header" />
      <div className="skeleton-content">
        <div className="skeleton-sidebar" />
        <div className="skeleton-main" />
      </div>
    </div>
  )
}

// 特定组件的骨架屏
function DashboardSkeleton() {
  return (
    <div className="dashboard-skeleton">
      {Array.from({ length: 6 }).map((_, i) => (
        <div key={i} className="chart-skeleton" />
      ))}
    </div>
  )
}
```

**React 19的新特性集成：**

```jsx
// 使用React 19的useActionState
import { useActionState } from 'react'

function ContactForm() {
  const [state, formAction] = useActionState(async (prevState, formData) => {
    try {
      await submitContact({
        name: formData.get('name'),
        email: formData.get('email'),
        message: formData.get('message')
      })
      return { success: true, message: '提交成功！' }
    } catch (error) {
      return { success: false, message: error.message }
    }
  }, { success: null, message: '' })
  
  return (
    <form action={formAction}>
      {state.message && (
        <div className={`alert ${state.success ? 'success' : 'error'}`}>
          {state.message}
        </div>
      )}
      
      <input name="name" placeholder="姓名" required />
      <input name="email" type="email" placeholder="邮箱" required />
      <textarea name="message" placeholder="消息" required />
      
      <button type="submit">
        提交
      </button>
    </form>
  )
}

// React 19的use Hook用法
import { use, Suspense } from 'react'

function UserProfile({ userPromise }) {
  // use Hook可以在组件中消费Promise
  const user = use(userPromise)
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  )
}

function UserPage({ userId }) {
  // 创建用户数据Promise
  const userPromise = useMemo(() => 
    fetchUser(userId), [userId]
  )
  
  return (
    <Suspense fallback={<div>加载用户信息...</div>}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  )
}
```

### 6.2 现代化性能优化策略

**基于Vite的构建时优化：**

```typescript
// vite.config.ts - 高级性能优化配置
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'
import { splitVendorChunkPlugin } from 'vite'
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    react({
      // 启用React DevTools profiling
      plugins: [
        ['@swc/plugin-react-remove-properties', {
          properties: ['data-testid']
        }]
      ]
    }),
    
    // 智能vendor分包
    splitVendorChunkPlugin(),
    
    // 构建分析
    visualizer({
      filename: 'dist/stats.html',
      open: true,
      gzipSize: true,
      brotliSize: true,
    })
  ],
  
  build: {
    // 代码分割策略优化
    rollupOptions: {
      output: {
        manualChunks: {
          // React核心
          'react-core': ['react', 'react-dom'],
          // 路由相关
          'router': ['react-router-dom', 'history'],
          // 状态管理
          'state': ['zustand', '@reduxjs/toolkit'],
          // UI组件库
          'ui-vendor': ['@mui/material', '@mui/icons-material'],
          // 工具库
          'utils': ['lodash-es', 'date-fns', 'ramda'],
          // 图表库
          'charts': ['recharts', 'chart.js', 'echarts'],
        },
        // 优化chunk命名
        chunkFileNames: (chunkInfo) => {
          const facadeModuleId = chunkInfo.facadeModuleId
          if (facadeModuleId) {
            const moduleName = facadeModuleId.split('/').pop()?.replace(/\.[^.]*$/, '')
            return `js/${moduleName}-[hash].js`
          }
          return 'js/[name]-[hash].js'
        },
        assetFileNames: (assetInfo) => {
          const extType = assetInfo.name?.split('.').pop()
          if (/png|jpe?g|svg|gif|tiff|bmp|ico/i.test(extType || '')) {
            return `images/[name]-[hash].[ext]`
          }
          if (/woff2?|eot|ttf|otf/i.test(extType || '')) {
            return `fonts/[name]-[hash].[ext]`
          }
          return `assets/[name]-[hash].[ext]`
        }
      },
    },
    
    // 压缩优化
    minify: 'esbuild',
    target: 'es2020',
    
    // CSS代码分割
    cssCodeSplit: true,
    
    // 资源内联阈值
    assetsInlineLimit: 4096,
    
    // 开启压缩
    reportCompressedSize: true,
  },
  
  // 依赖预构建优化
  optimizeDeps: {
    include: [
      'react/jsx-runtime',
      'react-dom/client',
      'react-router-dom',
      'zustand',
      '@mui/material/styles',
      '@emotion/react',
      '@emotion/styled',
    ],
    esbuildOptions: {
      target: 'es2020',
      define: {
        global: 'globalThis'
      },
    },
  },
})
```

**运行时性能优化策略：**

```jsx
import { memo, useMemo, useCallback, startTransition } from 'react'
import { useVirtualizer } from '@tanstack/react-virtual'

// 虚拟滚动优化大列表
function VirtualizedList({ items, height = 400 }) {
  const parentRef = useRef()
  
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
    overscan: 5, // 预渲染额外行数
  })
  
  return (
    <div
      ref={parentRef}
      style={{ height, overflow: 'auto' }}
    >
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          width: '100%',
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualItem.size}px`,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            <ListItem item={items[virtualItem.index]} />
          </div>
        ))}
      </div>
    </div>
  )
}

// 智能memo组件
const ListItem = memo(({ item }) => {
  return (
    <div className="list-item">
      <h3>{item.title}</h3>
      <p>{item.description}</p>
    </div>
  )
}, (prevProps, nextProps) => {
  // 自定义比较逻辑
  return prevProps.item.id === nextProps.item.id &&
         prevProps.item.title === nextProps.item.title
})

// 复杂计算的优化
function DataVisualization({ rawData, filters, sortBy }) {
  // 使用useMemo缓存昂贵的计算
  const processedData = useMemo(() => {
    console.log('Processing data...') // 只在依赖变化时执行
    
    return rawData
      .filter(item => {
        return Object.entries(filters).every(([key, value]) => {
          if (!value) return true
          return item[key]?.toString().toLowerCase().includes(value.toLowerCase())
        })
      })
      .sort((a, b) => {
        if (sortBy.direction === 'asc') {
          return a[sortBy.field] > b[sortBy.field] ? 1 : -1
        }
        return a[sortBy.field] < b[sortBy.field] ? 1 : -1
      })
  }, [rawData, filters, sortBy])
  
  // 稳定的事件处理器
  const handleItemClick = useCallback((item) => {
    startTransition(() => {
      // 非紧急更新
      onItemSelect(item)
    })
  }, [onItemSelect])
  
  return (
    <VirtualizedList 
      items={processedData}
      onItemClick={handleItemClick}
    />
  )
}
```

**Core Web Vitals优化实践：**

```jsx
// 性能监控组件
function PerformanceMonitor() {
  useEffect(() => {
    // 监控Core Web Vitals
    import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
      getCLS(console.log)
      getFID(console.log)
      getFCP(console.log)
      getLCP(console.log)
      getTTFB(console.log)
    })
    
    // 监控长任务
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((entryList) => {
        const entries = entryList.getEntries()
        entries.forEach((entry) => {
          if (entry.duration > 50) {
            console.warn('Long task detected:', entry)
            // 上报长任务
            analytics.track('long_task', {
              duration: entry.duration,
              startTime: entry.startTime
            })
          }
        })
      })
      
      observer.observe({ entryTypes: ['longtask'] })
      
      return () => observer.disconnect()
    }
  }, [])
  
  return null
}

// 图片懒加载优化
function OptimizedImage({ src, alt, className, ...props }) {
  const [imageSrc, setImageSrc] = useState()
  const [imageRef, inView] = useInView({
    triggerOnce: true,
    threshold: 0.1,
  })
  
  useEffect(() => {
    if (inView && src) {
      // 预加载高优先级图片
      const img = new Image()
      img.onload = () => setImageSrc(src)
      img.src = src
    }
  }, [inView, src])
  
  return (
    <div ref={imageRef} className={className}>
      {imageSrc ? (
        <img 
          src={imageSrc} 
          alt={alt} 
          loading="lazy"
          decoding="async"
          {...props}
        />
      ) : (
        <div className="image-placeholder" />
      )}
    </div>
  )
}

// 字体加载优化
function FontOptimizer() {
  useEffect(() => {
    // 预加载关键字体
    const link = document.createElement('link')
    link.rel = 'preload'
    link.href = '/fonts/inter-var.woff2'
    link.as = 'font'
    link.type = 'font/woff2'
    link.crossOrigin = 'anonymous'
    document.head.appendChild(link)
    
    // 字体加载完成后移除fallback样式
    document.fonts.ready.then(() => {
      document.documentElement.classList.add('fonts-loaded')
    })
  }, [])
  
  return null
}
```

## 7. 现代化测试策略

### 错误边界与监控
**完整的错误处理体系：**
```jsx
// components/ErrorBoundary.jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { 
      hasError: false, 
      error: null, 
      errorInfo: null 
    };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    this.setState({
      error: error,
      errorInfo: errorInfo
    });
    
    // 上报错误到监控系统
    errorMonitoringService.captureException(error, {
      extra: {
        componentStack: errorInfo.componentStack
      }
    });
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong.</h2>
          <details>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo.componentStack}
          </details>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// 使用示例
function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<LoadingSpinner />}>
        <Router>
          <Routes>
            <Route path="/" element={<Home />} />
            {/* 其他路由 */}
          </Routes>
        </Router>
      </Suspense>
    </ErrorBoundary>
  );
}
```

### 性能监控集成
**React性能监控：**
```jsx
// hooks/usePerformance.js
function usePerformance(componentName) {
  const mountTime = useRef(performance.now());
  const renderCount = useRef(0);
  
  useEffect(() => {
    const measure = () => {
      const now = performance.now();
      const renderTime = now - mountTime.current;
      
      renderCount.current++;
      
      // 发送性能数据
      performanceMonitor.trackRender({
        component: componentName,
        renderTime: renderTime,
        renderCount: renderCount.current,
        timestamp: now
      });
      
      mountTime.current = now;
    };
    
    measure();
  });
}

// 在组件中使用
function ExpensiveComponent() {
  usePerformance('ExpensiveComponent');
  
  // 组件逻辑
  return <div>...</div>;
}
```

## 总结
React工程实践面试题主要考察项目架构设计、状态管理策略、路由管理、测试体系、部署流程和监控系统等方面的实践经验。高级开发者需要具备完整的工程化思维，能够设计可维护、可扩展的React应用架构。
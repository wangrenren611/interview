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

### 组件设计原则
**单一职责原则实现：**
```jsx
// 不好的设计：组件承担过多职责
function UserProfile({ user, onEdit, onDelete, onMessage }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <img src={user.avatar} alt="Avatar" />
      <button onClick={() => onEdit(user.id)}>Edit</button>
      <button onClick={() => onDelete(user.id)}>Delete</button>
      <button onClick={() => onMessage(user.id)}>Message</button>
    </div>
  );
}

// 好的设计：职责分离
function UserAvatar({ user }) {
  return <img src={user.avatar} alt={user.name} />;
}

function UserInfo({ user }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

function UserActions({ userId, onEdit, onDelete, onMessage }) {
  return (
    <div>
      <button onClick={() => onEdit(userId)}>Edit</button>
      <button onClick={() => onDelete(userId)}>Delete</button>
      <button onClick={() => onMessage(userId)}>Message</button>
    </div>
  );
}

// 组合使用
function UserProfile({ user, ...actions }) {
  return (
    <div>
      <UserAvatar user={user} />
      <UserInfo user={user} />
      <UserActions userId={user.id} {...actions} />
    </div>
  );
}
```

## 2. 状态管理策略

### 状态管理方案选择
**根据项目复杂度选择：**

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

**中型项目 - Zustand：**
```jsx
import { create } from 'zustand';

const useStore = create((set, get) => ({
  user: null,
  theme: 'light',
  
  // Actions
  setUser: (user) => set({ user }),
  setTheme: (theme) => set({ theme }),
  
  login: async (credentials) => {
    const user = await authService.login(credentials);
    set({ user });
  },
  
  logout: () => {
    authService.logout();
    set({ user: null });
  },
  
  // Derived state
  isAuthenticated: () => !!get().user,
  
  // Async action with error handling
  fetchUser: async (userId) => {
    try {
      const user = await userService.getUser(userId);
      set({ user });
    } catch (error) {
      console.error('Failed to fetch user:', error);
    }
  }
}));

// 使用示例
function UserProfile() {
  const { user, fetchUser, isAuthenticated } = useStore();
  
  useEffect(() => {
    if (isAuthenticated()) {
      fetchUser('current');
    }
  }, [fetchUser, isAuthenticated]);
  
  return <div>{user?.name}</div>;
}
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

### 组件测试体系
**测试金字塔实现：**

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

## 5. 部署与CI/CD

### Docker容器化部署
**Dockerfile配置：**
```dockerfile
# 多阶段构建
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# 生产环境
FROM nginx:alpine

COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**nginx配置优化：**
```nginx
# nginx.conf
events {}

http {
  include /etc/nginx/mime.types;
  
  server {
    listen 80;
    
    # Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript;
    
    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
      expires 1y;
      add_header Cache-Control "public, immutable";
    }
    
    # SPA路由处理
    location / {
      try_files $uri $uri/ /index.html;
    }
  }
}
```

### GitHub Actions CI/CD
**自动化部署流水线：**
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test -- --coverage --watchAll=false
    
    - name: Build application
      run: npm run build
    
    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build
        path: build/
  
  deploy:
    runs-on: ubuntu-latest
    needs: test
    
    steps:
    - name: Download build artifacts
      uses: actions/download-artifact@v3
      with:
        name: build
        path: build/
    
    - name: Deploy to production
      uses: appleboy/scp-action@v0.1.3
      with:
        host: ${{ secrets.PRODUCTION_HOST }}
        username: ${{ secrets.PRODUCTION_USER }}
        key: ${{ secrets.SSH_KEY }}
        source: "build/*"
        target: "/var/www/app"
    
    - name: Restart nginx
      uses: appleboy/ssh-action@v0.1.6
      with:
        host: ${{ secrets.PRODUCTION_HOST }}
        username: ${{ secrets.PRODUCTION_USER }}
        key: ${{ secrets.SSH_KEY }}
        script: |
          sudo systemctl restart nginx
          echo "Deployment completed successfully"
```

## 6. 监控与错误处理

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
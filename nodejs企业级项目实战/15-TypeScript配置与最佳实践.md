# 15-TypeScript配置与最佳实践

## 1. TypeScript项目配置详解

### 1.1 基础配置文件

#### 1.1.1 tsconfig.json 详细配置

```json
{
  "compilerOptions": {
    // 基础配置
    "target": "ES2020",                    // 编译目标版本
    "module": "commonjs",                  // 模块系统
    "lib": ["ES2020"],                     // 包含的库文件
    "outDir": "./dist",                    // 输出目录
    "rootDir": "./src",                    // 源码目录
    "sourceMap": true,                     // 生成source map
    "declaration": true,                   // 生成声明文件
    "declarationMap": true,                // 生成声明文件的source map
    
    // 严格模式配置
    "strict": true,                        // 启用所有严格检查
    "noImplicitAny": true,                 // 禁止隐式any
    "noImplicitReturns": true,             // 禁止隐式返回
    "noImplicitThis": true,                // 禁止隐式this
    "noUnusedLocals": true,                // 禁止未使用的局部变量
    "noUnusedParameters": true,            // 禁止未使用的参数
    "exactOptionalPropertyTypes": true,    // 精确的可选属性类型
    "noImplicitOverride": true,            // 禁止隐式override
    "noPropertyAccessFromIndexSignature": true, // 禁止从索引签名访问属性
    "noUncheckedIndexedAccess": true,      // 禁止未检查的索引访问
    
    // 模块解析配置
    "moduleResolution": "node",            // 模块解析策略
    "esModuleInterop": true,               // ES模块互操作
    "allowSyntheticDefaultImports": true,  // 允许合成默认导入
    "resolveJsonModule": true,             // 解析JSON模块
    "skipLibCheck": true,                  // 跳过库文件检查
    "forceConsistentCasingInFileNames": true, // 强制文件名大小写一致
    
    // 实验性功能
    "experimentalDecorators": true,        // 实验性装饰器
    "emitDecoratorMetadata": true,         // 发射装饰器元数据
    
    // 路径映射
    "baseUrl": "./src",
    "paths": {
      "@/*": ["*"],
      "@/app/*": ["app/*"],
      "@/config/*": ["config/*"],
      "@/middleware/*": ["middleware/*"],
      "@/utils/*": ["utils/*"],
      "@/types/*": ["types/*"]
    }
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "tests",
    "**/*.test.ts",
    "**/*.spec.ts"
  ],
  "ts-node": {
    "esm": true,
    "experimentalSpecifierResolution": "node"
  }
}
```

#### 1.1.2 环境特定配置

**tsconfig.dev.json（开发环境）**
```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "sourceMap": true,
    "removeComments": false,
    "noEmit": true
  },
  "include": [
    "src/**/*",
    "tests/**/*"
  ]
}
```

**tsconfig.prod.json（生产环境）**
```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "sourceMap": false,
    "removeComments": true,
    "declaration": false,
    "declarationMap": false
  },
  "exclude": [
    "node_modules",
    "dist",
    "tests",
    "**/*.test.ts",
    "**/*.spec.ts",
    "**/*.dev.ts"
  ]
}
```

### 1.2 开发工具配置

#### 1.2.1 ESLint TypeScript配置

**.eslintrc.js**
```typescript
// .eslintrc.ts
import type { Linter } from 'eslint';

const config: Linter.Config = {
  env: {
    browser: false,
    es2021: true,
    node: true,
    jest: true
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    '@typescript-eslint/recommended-requiring-type-checking',
    'prettier'
  ],
  plugins: [
    '@typescript-eslint',
    'prettier'
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2021,
    sourceType: 'module',
    project: './tsconfig.json',
    tsconfigRootDir: __dirname
  },
  rules: {
    // Prettier集成
    'prettier/prettier': 'error',
    
    // 基础规则
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-unused-vars': 'off',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    
    // TypeScript特定规则
    '@typescript-eslint/explicit-function-return-type': 'warn',
    '@typescript-eslint/explicit-module-boundary-types': 'warn',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/no-non-null-assertion': 'error',
    '@typescript-eslint/prefer-nullish-coalescing': 'error',
    '@typescript-eslint/prefer-optional-chain': 'error',
    '@typescript-eslint/no-unnecessary-type-assertion': 'error',
    '@typescript-eslint/no-floating-promises': 'error',
    '@typescript-eslint/await-thenable': 'error',
    '@typescript-eslint/no-misused-promises': 'error',
    
    // 代码质量规则
    'no-var': 'error',
    'prefer-const': 'error',
    'require-await': 'error',
    'no-return-await': 'error'
  },
  ignorePatterns: ['dist/', 'node_modules/', '*.js', '*.d.ts']
};
```

#### 1.2.2 Jest TypeScript配置

**jest.config.js**
```typescript
// jest.config.ts
import type { Config } from 'jest';

const config: Config = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src', '<rootDir>/tests'],
  testMatch: [
    '**/__tests__/**/*.ts',
    '**/?(*.)+(spec|test).ts'
  ],
  transform: {
    '^.+\\.ts$': 'ts-jest'
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/*.test.ts',
    '!src/**/*.spec.ts'
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1'
  }
};
```

## 2. TypeScript最佳实践

### 2.1 类型定义策略

#### 2.1.1 接口设计原则

```typescript
// ✅ 好的接口设计
interface User {
  readonly id: number;           // 只读属性
  username: string;
  email: string;
  profile?: UserProfile;         // 可选属性
  roles: readonly Role[];        // 只读数组
}

// ❌ 避免的设计
interface BadUser {
  id: any;                       // 避免使用any
  username: string | null;       // 避免联合类型，使用可选属性
  [key: string]: any;            // 避免索引签名
}

// ✅ 使用泛型提高复用性
interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
  errors?: string[];
}

// 使用示例
type UserResponse = ApiResponse<User>;
type ProductListResponse = ApiResponse<Product[]>;
```

#### 2.1.2 类型别名与接口选择

```typescript
// 接口：用于定义对象结构
interface DatabaseConfig {
  host: string;
  port: number;
  database: string;
}

// 类型别名：用于联合类型、复杂类型
type Status = 'pending' | 'approved' | 'rejected';
type EventHandler<T> = (event: T) => void;
type Config = DatabaseConfig & { timeout: number };

// 条件类型
type NonNullable<T> = T extends null | undefined ? never : T;
type ApiEndpoint<T> = T extends 'user' ? '/api/users' : '/api/unknown';
```

### 2.2 错误处理模式

#### 2.2.1 Result模式

```typescript
// 定义Result类型
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

// 使用示例
async function getUserById(id: number): Promise<Result<User, string>> {
  try {
    const user = await userRepository.findById(id);
    if (!user) {
      return { success: false, error: 'User not found' };
    }
    return { success: true, data: user };
  } catch (error) {
    return { success: false, error: 'Database error' };
  }
}

// 使用Result
const result = await getUserById(1);
if (result.success) {
  console.log(result.data.username); // TypeScript知道data存在
} else {
  console.error(result.error); // TypeScript知道error存在
}
```

#### 2.2.2 自定义错误类

```typescript
// 基础错误类
abstract class AppError extends Error {
  abstract readonly statusCode: number;
  abstract readonly isOperational: boolean;
  
  constructor(message: string, public readonly context?: any) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

// 具体错误类
class ValidationError extends AppError {
  readonly statusCode = 400;
  readonly isOperational = true;
  
  constructor(message: string, public readonly field?: string) {
    super(message, { field });
  }
}

class NotFoundError extends AppError {
  readonly statusCode = 404;
  readonly isOperational = true;
}

// 使用示例
function validateUser(userData: any): User {
  if (!userData.username) {
    throw new ValidationError('Username is required', 'username');
  }
  if (!userData.email) {
    throw new ValidationError('Email is required', 'email');
  }
  return userData as User;
}
```

### 2.3 依赖注入模式

#### 2.3.1 服务容器

```typescript
// 服务容器接口
interface ServiceContainer {
  register<T>(token: string, factory: () => T): void;
  get<T>(token: string): T;
  isRegistered(token: string): boolean;
}

// 实现
class DIContainer implements ServiceContainer {
  private services = new Map<string, any>();
  private factories = new Map<string, () => any>();

  register<T>(token: string, factory: () => T): void {
    this.factories.set(token, factory);
  }

  get<T>(token: string): T {
    if (this.services.has(token)) {
      return this.services.get(token);
    }

    const factory = this.factories.get(token);
    if (!factory) {
      throw new Error(`Service ${token} not registered`);
    }

    const service = factory();
    this.services.set(token, service);
    return service;
  }

  isRegistered(token: string): boolean {
    return this.factories.has(token);
  }
}

// 使用示例
const container = new DIContainer();

// 注册服务
container.register('userService', () => new UserService());
container.register('emailService', () => new EmailService());

// 获取服务
const userService = container.get<UserService>('userService');
```

#### 2.3.2 装饰器注入

```typescript
// 装饰器定义
const services = new Map<string, any>();

function Injectable(token: string) {
  return function <T extends { new (...args: any[]): {} }>(constructor: T) {
    services.set(token, constructor);
    return constructor;
  };
}

function Inject(token: string) {
  return function (target: any, propertyKey: string | symbol | undefined, parameterIndex: number) {
    const existingTokens = Reflect.getMetadata('design:paramtypes', target) || [];
    existingTokens[parameterIndex] = token;
    Reflect.defineMetadata('design:paramtypes', existingTokens, target);
  };
}

// 使用示例
@Injectable('userService')
class UserService {
  async getUser(id: number): Promise<User> {
    // 实现
    return {} as User;
  }
}

class UserController {
  constructor(
    @Inject('userService') private userService: UserService
  ) {}
}
```

### 2.4 性能优化技巧

#### 2.4.1 类型推断优化

```typescript
// ✅ 利用类型推断
const users = [
  { id: 1, name: 'John', age: 30 },
  { id: 2, name: 'Jane', age: 25 }
]; // TypeScript自动推断为 { id: number; name: string; age: number; }[]

// ❌ 不必要的类型注解
const users: Array<{ id: number; name: string; age: number }> = [
  { id: 1, name: 'John', age: 30 },
  { id: 2, name: 'Jane', age: 25 }
];

// ✅ 使用const断言
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
} as const; // 类型为 { readonly apiUrl: "https://api.example.com"; readonly timeout: 5000; }
```

#### 2.4.2 编译优化

```typescript
// 使用项目引用
// tsconfig.json
{
  "references": [
    { "path": "./src" },
    { "path": "./tests" }
  ]
}

// src/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true
  }
}

// 使用增量编译
// tsconfig.json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  }
}
```

## 3. 企业级项目中的TypeScript应用

### 3.1 配置管理

```typescript
// src/config/types.ts
export interface DatabaseConfig {
  host: string;
  port: number;
  database: string;
  username: string;
  password: string;
  pool: {
    max: number;
    min: number;
    acquire: number;
    idle: number;
  };
}

export interface RedisConfig {
  host: string;
  port: number;
  password?: string;
  db: number;
}

export interface AppConfig {
  name: string;
  version: string;
  port: number;
  env: 'development' | 'production' | 'test';
  database: DatabaseConfig;
  redis: RedisConfig;
}

// src/config/index.ts
import { config } from 'dotenv';
import { AppConfig } from './types';

config();

const appConfig: AppConfig = {
  name: process.env.APP_NAME || 'ECommerce API',
  version: process.env.APP_VERSION || '1.0.0',
  port: parseInt(process.env.PORT || '3000'),
  env: (process.env.NODE_ENV as any) || 'development',
  database: {
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT || '3306'),
    database: process.env.DB_NAME || '',
    username: process.env.DB_USER || '',
    password: process.env.DB_PASSWORD || '',
    pool: {
      max: 20,
      min: 5,
      acquire: 30000,
      idle: 10000
    }
  },
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT || '6379'),
    password: process.env.REDIS_PASSWORD,
    db: parseInt(process.env.REDIS_DB || '0')
  }
};

export default appConfig;
```

### 3.2 API接口类型定义

```typescript
// src/types/api.types.ts
export interface ApiRequest<T = any> {
  body: T;
  params: Record<string, string>;
  query: Record<string, string>;
  headers: Record<string, string>;
}

export interface ApiResponse<T = any> {
  success: boolean;
  data?: T;
  message?: string;
  errors?: string[];
  pagination?: {
    page: number;
    limit: number;
    total: number;
    pages: number;
  };
}

// 具体API类型
export interface CreateUserRequest {
  username: string;
  email: string;
  password: string;
}

export interface UpdateUserRequest {
  username?: string;
  email?: string;
}

export interface UserResponse {
  id: number;
  username: string;
  email: string;
  status: string;
  created_at: string;
  updated_at: string;
}

// 控制器类型
export interface UserController {
  createUser(req: ApiRequest<CreateUserRequest>): Promise<ApiResponse<UserResponse>>;
  updateUser(req: ApiRequest<UpdateUserRequest>): Promise<ApiResponse<UserResponse>>;
  getUserById(req: ApiRequest): Promise<ApiResponse<UserResponse>>;
  deleteUser(req: ApiRequest): Promise<ApiResponse<void>>;
}
```

### 3.3 数据库模型类型

```typescript
// src/types/database.types.ts
export interface BaseEntity {
  id: number;
  created_at: Date;
  updated_at: Date;
}

export interface UserEntity extends BaseEntity {
  username: string;
  email: string;
  password_hash: string;
  salt: string;
  status: 'active' | 'inactive' | 'banned';
  email_verified: boolean;
}

export interface ProductEntity extends BaseEntity {
  sku: string;
  name: string;
  description?: string;
  price: number;
  category_id: number;
  status: 'draft' | 'active' | 'inactive';
  images?: string[];
  attributes?: Record<string, any>;
}

export interface OrderEntity extends BaseEntity {
  order_number: string;
  user_id: number;
  status: 'pending' | 'confirmed' | 'processing' | 'shipped' | 'delivered' | 'cancelled';
  payment_status: 'pending' | 'paid' | 'failed' | 'refunded';
  subtotal: number;
  shipping_amount: number;
  discount_amount: number;
  total_amount: number;
  shipping_address: Record<string, any>;
  notes?: string;
}

// 查询选项类型
export interface QueryOptions {
  page?: number;
  limit?: number;
  sort?: string;
  order?: 'ASC' | 'DESC';
  where?: Record<string, any>;
  include?: string[];
}

export interface PaginatedResult<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    pages: number;
  };
}
```

## 4. 总结

TypeScript在企业级项目中的应用不仅仅是类型检查，更是一种开发范式和工程实践。通过合理的配置和最佳实践，可以显著提升代码质量、开发效率和团队协作效果。

### 4.1 关键要点

1. **严格配置**：启用TypeScript严格模式，确保类型安全
2. **合理设计**：接口优先，合理使用类型别名和泛型
3. **错误处理**：统一的错误处理模式和类型定义
4. **性能优化**：利用类型推断和编译优化提升性能

### 4.2 持续改进

- 定期更新TypeScript版本和配置
- 收集团队反馈，优化开发体验
- 建立代码审查流程，确保类型安全
- 持续学习TypeScript新特性和最佳实践

# 07-API设计与文档

## 1. API设计概述

### 1.1 API设计的重要性

在企业级应用开发中，良好的API设计是确保系统可维护性、可扩展性和易用性的关键因素。优秀的API设计能够：

1. **提升开发效率**：清晰的API接口定义能够降低前后端协作成本
2. **增强系统稳定性**：规范的API设计能够减少错误和异常情况
3. **提高可维护性**：良好的设计模式使API易于理解和修改
4. **促进团队协作**：标准的API文档能够提高团队沟通效率

### 1.2 API设计原则

#### 1.2.1 一致性原则
在整个API设计中保持一致的命名规范、错误处理方式和响应格式。

#### 1.2.2 简洁性原则
API设计应尽量简洁明了，避免过度复杂的设计。

#### 1.2.3 可扩展性原则
API设计应考虑未来的扩展需求，预留扩展空间。

#### 1.2.4 安全性原则
API设计必须考虑安全性，包括认证、授权和数据保护。

## 2. RESTful API设计最佳实践

### 2.1 REST架构约束

REST（Representational State Transfer）是一种软件架构风格，具有以下约束条件：

1. **客户端-服务器分离**：关注点分离，提高可移植性
2. **无状态性**：每个请求都包含处理该请求所需的所有信息
3. **可缓存性**：响应可以被标记为可缓存或不可缓存
4. **统一接口**：通过标准接口访问资源
5. **分层系统**：客户端通常无法判断是否直接连接到终端服务器
6. **按需代码**（可选）：服务器可以临时向客户端传输可执行代码

### 2.2 资源设计

#### 2.2.1 资源命名规范

```javascript
// ✅ 推荐的资源命名
GET /api/users          // 获取用户列表
GET /api/users/123      // 获取特定用户
POST /api/users         // 创建新用户
PUT /api/users/123      // 更新用户信息
DELETE /api/users/123   // 删除用户

// ❌ 不推荐的命名
GET /api/getUsers
GET /api/UserList
POST /api/createUser
```

#### 2.2.2 使用名词而非动词

```javascript
// ✅ 正确的资源命名
GET /api/products
POST /api/products
PUT /api/products/123

// ❌ 错误的命名
GET /api/getProducts
POST /api/createProduct
PUT /api/updateProduct/123
```

#### 2.2.3 使用复数形式

```javascript
// ✅ 使用复数形式
GET /api/users
GET /api/products
GET /api/orders

// ❌ 使用单数形式
GET /api/user
GET /api/product
GET /api/order
```

### 2.3 HTTP方法使用

#### 2.3.1 标准HTTP方法

| 方法 | 用途 | 幂等性 | 安全性 |
|------|------|--------|--------|
| GET | 获取资源 | 是 | 是 |
| POST | 创建资源 | 否 | 否 |
| PUT | 更新资源（全量） | 是 | 否 |
| PATCH | 更新资源（部分） | 否 | 否 |
| DELETE | 删除资源 | 是 | 否 |

#### 2.3.2 方法使用示例

```javascript
// 获取用户列表
GET /api/users

// 获取特定用户
GET /api/users/123

// 创建新用户
POST /api/users
{
  "name": "张三",
  "email": "zhangsan@example.com"
}

// 更新用户信息（全量更新）
PUT /api/users/123
{
  "name": "张三",
  "email": "zhangsan_new@example.com"
}

// 更新用户信息（部分更新）
PATCH /api/users/123
{
  "email": "zhangsan_updated@example.com"
}

// 删除用户
DELETE /api/users/123
```

### 2.4 状态码设计

#### 2.4.1 常用HTTP状态码

```javascript
// 2xx 成功状态码
200 OK              // 请求成功
201 Created         // 资源创建成功
204 No Content      // 请求成功但无返回内容

// 4xx 客户端错误状态码
400 Bad Request     // 请求参数错误
401 Unauthorized    // 未认证
403 Forbidden       // 无权限访问
404 Not Found       // 资源不存在
409 Conflict        // 资源冲突

// 5xx 服务器错误状态码
500 Internal Server Error  // 服务器内部错误
502 Bad Gateway            // 网关错误
503 Service Unavailable    // 服务不可用
```

#### 2.4.2 状态码使用示例

```javascript
// 成功响应
HTTP/1.1 200 OK
Content-Type: application/json

{
  "success": true,
  "data": {
    "id": 123,
    "name": "张三",
    "email": "zhangsan@example.com"
  }
}

// 创建资源成功
HTTP/1.1 201 Created
Content-Type: application/json

{
  "success": true,
  "data": {
    "id": 124,
    "name": "李四",
    "email": "lisi@example.com"
  }
}

// 参数验证失败
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "success": false,
  "message": "参数验证失败",
  "errors": [
    {
      "field": "email",
      "message": "邮箱格式不正确"
    }
  ]
}

// 认证失败
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{
  "success": false,
  "message": "认证失败，请提供有效的访问令牌"
}

// 资源不存在
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "success": false,
  "message": "请求的资源不存在"
}
```

### 2.5 请求和响应设计

#### 2.5.1 统一响应格式

```javascript
// 成功响应格式
{
  "success": true,
  "data": {
    // 业务数据
  },
  "message": "操作成功"
}

// 失败响应格式
{
  "success": false,
  "message": "错误描述",
  "errors": [
    {
      "field": "字段名",
      "message": "错误描述"
    }
  ]
}

// 分页响应格式
{
  "success": true,
  "data": [
    // 数据列表
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "pages": 10
  }
}
```

#### 2.5.2 请求参数设计

```javascript
// 查询参数
GET /api/users?page=1&limit=10&sort=created_at&order=desc&name=张三

// 请求体参数
POST /api/users
Content-Type: application/json

{
  "name": "张三",
  "email": "zhangsan@example.com",
  "profile": {
    "age": 25,
    "address": "北京市朝阳区"
  }
}
```

### 2.6 错误处理

#### 2.6.1 统一错误处理机制

```javascript
// 错误响应格式
{
  "success": false,
  "code": "USER_NOT_FOUND",
  "message": "用户不存在",
  "details": {
    "userId": 123
  },
  "timestamp": "2025-01-01T12:00:00Z"
}
```

#### 2.6.2 错误码设计

```javascript
// 错误码分类
const ERROR_CODES = {
  // 通用错误
  COMMON: {
    INVALID_PARAMS: 'COMMON_INVALID_PARAMS',
    UNAUTHORIZED: 'COMMON_UNAUTHORIZED',
    FORBIDDEN: 'COMMON_FORBIDDEN',
    NOT_FOUND: 'COMMON_NOT_FOUND',
    INTERNAL_ERROR: 'COMMON_INTERNAL_ERROR'
  },
  
  // 用户相关错误
  USER: {
    NOT_FOUND: 'USER_NOT_FOUND',
    ALREADY_EXISTS: 'USER_ALREADY_EXISTS',
    INVALID_CREDENTIALS: 'USER_INVALID_CREDENTIALS'
  },
  
  // 订单相关错误
  ORDER: {
    NOT_FOUND: 'ORDER_NOT_FOUND',
    ALREADY_PAID: 'ORDER_ALREADY_PAID',
    OUT_OF_STOCK: 'ORDER_OUT_OF_STOCK'
  }
};
```

## 3. API文档生成（Swagger/OpenAPI）

### 3.1 OpenAPI规范简介

OpenAPI规范（以前称为Swagger规范）是RESTful API的API描述格式。OpenAPI文档允许开发人员描述整个API，包括：

1. **可用路径**（/users）和**操作**（GET /users）
2. **操作参数**：每个操作的输入和输出
3. **认证方法**
4. **联系信息、许可、使用条款和其他信息**

### 3.2 Swagger集成

#### 3.2.1 安装依赖

```bash
npm install swagger-jsdoc swagger-ui-koa
```

#### 3.2.2 Swagger配置

```javascript
// src/config/swagger.js
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: '电商系统API文档',
      version: '1.0.0',
      description: '企业级电商系统API接口文档',
      contact: {
        name: 'API Support',
        email: 'support@example.com'
      }
    },
    servers: [
      {
        url: 'http://localhost:3000/api',
        description: '开发环境'
      },
      {
        url: 'https://api.example.com/api',
        description: '生产环境'
      }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      }
    },
    security: [
      {
        bearerAuth: []
      }
    ]
  },
  apis: ['./src/routes/*.js', './src/app/controllers/*.js']
};

module.exports = swaggerJsdoc(options);
```

#### 3.2.3 Swagger UI集成

```javascript
// src/app.js
const Koa = require('koa');
const Router = require('koa-router');
const swaggerUi = require('swagger-ui-koa');
const swaggerSpec = require('./config/swagger');

const app = new Koa();
const router = new Router();

// 集成Swagger UI
router.get('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));

app.use(router.routes());
```

### 3.3 API文档注解

#### 3.3.1 路由注解

```javascript
// src/routes/user.routes.js
/**
 * @swagger
 * tags:
 *   name: Users
 *   description: 用户管理
 */

/**
 * @swagger
 * /users:
 *   get:
 *     summary: 获取用户列表
 *     tags: [Users]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           minimum: 1
 *         description: 页码
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           minimum: 1
 *           maximum: 100
 *         description: 每页数量
 *     responses:
 *       "200":
 *         description: 成功获取用户列表
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *                 pagination:
 *                   $ref: '#/components/schemas/Pagination'
 *       "401":
 *         $ref: '#/components/responses/Unauthorized'
 *       "500":
 *         $ref: '#/components/responses/InternalServerError'
 */
router.get('/users', userController.list);

/**
 * @swagger
 * /users/{id}:
 *   get:
 *     summary: 获取用户详情
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: integer
 *         description: 用户ID
 *     responses:
 *       "200":
 *         description: 成功获取用户详情
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *       "401":
 *         $ref: '#/components/responses/Unauthorized'
 *       "404":
 *         $ref: '#/components/responses/NotFound'
 *       "500":
 *         $ref: '#/components/responses/InternalServerError'
 */
router.get('/users/:id', userController.show);
```

#### 3.3.2 数据模型定义

```javascript
// src/app/models/user.model.js
/**
 * @swagger
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       required:
 *         - name
 *         - email
 *       properties:
 *         id:
 *           type: integer
 *           description: 用户ID
 *         name:
 *           type: string
 *           description: 用户姓名
 *         email:
 *           type: string
 *           format: email
 *           description: 用户邮箱
 *         createdAt:
 *           type: string
 *           format: date-time
 *           description: 创建时间
 *         updatedAt:
 *           type: string
 *           format: date-time
 *           description: 更新时间
 *       example:
 *         id: 1
 *         name: "张三"
 *         email: "zhangsan@example.com"
 *         createdAt: "2025-01-01T00:00:00.000Z"
 *         updatedAt: "2025-01-01T00:00:00.000Z"
 * 
 *     Pagination:
 *       type: object
 *       properties:
 *         page:
 *           type: integer
 *           description: 当前页码
 *         limit:
 *           type: integer
 *           description: 每页数量
 *         total:
 *           type: integer
 *           description: 总记录数
 *         pages:
 *           type: integer
 *           description: 总页数
 * 
 *   responses:
 *     Unauthorized:
 *       description: 认证失败
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               success:
 *                 type: boolean
 *                 example: false
 *               message:
 *                 type: string
 *                 example: "认证失败，请提供有效的访问令牌"
 * 
 *     NotFound:
 *       description: 资源不存在
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               success:
 *                 type: boolean
 *                 example: false
 *               message:
 *                 type: string
 *                 example: "请求的资源不存在"
 * 
 *     InternalServerError:
 *       description: 服务器内部错误
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               success:
 *                 type: boolean
 *                 example: false
 *               message:
 *                 type: string
 *                 example: "服务器内部错误"
 */
```

### 3.4 API文档最佳实践

#### 3.4.1 文档完整性

确保每个API端点都有完整的文档描述，包括：

1. **端点描述**：清晰说明端点的功能
2. **请求参数**：详细说明所有请求参数
3. **响应格式**：提供成功和失败的响应示例
4. **错误码说明**：列出可能的错误码和含义

#### 3.4.2 示例数据

```javascript
/**
 * @swagger
 * /users:
 *   post:
 *     summary: 创建新用户
 *     tags: [Users]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *             properties:
 *               name:
 *                 type: string
 *                 description: 用户姓名
 *               email:
 *                 type: string
 *                 format: email
 *                 description: 用户邮箱
 *           example:
 *             name: "李四"
 *             email: "lisi@example.com"
 *     responses:
 *       "201":
 *         description: 用户创建成功
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *             example:
 *               success: true
 *               data:
 *                 id: 123
 *                 name: "李四"
 *                 email: "lisi@example.com"
 *                 createdAt: "2025-01-01T12:00:00.000Z"
 *                 updatedAt: "2025-01-01T12:00:00.000Z"
 */
```

#### 3.4.3 版本控制

```javascript
/**
 * @swagger
 * /v1/users:
 *   get:
 *     summary: 获取用户列表 (v1版本)
 *     tags: [Users]
 *     deprecated: true
 *     # ...
 * 
 * /v2/users:
 *   get:
 *     summary: 获取用户列表 (v2版本)
 *     tags: [Users]
 *     # ...
 */
```

## 4. API版本管理

### 4.1 版本管理策略

在API开发中，版本管理是确保向后兼容性和平滑升级的关键。常见的版本管理策略包括：

### 4.2 URI版本控制

#### 4.2.1 路径版本控制

```javascript
// 版本控制示例
GET /api/v1/users        // v1版本
GET /api/v2/users        // v2版本
POST /api/v1/products    // v1版本
POST /api/v2/products    // v2版本
```

#### 4.2.2 实现方式

```javascript
// src/routes/index.js
const Router = require('koa-router');
const v1Router = require('./v1');
const v2Router = require('./v2');

const router = new Router();

// 挂载不同版本的路由
router.use('/api/v1', v1Router.routes());
router.use('/api/v2', v2Router.routes());

module.exports = router;
```

```javascript
// src/routes/v1/user.routes.js
const Router = require('koa-router');
const userController = require('../../app/controllers/v1/user.controller');

const router = new Router({ prefix: '/users' });

router.get('/', userController.list);    // GET /api/v1/users
router.get('/:id', userController.show); // GET /api/v1/users/123

module.exports = router;
```

```javascript
// src/routes/v2/user.routes.js
const Router = require('koa-router');
const userController = require('../../app/controllers/v2/user.controller');

const router = new Router({ prefix: '/users' });

router.get('/', userController.list);    // GET /api/v2/users
router.get('/:id', userController.show); // GET /api/v2/users/123
router.get('/profile', userController.profile); // 新增端点

module.exports = router;
```

### 4.3 Header版本控制

#### 4.3.1 实现方式

```javascript
// src/middleware/version.middleware.js
const versionMiddleware = async (ctx, next) => {
  const apiVersion = ctx.headers['api-version'] || 'v1';
  
  // 根据版本号设置相应的控制器
  ctx.state.apiVersion = apiVersion;
  
  await next();
};

module.exports = versionMiddleware;
```

```javascript
// src/routes/user.routes.js
const Router = require('koa-router');
const versionMiddleware = require('../middleware/version.middleware');
const v1UserController = require('../app/controllers/v1/user.controller');
const v2UserController = require('../app/controllers/v2/user.controller');

const router = new Router({ prefix: '/users' });

router.use(versionMiddleware);

router.get('/', async (ctx) => {
  if (ctx.state.apiVersion === 'v2') {
    await v2UserController.list(ctx);
  } else {
    await v1UserController.list(ctx);
  }
});

module.exports = router;
```

### 4.4 版本管理最佳实践

#### 4.4.1 版本号规范

采用语义化版本控制（Semantic Versioning）：

- **主版本号**：当你做了不兼容的API修改
- **次版本号**：当你做了向下兼容的功能性新增
- **修订号**：当你做了向下兼容的问题修正

```javascript
// 版本号示例
v1.0.0  // 初始版本
v1.0.1  // 问题修复
v1.1.0  // 新增功能
v2.0.0  // 不兼容的API修改
```

#### 4.4.2 版本兼容性

```javascript
// src/app/controllers/v2/user.controller.js
const v1UserController = require('../v1/user.controller');

class UserController {
  // 新版本的列表接口
  async list(ctx) {
    // 新增功能：支持按创建时间排序
    const { sort = 'created_at', order = 'desc' } = ctx.query;
    
    // 调用v1版本的核心逻辑
    const result = await v1UserController.getListData(ctx.query);
    
    // 应用新的排序逻辑
    if (sort === 'created_at') {
      result.data.sort((a, b) => {
        const dateA = new Date(a.createdAt);
        const dateB = new Date(b.createdAt);
        return order === 'asc' ? dateA - dateB : dateB - dateA;
      });
    }
    
    ctx.body = {
      success: true,
      data: result.data,
      pagination: result.pagination
    };
  }
  
  // 新增的用户资料接口
  async profile(ctx) {
    // 新功能实现
    ctx.body = {
      success: true,
      data: {
        id: ctx.user.id,
        name: ctx.user.name,
        email: ctx.user.email,
        preferences: ctx.user.preferences
      }
    };
  }
}

module.exports = new UserController();
```

#### 4.4.3 版本废弃策略

```javascript
// src/routes/v1/user.routes.js
/**
 * @swagger
 * /users:
 *   get:
 *     summary: 获取用户列表 (已废弃)
 *     deprecated: true
 *     description: 此接口已废弃，请使用 /api/v2/users
 *     # ...
 */
```

```javascript
// src/middleware/deprecation.middleware.js
const deprecationMiddleware = async (ctx, next) => {
  // 检查是否使用了废弃的API版本
  if (ctx.path.startsWith('/api/v1')) {
    ctx.set('Warning', '299 - "API v1 is deprecated, please migrate to v2"');
  }
  
  await next();
};

module.exports = deprecationMiddleware;
```

## 5. API测试与验证

### 5.1 API测试策略

#### 5.1.1 单元测试

```javascript
// tests/unit/user.controller.test.js
const request = require('supertest');
const app = require('../../src/app');

describe('用户控制器测试', () => {
  describe('GET /api/users', () => {
    it('应该返回用户列表', async () => {
      const response = await request(app)
        .get('/api/users')
        .set('Authorization', 'Bearer valid-token')
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(Array.isArray(response.body.data)).toBe(true);
    });
    
    it('未认证时应该返回401', async () => {
      await request(app)
        .get('/api/users')
        .expect(401);
    });
  });
  
  describe('GET /api/users/:id', () => {
    it('应该返回特定用户信息', async () => {
      const userId = 123;
      const response = await request(app)
        .get(`/api/users/${userId}`)
        .set('Authorization', 'Bearer valid-token')
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(response.body.data.id).toBe(userId);
    });
    
    it('用户不存在时应该返回404', async () => {
      const userId = 999999;
      await request(app)
        .get(`/api/users/${userId}`)
        .set('Authorization', 'Bearer valid-token')
        .expect(404);
    });
  });
});
```

#### 5.1.2 集成测试

```javascript
// tests/integration/user.api.test.js
const request = require('supertest');
const app = require('../../src/app');
const { User } = require('../../src/app/models');

describe('用户API集成测试', () => {
  beforeEach(async () => {
    // 清理测试数据
    await User.destroy({ where: {} });
  });
  
  describe('POST /api/users', () => {
    it('应该成功创建新用户', async () => {
      const userData = {
        name: '测试用户',
        email: 'test@example.com'
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .set('Authorization', 'Bearer admin-token')
        .expect(201);
      
      expect(response.body.success).toBe(true);
      expect(response.body.data.name).toBe(userData.name);
      expect(response.body.data.email).toBe(userData.email);
      
      // 验证数据库中确实创建了用户
      const user = await User.findByPk(response.body.data.id);
      expect(user).not.toBeNull();
      expect(user.name).toBe(userData.name);
    });
  });
  
  describe('PUT /api/users/:id', () => {
    it('应该成功更新用户信息', async () => {
      // 先创建一个用户
      const user = await User.create({
        name: '原始用户',
        email: 'original@example.com'
      });
      
      const updatedData = {
        name: '更新后的用户',
        email: 'updated@example.com'
      };
      
      const response = await request(app)
        .put(`/api/users/${user.id}`)
        .send(updatedData)
        .set('Authorization', 'Bearer admin-token')
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(response.body.data.name).toBe(updatedData.name);
      
      // 验证数据库中的数据已更新
      const updatedUser = await User.findByPk(user.id);
      expect(updatedUser.name).toBe(updatedData.name);
      expect(updatedUser.email).toBe(updatedData.email);
    });
  });
});
```

### 5.2 API验证工具

#### 5.2.1 请求验证中间件

```javascript
// src/middleware/validation.middleware.js
const Joi = require('joi');

const validationMiddleware = (schema) => {
  return async (ctx, next) => {
    try {
      // 验证请求参数
      if (schema.params && ctx.params) {
        await schema.params.validateAsync(ctx.params);
      }
      
      // 验证查询参数
      if (schema.query && ctx.query) {
        await schema.query.validateAsync(ctx.query);
      }
      
      // 验证请求体
      if (schema.body && ctx.request.body) {
        await schema.body.validateAsync(ctx.request.body);
      }
      
      await next();
    } catch (error) {
      ctx.status = 400;
      ctx.body = {
        success: false,
        message: '参数验证失败',
        errors: error.details.map(detail => ({
          field: detail.path.join('.'),
          message: detail.message
        }))
      };
    }
  };
};

module.exports = validationMiddleware;
```

#### 5.2.2 使用验证中间件

```javascript
// src/app/validators/user.validator.js
const Joi = require('joi');

const userValidator = {
  // 创建用户验证
  create: Joi.object({
    name: Joi.string().min(2).max(50).required().messages({
      'string.min': '姓名至少2个字符',
      'string.max': '姓名最多50个字符',
      'any.required': '姓名是必填项'
    }),
    email: Joi.string().email().required().messages({
      'string.email': '邮箱格式不正确',
      'any.required': '邮箱是必填项'
    })
  }),
  
  // 更新用户验证
  update: Joi.object({
    name: Joi.string().min(2).max(50).optional().messages({
      'string.min': '姓名至少2个字符',
      'string.max': '姓名最多50个字符'
    }),
    email: Joi.string().email().optional().messages({
      'string.email': '邮箱格式不正确'
    })
  }).min(1), // 至少有一个字段
  
  // 查询参数验证
  list: Joi.object({
    page: Joi.number().integer().min(1).default(1).messages({
      'number.integer': '页码必须是整数',
      'number.min': '页码至少为1'
    }),
    limit: Joi.number().integer().min(1).max(100).default(10).messages({
      'number.integer': '每页数量必须是整数',
      'number.min': '每页数量至少为1',
      'number.max': '每页数量最多为100'
    })
  })
};

module.exports = userValidator;
```

```javascript
// src/routes/user.routes.js
const Router = require('koa-router');
const userController = require('../app/controllers/user.controller');
const validationMiddleware = require('../middleware/validation.middleware');
const { userValidator } = require('../app/validators/user.validator');

const router = new Router({ prefix: '/users' });

// 使用验证中间件
router.get('/', 
  validationMiddleware({ query: userValidator.list }),
  userController.list
);

router.post('/', 
  validationMiddleware({ body: userValidator.create }),
  userController.create
);

router.put('/:id', 
  validationMiddleware({ 
    params: Joi.object({ id: Joi.number().integer().required() }),
    body: userValidator.update 
  }),
  userController.update
);

module.exports = router;
```

## 6. API性能优化

### 6.1 响应缓存

#### 6.1.1 HTTP缓存头设置

```javascript
// src/middleware/cache.middleware.js
const cacheMiddleware = (maxAge = 300) => {
  return async (ctx, next) => {
    // 设置缓存头
    ctx.set('Cache-Control', `public, max-age=${maxAge}`);
    ctx.set('ETag', `"${Date.now()}"`);
    
    await next();
  };
};

module.exports = cacheMiddleware;
```

#### 6.1.2 条件请求处理

```javascript
// src/middleware/conditional.middleware.js
const conditionalMiddleware = async (ctx, next) => {
  // 检查If-None-Match头
  const ifNoneMatch = ctx.headers['if-none-match'];
  
  await next();
  
  // 生成ETag
  const etag = `"${JSON.stringify(ctx.body).length}"`;
  ctx.set('ETag', etag);
  
  // 如果ETag匹配，返回304
  if (ifNoneMatch === etag) {
    ctx.status = 304;
    ctx.body = null;
  }
};

module.exports = conditionalMiddleware;
```

### 6.2 分页优化

#### 6.2.1 游标分页

```javascript
// src/app/services/user.service.js
class UserService {
  // 基于游标的分页
  async getUsersWithCursor(options = {}) {
    const {
      limit = 10,
      cursor, // 游标（通常是最后一条记录的ID）
      sort = 'id',
      order = 'ASC'
    } = options;
    
    const where = {};
    
    // 如果有游标，添加条件
    if (cursor) {
      where[sort] = order === 'ASC' ? { [Op.gt]: cursor } : { [Op.lt]: cursor };
    }
    
    const users = await User.findAll({
      where,
      limit: parseInt(limit) + 1, // 多查一条记录判断是否有下一页
      order: [[sort, order]]
    });
    
    const hasNextPage = users.length > limit;
    
    // 如果有多查的记录，移除它
    if (hasNextPage) {
      users.pop();
    }
    
    // 获取下一个游标
    const nextCursor = users.length > 0 ? users[users.length - 1][sort] : null;
    
    return {
      data: users,
      pagination: {
        hasNextPage,
        nextCursor
      }
    };
  }
}

module.exports = new UserService();
```

#### 6.2.2 分页响应

```javascript
// src/app/controllers/user.controller.js
class UserController {
  async list(ctx) {
    const {
      page,
      limit,
      cursor, // 支持游标分页
      sort = 'id',
      order = 'ASC'
    } = ctx.query;
    
    let result;
    
    // 根据是否有游标选择分页方式
    if (cursor) {
      result = await userService.getUsersWithCursor({
        limit,
        cursor,
        sort,
        order
      });
    } else {
      result = await userService.getUsersWithPagination({
        page,
        limit,
        sort,
        order
      });
    }
    
    ctx.body = {
      success: true,
      data: result.data,
      pagination: result.pagination
    };
  }
}
```

## 7. 总结

本文档详细介绍了企业级电商系统的API设计与文档管理。通过遵循RESTful API设计最佳实践、集成Swagger文档生成工具和实施合理的版本管理策略，我们能够构建出高质量、易维护的API系统。

### 7.1 关键要点

1. **RESTful设计原则**：遵循标准的REST约束，使用合适的HTTP方法和状态码
2. **统一响应格式**：建立一致的请求和响应格式，提高API易用性
3. **完善的文档系统**：通过Swagger/OpenAPI规范生成完整的API文档
4. **版本管理策略**：实施合理的API版本控制，确保向后兼容性
5. **测试与验证**：建立完整的API测试体系，确保API质量
6. **性能优化**：通过缓存、分页等技术优化API性能

### 7.2 下一步学习

- 实现中间件与插件开发
- 构建缓存与性能优化方案
- 实现消息队列与异步处理
- 完善测试策略与实践

继续阅读后续文档，深入学习电商系统的其他核心功能模块！
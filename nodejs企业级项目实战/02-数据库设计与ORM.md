# 02-数据库设计与ORM

## 1. 数据库设计原则

### 1.0 TypeScript与数据库ORM集成

在开始数据库设计之前，我们先了解TypeScript如何与Sequelize ORM集成，提供类型安全的数据库操作：

#### 1.0.1 Sequelize TypeScript配置

```typescript
// src/database/connection.ts
import { Sequelize, Options } from 'sequelize';
import config from '../config';

const sequelizeOptions: Options = {
  host: config.database.host,
  port: config.database.port,
  dialect: 'mysql' as const,
  timezone: config.database.timezone,
  logging: config.database.logging,
  pool: config.database.pool,
  define: {
    timestamps: true,
    underscored: true,
    freezeTableName: true,
    paranoid: false
  }
};

const sequelize = new Sequelize(
  config.database.database,
  config.database.username,
  config.database.password,
  sequelizeOptions
);

export { sequelize };
export default sequelize;
```

#### 1.0.2 类型定义示例

```typescript
// src/app/types/user.types.ts
export interface UserAttributes {
  id: number;
  username: string;
  email: string;
  phone?: string;
  password_hash: string;
  salt: string;
  status: 'active' | 'inactive' | 'banned';
  email_verified: boolean;
  created_at: Date;
  updated_at: Date;
}

export interface UserCreationAttributes extends Omit<UserAttributes, 'id' | 'created_at' | 'updated_at'> {
  password: string; // 创建时使用明文密码
}

export interface UserInstance extends UserAttributes {
  validatePassword(password: string): Promise<boolean>;
  getRoles(): Promise<RoleInstance[]>;
  hasPermission(permissionName: string): Promise<boolean>;
}
```

### 1.1 关系型数据库三范式

#### 1.1.1 第一范式（1NF）：原子性
```sql
-- ❌ 违反第一范式
CREATE TABLE users (
    id INT PRIMARY KEY,
    contact VARCHAR(200) -- 包含电话和邮箱
);

-- ✅ 符合第一范式  
CREATE TABLE users (
    id INT PRIMARY KEY,
    phone VARCHAR(20),
    email VARCHAR(100)
);
```

#### 1.1.2 第二范式（2NF）：完全依赖
```sql
-- ✅ 订单商品表设计
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    PRIMARY KEY(order_id, product_id)
);
```

#### 1.1.3 第三范式（3NF）：消除传递依赖
```sql
-- ✅ 订单表设计
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    total_amount DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### 1.2 索引设计策略

```sql
-- 主键索引
CREATE TABLE products (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    sku VARCHAR(50) UNIQUE NOT NULL,
    INDEX idx_sku (sku)
);

-- 复合索引优化
CREATE INDEX idx_user_status_created ON orders (
    user_id, status, created_at DESC
);
```

## 2. 电商系统核心表设计

### 2.1 用户相关表

#### 2.1.1 用户主表
```sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20) UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    salt VARCHAR(50) NOT NULL,
    status ENUM('active', 'inactive', 'banned') DEFAULT 'active',
    email_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_username (username),
    INDEX idx_email (email),
    INDEX idx_status (status)
) ENGINE=InnoDB CHARSET=utf8mb4;
```

#### 2.1.2 角色权限表
```sql
-- 角色表
CREATE TABLE roles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    display_name VARCHAR(100) NOT NULL,
    description TEXT
) ENGINE=InnoDB CHARSET=utf8mb4;

-- 权限表
CREATE TABLE permissions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    resource VARCHAR(100) NOT NULL,
    action VARCHAR(50) NOT NULL
) ENGINE=InnoDB CHARSET=utf8mb4;

-- 用户角色关联表
CREATE TABLE user_roles (
    user_id BIGINT NOT NULL,
    role_id INT NOT NULL,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
) ENGINE=InnoDB CHARSET=utf8mb4;

-- 角色权限关联表
CREATE TABLE role_permissions (
    role_id INT NOT NULL,
    permission_id INT NOT NULL,
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE,
    FOREIGN KEY (permission_id) REFERENCES permissions(id) ON DELETE CASCADE
) ENGINE=InnoDB CHARSET=utf8mb4;
```

### 2.2 商品相关表

#### 2.2.1 商品分类表
```sql
CREATE TABLE categories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    parent_id INT DEFAULT 0,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    sort_order INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_parent_id (parent_id),
    INDEX idx_slug (slug)
) ENGINE=InnoDB CHARSET=utf8mb4;
```

#### 2.2.2 商品主表
```sql
CREATE TABLE products (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    sku VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(200) NOT NULL,
    slug VARCHAR(200) UNIQUE NOT NULL,
    description TEXT,
    category_id INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    images JSON,
    attributes JSON,
    status ENUM('draft', 'active', 'inactive') DEFAULT 'draft',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_sku (sku),
    INDEX idx_category_id (category_id),
    INDEX idx_status (status),
    FULLTEXT idx_search (name, description),
    
    FOREIGN KEY (category_id) REFERENCES categories(id)
) ENGINE=InnoDB CHARSET=utf8mb4;
```

#### 2.2.3 库存表
```sql
CREATE TABLE inventories (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL DEFAULT 0,
    reserved_quantity INT NOT NULL DEFAULT 0,
    available_quantity INT GENERATED ALWAYS AS (quantity - reserved_quantity),
    reorder_point INT DEFAULT 0,
    
    UNIQUE KEY uk_product (product_id),
    INDEX idx_available_quantity (available_quantity),
    
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
) ENGINE=InnoDB CHARSET=utf8mb4;
```

### 2.3 订单相关表

#### 2.3.1 订单主表
```sql
CREATE TABLE orders (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    user_id BIGINT NOT NULL,
    status ENUM('pending', 'confirmed', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    payment_status ENUM('pending', 'paid', 'failed', 'refunded') DEFAULT 'pending',
    
    subtotal DECIMAL(10,2) NOT NULL,
    shipping_amount DECIMAL(10,2) DEFAULT 0,
    discount_amount DECIMAL(10,2) DEFAULT 0,
    total_amount DECIMAL(10,2) NOT NULL,
    
    shipping_address JSON NOT NULL,
    notes TEXT,
    
    confirmed_at TIMESTAMP NULL,
    shipped_at TIMESTAMP NULL,
    delivered_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_order_number (order_number),
    INDEX idx_user_id (user_id),
    INDEX idx_status (status),
    
    FOREIGN KEY (user_id) REFERENCES users(id)
) ENGINE=InnoDB CHARSET=utf8mb4;
```

#### 2.3.2 订单商品表
```sql
CREATE TABLE order_items (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    sku VARCHAR(100) NOT NULL,
    name VARCHAR(200) NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    quantity INT NOT NULL,
    total_price DECIMAL(10,2) NOT NULL,
    
    INDEX idx_order_id (order_id),
    INDEX idx_product_id (product_id),
    
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id)
) ENGINE=InnoDB CHARSET=utf8mb4;
```

## 3. Sequelize ORM配置

### 3.1 数据库连接配置

```typescript
// src/database/connection.ts
import { Sequelize, Options } from 'sequelize';
import config from '../config';

const sequelizeOptions: Options = {
  host: config.database.host,
  port: config.database.port,
  dialect: 'mysql' as const,
  timezone: config.database.timezone,
  logging: config.database.logging,
  
  pool: {
    max: 20,
    min: 5,
    acquire: 30000,
    idle: 10000
  },
  
  define: {
    timestamps: true,
    underscored: true,
    freezeTableName: true,
    paranoid: false
  }
};

const sequelize = new Sequelize(
  config.database.database,
  config.database.username,
  config.database.password,
  sequelizeOptions
);

const testConnection = async (): Promise<void> => {
  try {
    await sequelize.authenticate();
    console.log('✅ Database connection established');
  } catch (error: any) {
    console.error('❌ Database connection failed:', error.message);
    throw error;
  }
};

export { sequelize, testConnection };
export default sequelize;
```

### 3.2 模型基类设计

```typescript
// src/app/models/base.model.ts
import { Model, FindAndCountOptions, Transaction, UpsertOptions } from 'sequelize';
import { sequelize } from '../../database/connection';

export interface PaginationOptions {
  page?: number;
  limit?: number;
  where?: any;
  include?: any[];
  order?: any[];
}

export interface PaginationResult<T> {
  data: T[];
  pagination: {
    total: number;
    page: number;
    limit: number;
    pages: number;
  };
}

export interface UpsertBatchOptions extends Omit<UpsertOptions, 'transaction'> {
  transaction?: Transaction;
}

export interface UpsertBatchResult<T> {
  instance: T;
  created: boolean;
}

class BaseModel extends Model {
  static async findWithPagination<T extends BaseModel>(
    this: typeof BaseModel,
    options: PaginationOptions = {}
  ): Promise<PaginationResult<T>> {
    const {
      page = 1,
      limit = 10,
      where = {},
      include = [],
      order = [['created_at', 'DESC']]
    } = options;

    const offset = (page - 1) * limit;

    const result = await this.findAndCountAll({
      where,
      include,
      order,
      limit: parseInt(limit.toString()),
      offset: parseInt(offset.toString()),
      distinct: true
    } as FindAndCountOptions);

    return {
      data: result.rows as T[],
      pagination: {
        total: result.count,
        page: parseInt(page.toString()),
        limit: parseInt(limit.toString()),
        pages: Math.ceil(result.count / limit)
      }
    };
  }

  static async upsertBatch<T extends BaseModel>(
    this: typeof BaseModel,
    records: any[],
    options: UpsertBatchOptions = {}
  ): Promise<UpsertBatchResult<T>[]> {
    const transaction = options.transaction || await sequelize.transaction();
    
    try {
      const results: UpsertBatchResult<T>[] = [];
      
      for (const record of records) {
        const [instance, created] = await this.upsert(record, {
          transaction,
          ...options
        });
        results.push({ instance: instance as T, created });
      }
      
      if (!options.transaction) {
        await transaction.commit();
      }
      
      return results;
    } catch (error) {
      if (!options.transaction) {
        await transaction.rollback();
      }
      throw error;
    }
  }

  toJSON(): any {
    const values = { ...this.get() };
    delete values.password_hash;
    delete values.salt;
    return values;
  }
}

export default BaseModel;
```

### 3.3 核心模型实现

#### 3.3.1 用户模型
```typescript
// src/app/models/user.model.ts
import { DataTypes, ModelCtor, Model } from 'sequelize';
import bcrypt from 'bcryptjs';
import BaseModel from './base.model';
import { sequelize } from '../../database/connection';
import { UserAttributes, UserCreationAttributes, UserInstance } from '../types/user.types';

class User extends BaseModel implements UserInstance {
  declare id: number;
  declare username: string;
  declare email: string;
  declare phone?: string;
  declare password_hash: string;
  declare salt: string;
  declare status: 'active' | 'inactive' | 'banned';
  declare email_verified: boolean;
  declare created_at: Date;
  declare updated_at: Date;

  async validatePassword(password: string): Promise<boolean> {
    return bcrypt.compare(password, this.password_hash);
  }

  static async hashPassword(password: string): Promise<{ hash: string; salt: string }> {
    const salt = await bcrypt.genSalt(12);
    return {
      hash: await bcrypt.hash(password, salt),
      salt
    };
  }

  async getRoles(): Promise<any[]> {
    return await (this as any).getRoleList({
      attributes: ['id', 'name', 'display_name']
    });
  }

  async hasPermission(permissionName: string): Promise<boolean> {
    const roles = await this.getRoles();
    
    for (const role of roles) {
      const permissions = await (role as any).getPermissionList({
        where: { name: permissionName }
      });
      
      if (permissions.length > 0) {
        return true;
      }
    }
    
    return false;
  }
}

User.init({
  username: {
    type: DataTypes.STRING(50),
    allowNull: false,
    unique: true,
    validate: {
      len: [3, 50],
      isAlphanumeric: true
    }
  },
  email: {
    type: DataTypes.STRING(100),
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true
    }
  },
  phone: {
    type: DataTypes.STRING(20),
    allowNull: true,
    unique: true
  },
  password_hash: {
    type: DataTypes.STRING(255),
    allowNull: false
  },
  salt: {
    type: DataTypes.STRING(50),
    allowNull: false
  },
  status: {
    type: DataTypes.ENUM('active', 'inactive', 'banned'),
    defaultValue: 'active'
  },
  email_verified: {
    type: DataTypes.BOOLEAN,
    defaultValue: false
  }
}, {
  sequelize,
  tableName: 'users',
  paranoid: true,
  hooks: {
    beforeCreate: async (user: User): Promise<void> => {
      if (user.password_hash && !user.salt) {
        const { hash, salt } = await User.hashPassword(user.password_hash);
        user.password_hash = hash;
        user.salt = salt;
      }
    }
  }
});

export default User;
export type UserModel = ModelCtor<User>;
```

#### 3.3.2 商品模型
```typescript
// src/app/models/product.model.ts
import { DataTypes, Model, Optional } from 'sequelize';
import BaseModel from './base.model';
import { ProductAttributes, ProductCreationAttributes } from '../types/product.types';

// 商品模型接口
interface ProductInstance extends Model<ProductAttributes, ProductCreationAttributes>, ProductAttributes {
  // 实例方法
  hasEnoughStock(quantity: number): Promise<boolean>;
  getStats(): Promise<ProductStats>;
  getVariants(): Promise<ProductVariant[]>;
  getInventory(): Promise<ProductInventory | null>;
  
  // 关联方法
  countOrderItemList(): Promise<number>;
  countReviewList(): Promise<number>;
}

// 商品统计接口
interface ProductStats {
  orderCount: number;
  reviewCount: number;
  totalSales: number;
  averageRating: number;
}

class Product extends BaseModel<ProductAttributes, ProductCreationAttributes> {
  declare id: number;
  declare name: string;
  declare description: string;
  declare price: number;
  declare category_id: number;
  declare brand_id: number;
  declare status: ProductStatus;
  declare created_at: Date;
  declare updated_at: Date;

  /**
   * 检查是否有足够库存
   * @param quantity 需要数量
   * @returns 是否有足够库存
   */
  async hasEnoughStock(quantity: number): Promise<boolean> {
    const inventory = await this.getInventory();
    return inventory ? inventory.available_quantity >= quantity : false;
  }

  /**
   * 获取商品统计信息
   * @returns 商品统计
   */
  async getStats(): Promise<ProductStats> {
    const [orderCount, reviewCount] = await Promise.all([
      this.countOrderItemList(),
      this.countReviewList ? this.countReviewList() : 0
    ]);

    return {
      order_count: orderCount,
      review_count: reviewCount
    };
  }
}

// 初始化商品模型
Product.init({
  sku: {
    type: DataTypes.STRING(100),
    allowNull: false,
    unique: true
  },
  name: {
    type: DataTypes.STRING(200),
    allowNull: false,
    validate: {
      len: [1, 200]
    }
  },
  slug: {
    type: DataTypes.STRING(200),
    allowNull: false,
    unique: true
  },
  description: {
    type: DataTypes.TEXT,
    allowNull: true
  },
  category_id: {
    type: DataTypes.INTEGER,
    allowNull: false
  },
  price: {
    type: DataTypes.DECIMAL(10, 2),
    allowNull: false,
    validate: {
      min: 0
    }
  },
  images: {
    type: DataTypes.JSON,
    allowNull: true
  },
  attributes: {
    type: DataTypes.JSON,
    allowNull: true
  },
  status: {
    type: DataTypes.ENUM('draft', 'active', 'inactive'),
    defaultValue: 'draft'
  }
}, {
  sequelize,
  tableName: 'products',
  indexes: [
    { fields: ['sku'] },
    { fields: ['category_id'] },
    { fields: ['status'] }
  ]
});

module.exports = Product;
```

### 3.4 模型关联关系

```typescript
// src/app/models/index.ts
import { Sequelize } from 'sequelize';
import User from './user.model';
import Role from './role.model';
import Permission from './permission.model';
import Product from './product.model';
import Category from './category.model';
import Order from './order.model';
import OrderItem from './order-item.model';
import Inventory from './inventory.model';

// 用户角色关联 - 多对多
User.belongsToMany(Role, {
  through: 'user_roles',
  foreignKey: 'user_id',
  as: 'roleList'
});

Role.belongsToMany(User, {
  through: 'user_roles',
  foreignKey: 'role_id',
  as: 'userList'
});

// 角色权限关联 - 多对多
Role.belongsToMany(Permission, {
  through: 'role_permissions',
  foreignKey: 'role_id',
  as: 'permissionList'
});

// 商品分类关联
Category.hasMany(Product, {
  foreignKey: 'category_id',
  as: 'productList'
});

Product.belongsTo(Category, {
  foreignKey: 'category_id',
  as: 'category'
});

// 商品库存关联
Product.hasOne(Inventory, {
  foreignKey: 'product_id',
  as: 'inventory'
});

Inventory.belongsTo(Product, {
  foreignKey: 'product_id',
  as: 'product'
});

// 订单关联
User.hasMany(Order, {
  foreignKey: 'user_id',
  as: 'orderList'
});

Order.belongsTo(User, {
  foreignKey: 'user_id',
  as: 'user'
});

Order.hasMany(OrderItem, {
  foreignKey: 'order_id',
  as: 'itemList'
});

OrderItem.belongsTo(Order, {
  foreignKey: 'order_id',
  as: 'order'
});

OrderItem.belongsTo(Product, {
  foreignKey: 'product_id',
  as: 'product'
});

module.exports = {
  User,
  Role,
  Permission,
  Product,
  Category,
  Order,
  OrderItem,
  Inventory
};
```

## 4. 数据库迁移与种子数据

### 4.1 Sequelize CLI配置

```typescript
// .sequelizerc
import * as path from 'path';

// Sequelize CLI配置接口
interface SequelizeConfig {
  'config': string;
  'models-path': string;
  'seeders-path': string;
  'migrations-path': string;
}

const config: SequelizeConfig = {
  'config': path.resolve('src', 'config', 'database.ts'),
  'models-path': path.resolve('src', 'app', 'models'),
  'seeders-path': path.resolve('src', 'database', 'seeders'),
  'migrations-path': path.resolve('src', 'database', 'migrations')
};

export default config;
```

### 4.2 数据库迁移

```typescript
// src/database/migrations/20241201000001-create-users.ts
import { QueryInterface, DataTypes } from 'sequelize';

// 迁移接口
interface Migration {
  up(queryInterface: QueryInterface, Sequelize: typeof DataTypes): Promise<void>;
  down(queryInterface: QueryInterface, Sequelize: typeof DataTypes): Promise<void>;
}

const migration: Migration = {
  async up(queryInterface: QueryInterface, Sequelize: typeof DataTypes): Promise<void> {
    await queryInterface.createTable('users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.BIGINT
      },
      username: {
        type: Sequelize.STRING(50),
        allowNull: false,
        unique: true
      },
      email: {
        type: Sequelize.STRING(100),
        allowNull: false,
        unique: true
      },
      password_hash: {
        type: Sequelize.STRING(255),
        allowNull: false
      },
      salt: {
        type: Sequelize.STRING(50),
        allowNull: false
      },
      status: {
        type: Sequelize.ENUM('active', 'inactive', 'banned'),
        defaultValue: 'active'
      },
      created_at: {
        allowNull: false,
        type: Sequelize.DATE,
        defaultValue: Sequelize.literal('CURRENT_TIMESTAMP')
      },
      updated_at: {
        allowNull: false,
        type: Sequelize.DATE,
        defaultValue: Sequelize.literal('CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP')
      }
    });

    await queryInterface.addIndex('users', ['username']);
    await queryInterface.addIndex('users', ['email']);
  },

  async down(queryInterface: QueryInterface, Sequelize: typeof DataTypes): Promise<void> {
    await queryInterface.dropTable('users');
  }
};

export default migration;
```

### 4.3 种子数据

```typescript
// src/database/seeders/20241201000001-init-roles.ts
import { QueryInterface, DataTypes } from 'sequelize';

// 种子数据接口
interface Seeder {
  up(queryInterface: QueryInterface, Sequelize: typeof DataTypes): Promise<void>;
  down(queryInterface: QueryInterface, Sequelize: typeof DataTypes): Promise<void>;
}

const seeder: Seeder = {
  async up(queryInterface: QueryInterface, Sequelize: typeof DataTypes): Promise<void> {
    await queryInterface.bulkInsert('roles', [
      {
        id: 1,
        name: 'admin',
        display_name: '管理员',
        description: '系统管理员'
      },
      {
        id: 2,
        name: 'customer',
        display_name: '客户',
        description: '普通客户'
      }
    ]);

    await queryInterface.bulkInsert('permissions', [
      {
        id: 1,
        name: 'user.create',
        resource: 'user',
        action: 'create'
      },
      {
        id: 2,
        name: 'product.manage',
        resource: 'product',
        action: 'manage'
      }
    ]);

    await queryInterface.bulkInsert('role_permissions', [
      { role_id: 1, permission_id: 1 },
      { role_id: 1, permission_id: 2 },
      { role_id: 2, permission_id: 2 }
    ]);
  },

  async down(queryInterface: QueryInterface, Sequelize: typeof DataTypes): Promise<void> {
    await queryInterface.bulkDelete('role_permissions', null, {});
    await queryInterface.bulkDelete('permissions', null, {});
    await queryInterface.bulkDelete('roles', null, {});
  }
};

export default seeder;
```

## 5. 数据库操作最佳实践

### 5.1 事务处理

```typescript
// 订单创建事务示例
import { Transaction } from 'sequelize';
import { OrderAttributes, OrderCreationAttributes } from '../types/order.types';
import { OrderItemAttributes, OrderItemCreationAttributes } from '../types/order.types';

// 订单数据接口
interface OrderData extends OrderCreationAttributes {
  user_id: number;
  total_amount: number;
  status: string;
}

// 订单商品数据接口
interface OrderItemData extends OrderItemCreationAttributes {
  product_id: number;
  quantity: number;
  unit_price: number;
}

const createOrderWithItems = async (
  orderData: OrderData, 
  items: OrderItemData[]
): Promise<Order> => {
  return await sequelize.transaction(async (t: Transaction): Promise<Order> => {
    // 创建订单
    const order = await Order.create(orderData, { transaction: t });
    
    // 创建订单商品
    const orderItems: OrderItemCreationAttributes[] = items.map(item => ({
      ...item,
      order_id: order.id
    }));
    
    await OrderItem.bulkCreate(orderItems, { transaction: t });
    
    // 更新库存
    for (const item of items) {
      await Inventory.decrement('quantity', {
        by: item.quantity,
        where: { product_id: item.product_id },
        transaction: t
      });
    }
    
    return order;
  });
};
```

### 5.2 查询优化

```typescript
// 预加载关联数据，避免N+1查询
const getOrdersWithDetails = async (userId: number): Promise<Order[]> => {
  return await Order.findAll({
    where: { user_id: userId },
    include: [
      {
        model: OrderItem,
        as: 'itemList',
        include: [
          {
            model: Product,
            as: 'product',
            attributes: ['id', 'name', 'sku', 'images']
          }
        ]
      },
      {
        model: User,
        as: 'user',
        attributes: ['id', 'username', 'email']
      }
    ],
    order: [['created_at', 'DESC']]
  });
};
```

### 5.3 缓存策略

```typescript
// Redis缓存示例
import { redis } from '../utils/redis.client';
import { Product, Category, Inventory } from '../models';
import { ProductInstance } from '../types/product.types';

/**
 * 获取商品信息（带缓存）
 * @param productId 商品ID
 * @returns 商品信息
 */
const getProductWithCache = async (productId: number): Promise<ProductInstance | null> => {
  const cacheKey = `product:${productId}`;
  
  // 尝试从缓存获取
  let product: ProductInstance | null = null;
  const cachedData = await redis.get(cacheKey);
  
  if (cachedData) {
    product = JSON.parse(cachedData) as ProductInstance;
    return product;
  }
  
  // 从数据库获取
  product = await Product.findByPk(productId, {
    include: [
      { model: Category, as: 'category' },
      { model: Inventory, as: 'inventory' }
    ]
  });
  
  if (product) {
    // 缓存1小时
    await redis.setex(cacheKey, 3600, JSON.stringify(product));
  }
  
  return product;
};
```

## 6. 总结

本文档详细介绍了企业级电商系统的数据库设计原则和ORM最佳实践。通过规范的表设计、合理的索引策略、完善的模型关联和高效的查询优化，为系统提供了稳定可靠的数据层支撑。

### 6.1 关键要点

1. **规范的数据库设计**：遵循三范式原则，合理设计表结构和索引
2. **完善的ORM模型**：使用Sequelize实现模型定义和关联关系
3. **高效的查询优化**：通过预加载、缓存等策略提升查询性能
4. **可靠的事务管理**：确保数据一致性和完整性

### 6.2 下一步学习

- 实现用户认证与权限管理系统
- 构建RESTful API接口
- 开发商品和订单管理功能

继续阅读下一章节，深入学习用户认证与权限管理的实现！
# 基本操作与CRUD示例

## 目录
1. [数据库连接管理](#数据库连接管理)
2. [表结构设计与创建](#表结构设计与创建)
3. [数据插入操作](#数据插入操作)
4. [数据查询操作](#数据查询操作)
5. [数据更新操作](#数据更新操作)
6. [数据删除操作](#数据删除操作)
7. [事务处理](#事务处理)
8. [错误处理机制](#错误处理机制)

---

## 数据库连接管理

### 1. 基本连接方式

**sqlite3库连接**：
```javascript
const sqlite3 = require('sqlite3').verbose();

// 文件数据库
const db = new sqlite3.Database('./app.db', (err) => {
    if (err) {
        console.error('连接失败:', err.message);
    } else {
        console.log('数据库连接成功');
    }
});

// 内存数据库
const memoryDb = new sqlite3.Database(':memory:');

// 临时数据库
const tempDb = new sqlite3.Database('');
```

**better-sqlite3库连接**：
```javascript
const Database = require('better-sqlite3');

// 文件数据库
const db = new Database('./app.db');

// 内存数据库
const memoryDb = new Database(':memory:');

// 只读模式
const readOnlyDb = new Database('./app.db', { readonly: true });
```

### 2. 连接配置

**sqlite3配置**：
```javascript
const db = new sqlite3.Database('./app.db', sqlite3.OPEN_READWRITE | sqlite3.OPEN_CREATE, (err) => {
    if (err) {
        console.error('连接失败:', err.message);
    } else {
        // 启用WAL模式
        db.run('PRAGMA journal_mode = WAL');
        // 启用外键约束
        db.run('PRAGMA foreign_keys = ON');
        // 设置同步模式
        db.run('PRAGMA synchronous = NORMAL');
    }
});
```

**better-sqlite3配置**：
```javascript
const db = new Database('./app.db', {
    verbose: console.log,  // 调试模式
    readonly: false,       // 读写模式
    fileMustExist: false   // 文件不存在时创建
});

// 设置PRAGMA
db.pragma('journal_mode = WAL');
db.pragma('foreign_keys = ON');
db.pragma('synchronous = NORMAL');
```

---

## 表结构设计与创建

### 1. 基本表创建

**用户表示例**：
```javascript
// sqlite3方式
db.run(`
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE NOT NULL,
        email TEXT UNIQUE NOT NULL,
        password_hash TEXT NOT NULL,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
`, (err) => {
    if (err) {
        console.error('创建表失败:', err.message);
    } else {
        console.log('用户表创建成功');
    }
});

// better-sqlite3方式
db.exec(`
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE NOT NULL,
        email TEXT UNIQUE NOT NULL,
        password_hash TEXT NOT NULL,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
`);
```

### 2. 复杂表结构

**文章和评论系统**：
```javascript
// 文章表
db.run(`
    CREATE TABLE IF NOT EXISTS posts (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        content TEXT,
        author_id INTEGER NOT NULL,
        status TEXT DEFAULT 'draft' CHECK(status IN ('draft', 'published', 'archived')),
        view_count INTEGER DEFAULT 0,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        FOREIGN KEY (author_id) REFERENCES users (id) ON DELETE CASCADE
    )
`);

// 评论表
db.run(`
    CREATE TABLE IF NOT EXISTS comments (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        post_id INTEGER NOT NULL,
        user_id INTEGER NOT NULL,
        content TEXT NOT NULL,
        parent_id INTEGER,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        FOREIGN KEY (post_id) REFERENCES posts (id) ON DELETE CASCADE,
        FOREIGN KEY (user_id) REFERENCES users (id) ON DELETE CASCADE,
        FOREIGN KEY (parent_id) REFERENCES comments (id) ON DELETE CASCADE
    )
`);

// 标签表
db.run(`
    CREATE TABLE IF NOT EXISTS tags (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT UNIQUE NOT NULL,
        description TEXT,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
`);

// 文章标签关联表
db.run(`
    CREATE TABLE IF NOT EXISTS post_tags (
        post_id INTEGER NOT NULL,
        tag_id INTEGER NOT NULL,
        PRIMARY KEY (post_id, tag_id),
        FOREIGN KEY (post_id) REFERENCES posts (id) ON DELETE CASCADE,
        FOREIGN KEY (tag_id) REFERENCES tags (id) ON DELETE CASCADE
    )
`);
```

### 3. 索引创建

```javascript
// 创建索引提高查询性能
db.run('CREATE INDEX IF NOT EXISTS idx_users_email ON users (email)');
db.run('CREATE INDEX IF NOT EXISTS idx_posts_author ON posts (author_id)');
db.run('CREATE INDEX IF NOT EXISTS idx_posts_status ON posts (status)');
db.run('CREATE INDEX IF NOT EXISTS idx_comments_post ON comments (post_id)');
db.run('CREATE INDEX IF NOT EXISTS idx_comments_user ON comments (user_id)');
```

---

## 数据插入操作

### 1. 单条数据插入

**sqlite3方式**：
```javascript
// 使用参数化查询防止SQL注入
const insertUser = db.prepare('INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)');

insertUser.run(['john_doe', 'john@example.com', 'hashed_password'], function(err) {
    if (err) {
        console.error('插入失败:', err.message);
    } else {
        console.log('用户插入成功, ID:', this.lastID);
    }
});

// 不使用预编译语句
db.run(
    'INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)',
    ['jane_doe', 'jane@example.com', 'hashed_password'],
    function(err) {
        if (err) {
            console.error('插入失败:', err.message);
        } else {
            console.log('用户插入成功, ID:', this.lastID);
        }
    }
);
```

**better-sqlite3方式**：
```javascript
// 预编译语句（推荐）
const insertUser = db.prepare('INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)');

try {
    const result = insertUser.run('john_doe', 'john@example.com', 'hashed_password');
    console.log('用户插入成功, ID:', result.lastInsertRowid);
} catch (error) {
    console.error('插入失败:', error.message);
}

// 直接执行
try {
    const result = db.run(
        'INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)',
        'jane_doe', 'jane@example.com', 'hashed_password'
    );
    console.log('用户插入成功, ID:', result.lastInsertRowid);
} catch (error) {
    console.error('插入失败:', error.message);
}
```

### 2. 批量数据插入

**sqlite3方式**：
```javascript
// 使用事务提高性能
db.serialize(() => {
    db.run('BEGIN TRANSACTION');
    
    const stmt = db.prepare('INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)');
    
    const users = [
        ['user1', 'user1@example.com', 'hash1'],
        ['user2', 'user2@example.com', 'hash2'],
        ['user3', 'user3@example.com', 'hash3']
    ];
    
    users.forEach(user => {
        stmt.run(user, (err) => {
            if (err) {
                console.error('批量插入失败:', err.message);
                db.run('ROLLBACK');
            }
        });
    });
    
    stmt.finalize();
    db.run('COMMIT', (err) => {
        if (err) {
            console.error('提交失败:', err.message);
        } else {
            console.log('批量插入成功');
        }
    });
});
```

**better-sqlite3方式**：
```javascript
// 使用事务和预编译语句
const insertUser = db.prepare('INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)');

const insertMany = db.transaction((users) => {
    for (const user of users) {
        insertUser.run(user);
    }
});

try {
    const users = [
        ['user1', 'user1@example.com', 'hash1'],
        ['user2', 'user2@example.com', 'hash2'],
        ['user3', 'user3@example.com', 'hash3']
    ];
    
    insertMany(users);
    console.log('批量插入成功');
} catch (error) {
    console.error('批量插入失败:', error.message);
}
```

---

## 数据查询操作

### 1. 基本查询

**sqlite3方式**：
```javascript
// 查询所有用户
db.all('SELECT * FROM users', (err, rows) => {
    if (err) {
        console.error('查询失败:', err.message);
    } else {
        console.log('所有用户:', rows);
    }
});

// 查询单个用户
db.get('SELECT * FROM users WHERE id = ?', [1], (err, row) => {
    if (err) {
        console.error('查询失败:', err.message);
    } else {
        console.log('用户信息:', row);
    }
});

// 条件查询
db.all('SELECT * FROM users WHERE created_at > ?', ['2024-01-01'], (err, rows) => {
    if (err) {
        console.error('查询失败:', err.message);
    } else {
        console.log('新用户:', rows);
    }
});
```

**better-sqlite3方式**：
```javascript
// 查询所有用户
try {
    const users = db.prepare('SELECT * FROM users').all();
    console.log('所有用户:', users);
} catch (error) {
    console.error('查询失败:', error.message);
}

// 查询单个用户
try {
    const user = db.prepare('SELECT * FROM users WHERE id = ?').get(1);
    console.log('用户信息:', user);
} catch (error) {
    console.error('查询失败:', error.message);
}

// 条件查询
try {
    const newUsers = db.prepare('SELECT * FROM users WHERE created_at > ?').all('2024-01-01');
    console.log('新用户:', newUsers);
} catch (error) {
    console.error('查询失败:', error.message);
}
```

### 2. 复杂查询

**JOIN查询**：
```javascript
// 查询文章及其作者信息
const query = `
    SELECT 
        p.id,
        p.title,
        p.content,
        p.created_at,
        u.username as author_name,
        u.email as author_email
    FROM posts p
    JOIN users u ON p.author_id = u.id
    WHERE p.status = 'published'
    ORDER BY p.created_at DESC
`;

// sqlite3方式
db.all(query, (err, rows) => {
    if (err) {
        console.error('查询失败:', err.message);
    } else {
        console.log('文章列表:', rows);
    }
});

// better-sqlite3方式
try {
    const posts = db.prepare(query).all();
    console.log('文章列表:', posts);
} catch (error) {
    console.error('查询失败:', error.message);
}
```

**聚合查询**：
```javascript
// 统计查询
const statsQuery = `
    SELECT 
        COUNT(*) as total_users,
        COUNT(CASE WHEN created_at > date('now', '-30 days') THEN 1 END) as new_users,
        COUNT(CASE WHEN created_at > date('now', '-7 days') THEN 1 END) as weekly_users
    FROM users
`;

// sqlite3方式
db.get(statsQuery, (err, row) => {
    if (err) {
        console.error('统计查询失败:', err.message);
    } else {
        console.log('用户统计:', row);
    }
});

// better-sqlite3方式
try {
    const stats = db.prepare(statsQuery).get();
    console.log('用户统计:', stats);
} catch (error) {
    console.error('统计查询失败:', error.message);
}
```

---

## 数据更新操作

### 1. 单条数据更新

**sqlite3方式**：
```javascript
// 更新用户信息
db.run(
    'UPDATE users SET email = ?, updated_at = CURRENT_TIMESTAMP WHERE id = ?',
    ['new_email@example.com', 1],
    function(err) {
        if (err) {
            console.error('更新失败:', err.message);
        } else {
            console.log('更新成功, 影响行数:', this.changes);
        }
    }
);
```

**better-sqlite3方式**：
```javascript
// 更新用户信息
try {
    const result = db.prepare('UPDATE users SET email = ?, updated_at = CURRENT_TIMESTAMP WHERE id = ?')
        .run('new_email@example.com', 1);
    console.log('更新成功, 影响行数:', result.changes);
} catch (error) {
    console.error('更新失败:', error.message);
}
```

### 2. 批量更新

**sqlite3方式**：
```javascript
// 批量更新用户状态
db.serialize(() => {
    db.run('BEGIN TRANSACTION');
    
    const stmt = db.prepare('UPDATE users SET status = ? WHERE id = ?');
    
    const updates = [
        ['active', 1],
        ['inactive', 2],
        ['active', 3]
    ];
    
    updates.forEach(update => {
        stmt.run(update, (err) => {
            if (err) {
                console.error('批量更新失败:', err.message);
                db.run('ROLLBACK');
            }
        });
    });
    
    stmt.finalize();
    db.run('COMMIT', (err) => {
        if (err) {
            console.error('提交失败:', err.message);
        } else {
            console.log('批量更新成功');
        }
    });
});
```

**better-sqlite3方式**：
```javascript
// 批量更新
const updateUser = db.prepare('UPDATE users SET status = ? WHERE id = ?');

const updateMany = db.transaction((updates) => {
    for (const update of updates) {
        updateUser.run(update);
    }
});

try {
    const updates = [
        ['active', 1],
        ['inactive', 2],
        ['active', 3]
    ];
    
    updateMany(updates);
    console.log('批量更新成功');
} catch (error) {
    console.error('批量更新失败:', error.message);
}
```

---

## 数据删除操作

### 1. 单条数据删除

**sqlite3方式**：
```javascript
// 删除用户
db.run('DELETE FROM users WHERE id = ?', [1], function(err) {
    if (err) {
        console.error('删除失败:', err.message);
    } else {
        console.log('删除成功, 影响行数:', this.changes);
    }
});
```

**better-sqlite3方式**：
```javascript
// 删除用户
try {
    const result = db.prepare('DELETE FROM users WHERE id = ?').run(1);
    console.log('删除成功, 影响行数:', result.changes);
} catch (error) {
    console.error('删除失败:', error.message);
}
```

### 2. 批量删除

**sqlite3方式**：
```javascript
// 删除过期数据
db.run('DELETE FROM sessions WHERE expires_at < ?', [Date.now()], function(err) {
    if (err) {
        console.error('批量删除失败:', err.message);
    } else {
        console.log('批量删除成功, 影响行数:', this.changes);
    }
});
```

**better-sqlite3方式**：
```javascript
// 删除过期数据
try {
    const result = db.prepare('DELETE FROM sessions WHERE expires_at < ?').run(Date.now());
    console.log('批量删除成功, 影响行数:', result.changes);
} catch (error) {
    console.error('批量删除失败:', error.message);
}
```

---

## 事务处理

### 1. 基本事务

**sqlite3方式**：
```javascript
// 转账示例
db.serialize(() => {
    db.run('BEGIN TRANSACTION');
    
    // 减少发送方余额
    db.run('UPDATE accounts SET balance = balance - ? WHERE id = ?', [100, 1], (err) => {
        if (err) {
            console.error('转账失败:', err.message);
            db.run('ROLLBACK');
            return;
        }
        
        // 增加接收方余额
        db.run('UPDATE accounts SET balance = balance + ? WHERE id = ?', [100, 2], (err) => {
            if (err) {
                console.error('转账失败:', err.message);
                db.run('ROLLBACK');
            } else {
                db.run('COMMIT', (err) => {
                    if (err) {
                        console.error('提交失败:', err.message);
                    } else {
                        console.log('转账成功');
                    }
                });
            }
        });
    });
});
```

**better-sqlite3方式**：
```javascript
// 转账示例
const transfer = db.transaction((fromId, toId, amount) => {
    const reduceBalance = db.prepare('UPDATE accounts SET balance = balance - ? WHERE id = ?');
    const increaseBalance = db.prepare('UPDATE accounts SET balance = balance + ? WHERE id = ?');
    
    reduceBalance.run(amount, fromId);
    increaseBalance.run(amount, toId);
});

try {
    transfer(1, 2, 100);
    console.log('转账成功');
} catch (error) {
    console.error('转账失败:', error.message);
}
```

---

## 错误处理机制

### 1. 基本错误处理

**sqlite3错误处理**：
```javascript
function handleDatabaseError(err, operation) {
    if (err) {
        console.error(`${operation}失败:`, err.message);
        
        // 根据错误类型进行不同处理
        if (err.code === 'SQLITE_CONSTRAINT') {
            console.error('数据约束错误');
        } else if (err.code === 'SQLITE_BUSY') {
            console.error('数据库忙碌，请稍后重试');
        } else if (err.code === 'SQLITE_LOCKED') {
            console.error('数据库被锁定');
        }
        
        return false;
    }
    return true;
}

// 使用示例
db.run('INSERT INTO users (username, email) VALUES (?, ?)', ['test', 'test@example.com'], function(err) {
    if (handleDatabaseError(err, '插入用户')) {
        console.log('用户插入成功, ID:', this.lastID);
    }
});
```

**better-sqlite3错误处理**：
```javascript
function handleDatabaseError(error, operation) {
    console.error(`${operation}失败:`, error.message);
    
    if (error.code === 'SQLITE_CONSTRAINT') {
        console.error('数据约束错误');
    } else if (error.code === 'SQLITE_BUSY') {
        console.error('数据库忙碌，请稍后重试');
    }
    
    return false;
}

// 使用示例
try {
    const result = db.prepare('INSERT INTO users (username, email) VALUES (?, ?)')
        .run('test', 'test@example.com');
    console.log('用户插入成功, ID:', result.lastInsertRowid);
} catch (error) {
    handleDatabaseError(error, '插入用户');
}
```

### 2. 高级错误处理

**错误重试机制**：
```javascript
async function retryOperation(operation, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await operation();
        } catch (error) {
            if (error.code === 'SQLITE_BUSY' && i < maxRetries - 1) {
                console.log(`操作失败，${1000 * (i + 1)}ms后重试...`);
                await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
                continue;
            }
            throw error;
        }
    }
}

// 使用示例
async function insertUserWithRetry(userData) {
    return await retryOperation(async () => {
        return new Promise((resolve, reject) => {
            db.run('INSERT INTO users (username, email) VALUES (?, ?)', 
                [userData.username, userData.email], 
                function(err) {
                    if (err) reject(err);
                    else resolve(this.lastID);
                }
            );
        });
    });
}
```

---

## 总结

通过本章的学习，您已经掌握了Node.js中SQLite的基本操作：

1. **数据库连接**：文件数据库、内存数据库、连接配置
2. **表结构设计**：基本表、复杂表、索引创建
3. **CRUD操作**：增删改查的完整实现
4. **事务处理**：确保数据一致性
5. **错误处理**：完善的错误处理机制

这些基础知识为后续学习高级特性和实际项目开发奠定了坚实基础。

---

*下一章：[高级特性与性能优化](./04-高级特性与性能优化.md)*

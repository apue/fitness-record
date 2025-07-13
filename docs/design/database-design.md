# Fitness Record 数据库设计文档

## 1. 数据库概述

### 1.1 数据库选择
- **开发环境**: SQLite (轻量级，便于开发)
- **生产环境**: PostgreSQL (高性能，支持复杂查询)
- **ORM**: SQLx (Rust原生异步ORM)

### 1.2 设计原则
- 遵循第三范式
- 合理使用索引优化查询性能
- 支持数据完整性和一致性
- 考虑未来扩展性

## 2. 数据库表结构

### 2.1 用户表 (users)
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    avatar_url VARCHAR(500),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_created_at ON users(created_at);
```

### 2.2 训练表 (trainings)
```sql
CREATE TABLE trainings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    date DATE NOT NULL,
    notes TEXT,
    total_volume DECIMAL(10,2) DEFAULT 0,
    peak_weight DECIMAL(8,2) DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_trainings_user_id ON trainings(user_id);
CREATE INDEX idx_trainings_date ON trainings(date);
CREATE INDEX idx_trainings_user_date ON trainings(user_id, date);
```

### 2.3 训练项目表 (exercises)
```sql
CREATE TABLE exercises (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    training_id UUID NOT NULL REFERENCES trainings(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    weight DECIMAL(8,2) NOT NULL,
    reps INTEGER NOT NULL,
    sets INTEGER NOT NULL,
    rest_time INTEGER, -- 休息时间（秒）
    notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_exercises_training_id ON exercises(training_id);
CREATE INDEX idx_exercises_name ON exercises(name);
CREATE INDEX idx_exercises_weight ON exercises(weight);
```

### 2.4 训练模板表 (templates)
```sql
CREATE TABLE templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_templates_user_id ON templates(user_id);
CREATE INDEX idx_templates_name ON templates(name);
```

### 2.5 模板项目表 (template_exercises)
```sql
CREATE TABLE template_exercises (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    template_id UUID NOT NULL REFERENCES templates(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    default_weight DECIMAL(8,2),
    default_reps INTEGER,
    default_sets INTEGER,
    default_rest_time INTEGER,
    order_index INTEGER NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_template_exercises_template_id ON template_exercises(template_id);
CREATE INDEX idx_template_exercises_order ON template_exercises(template_id, order_index);
```

### 2.6 用户会话表 (user_sessions)
```sql
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_user_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_user_sessions_token_hash ON user_sessions(token_hash);
CREATE INDEX idx_user_sessions_expires_at ON user_sessions(expires_at);
```

## 3. 数据关系图

```
users (用户)
├── trainings (训练记录) 1:N
│   └── exercises (训练项目) 1:N
├── templates (训练模板) 1:N
│   └── template_exercises (模板项目) 1:N
└── user_sessions (用户会话) 1:N
```

## 4. 数据模型定义

### 4.1 用户模型
```rust
#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
pub struct User {
    pub id: Uuid,
    pub username: String,
    pub email: String,
    #[serde(skip_serializing)]
    pub password_hash: String,
    pub avatar_url: Option<String>,
    pub is_active: bool,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct CreateUserRequest {
    pub username: String,
    pub email: String,
    pub password: String,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct UpdateUserRequest {
    pub username: Option<String>,
    pub avatar_url: Option<String>,
}
```

### 4.2 训练模型
```rust
#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
pub struct Training {
    pub id: Uuid,
    pub user_id: Uuid,
    pub date: NaiveDate,
    pub notes: Option<String>,
    pub total_volume: Decimal,
    pub peak_weight: Decimal,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct CreateTrainingRequest {
    pub date: NaiveDate,
    pub notes: Option<String>,
    pub exercises: Vec<CreateExerciseRequest>,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct TrainingWithExercises {
    #[serde(flatten)]
    pub training: Training,
    pub exercises: Vec<Exercise>,
}
```

### 4.3 训练项目模型
```rust
#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
pub struct Exercise {
    pub id: Uuid,
    pub training_id: Uuid,
    pub name: String,
    pub weight: Decimal,
    pub reps: i32,
    pub sets: i32,
    pub rest_time: Option<i32>,
    pub notes: Option<String>,
    pub created_at: DateTime<Utc>,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct CreateExerciseRequest {
    pub name: String,
    pub weight: f64,
    pub reps: i32,
    pub sets: i32,
    pub rest_time: Option<i32>,
    pub notes: Option<String>,
}
```

### 4.4 训练模板模型
```rust
#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
pub struct Template {
    pub id: Uuid,
    pub user_id: Uuid,
    pub name: String,
    pub description: Option<String>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct CreateTemplateRequest {
    pub name: String,
    pub description: Option<String>,
    pub exercises: Vec<CreateTemplateExerciseRequest>,
}
```

## 5. 数据库查询优化

### 5.1 常用查询索引
```sql
-- 用户训练记录查询
CREATE INDEX idx_trainings_user_date_desc ON trainings(user_id, date DESC);

-- 训练项目统计查询
CREATE INDEX idx_exercises_name_weight ON exercises(name, weight);

-- 模板查询
CREATE INDEX idx_templates_user_created ON templates(user_id, created_at DESC);
```

### 5.2 复杂查询优化
```sql
-- 获取用户训练统计
SELECT 
    COUNT(*) as total_trainings,
    SUM(total_volume) as total_volume,
    AVG(total_volume) as average_volume,
    MAX(peak_weight) as peak_weight
FROM trainings 
WHERE user_id = $1 
    AND date BETWEEN $2 AND $3;

-- 获取训练项目使用频率
SELECT 
    name,
    COUNT(*) as usage_count,
    SUM(weight * reps * sets) as total_volume,
    MAX(weight) as peak_weight,
    AVG(weight) as average_weight
FROM exercises e
JOIN trainings t ON e.training_id = t.id
WHERE t.user_id = $1 
    AND t.date BETWEEN $2 AND $3
GROUP BY name
ORDER BY usage_count DESC;
```

## 6. 数据迁移策略

### 6.1 迁移文件结构
```
database/
├── migrations/
│   ├── 001_create_users.sql
│   ├── 002_create_trainings.sql
│   ├── 003_create_exercises.sql
│   ├── 004_create_templates.sql
│   ├── 005_create_template_exercises.sql
│   └── 006_create_user_sessions.sql
└── connection.rs
```

### 6.2 迁移执行
```rust
pub async fn run_migrations(pool: &PgPool) -> Result<(), sqlx::Error> {
    sqlx::migrate!("./database/migrations").run(pool).await
}
```

## 7. 数据备份和恢复

### 7.1 备份策略
- 每日自动备份
- 保留30天备份文件
- 异地备份存储

### 7.2 恢复流程
1. 停止应用服务
2. 恢复数据库备份
3. 运行数据迁移
4. 验证数据完整性
5. 重启应用服务

## 8. 数据安全

### 8.1 敏感数据加密
- 用户密码使用bcrypt加密
- JWT token哈希存储
- 数据库连接使用SSL

### 8.2 数据访问控制
- 用户只能访问自己的数据
- 数据库用户权限最小化
- 定期审计数据访问日志

## 9. 性能监控

### 9.1 查询性能监控
- 慢查询日志记录
- 查询执行计划分析
- 索引使用情况监控

### 9.2 数据库连接池
```rust
pub async fn create_connection_pool() -> Result<PgPool, sqlx::Error> {
    let database_url = std::env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    
    PgPoolOptions::new()
        .max_connections(20)
        .min_connections(5)
        .connect(&database_url)
        .await
}
```

## 10. 数据一致性

### 10.1 事务处理
```rust
pub async fn create_training_with_exercises(
    pool: &PgPool,
    user_id: Uuid,
    training: CreateTrainingRequest,
) -> Result<Training, sqlx::Error> {
    let mut tx = pool.begin().await?;
    
    // 创建训练记录
    let training_id = sqlx::query!(
        "INSERT INTO trainings (user_id, date, notes) VALUES ($1, $2, $3) RETURNING id",
        user_id,
        training.date,
        training.notes
    )
    .fetch_one(&mut *tx)
    .await?
    .id;
    
    // 创建训练项目
    for exercise in training.exercises {
        sqlx::query!(
            "INSERT INTO exercises (training_id, name, weight, reps, sets, rest_time, notes) 
             VALUES ($1, $2, $3, $4, $5, $6, $7)",
            training_id,
            exercise.name,
            exercise.weight,
            exercise.reps,
            exercise.sets,
            exercise.rest_time,
            exercise.notes
        )
        .execute(&mut *tx)
        .await?;
    }
    
    tx.commit().await?;
    
    // 返回创建的训练记录
    get_training_by_id(pool, training_id).await
}
```

### 10.2 数据验证
- 输入数据验证
- 业务规则检查
- 外键约束保证 
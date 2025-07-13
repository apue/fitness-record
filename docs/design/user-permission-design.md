# Fitness Record 用户权限设计文档

## 1. 权限系统概述

### 1.1 设计目标
- 确保用户数据隔离和安全
- 支持细粒度的权限控制
- 便于未来功能扩展
- 符合最小权限原则

### 1.2 权限模型
采用基于角色的访问控制（RBAC）模型，结合资源所有权验证，确保用户只能访问自己的数据。

## 2. 用户角色定义

### 2.1 基础用户角色
```rust
#[derive(Debug, Clone, PartialEq)]
pub enum UserRole {
    User,      // 普通用户
    Premium,   // 高级用户（未来扩展）
    Admin,     // 管理员（系统管理）
}
```

### 2.2 角色权限矩阵
| 功能模块 | User | Premium | Admin |
|---------|------|---------|-------|
| 用户注册/登录 | ✅ | ✅ | ✅ |
| 个人资料管理 | ✅ | ✅ | ✅ |
| 训练记录管理 | ✅ | ✅ | ✅ |
| 数据分析查看 | ✅ | ✅ | ✅ |
| 训练模板管理 | ✅ | ✅ | ✅ |
| 高级分析功能 | ❌ | ✅ | ✅ |
| 数据导出功能 | ❌ | ✅ | ✅ |
| 系统管理功能 | ❌ | ❌ | ✅ |

## 3. 权限验证机制

### 3.1 JWT Token认证
```rust
#[derive(Debug, Serialize, Deserialize)]
pub struct Claims {
    pub sub: String,        // 用户ID
    pub username: String,    // 用户名
    pub role: UserRole,      // 用户角色
    pub exp: usize,          // 过期时间
    pub iat: usize,          // 签发时间
}

pub struct AuthenticatedUser {
    pub id: Uuid,
    pub username: String,
    pub role: UserRole,
}
```

### 3.2 认证中间件
```rust
pub async fn auth_middleware(
    mut req: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    // 从请求头获取token
    let auth_header = req
        .headers()
        .get("Authorization")
        .and_then(|h| h.to_str().ok())
        .and_then(|h| h.strip_prefix("Bearer "));

    let token = auth_header.ok_or(StatusCode::UNAUTHORIZED)?;
    
    // 验证token
    let claims = verify_token(token).map_err(|_| StatusCode::UNAUTHORIZED)?;
    
    // 将用户信息添加到请求扩展
    req.extensions_mut().insert(AuthenticatedUser {
        id: Uuid::parse_str(&claims.sub).unwrap(),
        username: claims.username,
        role: claims.role,
    });

    Ok(next.run(req).await)
}
```

### 3.3 资源所有权验证
```rust
pub async fn verify_resource_ownership<T>(
    user_id: Uuid,
    resource: &T,
) -> Result<(), AppError>
where
    T: HasOwner,
{
    if resource.owner_id() != user_id {
        return Err(AppError::Forbidden("无权访问此资源".into()));
    }
    Ok(())
}

// 为数据模型实现所有权检查
pub trait HasOwner {
    fn owner_id(&self) -> Uuid;
}

impl HasOwner for Training {
    fn owner_id(&self) -> Uuid {
        self.user_id
    }
}

impl HasOwner for Template {
    fn owner_id(&self) -> Uuid {
        self.user_id
    }
}
```

## 4. 权限控制实现

### 4.1 路由级权限控制
```rust
pub fn create_router() -> Router {
    Router::new()
        // 公开路由
        .route("/auth/register", post(register))
        .route("/auth/login", post(login))
        
        // 需要认证的路由
        .route_group("/api/v1")
        .layer(auth_middleware)
        .route("/users/me", get(get_user_info))
        .route("/users/me", put(update_user_info))
        .route("/trainings", get(list_trainings))
        .route("/trainings", post(create_training))
        .route("/trainings/:id", get(get_training))
        .route("/trainings/:id", put(update_training))
        .route("/trainings/:id", delete(delete_training))
        .route("/templates", get(list_templates))
        .route("/templates", post(create_template))
        .route("/analytics/statistics", get(get_statistics))
        .route("/analytics/volume-trend", get(get_volume_trend))
}
```

### 4.2 处理器级权限控制
```rust
pub async fn get_training(
    Path(training_id): Path<Uuid>,
    Extension(user): Extension<AuthenticatedUser>,
    Extension(pool): Extension<PgPool>,
) -> Result<Json<ApiResponse<TrainingWithExercises>>, AppError> {
    // 获取训练记录
    let training = get_training_by_id(&pool, training_id).await?;
    
    // 验证资源所有权
    verify_resource_ownership(user.id, &training).await?;
    
    // 获取训练项目
    let exercises = get_exercises_by_training_id(&pool, training_id).await?;
    
    let result = TrainingWithExercises {
        training,
        exercises,
    };
    
    Ok(Json(ApiResponse::success(result)))
}

pub async fn create_training(
    Extension(user): Extension<AuthenticatedUser>,
    Extension(pool): Extension<PgPool>,
    Json(request): Json<CreateTrainingRequest>,
) -> Result<Json<ApiResponse<Training>>, AppError> {
    // 验证用户权限
    if user.role == UserRole::Admin {
        return Err(AppError::Forbidden("管理员不能创建训练记录".into()));
    }
    
    // 创建训练记录
    let training = create_training_with_exercises(&pool, user.id, request).await?;
    
    Ok(Json(ApiResponse::success(training)))
}
```

### 4.3 角色级权限控制
```rust
pub async fn get_advanced_analytics(
    Extension(user): Extension<AuthenticatedUser>,
    Extension(pool): Extension<PgPool>,
) -> Result<Json<ApiResponse<AdvancedAnalytics>>, AppError> {
    // 检查用户角色
    match user.role {
        UserRole::Premium | UserRole::Admin => {
            let analytics = get_advanced_analytics_data(&pool, user.id).await?;
            Ok(Json(ApiResponse::success(analytics)))
        }
        UserRole::User => {
            Err(AppError::Forbidden("需要高级用户权限".into()))
        }
    }
}
```

## 5. 数据访问控制

### 5.1 数据库查询权限
```rust
pub async fn get_user_trainings(
    pool: &PgPool,
    user_id: Uuid,
    filters: TrainingFilters,
) -> Result<Vec<Training>, sqlx::Error> {
    // 确保只能查询自己的训练记录
    sqlx::query_as!(
        Training,
        r#"
        SELECT * FROM trainings 
        WHERE user_id = $1 
        AND ($2::date IS NULL OR date >= $2)
        AND ($3::date IS NULL OR date <= $3)
        ORDER BY date DESC
        "#,
        user_id,
        filters.start_date,
        filters.end_date
    )
    .fetch_all(pool)
    .await
}
```

### 5.2 数据修改权限
```rust
pub async fn update_training(
    pool: &PgPool,
    training_id: Uuid,
    user_id: Uuid,
    updates: UpdateTrainingRequest,
) -> Result<Training, AppError> {
    // 验证训练记录所有权
    let training = get_training_by_id(pool, training_id).await?;
    if training.user_id != user_id {
        return Err(AppError::Forbidden("无权修改此训练记录".into()));
    }
    
    // 执行更新
    update_training_record(pool, training_id, updates).await
}
```

## 6. 安全措施

### 6.1 密码安全
```rust
pub async fn hash_password(password: &str) -> Result<String, bcrypt::BcryptError> {
    let salt = bcrypt::gen_salt(bcrypt::DEFAULT_COST)?;
    bcrypt::hash(password, salt)
}

pub async fn verify_password(password: &str, hash: &str) -> Result<bool, bcrypt::BcryptError> {
    bcrypt::verify(password, hash)
}
```

### 6.2 Token安全
```rust
pub fn generate_token(user: &User) -> Result<String, jsonwebtoken::errors::Error> {
    let expiration = Utc::now()
        .checked_add_signed(chrono::Duration::hours(24))
        .expect("valid timestamp")
        .timestamp() as usize;

    let claims = Claims {
        sub: user.id.to_string(),
        username: user.username.clone(),
        role: UserRole::User, // 从数据库获取用户角色
        exp: expiration,
        iat: Utc::now().timestamp() as usize,
    };

    jsonwebtoken::encode(&Header::default(), &claims, &EncodingKey::from_secret(JWT_SECRET.as_ref()))
}

pub fn verify_token(token: &str) -> Result<Claims, jsonwebtoken::errors::Error> {
    let token_data = jsonwebtoken::decode::<Claims>(
        token,
        &DecodingKey::from_secret(JWT_SECRET.as_ref()),
        &Validation::default(),
    )?;
    
    Ok(token_data.claims)
}
```

### 6.3 输入验证
```rust
pub fn validate_training_request(request: &CreateTrainingRequest) -> Result<(), ValidationError> {
    // 验证日期
    if request.date > Utc::now().date_naive() {
        return Err(ValidationError::new("训练日期不能是未来日期"));
    }
    
    // 验证训练项目
    if request.exercises.is_empty() {
        return Err(ValidationError::new("至少需要一个训练项目"));
    }
    
    for exercise in &request.exercises {
        if exercise.weight <= 0.0 {
            return Err(ValidationError::new("重量必须大于0"));
        }
        if exercise.reps <= 0 {
            return Err(ValidationError::new("次数必须大于0"));
        }
        if exercise.sets <= 0 {
            return Err(ValidationError::new("组数必须大于0"));
        }
    }
    
    Ok(())
}
```

## 7. 审计日志

### 7.1 操作日志记录
```rust
#[derive(Debug, Serialize, Deserialize)]
pub struct AuditLog {
    pub id: Uuid,
    pub user_id: Uuid,
    pub action: String,
    pub resource_type: String,
    pub resource_id: Option<Uuid>,
    pub details: serde_json::Value,
    pub ip_address: String,
    pub user_agent: String,
    pub created_at: DateTime<Utc>,
}

pub async fn log_audit_event(
    pool: &PgPool,
    user_id: Uuid,
    action: &str,
    resource_type: &str,
    resource_id: Option<Uuid>,
    details: serde_json::Value,
    req: &Request,
) -> Result<(), sqlx::Error> {
    let ip_address = req
        .headers()
        .get("X-Forwarded-For")
        .or(req.headers().get("X-Real-IP"))
        .and_then(|h| h.to_str().ok())
        .unwrap_or("unknown");

    let user_agent = req
        .headers()
        .get("User-Agent")
        .and_then(|h| h.to_str().ok())
        .unwrap_or("unknown");

    sqlx::query!(
        r#"
        INSERT INTO audit_logs (user_id, action, resource_type, resource_id, details, ip_address, user_agent)
        VALUES ($1, $2, $3, $4, $5, $6, $7)
        "#,
        user_id,
        action,
        resource_type,
        resource_id,
        details,
        ip_address,
        user_agent
    )
    .execute(pool)
    .await?;

    Ok(())
}
```

## 8. 错误处理

### 8.1 权限错误类型
```rust
#[derive(Debug, thiserror::Error)]
pub enum AppError {
    #[error("认证失败: {0}")]
    Authentication(String),
    
    #[error("权限不足: {0}")]
    Forbidden(String),
    
    #[error("资源不存在: {0}")]
    NotFound(String),
    
    #[error("参数验证失败: {0}")]
    Validation(String),
    
    #[error("服务器内部错误: {0}")]
    Internal(String),
}

impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, error_code, message) = match self {
            AppError::Authentication(msg) => (StatusCode::UNAUTHORIZED, "AUTH_ERROR", msg),
            AppError::Forbidden(msg) => (StatusCode::FORBIDDEN, "FORBIDDEN", msg),
            AppError::NotFound(msg) => (StatusCode::NOT_FOUND, "NOT_FOUND", msg),
            AppError::Validation(msg) => (StatusCode::BAD_REQUEST, "VALIDATION_ERROR", msg),
            AppError::Internal(msg) => (StatusCode::INTERNAL_SERVER_ERROR, "INTERNAL_ERROR", msg),
        };

        let error_response = ErrorResponse {
            success: false,
            error: ErrorDetail {
                code: error_code.to_string(),
                message,
                details: None,
            },
            timestamp: Utc::now(),
        };

        (status, Json(error_response)).into_response()
    }
}
```

## 9. 权限测试

### 9.1 单元测试
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_verify_resource_ownership() {
        let user_id = Uuid::new_v4();
        let other_user_id = Uuid::new_v4();
        
        let training = Training {
            id: Uuid::new_v4(),
            user_id,
            date: Utc::now().date_naive(),
            notes: None,
            total_volume: Decimal::new(1000, 0),
            peak_weight: Decimal::new(50, 0),
            created_at: Utc::now(),
            updated_at: Utc::now(),
        };
        
        // 测试所有者访问
        assert!(verify_resource_ownership(user_id, &training).await.is_ok());
        
        // 测试非所有者访问
        assert!(verify_resource_ownership(other_user_id, &training).await.is_err());
    }
}
```

## 10. 未来扩展

### 10.1 权限系统扩展
- 支持更细粒度的权限控制
- 实现权限组和权限继承
- 支持动态权限配置

### 10.2 安全增强
- 实现多因素认证
- 支持OAuth2集成
- 添加API密钥管理 
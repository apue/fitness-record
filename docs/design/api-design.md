# Fitness Record API设计文档

## 1. API概述

### 1.1 设计原则
- 遵循RESTful API设计规范
- 使用HTTP状态码正确表示响应状态
- 统一的JSON响应格式
- 支持API版本控制
- 完整的错误处理机制

### 1.2 基础信息
- **基础URL**: `https://fitness-record.shuttle.app/api/v1`
- **认证方式**: Bearer Token (JWT)
- **数据格式**: JSON
- **字符编码**: UTF-8

### 1.3 通用响应格式
```json
{
  "success": true,
  "data": {},
  "message": "操作成功",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

### 1.4 错误响应格式
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "参数验证失败",
    "details": {}
  },
  "timestamp": "2024-01-01T00:00:00Z"
}
```

## 2. 认证API

### 2.1 用户注册
**POST** `/auth/register`

**请求体**:
```json
{
  "username": "string",
  "email": "string",
  "password": "string"
}
```

**响应**:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "username": "string",
      "email": "string",
      "created_at": "2024-01-01T00:00:00Z"
    },
    "token": "jwt_token_string"
  },
  "message": "注册成功"
}
```

### 2.2 用户登录
**POST** `/auth/login`

**请求体**:
```json
{
  "email": "string",
  "password": "string"
}
```

**响应**:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "username": "string",
      "email": "string"
    },
    "token": "jwt_token_string",
    "expires_at": "2024-01-01T00:00:00Z"
  },
  "message": "登录成功"
}
```

### 2.3 刷新Token
**POST** `/auth/refresh`

**请求头**:
```
Authorization: Bearer <refresh_token>
```

**响应**:
```json
{
  "success": true,
  "data": {
    "token": "new_jwt_token_string",
    "expires_at": "2024-01-01T00:00:00Z"
  },
  "message": "Token刷新成功"
}
```

### 2.4 用户登出
**POST** `/auth/logout`

**请求头**:
```
Authorization: Bearer <token>
```

**响应**:
```json
{
  "success": true,
  "message": "登出成功"
}
```

## 3. 用户管理API

### 3.1 获取用户信息
**GET** `/users/me`

**请求头**:
```
Authorization: Bearer <token>
```

**响应**:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "username": "string",
      "email": "string",
      "avatar_url": "string",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  }
}
```

### 3.2 更新用户信息
**PUT** `/users/me`

**请求头**:
```
Authorization: Bearer <token>
```

**请求体**:
```json
{
  "username": "string",
  "avatar_url": "string"
}
```

**响应**:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "username": "string",
      "email": "string",
      "avatar_url": "string",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  },
  "message": "用户信息更新成功"
}
```

## 4. 训练记录API

### 4.1 创建训练
**POST** `/trainings`

**请求头**:
```
Authorization: Bearer <token>
```

**请求体**:
```json
{
  "date": "2024-01-01",
  "notes": "string",
  "fatigue_level": 7,
  "exercises": [
    {
      "name": "坐姿划船",
      "weight": 50.0,
      "reps": 12,
      "sets": 3,
      "rest_time": 60,
      "notes": "string"
    }
  ]
}
```

**响应**:
```json
{
  "success": true,
  "data": {
    "training": {
      "id": "uuid",
      "user_id": "uuid",
      "date": "2024-01-01",
      "notes": "string",
      "total_volume": 1800.0,
      "peak_weight": 50.0,
      "fatigue_level": 7,
      "exercises": [],
      "created_at": "2024-01-01T00:00:00Z"
    }
  },
  "message": "训练记录创建成功"
}
```

### 4.2 获取训练列表
**GET** `/trainings`

**请求头**:
```
Authorization: Bearer <token>
```

**查询参数**:
- `page`: 页码 (默认: 1)
- `limit`: 每页数量 (默认: 20)
- `start_date`: 开始日期 (可选)
- `end_date`: 结束日期 (可选)
- `exercise_name`: 训练项目名称 (可选)

**响应**:
```json
{
  "success": true,
  "data": {
    "trainings": [
      {
        "id": "uuid",
        "date": "2024-01-01",
        "notes": "string",
        "total_volume": 1800.0,
        "peak_weight": 50.0,
        "fatigue_level": 7,
        "exercise_count": 3,
        "created_at": "2024-01-01T00:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 100,
      "total_pages": 5
    }
  }
}
```

### 4.3 获取训练详情
**GET** `/trainings/{training_id}`

**请求头**:
```
Authorization: Bearer <token>
```

**响应**:
```json
{
  "success": true,
  "data": {
    "training": {
              "id": "uuid",
        "date": "2024-01-01",
        "notes": "string",
        "total_volume": 1800.0,
        "peak_weight": 50.0,
        "fatigue_level": 7,
        "exercises": [
        {
          "id": "uuid",
          "name": "坐姿划船",
          "weight": 50.0,
          "reps": 12,
          "sets": 3,
          "rest_time": 60,
          "notes": "string"
        }
      ],
      "created_at": "2024-01-01T00:00:00Z"
    }
  }
}
```

### 4.4 更新训练
**PUT** `/trainings/{training_id}`

**请求头**:
```
Authorization: Bearer <token>
```

**请求体**:
```json
{
  "date": "2024-01-01",
  "notes": "string",
  "fatigue_level": 7,
  "exercises": []
}
```

**响应**:
```json
{
  "success": true,
  "data": {
    "training": {}
  },
  "message": "训练记录更新成功"
}
```

### 4.5 删除训练
**DELETE** `/trainings/{training_id}`

**请求头**:
```
Authorization: Bearer <token>
```

**响应**:
```json
{
  "success": true,
  "message": "训练记录删除成功"
}
```

## 5. 数据分析API

### 5.1 获取训练统计
**GET** `/analytics/statistics`

**请求头**:
```
Authorization: Bearer <token>
```

**查询参数**:
- `start_date`: 开始日期 (默认: 30天前)
- `end_date`: 结束日期 (默认: 今天)

**响应**:
```json
{
  "success": true,
  "data": {
    "total_trainings": 15,
    "total_volume": 27000.0,
    "average_volume": 1800.0,
    "peak_weight": 80.0,
    "average_fatigue_level": 6.8,
    "most_used_exercise": "坐姿划船",
    "training_frequency": {
      "weekly": 3.5,
      "monthly": 15
    }
  }
}
```

### 5.2 获取容量趋势
**GET** `/analytics/volume-trend`

**请求头**:
```
Authorization: Bearer <token>
```

**查询参数**:
- `start_date`: 开始日期
- `end_date`: 结束日期
- `exercise_name`: 训练项目名称 (可选)

**响应**:
```json
{
  "success": true,
  "data": {
    "trend": [
      {
        "date": "2024-01-01",
        "volume": 1800.0,
        "peak_weight": 50.0,
        "fatigue_level": 7,
        "training_count": 1
      }
    ]
  }
}
```

### 5.3 获取训练项目统计
**GET** `/analytics/exercises`

**请求头**:
```
Authorization: Bearer <token>
```

**查询参数**:
- `start_date`: 开始日期
- `end_date`: 结束日期

**响应**:
```json
{
  "success": true,
  "data": {
    "exercises": [
      {
        "name": "坐姿划船",
        "usage_count": 10,
        "total_volume": 6000.0,
        "peak_weight": 60.0,
        "average_weight": 50.0
      }
    ]
  }
}
```

## 6. 训练模板API

### 6.1 创建训练模板
**POST** `/templates`

**请求头**:
```
Authorization: Bearer <token>
```

**请求体**:
```json
{
  "name": "胸部训练",
  "description": "string",
  "exercises": [
    {
      "name": "哑铃推胸",
      "default_weight": 30.0,
      "default_reps": 10,
      "default_sets": 3,
      "default_rest_time": 90
    }
  ]
}
```

**响应**:
```json
{
  "success": true,
  "data": {
    "template": {
      "id": "uuid",
      "name": "胸部训练",
      "description": "string",
      "exercises": [],
      "created_at": "2024-01-01T00:00:00Z"
    }
  },
  "message": "训练模板创建成功"
}
```

### 6.2 获取模板列表
**GET** `/templates`

**请求头**:
```
Authorization: Bearer <token>
```

**响应**:
```json
{
  "success": true,
  "data": {
    "templates": [
      {
        "id": "uuid",
        "name": "胸部训练",
        "description": "string",
        "exercise_count": 3,
        "created_at": "2024-01-01T00:00:00Z"
      }
    ]
  }
}
```

### 6.3 应用模板到训练
**POST** `/trainings/from-template/{template_id}`

**请求头**:
```
Authorization: Bearer <token>
```

**请求体**:
```json
{
  "date": "2024-01-01",
  "notes": "string"
}
```

**响应**:
```json
{
  "success": true,
  "data": {
    "training": {}
  },
  "message": "训练记录创建成功"
}
```

## 7. 错误码定义

### 7.1 通用错误码
- `400`: 请求参数错误
- `401`: 未认证
- `403`: 无权限
- `404`: 资源不存在
- `422`: 数据验证失败
- `500`: 服务器内部错误

### 7.2 业务错误码
- `AUTH_INVALID_CREDENTIALS`: 用户名或密码错误
- `AUTH_TOKEN_EXPIRED`: Token已过期
- `AUTH_TOKEN_INVALID`: Token无效
- `USER_EMAIL_EXISTS`: 邮箱已存在
- `USER_NOT_FOUND`: 用户不存在
- `TRAINING_NOT_FOUND`: 训练记录不存在
- `TEMPLATE_NOT_FOUND`: 训练模板不存在

## 8. 限流策略

### 8.1 API限流
- 认证API: 10次/分钟
- 其他API: 100次/分钟
- 文件上传: 5次/分钟

### 8.2 限流响应
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "请求过于频繁，请稍后再试",
    "retry_after": 60
  }
}
```

## 9. 版本控制

### 9.1 版本策略
- 使用URL路径版本控制: `/api/v1/`
- 向后兼容性保证
- 废弃API提前通知

### 9.2 版本迁移
- 提供迁移指南
- 支持多版本并行运行
- 逐步淘汰旧版本 
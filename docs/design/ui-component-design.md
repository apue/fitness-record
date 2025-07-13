# Fitness Record 页面组件设计文档

## 1. 设计概述

### 1.1 设计原则
- **移动端优先**: 所有组件都优先考虑移动端体验
- **响应式设计**: 自适应不同屏幕尺寸
- **简洁易用**: 界面简洁，操作直观
- **性能优化**: 快速加载，流畅交互

### 1.2 技术栈
- **HTML5**: 语义化标签
- **CSS3**: Flexbox/Grid布局，CSS变量
- **JavaScript**: 原生JS，模块化设计
- **图表库**: Chart.js
- **图标**: Heroicons

### 1.3 设计系统
```css
:root {
  /* 颜色系统 */
  --primary-color: #3b82f6;
  --primary-dark: #2563eb;
  --secondary-color: #64748b;
  --success-color: #10b981;
  --warning-color: #f59e0b;
  --error-color: #ef4444;
  
  /* 中性色 */
  --gray-50: #f8fafc;
  --gray-100: #f1f5f9;
  --gray-200: #e2e8f0;
  --gray-300: #cbd5e1;
  --gray-400: #94a3b8;
  --gray-500: #64748b;
  --gray-600: #475569;
  --gray-700: #334155;
  --gray-800: #1e293b;
  --gray-900: #0f172a;
  
  /* 字体系统 */
  --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 1.875rem;
  
  /* 间距系统 */
  --spacing-1: 0.25rem;
  --spacing-2: 0.5rem;
  --spacing-3: 0.75rem;
  --spacing-4: 1rem;
  --spacing-6: 1.5rem;
  --spacing-8: 2rem;
  --spacing-12: 3rem;
  
  /* 圆角 */
  --radius-sm: 0.25rem;
  --radius-md: 0.375rem;
  --radius-lg: 0.5rem;
  --radius-xl: 0.75rem;
  
  /* 阴影 */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
}
```

## 2. 页面结构

### 2.1 应用布局
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fitness Record</title>
    <link rel="stylesheet" href="/css/main.css">
</head>
<body>
    <!-- 导航栏 -->
    <nav class="navbar">
        <div class="navbar-brand">
            <h1>Fitness Record</h1>
        </div>
        <div class="navbar-menu">
            <a href="/dashboard" class="nav-link">仪表板</a>
            <a href="/trainings" class="nav-link">训练记录</a>
            <a href="/analytics" class="nav-link">数据分析</a>
            <a href="/profile" class="nav-link">个人资料</a>
        </div>
        <div class="navbar-user">
            <button class="btn-user" id="userMenuBtn">
                <img src="/images/avatar.png" alt="用户头像" class="avatar">
            </button>
        </div>
    </nav>

    <!-- 主要内容区域 -->
    <main class="main-content">
        <div id="app"></div>
    </main>

    <!-- 移动端底部导航 -->
    <nav class="mobile-nav">
        <a href="/dashboard" class="mobile-nav-item">
            <svg class="icon"><!-- 仪表板图标 --></svg>
            <span>仪表板</span>
        </a>
        <a href="/trainings" class="mobile-nav-item">
            <svg class="icon"><!-- 训练图标 --></svg>
            <span>训练</span>
        </a>
        <a href="/analytics" class="mobile-nav-item">
            <svg class="icon"><!-- 分析图标 --></svg>
            <span>分析</span>
        </a>
        <a href="/profile" class="mobile-nav-item">
            <svg class="icon"><!-- 用户图标 --></svg>
            <span>我的</span>
        </a>
    </nav>

    <script src="/js/app.js" type="module"></script>
</body>
</html>
```

## 3. 核心组件

### 3.1 认证组件

#### 登录表单
```html
<div class="auth-container">
    <div class="auth-card">
        <h2 class="auth-title">登录</h2>
        <form class="auth-form" id="loginForm">
            <div class="form-group">
                <label for="email">邮箱</label>
                <input type="email" id="email" name="email" required>
            </div>
            <div class="form-group">
                <label for="password">密码</label>
                <input type="password" id="password" name="password" required>
            </div>
            <button type="submit" class="btn btn-primary btn-full">登录</button>
        </form>
        <div class="auth-links">
            <a href="/register">注册新账户</a>
            <a href="/forgot-password">忘记密码？</a>
        </div>
    </div>
</div>
```

#### 注册表单
```html
<div class="auth-container">
    <div class="auth-card">
        <h2 class="auth-title">注册</h2>
        <form class="auth-form" id="registerForm">
            <div class="form-group">
                <label for="username">用户名</label>
                <input type="text" id="username" name="username" required>
            </div>
            <div class="form-group">
                <label for="email">邮箱</label>
                <input type="email" id="email" name="email" required>
            </div>
            <div class="form-group">
                <label for="password">密码</label>
                <input type="password" id="password" name="password" required>
            </div>
            <div class="form-group">
                <label for="confirmPassword">确认密码</label>
                <input type="password" id="confirmPassword" name="confirmPassword" required>
            </div>
            <button type="submit" class="btn btn-primary btn-full">注册</button>
        </form>
        <div class="auth-links">
            <a href="/login">已有账户？登录</a>
        </div>
    </div>
</div>
```

### 3.2 训练记录组件

#### 新建训练按钮
```html
<div class="new-training-section">
    <button class="btn btn-primary btn-large" id="newTrainingBtn">
        <svg class="icon icon-lg"><!-- 加号图标 --></svg>
        <span>新建训练</span>
    </button>
</div>
```

#### 训练表单
```html
<div class="training-form" id="trainingForm">
    <div class="form-header">
        <h3>新建训练</h3>
        <button class="btn-close" id="closeForm">&times;</button>
    </div>
    
    <form>
        <div class="form-group">
            <label for="trainingDate">训练日期</label>
            <input type="date" id="trainingDate" name="date" required>
        </div>
        
        <div class="form-group">
            <label for="trainingNotes">备注</label>
            <textarea id="trainingNotes" name="notes" rows="3"></textarea>
        </div>
        
        <div class="exercises-section">
            <h4>训练项目</h4>
            <div class="exercises-list" id="exercisesList">
                <!-- 动态生成的训练项目 -->
            </div>
            <button type="button" class="btn btn-secondary" id="addExerciseBtn">
                <svg class="icon"><!-- 加号图标 --></svg>
                添加项目
            </button>
        </div>
        
        <div class="form-actions">
            <button type="submit" class="btn btn-primary">保存训练</button>
            <button type="button" class="btn btn-secondary" id="cancelBtn">取消</button>
        </div>
    </form>
</div>
```

#### 训练项目组件
```html
<div class="exercise-item" data-exercise-id="">
    <div class="exercise-header">
        <input type="text" class="exercise-name" placeholder="项目名称" required>
        <button class="btn-remove" data-action="remove-exercise">&times;</button>
    </div>
    
    <div class="exercise-details">
        <div class="form-row">
            <div class="form-group">
                <label>重量 (kg)</label>
                <input type="number" class="exercise-weight" min="0" step="0.5" required>
            </div>
            <div class="form-group">
                <label>次数</label>
                <input type="number" class="exercise-reps" min="1" required>
            </div>
            <div class="form-group">
                <label>组数</label>
                <input type="number" class="exercise-sets" min="1" required>
            </div>
        </div>
        
        <div class="form-row">
            <div class="form-group">
                <label>休息时间 (秒)</label>
                <input type="number" class="exercise-rest" min="0">
            </div>
            <div class="form-group">
                <label>备注</label>
                <input type="text" class="exercise-notes">
            </div>
        </div>
    </div>
</div>
```

### 3.3 训练列表组件

#### 训练记录卡片
```html
<div class="training-card" data-training-id="">
    <div class="training-header">
        <div class="training-date">
            <span class="date-day">15</span>
            <span class="date-month">一月</span>
        </div>
        <div class="training-stats">
            <div class="stat-item">
                <span class="stat-label">总容量</span>
                <span class="stat-value">1,800 kg</span>
            </div>
            <div class="stat-item">
                <span class="stat-label">峰值</span>
                <span class="stat-value">50 kg</span>
            </div>
        </div>
    </div>
    
    <div class="training-exercises">
        <div class="exercise-summary">
            <span class="exercise-name">坐姿划船</span>
            <span class="exercise-detail">50kg × 12 × 3</span>
        </div>
        <div class="exercise-summary">
            <span class="exercise-name">哑铃推胸</span>
            <span class="exercise-detail">30kg × 10 × 3</span>
        </div>
    </div>
    
    <div class="training-actions">
        <button class="btn btn-sm btn-secondary" data-action="view-training">查看详情</button>
        <button class="btn btn-sm btn-primary" data-action="edit-training">编辑</button>
        <button class="btn btn-sm btn-danger" data-action="delete-training">删除</button>
    </div>
</div>
```

### 3.4 数据分析组件

#### 统计卡片
```html
<div class="stats-grid">
    <div class="stat-card">
        <div class="stat-icon">
            <svg class="icon"><!-- 训练图标 --></svg>
        </div>
        <div class="stat-content">
            <h3 class="stat-value">15</h3>
            <p class="stat-label">本月训练次数</p>
        </div>
    </div>
    
    <div class="stat-card">
        <div class="stat-icon">
            <svg class="icon"><!-- 重量图标 --></svg>
        </div>
        <div class="stat-content">
            <h3 class="stat-value">27,000 kg</h3>
            <p class="stat-label">总训练容量</p>
        </div>
    </div>
    
    <div class="stat-card">
        <div class="stat-icon">
            <svg class="icon"><!-- 峰值图标 --></svg>
        </div>
        <div class="stat-content">
            <h3 class="stat-value">80 kg</h3>
            <p class="stat-label">峰值重量</p>
        </div>
    </div>
    
    <div class="stat-card">
        <div class="stat-icon">
            <svg class="icon"><!-- 频率图标 --></svg>
        </div>
        <div class="stat-content">
            <h3 class="stat-value">3.5</h3>
            <p class="stat-label">周训练频率</p>
        </div>
    </div>
</div>
```

#### 图表组件
```html
<div class="chart-container">
    <div class="chart-header">
        <h3>训练容量趋势</h3>
        <div class="chart-controls">
            <select class="chart-period" id="chartPeriod">
                <option value="7">最近7天</option>
                <option value="30" selected>最近30天</option>
                <option value="90">最近90天</option>
            </select>
        </div>
    </div>
    <div class="chart-wrapper">
        <canvas id="volumeChart"></canvas>
    </div>
</div>
```

### 3.5 模板组件

#### 模板选择器
```html
<div class="template-selector">
    <h4>使用训练模板</h4>
    <div class="template-list" id="templateList">
        <div class="template-item" data-template-id="">
            <div class="template-info">
                <h5 class="template-name">胸部训练</h5>
                <p class="template-description">包含哑铃推胸、俯卧撑等</p>
                <span class="template-exercise-count">3个项目</span>
            </div>
            <button class="btn btn-primary btn-sm" data-action="use-template">
                使用模板
            </button>
        </div>
    </div>
</div>
```

## 4. 响应式设计

### 4.1 断点定义
```css
/* 移动端 */
@media (max-width: 640px) {
    .container {
        padding: var(--spacing-4);
    }
    
    .navbar-menu {
        display: none;
    }
    
    .mobile-nav {
        display: flex;
    }
}

/* 平板 */
@media (min-width: 641px) and (max-width: 1024px) {
    .container {
        padding: var(--spacing-6);
    }
}

/* 桌面端 */
@media (min-width: 1025px) {
    .container {
        padding: var(--spacing-8);
    }
    
    .mobile-nav {
        display: none;
    }
}
```

### 4.2 移动端优化
```css
/* 触摸友好的按钮 */
.btn {
    min-height: 44px;
    padding: var(--spacing-3) var(--spacing-4);
}

/* 移动端表单优化 */
.form-group input,
.form-group textarea {
    font-size: 16px; /* 防止iOS缩放 */
}

/* 移动端导航 */
.mobile-nav {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    background: white;
    border-top: 1px solid var(--gray-200);
    padding: var(--spacing-2);
    z-index: 1000;
}

.mobile-nav-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: var(--spacing-2);
    text-decoration: none;
    color: var(--gray-600);
    font-size: var(--font-size-xs);
}

.mobile-nav-item.active {
    color: var(--primary-color);
}
```

## 5. 交互设计

### 5.1 加载状态
```html
<div class="loading-spinner" id="loadingSpinner">
    <div class="spinner"></div>
    <p>加载中...</p>
</div>
```

### 5.2 错误处理
```html
<div class="error-message" id="errorMessage">
    <svg class="icon icon-error"><!-- 错误图标 --></svg>
    <p class="error-text">操作失败，请重试</p>
    <button class="btn btn-primary" onclick="retry()">重试</button>
</div>
```

### 5.3 成功反馈
```html
<div class="success-toast" id="successToast">
    <svg class="icon icon-success"><!-- 成功图标 --></svg>
    <span class="toast-message">操作成功</span>
</div>
```

## 6. 性能优化

### 6.1 图片优化
```html
<!-- 响应式图片 -->
<img src="/images/avatar-small.jpg" 
     srcset="/images/avatar-small.jpg 1x, /images/avatar-large.jpg 2x"
     alt="用户头像"
     loading="lazy">
```

### 6.2 代码分割
```javascript
// 按需加载组件
const loadChartComponent = async () => {
    const { Chart } = await import('./components/chart.js');
    return Chart;
};
```

### 6.3 缓存策略
```javascript
// 本地存储缓存
const cache = {
    set: (key, data, ttl = 3600000) => {
        const item = {
            data,
            timestamp: Date.now(),
            ttl
        };
        localStorage.setItem(key, JSON.stringify(item));
    },
    
    get: (key) => {
        const item = localStorage.getItem(key);
        if (!item) return null;
        
        const { data, timestamp, ttl } = JSON.parse(item);
        if (Date.now() - timestamp > ttl) {
            localStorage.removeItem(key);
            return null;
        }
        
        return data;
    }
};
```

## 7. 无障碍设计

### 7.1 语义化标签
```html
<main role="main">
    <section aria-labelledby="training-heading">
        <h2 id="training-heading">训练记录</h2>
        <!-- 内容 -->
    </section>
</main>
```

### 7.2 键盘导航
```css
/* 焦点样式 */
.btn:focus,
input:focus,
textarea:focus {
    outline: 2px solid var(--primary-color);
    outline-offset: 2px;
}
```

### 7.3 屏幕阅读器支持
```html
<button aria-label="添加训练项目" class="btn btn-secondary">
    <svg class="icon" aria-hidden="true"><!-- 加号图标 --></svg>
    <span>添加项目</span>
</button>
```

## 8. 测试策略

### 8.1 组件测试
```javascript
// 测试训练表单组件
describe('TrainingForm', () => {
    test('应该正确验证表单数据', () => {
        const form = new TrainingForm();
        const data = {
            date: '2024-01-15',
            exercises: [
                { name: '坐姿划船', weight: 50, reps: 12, sets: 3 }
            ]
        };
        
        expect(form.validate(data)).toBe(true);
    });
});
```

### 8.2 响应式测试
- 使用浏览器开发者工具测试不同屏幕尺寸
- 测试触摸设备交互
- 验证键盘导航功能

### 8.3 性能测试
- 使用Lighthouse进行性能审计
- 测试页面加载时间
- 验证内存使用情况 
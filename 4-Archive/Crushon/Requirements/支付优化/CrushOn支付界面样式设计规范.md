# CrushOn.AI 支付界面样式设计规范

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**文档类型**: UI设计规范  
**设计负责人**: UI设计团队  
**品牌风格**: 现代、可信、友好

## 一、设计系统概览

### 1.1 设计原则

| 原则 | 描述 | 视觉体现 |
|-----|------|----------|
| **清晰性** | 信息层级分明，重点突出 | 标题、正文、辅助文字的大小对比 |
| **一致性** | 统一的视觉语言和交互模式 | 组件样式、颜色、间距标准化 |
| **现代感** | 时尚简约的视觉风格 | 扁平化设计、渐变色、圆角 |
| **可信度** | 营造安全可靠的支付环境 | 安全图标、认证标识、专业配色 |
| **友好性** | 温暖亲切的用户体验 | 柔和的色彩、圆润的形状、积极的反馈 |

### 1.2 设计栅格

```css
/* 响应式栅格系统 */
:root {
  --grid-columns: 12;
  --grid-gutter: 24px;
  --container-padding: 16px;
  
  /* 断点定义 */
  --breakpoint-xs: 375px;
  --breakpoint-sm: 768px;
  --breakpoint-md: 1024px;
  --breakpoint-lg: 1440px;
  --breakpoint-xl: 1920px;
}

/* 容器宽度 */
.container {
  width: 100%;
  margin: 0 auto;
  padding: 0 var(--container-padding);
  
  @media (min-width: 768px) {
    max-width: 720px;
  }
  
  @media (min-width: 1024px) {
    max-width: 960px;
  }
  
  @media (min-width: 1440px) {
    max-width: 1200px;
  }
}
```

## 二、颜色系统

### 2.1 品牌色板

```css
:root {
  /* 主色 - 紫色渐变 */
  --primary-gradient: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  --primary-500: #667eea;
  --primary-600: #5a67d8;
  --primary-400: #7f9cf5;
  
  /* 辅助色 - 成功/警告/错误 */
  --success-500: #48bb78;
  --success-100: #f0fff4;
  --warning-500: #ed8936;
  --warning-100: #fffdf7;
  --error-500: #f56565;
  --error-100: #fff5f5;
  
  /* 中性色 */
  --gray-900: #1a202c;
  --gray-800: #2d3748;
  --gray-700: #4a5568;
  --gray-600: #718096;
  --gray-500: #a0aec0;
  --gray-400: #cbd5e0;
  --gray-300: #e2e8f0;
  --gray-200: #edf2f7;
  --gray-100: #f7fafc;
  --white: #ffffff;
  
  /* 特殊用途色 */
  --background: #f8f9fc;
  --surface: #ffffff;
  --text-primary: var(--gray-900);
  --text-secondary: var(--gray-600);
  --text-muted: var(--gray-500);
  --border: var(--gray-300);
}

/* 暗色模式 */
@media (prefers-color-scheme: dark) {
  :root {
    --background: #1a202c;
    --surface: #2d3748;
    --text-primary: #f7fafc;
    --text-secondary: #cbd5e0;
    --text-muted: #a0aec0;
    --border: #4a5568;
  }
}
```

### 2.2 颜色使用规范

| 颜色类型 | 使用场景 | 示例 |
|---------|---------|------|
| **主色调** | CTA按钮、链接、选中状态 | 支付按钮、套餐选中边框 |
| **成功色** | 成功提示、正向反馈 | 支付成功、验证通过 |
| **警告色** | 警告信息、重要提示 | 优惠即将过期、账户余额不足 |
| **错误色** | 错误提示、验证失败 | 支付失败、表单错误 |
| **中性色** | 文字、边框、背景 | 正文、分割线、卡片背景 |

## 三、字体系统

### 3.1 字体族定义

```css
:root {
  /* 主字体栈 */
  --font-family-base: -apple-system, BlinkMacSystemFont, "Segoe UI", 
                       "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei",
                       "Helvetica Neue", Helvetica, Arial, sans-serif;
  
  /* 等宽字体 */
  --font-family-mono: "SF Mono", Monaco, "Inconsolata", "Fira Code", 
                       "Courier New", monospace;
  
  /* 数字字体 */
  --font-family-number: "Roboto", "DIN Pro", var(--font-family-base);
}
```

### 3.2 字体大小和行高

```css
:root {
  /* 字体大小 */
  --text-xs: 12px;
  --text-sm: 14px;
  --text-base: 16px;
  --text-lg: 18px;
  --text-xl: 20px;
  --text-2xl: 24px;
  --text-3xl: 30px;
  --text-4xl: 36px;
  --text-5xl: 48px;
  
  /* 行高 */
  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;
  --leading-loose: 2;
  
  /* 字重 */
  --font-light: 300;
  --font-normal: 400;
  --font-medium: 500;
  --font-semibold: 600;
  --font-bold: 700;
}
```

### 3.3 文字样式定义

```css
/* 标题样式 */
.h1 {
  font-size: var(--text-4xl);
  font-weight: var(--font-bold);
  line-height: var(--leading-tight);
  color: var(--text-primary);
  letter-spacing: -0.02em;
}

.h2 {
  font-size: var(--text-3xl);
  font-weight: var(--font-semibold);
  line-height: var(--leading-tight);
  color: var(--text-primary);
  letter-spacing: -0.01em;
}

.h3 {
  font-size: var(--text-2xl);
  font-weight: var(--font-semibold);
  line-height: var(--leading-normal);
  color: var(--text-primary);
}

/* 正文样式 */
.body-large {
  font-size: var(--text-lg);
  line-height: var(--leading-relaxed);
  color: var(--text-primary);
}

.body-regular {
  font-size: var(--text-base);
  line-height: var(--leading-normal);
  color: var(--text-primary);
}

.body-small {
  font-size: var(--text-sm);
  line-height: var(--leading-normal);
  color: var(--text-secondary);
}

/* 价格显示 */
.price {
  font-family: var(--font-family-number);
  font-weight: var(--font-bold);
  font-feature-settings: "tnum";
  
  &.price-large {
    font-size: var(--text-4xl);
  }
  
  &.price-regular {
    font-size: var(--text-2xl);
  }
  
  &.price-small {
    font-size: var(--text-lg);
  }
}
```

## 四、间距系统

### 4.1 间距规范

```css
:root {
  /* 基础间距单位 */
  --spacing-base: 8px;
  
  /* 间距尺寸 */
  --spacing-0: 0;
  --spacing-1: calc(var(--spacing-base) * 0.5);   /* 4px */
  --spacing-2: var(--spacing-base);                /* 8px */
  --spacing-3: calc(var(--spacing-base) * 1.5);   /* 12px */
  --spacing-4: calc(var(--spacing-base) * 2);     /* 16px */
  --spacing-5: calc(var(--spacing-base) * 2.5);   /* 20px */
  --spacing-6: calc(var(--spacing-base) * 3);     /* 24px */
  --spacing-8: calc(var(--spacing-base) * 4);     /* 32px */
  --spacing-10: calc(var(--spacing-base) * 5);    /* 40px */
  --spacing-12: calc(var(--spacing-base) * 6);    /* 48px */
  --spacing-16: calc(var(--spacing-base) * 8);    /* 64px */
  --spacing-20: calc(var(--spacing-base) * 10);   /* 80px */
  --spacing-24: calc(var(--spacing-base) * 12);   /* 96px */
}
```

### 4.2 组件间距规范

| 组件类型 | 内边距 | 外边距 | 元素间距 |
|---------|--------|--------|----------|
| **卡片** | 24px | 16px底部 | 内部元素16px |
| **按钮** | 16px 32px | 8px | - |
| **输入框** | 12px 16px | 16px底部 | 标签8px |
| **模态框** | 32px | - | 内容区24px |
| **列表项** | 16px | - | 项目间8px |

## 五、组件样式

### 5.1 按钮组件

```css
/* 基础按钮样式 */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: var(--spacing-3) var(--spacing-6);
  font-size: var(--text-base);
  font-weight: var(--font-medium);
  line-height: 1;
  border-radius: 8px;
  cursor: pointer;
  transition: all 200ms ease;
  outline: none;
  border: 1px solid transparent;
  
  &:focus-visible {
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  }
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

/* 主按钮 */
.btn-primary {
  background: var(--primary-gradient);
  color: white;
  
  &:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(102, 126, 234, 0.25);
  }
  
  &:active:not(:disabled) {
    transform: translateY(0);
  }
}

/* 次要按钮 */
.btn-secondary {
  background: var(--gray-100);
  color: var(--text-primary);
  border-color: var(--border);
  
  &:hover:not(:disabled) {
    background: var(--gray-200);
  }
}

/* 文字按钮 */
.btn-text {
  background: transparent;
  color: var(--primary-500);
  padding: var(--spacing-2) var(--spacing-3);
  
  &:hover:not(:disabled) {
    background: rgba(102, 126, 234, 0.08);
  }
}

/* 按钮尺寸 */
.btn-small {
  padding: var(--spacing-2) var(--spacing-4);
  font-size: var(--text-sm);
}

.btn-large {
  padding: var(--spacing-4) var(--spacing-8);
  font-size: var(--text-lg);
  border-radius: 12px;
}

/* 按钮状态 */
.btn-loading {
  position: relative;
  color: transparent;
  
  &::after {
    content: '';
    position: absolute;
    width: 20px;
    height: 20px;
    top: 50%;
    left: 50%;
    margin: -10px 0 0 -10px;
    border: 2px solid white;
    border-top-color: transparent;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

### 5.2 输入框组件

```css
/* 输入框容器 */
.input-group {
  margin-bottom: var(--spacing-4);
}

/* 标签 */
.input-label {
  display: block;
  font-size: var(--text-sm);
  font-weight: var(--font-medium);
  color: var(--text-primary);
  margin-bottom: var(--spacing-2);
  
  &.required::after {
    content: '*';
    color: var(--error-500);
    margin-left: 4px;
  }
}

/* 输入框 */
.input {
  width: 100%;
  padding: var(--spacing-3) var(--spacing-4);
  font-size: var(--text-base);
  color: var(--text-primary);
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 8px;
  transition: all 200ms ease;
  
  &::placeholder {
    color: var(--text-muted);
  }
  
  &:hover {
    border-color: var(--gray-400);
  }
  
  &:focus {
    outline: none;
    border-color: var(--primary-500);
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  }
  
  &.input-error {
    border-color: var(--error-500);
    
    &:focus {
      box-shadow: 0 0 0 3px rgba(245, 101, 101, 0.1);
    }
  }
  
  &.input-success {
    border-color: var(--success-500);
    
    &:focus {
      box-shadow: 0 0 0 3px rgba(72, 187, 120, 0.1);
    }
  }
}

/* 帮助文字 */
.input-help {
  font-size: var(--text-sm);
  color: var(--text-muted);
  margin-top: var(--spacing-2);
}

/* 错误提示 */
.input-error-message {
  font-size: var(--text-sm);
  color: var(--error-500);
  margin-top: var(--spacing-2);
  display: flex;
  align-items: center;
  
  &::before {
    content: '⚠';
    margin-right: var(--spacing-1);
  }
}

/* 特殊输入框 - 信用卡 */
.input-credit-card {
  font-family: var(--font-family-mono);
  letter-spacing: 0.1em;
  
  &.with-icon {
    padding-left: var(--spacing-12);
    background-image: var(--card-icon);
    background-repeat: no-repeat;
    background-position: var(--spacing-4) center;
    background-size: 24px;
  }
}
```

### 5.3 卡片组件

```css
/* 基础卡片 */
.card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: var(--spacing-6);
  transition: all 200ms ease;
  
  &.card-hover:hover {
    transform: translateY(-4px);
    box-shadow: 0 12px 24px rgba(0, 0, 0, 0.08);
  }
}

/* 套餐卡片 */
.plan-card {
  position: relative;
  padding: var(--spacing-8);
  text-align: center;
  
  &.recommended {
    border-color: var(--primary-500);
    box-shadow: 0 0 0 2px var(--primary-500);
    
    &::before {
      content: '推荐';
      position: absolute;
      top: -12px;
      left: 50%;
      transform: translateX(-50%);
      background: var(--primary-gradient);
      color: white;
      padding: 4px 16px;
      border-radius: 20px;
      font-size: var(--text-sm);
      font-weight: var(--font-semibold);
    }
  }
  
  .plan-name {
    font-size: var(--text-2xl);
    font-weight: var(--font-bold);
    margin-bottom: var(--spacing-2);
  }
  
  .plan-price {
    font-size: var(--text-5xl);
    font-weight: var(--font-bold);
    background: var(--primary-gradient);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    margin-bottom: var(--spacing-4);
    
    .currency {
      font-size: var(--text-2xl);
      vertical-align: super;
    }
    
    .period {
      font-size: var(--text-lg);
      color: var(--text-secondary);
      -webkit-text-fill-color: initial;
    }
  }
  
  .plan-features {
    list-style: none;
    padding: 0;
    margin: var(--spacing-6) 0;
    
    li {
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: var(--spacing-3);
      color: var(--text-secondary);
      
      &::before {
        content: '✓';
        color: var(--success-500);
        margin-right: var(--spacing-2);
        font-weight: var(--font-bold);
      }
    }
  }
}

/* 支付方式卡片 */
.payment-method-card {
  display: flex;
  align-items: center;
  padding: var(--spacing-4);
  border: 2px solid var(--border);
  border-radius: 8px;
  cursor: pointer;
  transition: all 200ms ease;
  
  &:hover {
    border-color: var(--gray-400);
    background: var(--gray-50);
  }
  
  &.selected {
    border-color: var(--primary-500);
    background: rgba(102, 126, 234, 0.05);
  }
  
  .payment-icon {
    width: 48px;
    height: 32px;
    margin-right: var(--spacing-4);
    object-fit: contain;
  }
  
  .payment-info {
    flex: 1;
    
    .payment-name {
      font-weight: var(--font-medium);
      margin-bottom: var(--spacing-1);
    }
    
    .payment-description {
      font-size: var(--text-sm);
      color: var(--text-secondary);
    }
  }
  
  .payment-badge {
    padding: 4px 12px;
    background: var(--success-100);
    color: var(--success-500);
    border-radius: 20px;
    font-size: var(--text-xs);
    font-weight: var(--font-semibold);
  }
}
```

### 5.4 模态框组件

```css
/* 模态框遮罩 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 200ms ease;
}

/* 模态框容器 */
.modal {
  background: var(--surface);
  border-radius: 16px;
  max-width: 480px;
  width: 90%;
  max-height: 90vh;
  overflow: auto;
  animation: slideUp 300ms ease;
  
  .modal-header {
    padding: var(--spacing-6);
    border-bottom: 1px solid var(--border);
    
    .modal-title {
      font-size: var(--text-xl);
      font-weight: var(--font-semibold);
      color: var(--text-primary);
    }
    
    .modal-close {
      position: absolute;
      top: var(--spacing-4);
      right: var(--spacing-4);
      width: 32px;
      height: 32px;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 50%;
      background: var(--gray-100);
      cursor: pointer;
      transition: all 200ms ease;
      
      &:hover {
        background: var(--gray-200);
      }
    }
  }
  
  .modal-body {
    padding: var(--spacing-6);
  }
  
  .modal-footer {
    padding: var(--spacing-6);
    border-top: 1px solid var(--border);
    display: flex;
    justify-content: flex-end;
    gap: var(--spacing-3);
  }
}

/* 动画 */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

### 5.5 进度指示器

```css
/* 线性进度条 */
.progress-bar {
  height: 8px;
  background: var(--gray-200);
  border-radius: 4px;
  overflow: hidden;
  
  .progress-fill {
    height: 100%;
    background: var(--primary-gradient);
    border-radius: 4px;
    transition: width 300ms ease;
    position: relative;
    
    &::after {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: linear-gradient(
        90deg,
        transparent,
        rgba(255, 255, 255, 0.3),
        transparent
      );
      animation: shimmer 2s infinite;
    }
  }
}

/* 环形进度条 */
.progress-circle {
  --size: 120px;
  --stroke: 8px;
  
  width: var(--size);
  height: var(--size);
  position: relative;
  
  svg {
    width: 100%;
    height: 100%;
    transform: rotate(-90deg);
    
    circle {
      fill: none;
      stroke-width: var(--stroke);
      
      &.progress-bg {
        stroke: var(--gray-200);
      }
      
      &.progress-fill {
        stroke: url(#gradient);
        stroke-linecap: round;
        stroke-dasharray: var(--circumference);
        stroke-dashoffset: var(--offset);
        transition: stroke-dashoffset 300ms ease;
      }
    }
  }
  
  .progress-text {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: var(--text-2xl);
    font-weight: var(--font-bold);
    color: var(--primary-500);
  }
}

/* 步骤指示器 */
.steps {
  display: flex;
  align-items: center;
  justify-content: space-between;
  
  .step {
    flex: 1;
    text-align: center;
    position: relative;
    
    &:not(:last-child)::after {
      content: '';
      position: absolute;
      top: 20px;
      left: 50%;
      width: 100%;
      height: 2px;
      background: var(--gray-300);
      z-index: -1;
    }
    
    &.completed::after {
      background: var(--primary-500);
    }
    
    .step-circle {
      width: 40px;
      height: 40px;
      margin: 0 auto var(--spacing-2);
      border-radius: 50%;
      background: var(--gray-200);
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: var(--font-semibold);
      color: var(--text-secondary);
      transition: all 200ms ease;
      
      &.active {
        background: var(--primary-gradient);
        color: white;
        transform: scale(1.1);
      }
      
      &.completed {
        background: var(--success-500);
        color: white;
        
        &::before {
          content: '✓';
        }
      }
    }
    
    .step-label {
      font-size: var(--text-sm);
      color: var(--text-secondary);
      
      &.active {
        color: var(--text-primary);
        font-weight: var(--font-medium);
      }
    }
  }
}

@keyframes shimmer {
  to {
    transform: translateX(100%);
  }
}
```

## 六、动效规范

### 6.1 动效原则

| 原则 | 说明 | 时长建议 |
|-----|------|----------|
| **自然流畅** | 动效要符合物理规律 | 200-400ms |
| **目的明确** | 动效要服务于功能 | 根据重要性调整 |
| **性能优先** | 避免影响页面性能 | 优先使用transform |
| **可访问性** | 支持减少动效选项 | prefers-reduced-motion |

### 6.2 常用动效

```css
/* 缓动函数 */
:root {
  --ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ease-out: cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
  --ease-bounce: cubic-bezier(0.68, -0.55, 0.265, 1.55);
}

/* 淡入淡出 */
.fade-enter {
  opacity: 0;
}

.fade-enter-active {
  opacity: 1;
  transition: opacity 200ms var(--ease-out);
}

/* 滑动进入 */
.slide-enter {
  opacity: 0;
  transform: translateY(20px);
}

.slide-enter-active {
  opacity: 1;
  transform: translateY(0);
  transition: all 300ms var(--ease-out);
}

/* 缩放效果 */
.scale-enter {
  opacity: 0;
  transform: scale(0.9);
}

.scale-enter-active {
  opacity: 1;
  transform: scale(1);
  transition: all 200ms var(--ease-out);
}

/* 成功动画 */
@keyframes success-bounce {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.success-animation {
  animation: success-bounce 500ms var(--ease-bounce);
}
```

## 七、图标系统

### 7.1 图标规范

```css
/* 图标尺寸 */
.icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  
  &.icon-xs { width: 16px; height: 16px; }
  &.icon-sm { width: 20px; height: 20px; }
  &.icon-md { width: 24px; height: 24px; }
  &.icon-lg { width: 32px; height: 32px; }
  &.icon-xl { width: 48px; height: 48px; }
}

/* 图标颜色 */
.icon-primary { color: var(--primary-500); }
.icon-success { color: var(--success-500); }
.icon-warning { color: var(--warning-500); }
.icon-error { color: var(--error-500); }
.icon-muted { color: var(--text-muted); }
```

### 7.2 常用图标定义

```html
<!-- SVG Sprite -->
<svg style="display: none;">
  <!-- 支付成功 -->
  <symbol id="icon-success" viewBox="0 0 24 24">
    <circle cx="12" cy="12" r="10" fill="currentColor" opacity="0.1"/>
    <path d="M9 12l2 2 4-4" stroke="currentColor" stroke-width="2" 
          stroke-linecap="round" stroke-linejoin="round" fill="none"/>
  </symbol>
  
  <!-- 安全锁 -->
  <symbol id="icon-lock" viewBox="0 0 24 24">
    <rect x="5" y="11" width="14" height="10" rx="2" 
          fill="currentColor" opacity="0.1"/>
    <path d="M7 11V7a5 5 0 0110 0v4M5 11h14a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2z" 
          stroke="currentColor" stroke-width="2" fill="none"/>
  </symbol>
  
  <!-- 信用卡 -->
  <symbol id="icon-credit-card" viewBox="0 0 24 24">
    <rect x="2" y="5" width="20" height="14" rx="2" 
          fill="currentColor" opacity="0.1"/>
    <path d="M2 10h20M8 14h2M14 14h4" 
          stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
  </symbol>
</svg>

<!-- 使用图标 -->
<svg class="icon icon-md icon-success">
  <use href="#icon-success"></use>
</svg>
```

## 八、响应式设计

### 8.1 移动端适配

```css
/* 移动端优化 */
@media (max-width: 768px) {
  /* 字体大小调整 */
  .h1 { font-size: var(--text-3xl); }
  .h2 { font-size: var(--text-2xl); }
  .h3 { font-size: var(--text-xl); }
  
  /* 间距调整 */
  .container {
    padding: 0 var(--spacing-4);
  }
  
  .card {
    padding: var(--spacing-4);
    border-radius: 8px;
  }
  
  /* 按钮全宽 */
  .btn {
    width: 100%;
    justify-content: center;
  }
  
  /* 套餐卡片垂直排列 */
  .plan-cards {
    flex-direction: column;
    gap: var(--spacing-4);
  }
  
  /* 模态框全屏 */
  .modal {
    width: 100%;
    height: 100%;
    max-width: none;
    max-height: none;
    border-radius: 0;
  }
}

/* 平板端优化 */
@media (min-width: 768px) and (max-width: 1024px) {
  .plan-cards {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: var(--spacing-4);
  }
}
```

### 8.2 触摸优化

```css
/* 触摸目标最小尺寸 */
@media (pointer: coarse) {
  .btn,
  .input,
  .payment-method-card {
    min-height: 44px;
  }
  
  /* 增加点击区域 */
  .checkbox-label,
  .radio-label {
    padding: var(--spacing-3);
    margin: calc(var(--spacing-3) * -1);
  }
  
  /* 禁用悬停效果 */
  @media (hover: none) {
    .btn:hover,
    .card:hover {
      transform: none;
      box-shadow: none;
    }
  }
}
```

## 九、主题定制

### 9.1 CSS变量覆盖

```css
/* 主题定制示例 */
.theme-custom {
  /* 覆盖品牌色 */
  --primary-500: #8b5cf6;
  --primary-gradient: linear-gradient(135deg, #8b5cf6 0%, #ec4899 100%);
  
  /* 覆盖圆角 */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  
  /* 覆盖阴影 */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.1);
}
```

---

**文档状态**: ✅ 已完成  
**下一步**: 开始组件设计文档
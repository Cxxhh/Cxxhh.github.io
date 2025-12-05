# 嵌入式动画网页生成规范

## 一、概述

本文档规定了生成可无缝嵌入博客文章的交互式动画网页时必须遵守的规范。这些规则确保动画在iframe环境中能够正常工作，并保持良好的用户体验。

## 二、核心原则

1. **iframe兼容性**：动画必须能在iframe中正常运行
2. **自包含性**：动画应独立运行，不依赖外部资源（CDN除外）
3. **响应式设计**：适配不同屏幕尺寸
4. **性能优化**：避免阻塞主页面渲染
5. **交互友好**：确保所有交互功能在iframe中可用

## 三、禁止使用的API和特性

### 3.1 窗口和文档操作（❌ 禁止）

| API/特性 | 原因 | 替代方案 |
|----------|------|----------|
| `window.open()` | iframe中可能被阻止 | 使用内部模态框或提示 |
| `window.parent` | 跨域限制 | 避免访问父窗口 |
| `window.top` | 跨域限制 | 避免访问顶层窗口 |
| `window.location` | 会导航整个页面 | 使用 `history.pushState()` 或内部状态管理 |
| `document.write()` | 阻塞渲染，iframe中可能失效 | 使用DOM操作 |
| `document.domain` | 安全限制 | 避免使用 |

### 3.2 存储API（⚠️ 限制使用）

| API | 限制说明 | 替代方案 |
|-----|----------|----------|
| `localStorage` | 可能被浏览器阻止 | 使用内存状态，或添加try-catch |
| `sessionStorage` | 可能被浏览器阻止 | 使用内存状态 |
| `IndexedDB` | 权限问题 | 仅在必要时使用，添加错误处理 |
| `Cookies` | 第三方cookie限制 | 避免使用 |

**推荐做法：**
```javascript
// ❌ 错误：直接使用localStorage
localStorage.setItem('key', 'value');

// ✅ 正确：添加错误处理
try {
    localStorage.setItem('key', 'value');
} catch (e) {
    // 使用内存状态作为后备
    memoryState.key = 'value';
}
```

### 3.3 网络请求（⚠️ 限制使用）

| API | 限制说明 | 替代方案 |
|-----|----------|----------|
| `fetch()` 跨域请求 | CORS限制 | 使用JSONP或确保服务器支持CORS |
| `XMLHttpRequest` 跨域 | CORS限制 | 同上 |
| `WebSocket` | 可能被防火墙阻止 | 添加连接失败处理 |

**推荐做法：**
```javascript
// ✅ 正确：添加错误处理
fetch('/api/data')
    .then(response => response.json())
    .catch(error => {
        console.warn('网络请求失败，使用默认数据');
        return defaultData;
    });
```

### 3.4 全屏API（❌ 禁止）

| API | 原因 | 替代方案 |
|-----|------|----------|
| `element.requestFullscreen()` | iframe中需要用户手势，且可能被阻止 | 使用全屏模式CSS类 |
| `document.exitFullscreen()` | 同上 | 使用CSS控制 |

**替代实现：**
```javascript
// ❌ 错误
element.requestFullscreen();

// ✅ 正确：使用CSS全屏模式
element.classList.add('fullscreen-mode');
// CSS: .fullscreen-mode { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; z-index: 9999; }
```

### 3.5 剪贴板API（⚠️ 限制使用）

| API | 限制说明 | 替代方案 |
|-----|----------|----------|
| `navigator.clipboard.writeText()` | 需要用户手势，可能被阻止 | 使用 `document.execCommand('copy')` 或提示用户手动复制 |
| `navigator.clipboard.readText()` | 权限问题 | 使用输入框让用户粘贴 |

**推荐做法：**
```javascript
// ✅ 正确：添加降级方案
async function copyToClipboard(text) {
    try {
        await navigator.clipboard.writeText(text);
    } catch (e) {
        // 降级方案：使用execCommand
        const textarea = document.createElement('textarea');
        textarea.value = text;
        document.body.appendChild(textarea);
        textarea.select();
        document.execCommand('copy');
        document.body.removeChild(textarea);
    }
}
```

### 3.6 文件系统API（❌ 禁止）

| API | 原因 | 替代方案 |
|-----|------|----------|
| `FileSystem API` | 已废弃 | 使用FileReader API |
| `showOpenFilePicker()` | 权限问题 | 使用 `<input type="file">` |
| `showSaveFilePicker()` | 权限问题 | 使用下载链接 |

### 3.7 通知API（❌ 禁止）

| API | 原因 | 替代方案 |
|-----|------|----------|
| `Notification.requestPermission()` | 权限问题，用户体验差 | 使用页面内提示 |
| `new Notification()` | 同上 | 使用自定义通知组件 |

### 3.8 地理位置API（❌ 禁止）

| API | 原因 | 替代方案 |
|-----|------|----------|
| `navigator.geolocation` | 隐私问题，权限限制 | 使用输入框让用户输入坐标 |

### 3.9 设备API（❌ 禁止）

| API | 原因 | 替代方案 |
|-----|------|----------|
| `navigator.getUserMedia()` | 权限问题 | 使用模拟数据或占位符 |
| `DeviceOrientationEvent` | 权限问题 | 使用模拟数据 |
| `DeviceMotionEvent` | 权限问题 | 使用模拟数据 |

## 四、CSS样式规范

### 4.1 禁止的CSS特性

| 特性 | 原因 | 替代方案 |
|------|------|----------|
| `position: fixed` 相对于viewport | iframe中定位不准确 | 使用 `position: absolute` 相对于容器 |
| `vh`/`vw` 单位 | iframe中可能不准确 | 使用百分比或px |
| `@import` 外部样式 | 阻塞渲染 | 使用内联样式或CDN链接 |

### 4.2 必须的CSS设置

```css
/* ✅ 正确：确保动画容器适配iframe */
* {
    box-sizing: border-box;
}

html, body {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
    overflow: hidden; /* 防止iframe内滚动 */
    background-color: transparent; /* 透明背景，融入文章 */
}

/* 主容器 */
.main-container {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
}
```

### 4.3 响应式设计

```css
/* ✅ 正确：使用flex布局适配不同尺寸 */
.container {
    display: flex;
    flex-direction: column;
    min-height: 0; /* 重要：允许flex子元素缩小 */
}

.content-area {
    flex: 1;
    min-height: 0;
    overflow: auto;
}

.fixed-header {
    flex-shrink: 0; /* 固定头部不缩小 */
}
```

## 五、JavaScript规范

### 5.1 初始化规范

```javascript
// ✅ 正确：等待DOM加载完成
function init() {
    // 初始化代码
    resizeCanvas();
    setupEventListeners();
    requestAnimationFrame(loop);
}

// 使用DOMContentLoaded而不是window.onload（更快）
if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', init);
} else {
    init();
}
```

### 5.2 事件处理规范

```javascript
// ✅ 正确：使用事件委托，避免内存泄漏
document.addEventListener('click', (e) => {
    if (e.target.matches('.button')) {
        handleButtonClick(e.target);
    }
});

// ✅ 正确：移除事件监听器
function cleanup() {
    window.removeEventListener('resize', handleResize);
}
```

### 5.3 Canvas操作规范

```javascript
// ✅ 正确：响应式Canvas
function resizeCanvas() {
    const container = canvas.parentElement;
    const rect = container.getBoundingClientRect();
    canvas.width = rect.width;
    canvas.height = rect.height;
    // 重新绘制
    redraw();
}

// 监听容器大小变化（而不是window）
const resizeObserver = new ResizeObserver(() => {
    resizeCanvas();
});
resizeObserver.observe(canvas.parentElement);
```

### 5.4 动画循环规范

```javascript
// ✅ 正确：使用requestAnimationFrame
let animationId;
function loop() {
    updateLogic();
    render();
    animationId = requestAnimationFrame(loop);
}

function startAnimation() {
    if (!animationId) {
        loop();
    }
}

function stopAnimation() {
    if (animationId) {
        cancelAnimationFrame(animationId);
        animationId = null;
    }
}
```

## 六、HTML结构规范

### 6.1 基本结构模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>动画标题</title>
    <!-- 使用CDN资源，避免本地依赖 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* 内联样式，确保样式不丢失 */
        * { box-sizing: border-box; }
        html, body {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: transparent;
        }
    </style>
</head>
<body>
    <div class="main-container">
        <!-- 内容 -->
    </div>
    <script>
        // 内联脚本，确保功能完整
        // 初始化代码
    </script>
</body>
</html>
```

### 6.2 禁止的HTML特性

| 特性 | 原因 | 替代方案 |
|------|------|----------|
| `<base>` 标签 | 可能影响相对路径 | 使用绝对路径或CDN |
| `<meta http-equiv="refresh">` | 用户体验差 | 使用JavaScript定时器 |
| `<iframe>` 嵌套 | 性能问题，安全限制 | 避免嵌套iframe |

## 七、资源加载规范

### 7.1 外部资源

| 资源类型 | 推荐方式 | 禁止方式 |
|----------|----------|----------|
| CSS框架 | CDN链接 | 本地文件 |
| 字体 | Google Fonts CDN | 本地字体文件 |
| 图标 | SVG内联或CDN | 本地图片文件 |
| 图片 | Base64内联或CDN | 相对路径文件 |

### 7.2 资源加载示例

```html
<!-- ✅ 正确：使用CDN -->
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono&display=swap" rel="stylesheet">
<script src="https://cdn.tailwindcss.com"></script>

<!-- ✅ 正确：小图片使用Base64 -->
<img src="data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiLz4=">

<!-- ❌ 错误：使用相对路径 -->
<img src="./images/icon.png">
```

## 八、交互功能规范

### 8.1 按钮和控件

```javascript
// ✅ 正确：所有交互元素都应在iframe内工作
function handleButtonClick() {
    // 功能实现
}

// ✅ 正确：添加键盘支持
document.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' && e.target.matches('input')) {
        handleSubmit();
    }
});
```

### 8.2 表单处理

```javascript
// ✅ 正确：阻止默认提交行为
form.addEventListener('submit', (e) => {
    e.preventDefault();
    handleFormSubmit();
});
```

### 8.3 拖拽功能

```javascript
// ✅ 正确：使用Pointer Events（更好的跨设备支持）
element.addEventListener('pointerdown', handleDragStart);
element.addEventListener('pointermove', handleDrag);
element.addEventListener('pointerup', handleDragEnd);
```

## 九、性能优化规范

### 9.1 避免阻塞操作

```javascript
// ❌ 错误：同步阻塞操作
function processLargeData() {
    for (let i = 0; i < 1000000; i++) {
        // 长时间计算
    }
}

// ✅ 正确：使用requestIdleCallback或分帧处理
function processLargeData() {
    let index = 0;
    function processChunk() {
        const end = Math.min(index + 1000, 1000000);
        for (let i = index; i < end; i++) {
            // 处理数据
        }
        index = end;
        if (index < 1000000) {
            requestIdleCallback(processChunk);
        }
    }
    processChunk();
}
```

### 9.2 内存管理

```javascript
// ✅ 正确：清理事件监听器和引用
function cleanup() {
    // 移除事件监听器
    element.removeEventListener('click', handler);
    // 清理定时器
    clearInterval(timerId);
    // 清理Canvas上下文
    ctx = null;
    // 清理DOM引用
    element = null;
}
```

## 十、错误处理规范

### 10.1 全局错误处理

```javascript
// ✅ 正确：添加全局错误处理
window.addEventListener('error', (e) => {
    console.error('动画错误:', e.error);
    // 显示友好的错误提示
    showErrorMessage('动画加载失败，请刷新页面重试');
});

// ✅ 正确：处理未捕获的Promise错误
window.addEventListener('unhandledrejection', (e) => {
    console.error('未处理的Promise错误:', e.reason);
    e.preventDefault(); // 阻止默认错误输出
});
```

### 10.2 API调用错误处理

```javascript
// ✅ 正确：所有可能失败的API调用都要有错误处理
try {
    const data = JSON.parse(jsonString);
} catch (e) {
    console.warn('JSON解析失败，使用默认值');
    data = defaultData;
}
```

## 十一、测试检查清单

生成动画后，必须检查以下项目：

- [ ] 动画在iframe中正常显示
- [ ] 所有按钮和交互功能可用
- [ ] 响应式布局在不同尺寸下正常
- [ ] 没有控制台错误
- [ ] 动画性能流畅（60fps）
- [ ] 移动端触摸交互正常
- [ ] 键盘导航可用（如适用）
- [ ] 没有外部资源404错误
- [ ] 背景透明，融入文章
- [ ] 动画可以暂停/恢复（如适用）

## 十二、示例：正确的动画结构

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SPI动画演示</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        * { box-sizing: border-box; }
        html, body {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: transparent;
        }
        .container {
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- 内容 -->
    </div>
    <script>
        // 初始化代码
        function init() {
            // 使用相对尺寸，不依赖window
            resizeCanvas();
            setupEvents();
            startAnimation();
        }
        
        // 使用DOMContentLoaded
        if (document.readyState === 'loading') {
            document.addEventListener('DOMContentLoaded', init);
        } else {
            init();
        }
    </script>
</body>
</html>
```

## 十三、常见错误示例

### ❌ 错误示例1：使用window尺寸

```javascript
// 错误：iframe中window尺寸不准确
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// 正确：使用容器尺寸
const rect = canvas.parentElement.getBoundingClientRect();
canvas.width = rect.width;
canvas.height = rect.height;
```

### ❌ 错误示例2：访问父窗口

```javascript
// 错误：跨域限制
window.parent.postMessage('hello', '*');

// 正确：使用内部通信
eventBus.emit('message', 'hello');
```

### ❌ 错误示例3：使用localStorage无错误处理

```javascript
// 错误：可能抛出异常
localStorage.setItem('key', 'value');

// 正确：添加错误处理
try {
    localStorage.setItem('key', 'value');
} catch (e) {
    // 使用内存状态
    memoryState.key = 'value';
}
```

## 十四、总结

生成嵌入式动画网页时，核心原则是：

1. **自包含**：不依赖外部文件（CDN除外）
2. **iframe友好**：所有API都考虑iframe环境限制
3. **响应式**：使用相对尺寸和flex布局
4. **错误处理**：所有可能失败的API都要有降级方案
5. **性能优化**：避免阻塞操作，合理使用动画循环

遵循这些规范，可以确保生成的动画能够无缝嵌入博客文章，并在各种环境下正常工作。


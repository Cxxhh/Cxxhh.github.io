# 嵌入式动画快速参考

## 🚫 禁止使用的API（iframe中受限）

### 窗口操作
- ❌ `window.open()` - 使用内部模态框
- ❌ `window.parent` / `window.top` - 避免访问父窗口
- ❌ `window.location` - 使用内部状态管理
- ❌ `document.write()` - 使用DOM操作

### 存储API（需错误处理）
- ⚠️ `localStorage` / `sessionStorage` - 添加try-catch
- ⚠️ `IndexedDB` - 添加权限检查
- ❌ `Cookies` - 避免使用

### 全屏/权限API
- ❌ `element.requestFullscreen()` - 使用CSS全屏类
- ❌ `Notification API` - 使用页面内提示
- ❌ `navigator.geolocation` - 使用输入框
- ❌ `navigator.getUserMedia()` - 使用模拟数据

### 剪贴板（需降级方案）
- ⚠️ `navigator.clipboard` - 添加execCommand降级

## ✅ 必须的CSS设置

```css
* { box-sizing: border-box; }
html, body {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background-color: transparent;
}
```

## ✅ 必须的HTML结构

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>动画标题</title>
    <!-- CDN资源 -->
</head>
<body>
    <div class="container" style="width:100%;height:100%;">
        <!-- 内容 -->
    </div>
    <script>
        // 初始化代码
    </script>
</body>
</html>
```

## ✅ 响应式尺寸（使用容器而非window）

```javascript
// ❌ 错误
canvas.width = window.innerWidth;

// ✅ 正确
const rect = canvas.parentElement.getBoundingClientRect();
canvas.width = rect.width;
```

## ✅ 初始化方式

```javascript
// ✅ 使用DOMContentLoaded
if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', init);
} else {
    init();
}
```

## ✅ 资源加载（使用CDN或Base64）

```html
<!-- ✅ CDN -->
<script src="https://cdn.tailwindcss.com"></script>

<!-- ✅ Base64小图片 -->
<img src="data:image/svg+xml;base64,...">
```

## 📋 检查清单

- [ ] 背景透明（`background-color: transparent`）
- [ ] 无外部文件依赖（CDN除外）
- [ ] 使用容器尺寸而非window尺寸
- [ ] 所有API调用有错误处理
- [ ] 响应式布局（flex/百分比）
- [ ] 无控制台错误


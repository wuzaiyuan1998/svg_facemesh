# 🦎 六角恐龙 FaceMesh 动捕 - 二次开发指南

> 基于 Google MediaPipe FaceMesh 的实时面部捕捉 SVG 动画系统

**最后更新**：2026-03-12  
**版本**：v1.0 (精细分层版)

---

## 📖 目录

1. [项目概述](#项目概述)
2. [环境搭建](#环境搭建)
3. [项目结构](#项目结构)
4. [核心原理](#核心原理)
5. [自定义开发](#自定义开发)
6. [部署指南](#部署指南)
7. [性能优化](#性能优化)
8. [常见问题](#常见问题)

---

## 项目概述

### 功能特性

- ✅ **实时面部捕捉** - 468 个 FaceMesh 关键点
- ✅ **分层 SVG 动画** - 8 个独立图层，支持独立控制
- ✅ **表情映射** - 眨眼、张嘴、眼睛追踪、腮红
- ✅ **物理动画** - 尾巴/外鳃待机摆动
- ✅ **防抖动处理** - EMA 平滑 + 死区过滤
- ✅ **纯前端运行** - 无需后端服务器

### 技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| **MediaPipe FaceMesh** | 0.4.x | 面部关键点检测 |
| **SVG** | 1.1 | 角色渲染 |
| **Canvas 2D** | - | FaceMesh 网格调试显示 |
| **JavaScript** | ES6+ | 动捕逻辑 |

### 效果演示

```
摄像头 → FaceMesh 468 点 → 图层变换 → SVG 动画
                              ↓
                         尾巴摆动
                         外鳃摆动
                         泡泡动画
```

---

## 环境搭建

### 系统要求

| 组件 | 最低配置 | 推荐配置 |
|------|----------|----------|
| **操作系统** | Windows 10 / macOS 10.14 / Linux | 同上 |
| **CPU** | 双核 2.0GHz | 四核 2.5GHz+ |
| **内存** | 4GB | 8GB+ |
| **摄像头** | 480p @ 15fps | 720p @ 30fps |
| **浏览器** | Chrome 80+ | Chrome 最新版 |

### 开发工具

#### 1. 代码编辑器（任选）

```bash
# VS Code（推荐）
https://code.visualstudio.com/

# 必装扩展
- Live Server
- Prettier
- ESLint
```

#### 2. Node.js（可选，用于本地服务器）

```bash
# 下载安装
https://nodejs.org/

# 验证安装
node -v
npm -v
```

#### 3. Python（可选，用于本地服务器）

```bash
# 验证安装
python3 --version

# 如果没有，下载安装
https://www.python.org/
```

### 项目获取

```bash
# 方式 1：直接复制文件
cp -r /path/to/source /your/workspace/

# 方式 2：Git 克隆（如果有仓库）
git clone <repository-url>
cd liujiao-facemesh
```

### 目录结构

```
liujiao-facemesh/
├── liujiao-facemesh-pro.html      # 主程序（精细分层版）
├── liujiao-facemesh.html          # 基础版本
├── liujiao-facemesh-offline.html  # 离线版本
├── README.md                       # 使用手册
├── PACKAGING.md                    # 打包说明
└── DEV_GUIDE.md                    # 本文档
```

---

## 项目结构

### 核心文件解析

#### `liujiao-facemesh-pro.html`

```
HTML 结构（约 700 行）
├── <head>
│   ├── <style>              # CSS 样式（约 150 行）
│   └── <title>              # 页面标题
├── <body>
│   ├── <h1>                 # 标题
│   ├── .container           # 主容器
│   │   ├── .video-panel     # 视频面板
│   │   │   ├── <video>      # 摄像头画面
│   │   │   ├── <canvas>     # FaceMesh 网格
│   │   │   └── #fps         # FPS 计数器
│   │   └── .svg-panel       # SVG 面板
│   │       └── <svg>        # 六角恐龙 SVG（约 150 行）
│   │           ├── <defs>   # 渐变定义
│   │           ├── #layer-tail      # 尾巴层
│   │           ├── #layer-body-group # 身体组
│   │           │   ├── #layer-head  # 头部组
│   │           │   ├── #layer-blush # 腮红层
│   │           │   ├── #layer-belly # 腹部
│   │           │   └── #layer-limbs # 四肢
│   │           └── #bubble-container # 泡泡
│   ├── .controls            # 控制按钮
│   └── .info                # 说明信息
└── <script>                 # JavaScript 逻辑（约 400 行）
    ├── 变量声明
    ├── 平滑滤波函数
    ├── SVG 绘制函数
    ├── 表情映射函数
    ├── 初始化函数
    └── 事件监听
```

### SVG 图层结构

```xml
<svg viewBox="0 0 800 800">
  <!-- 图层 1: 尾巴（最底层） -->
  <g id="layer-tail">...</g>
  
  <!-- 图层 2: 身体组（整体动捕） -->
  <g id="layer-body-group">
    <!-- 图层 2.1: 头部组 -->
    <g id="layer-head">
      <path id="head-shape"/>      <!-- 头部形状 -->
      <g id="head-spots"/>          <!-- 头部斑点 -->
      <g id="layer-gills"/>         <!-- 外鳃 -->
      <g id="layer-eyes"/>          <!-- 眼睛 -->
      <g id="layer-mouth"/>         <!-- 嘴巴 -->
    </g>
    
    <!-- 图层 2.2: 腮红层（独立） -->
    <g id="layer-blush">...</g>
    
    <!-- 图层 2.3: 腹部 -->
    <g id="layer-belly">...</g>
    
    <!-- 图层 2.4: 四肢 -->
    <g id="layer-limbs">...</g>
  </g>
  
  <!-- 图层 3: 泡泡（最上层） -->
  <g id="bubble-container">...</g>
</svg>
```

---

## 核心原理

### 1. FaceMesh 关键点检测

```javascript
// MediaPipe FaceMesh 输出 468 个 3D 关键点
faceMesh.onResults((results) => {
  const landmarks = results.multiFaceLandmarks[0];
  // landmarks[i] = { x, y, z, visibility }
});
```

### 2. 关键点映射表

```javascript
// 眼睛
33   - 左眼左角
133  - 左眼右角
159  - 左眼上
145  - 左眼下
362  - 右眼左角
263  - 右眼右角
386  - 右眼上
374  - 右眼下
468  - 左眼虹膜
473  - 右眼虹膜

// 嘴巴
61   - 左嘴角
291  - 右嘴角
13   - 上唇中点
14   - 下唇中点
0    - 嘴巴中心

// 鼻子
1    - 鼻尖
4    - 鼻底
273  - 鼻左
6    - 鼻右

// 脸颊
116  - 左脸颊
345  - 右脸颊

// 头部
10   - 额头
152  - 下巴
```

### 3. 动捕变换流程

```
FaceMesh 关键点
    ↓
计算偏移量（基于关键点坐标）
    ↓
应用死区过滤 + EMA 平滑
    ↓
设置 SVG transform 属性
    ↓
浏览器渲染
```

### 4. 平滑滤波算法

```javascript
// 死区过滤（消除微小抖动）
function applyDeadZone(value, threshold) {
  if (Math.abs(value) < threshold) return 0;
  return value;
}

// EMA 平滑（指数移动平均）
function smoothValue(current, prev, factor) {
  return prev + (current - prev) * factor;
}

// 参数配置
const deadZone = 0.003;      // 死区阈值
const smoothingFactor = 0.3; // 平滑系数
```

### 5. 眨眼检测

```javascript
// 计算眼睛开合度（垂直距离 / 水平距离）
function getEyeOpenness(top, bottom, left, right) {
  const vertical = Math.abs(top.y - bottom.y);
  const horizontal = Math.abs(right.x - left.x);
  return vertical / horizontal;
}

// 眨眼判定
const avgEyeOpenness = (leftOpenness + rightOpenness) / 2;
const isBlinking = avgEyeOpenness < 0.32;  // 阈值可调
```

---

## 自定义开发

### 1. 修改角色外观

#### 更改颜色

```xml
<!-- 找到对应的 SVG 元素，修改 fill/stroke 属性 -->
<path id="head-shape" 
      fill="#FFD166"      <!-- 头部颜色 -->
      stroke="#000" 
      stroke-width="18"/>

<circle id="blush-left" 
        fill="#FFB3C1"    <!-- 腮红颜色 -->
        opacity="0.8"/>
```

#### 添加新元素

```xml
<!-- 在对应的图层组内添加 -->
<g id="layer-head">
  <!-- 添加新的装饰元素 -->
  <circle cx="450" cy="250" r="15" fill="#FFB347"/>
</g>
```

### 2. 添加新动捕功能

#### 步骤 1：在 SVG 中添加元素

```xml
<g id="layer-new-feature">
  <path id="new-part" d="..." fill="#..."/>
</g>
```

#### 步骤 2：在 JavaScript 中获取引用

```javascript
const svgElements = {
  // ... 现有元素
  newPart: document.getElementById('new-part'),
};
```

#### 步骤 3：在 applyExpression 中添加逻辑

```javascript
function applyExpression(landmarks) {
  // ... 现有代码
  
  // 新动捕逻辑
  const landmark = landmarks[关键点编号];
  if (landmark && svgElements.newPart) {
    const offsetX = (landmark.x - 0.5) * 100;
    const offsetY = (landmark.y - 0.5) * 100;
    svgElements.newPart.setAttribute('transform', 
      `translate(${offsetX}, ${offsetY})`);
  }
}
```

### 3. 调整动捕灵敏度

#### 修改死区阈值

```javascript
// 更灵敏（但可能抖动）
const deadZone = 0.001;

// 更稳定（但响应延迟）
const deadZone = 0.005;
```

#### 修改平滑系数

```javascript
// 快速响应（平滑度低）
const smoothingFactor = 0.5;

// 平滑稳定（响应慢）
const smoothingFactor = 0.1;
```

#### 修改眨眼阈值

```javascript
// 更容易眨眼
const isBlinking = avgEyeOpenness < 0.35;

// 更难眨眼
const isBlinking = avgEyeOpenness < 0.25;
```

### 4. 添加新动画

#### 待机动画示例

```javascript
// 在 animateTail 函数后添加新动画
let customAngle = 0;
let customTime = 0;

function animateCustom() {
  customTime += 0.03;
  customAngle = Math.sin(customTime) * 10;
  
  if (svgElements.newPart) {
    svgElements.newPart.setAttribute('transform', 
      `rotate(${customAngle} 400 400)`);
  }
  
  if (isRunning) {
    requestAnimationFrame(animateCustom);
  }
}

// 在 start() 函数中启动
function start() {
  // ... 现有代码
  animateCustom();  // 启动新动画
}
```

### 5. 修改图层顺序

```xml
<!-- SVG 中元素的顺序决定渲染层级 -->
<!-- 先定义的在下层，后定义的在上层 -->

<svg>
  <g id="layer-1">...</g>  <!-- 最底层 -->
  <g id="layer-2">...</g>
  <g id="layer-3">...</g>  <!-- 最上层 -->
</svg>
```

### 6. 调试技巧

#### 显示关键点编号

```javascript
// 在 drawFaceMesh 函数中添加
landmarks.forEach((landmark, i) => {
  if (i % 10 === 0) {  // 每 10 个点显示一个编号
    const x = landmark.x * canvas.width;
    const y = landmark.y * canvas.height;
    ctx.fillStyle = '#FFF';
    ctx.font = '10px Arial';
    ctx.fillText(i, x + 5, y - 5);
  }
});
```

#### 实时显示参数

```javascript
// 在页面添加显示区域
<div id="debug-info"></div>

// 在 applyExpression 中更新
document.getElementById('debug-info').innerHTML = `
  FaceX: ${faceX.toFixed(3)}<br>
  EyeOpenness: ${avgEyeOpenness.toFixed(3)}<br>
  Blinking: ${isBlinking}
`;
```

---

## 部署指南

### 方式 1：本地开发服务器（推荐）

#### 使用 Python

```bash
# 进入项目目录
cd /path/to/liujiao-facemesh

# 启动服务器
python3 -m http.server 8080

# 访问
http://localhost:8080/liujiao-facemesh-pro.html
```

#### 使用 Node.js

```bash
# 安装 http-server
npm install -g http-server

# 启动服务器
http-server . -p 8080

# 访问
http://localhost:8080/liujiao-facemesh-pro.html
```

#### 使用 VS Code Live Server

```
1. 安装 Live Server 扩展
2. 右键 liujiao-facemesh-pro.html
3. 选择 "Open with Live Server"
4. 自动在浏览器中打开
```

### 方式 2：直接打开文件

```bash
# 双击 HTML 文件
# 或
open liujiao-facemesh-pro.html  # macOS
start liujiao-facemesh-pro.html # Windows
```

**注意**：某些浏览器可能限制 `file://` 协议的摄像头访问，建议使用本地服务器。

### 方式 3：部署到 Web 服务器

#### Nginx 配置

```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    root /path/to/liujiao-facemesh;
    index liujiao-facemesh-pro.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    # 启用 HTTPS（推荐）
    # 需要配置 SSL 证书
}
```

#### Apache 配置

```apache
<VirtualHost *:80>
    ServerName your-domain.com
    DocumentRoot /path/to/liujiao-facemesh
    
    <Directory /path/to/liujiao-facemesh>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### 方式 4：静态托管服务

#### GitHub Pages

```bash
# 1. 创建 GitHub 仓库
# 2. 推送文件
git add .
git commit -m "Initial commit"
git push origin main

# 3. 启用 GitHub Pages
# Settings → Pages → Source: main branch
```

#### Vercel / Netlify

```bash
# 1. 安装 CLI
npm install -g vercel

# 2. 部署
vercel

# 3. 按提示完成部署
```

### HTTPS 要求

摄像头访问**必须**在以下环境之一：
- ✅ `https://` 开头的网站
- ✅ `http://localhost` 或 `http://127.0.0.1`
- ❌ `file://` 协议（部分浏览器限制）
- ❌ `http://` 公网 IP（必须 HTTPS）

---

## 性能优化

### 1. 降低模型复杂度

```javascript
faceMesh.setOptions({
  modelComplexity: 1,  // 0=轻量，1=标准，2=高精度
  minDetectionConfidence: 0.3,
  minTrackingConfidence: 0.3
});
```

### 2. 降低摄像头分辨率

```javascript
stream = await navigator.mediaDevices.getUserMedia({
  video: { 
    width: { ideal: 640 },   // 降低到 640x480
    height: { ideal: 480 }
  }
});
```

### 3. 减少绘制频率

```javascript
// 每 2 帧绘制一次网格
let frameSkip = 0;
faceMesh.onResults((results) => {
  frameSkip++;
  if (frameSkip % 2 === 0) {
    drawFaceMesh(results.multiFaceLandmarks[0]);
  }
});
```

### 4. 优化 SVG 元素数量

```xml
<!-- 移除不必要的元素 -->
<!-- 合并可以合并的路径 -->
<!-- 使用简单的几何形状代替复杂路径 -->
```

### 5. 使用 requestAnimationFrame

```javascript
// ✅ 正确：使用 rAF
function animate() {
  // 动画逻辑
  requestAnimationFrame(animate);
}

// ❌ 错误：使用 setInterval
setInterval(animate, 16);  // 不要这样做
```

---

## 常见问题

### Q1: 摄像头无法启动？

**A:** 检查以下几点：
1. 浏览器是否授予摄像头权限
2. 是否使用 HTTPS 或 localhost
3. 其他程序是否占用摄像头
4. 摄像头硬件是否正常

### Q2: FaceMesh 加载失败？

**A:** 
1. 检查网络连接（需要下载 5MB 模型）
2. 清除浏览器缓存后重试
3. 检查 CDN 是否可访问

### Q3: 动捕抖动严重？

**A:** 调整平滑参数：
```javascript
const deadZone = 0.005;      // 提高死区阈值
const smoothingFactor = 0.1; // 降低平滑系数
```

### Q4: 眨眼不触发？

**A:** 降低眨眼阈值：
```javascript
const isBlinking = avgEyeOpenness < 0.35;  // 提高阈值
```

### Q5: 性能卡顿？

**A:** 
1. 降低模型复杂度：`modelComplexity: 0`
2. 降低摄像头分辨率：`width: 320, height: 240`
3. 关闭其他浏览器标签页
4. 重启浏览器

### Q6: 如何修改角色？

**A:** 
1. 编辑 SVG 元素（颜色、形状）
2. 保持 `id` 不变（JavaScript 需要引用）
3. 测试动捕是否正常

### Q7: 如何添加新表情？

**A:** 
1. 在 SVG 中添加新元素
2. 在 JavaScript 中获取引用
3. 在 `applyExpression` 中添加映射逻辑
4. 参考现有代码结构

### Q8: 移动端能用吗？

**A:** 
- ✅ iOS Safari：支持（iOS 14+）
- ✅ Android Chrome：支持
- ⚠️ 性能可能较低，建议降低模型复杂度

---

## 参考资料

### 官方文档

- [MediaPipe FaceMesh](https://google.github.io/mediapipe/solutions/face_mesh)
- [SVG Transform](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute/transform)
- [Canvas API](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API)

### 相关项目

- [MediaPipe Examples](https://codepen.io/collection/DbxMqM)
- [SVG Animation](https://css-tricks.com/guide-svg-animations-smil/)

### 工具推荐

- [SVG Editor](https://svg-editor.github.io/svgedit/)
- [MediaPipe Studio](https://studio.mediapipe.dev/)

---

## 更新日志

### v1.0 (2026-03-12)
- ✅ 初始版本发布
- ✅ 8 层 SVG 分层结构
- ✅ 身体组整体联动
- ✅ 腮红独立动捕
- ✅ 眼睛眼眶独立平移
- ✅ 防抖动处理（EMA + 死区）
- ✅ 眨眼检测优化

---

## 技术支持

如有问题，请提供：
1. 浏览器版本
2. 操作系统版本
3. 控制台错误信息（F12）
4. 问题复现步骤

---

**🦎 祝你开发愉快！**

*最后更新：2026-03-12*

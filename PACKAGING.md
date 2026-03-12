# 📦 六角恐龙 FaceMesh 离线版 - 打包说明

## 📁 文件清单

### 核心文件
```
liujiao-facemesh-offline/
├── liujiao-facemesh-offline.html    # 离线版主程序
├── README.md                         # 使用手册
└── PACKAGING.md                      # 本文件（打包说明）
```

### 可选文件（开发版本）
```
├── liujiao-facemesh.html            # 在线版（CDN 加速）
├── face-mesh-lite.html              # 轻量面部追踪版
└── face-detection.html              # 超轻量检测版
```

---

## 🚀 快速使用

### 方式一：直接打开（推荐）

1. **双击** `liujiao-facemesh-offline.html`
2. 浏览器会自动打开
3. 点击 **"启动"** 按钮
4. 允许摄像头权限
5. 开始互动！

### 方式二：本地服务器

```bash
# 进入目录
cd liujiao-facemesh-offline

# 启动服务器（任选一种）

# Python 3
python3 -m http.server 8080

# Node.js
npx http-server . -p 8080

# 浏览器访问
http://localhost:8080/liujiao-facemesh-offline.html
```

---

## 📥 离线说明

### 首次运行（需要网络）

1. 打开 `liujiao-facemesh-offline.html`
2. 点击"启动"
3. 系统会自动下载 MediaPipe 模型（约 5MB）
4. 模型会缓存到浏览器
5. 加载完成后即可使用

### 完全离线（无需网络）

模型文件会缓存在浏览器中，刷新页面后可离线使用。

**浏览器缓存位置**：
- Chrome: `~/.cache/CacheStorage/`
- Firefox: `~/.mozilla/firefox/*/cache2/`
- Edge: `~/.cache/Microsoft/Edge/`

### 清除缓存

如需重新下载模型：
1. 按 `F12` 打开开发者工具
2. 右键刷新按钮 → **"清空缓存并硬性重新加载"**
3. 或清除浏览器缓存

---

## 📦 打包为 ZIP

### Windows

```powershell
# 方式一：右键压缩
# 1. 选中 liujiao-facemesh-offline 文件夹
# 2. 右键 → 发送到 → 压缩 (zipped) 文件夹

# 方式二：PowerShell
Compress-Archive -Path liujiao-facemesh-offline -DestinationPath liujiao-facemesh-offline.zip
```

### macOS / Linux

```bash
# 终端压缩
zip -r liujiao-facemesh-offline.zip liujiao-facemesh-offline/

# 或创建 tar.gz
tar -czvf liujiao-facemesh-offline.tar.gz liujiao-facemesh-offline/
```

### 文件大小

| 文件 | 大小 |
|------|------|
| HTML 文件 | ~25KB |
| README.md | ~6KB |
| **总计** | **~31KB** |

*注：MediaPipe 模型（5MB）会从 CDN 动态加载，不包含在压缩包中*

---

## 🌐 完全离线版本（包含模型）

如需**完全离线**（无任何网络依赖），需要下载 MediaPipe 模型文件：

### 下载模型文件

```bash
cd liujiao-facemesh-offline

# 下载 FaceMesh 库
curl -o face_mesh.js https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh.js

# 下载 WASM 模型（约 3MB）
curl -o face_mesh_solution_simd_wasm_bin.wasm \
     https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh_solution_simd_wasm_bin.wasm

# 下载资源文件（约 2MB）
curl -o face_mesh_solution_packed_assets.data \
     https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh_solution_packed_assets.data
```

### 修改 HTML 引用

编辑 `liujiao-facemesh-offline.html`，找到：

```javascript
locateFile: (file) => {
  return `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${file}`;
}
```

改为：

```javascript
locateFile: (file) => {
  return `./${file}`;  // 使用本地文件
}
```

### 完全离线包大小

| 文件 | 大小 |
|------|------|
| HTML + 文档 | ~31KB |
| face_mesh.js | ~200KB |
| *.wasm | ~3MB |
| *.data | ~2MB |
| **总计** | **~5.2MB** |

---

## ✅ 验证清单

打包前检查：

- [ ] `liujiao-facemesh-offline.html` 存在
- [ ] `README.md` 存在
- [ ] 可以用浏览器打开 HTML 文件
- [ ] 点击"启动"按钮正常
- [ ] 摄像头权限请求正常
- [ ] 面部检测功能正常
- [ ] 表情映射功能正常

---

## 📤 分发建议

### 通过飞书发送

1. 压缩为 ZIP 文件
2. 在飞书聊天窗口点击"文件"按钮
3. 选择 ZIP 文件上传
4. 发送给用户

### 通过邮件发送

1. 压缩为 ZIP 文件
2. 添加附件
3. 如果文件>25MB，使用网盘链接

### 通过 U 盘拷贝

1. 直接拷贝整个文件夹
2. 或压缩后拷贝 ZIP 文件

---

## 🔧 故障排除

### Q: 打开 HTML 文件后是空白？

A: 按 F12 查看控制台错误，可能是：
- 浏览器版本过旧 → 升级 Chrome/Edge
- 文件损坏 → 重新下载
- 路径问题 → 确保 HTML 文件在根目录

### Q: 摄像头无法启动？

A: 检查：
- 浏览器是否授予摄像头权限
- 其他程序是否占用摄像头
- 摄像头硬件是否正常

### Q: 模型加载失败？

A: 检查：
- 网络连接是否正常
- CDN 是否可访问
- 防火墙是否阻止

---

## 📞 技术支持

如有问题，请提供：
1. 浏览器版本（Chrome/Edge/Firefox）
2. 操作系统版本
3. 控制台错误信息（F12）
4. 问题复现步骤

---

**🦎 祝你使用愉快！**

*最后更新：2026-03-11*

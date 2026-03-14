# 宝塔面板部署指南

## 前提条件

- 已安装 Node.js (建议 v18 或 v20)
- 已安装 PM2 管理器（推荐）
- 项目已上传到服务器

## 方法 1：使用 PM2（推荐）

### 1. 上传项目文件

将整个项目上传到服务器，例如：
```
/www/wwwroot/dyyq/
```

### 2. 安装依赖

在 SSH 或宝塔终端中执行：
```bash
cd /www/wwwroot/dyyq/frontend

# 清理旧依赖（如果有）
rm -rf node_modules package-lock.json

# 安装依赖
npm install
```

### 3. 构建项目
```bash
npm run build
```

### 4. 配置 PM2

在宝塔面板 → 软件商店 → PM2 管理器：

**添加项目：**
- 项目名称：`dyyq-frontend`
- 运行目录：`/www/wwwroot/dyyq/frontend`
- 启动文件：`npm`
- 启动方式：`start`
- 端口：`3000`

或者使用命令行：
```bash
cd /www/wwwroot/dyyq/frontend
pm2 start npm --name "dyyq-frontend" -- start
pm2 save
```

### 5. 配置反向代理（Nginx）

在宝塔面板 → 网站 → 设置 → 反向代理：

**添加反向代理：**
- 代理名称：`dyyq-frontend`
- 目标 URL：`http://127.0.0.1:3000`
- 发送域名：`$host`

**代理配置：**
```nginx
location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_cache_bypass $http_upgrade;
}
```

---

## 方法 2：使用 Next.js 静态导出（无需 PM2）

如果只想部署静态文件：

### 1. 构建静态文件

在本地或服务器上：
```bash
cd frontend
npm install
npm run build
```

这会在 `frontend/out` 目录生成静态文件。

### 2. 上传静态文件

只上传 `frontend/out` 目录的内容到网站根目录：
```
/www/wwwroot/your-domain.com/
├── index.html
├── _next/
├── img/
├── fonts/
└── ...
```

### 3. 配置 Nginx（如果需要 SPA 路由）

在宝塔面板 → 网站 → 设置 → 配置文件：

添加以下内容（在 `location /` 之前）：
```nginx
location / {
    try_files $uri $uri/ /index.html;
}

# 缓存静态资源
location /_next/static/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

location /img/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

location /fonts/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

---

## 常见问题排查

### 问题 1：`sh: next: command not found`

**原因**：依赖未安装

**解决**：
```bash
cd frontend
npm install
```

### 问题 2：端口被占用

**检查**：
```bash
netstat -ntp | grep 3000
```

**解决**：修改端口或停止占用进程

### 问题 3：内存不足

**解决**：增加服务器内存或使用 swap
```bash
# 创建 2GB swap
dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

### 问题 4：构建失败

**检查 Node 版本**：
```bash
node -v  # 建议 v18 或 v20
```

**清理缓存重试**：
```bash
rm -rf .next node_modules package-lock.json
npm install
npm run build
```

---

## 性能优化建议

1. **启用 gzip 压缩**（宝塔面板默认已启用）
2. **配置 CDN**（如 Cloudflare、阿里云 CDN）
3. **启用 HTTP/2**
4. **配置图片缓存**（已在 Nginx 配置中）

---

## 监控和日志

### 查看 PM2 日志：
```bash
pm2 logs dyyq-frontend
```

### 重启项目：
```bash
pm2 restart dyyq-frontend
```

### 停止项目：
```bash
pm2 stop dyyq-frontend
```

---

## SSL 证书配置

在宝塔面板 → 网站 → 设置 → SSL：

1. 选择 Let's Encrypt
2. 点击申请
3. 开启强制 HTTPS

---

## 域名配置

将你的域名 `perfectlifeexperience.com` 的 A 记录指向服务器 IP。

---

**部署完成后访问：** https://perfectlifeexperience.com

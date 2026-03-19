# 部署指南

## 构建

```bash
npm install
npm run build
# 产物输出至 dist/
```

## 环境变量

在项目根目录创建 `.env.production`，按需填写：

```env
VITE_API_BASE_URL=https://api.example.com
```

> 不同环境对应不同文件：`.env.development` / `.env.production`

## 部署方式

### 静态托管（推荐）

将 `dist/` 目录上传至静态托管平台：

| 平台 | 说明 |
|------|------|
| Vercel | 连接 Git 仓库，自动构建部署 |
| Nginx | 见下方配置 |
| CDN | 上传 `dist/` 至 OSS / S3，配置自定义域名 |

### Nginx 配置示例

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/dist;
    index index.html;

    # SPA 路由支持
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

## 回滚

保留上一个版本的 `dist/` 产物，回滚时直接替换。

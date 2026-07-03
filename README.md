## Sub2API Docker Compose 部署流程总结

### 一、环境准备
1. **确认 Docker 环境**
   ```bash
   docker --version        # Docker 29.5.2
   docker-compose --version # Docker Compose v5.1.4
   ```

2. **检查端口占用**
   ```bash
   ss -tlnp | grep 8080
   ```
   确认 8080 端口未被占用。

### 二、下载部署脚本并执行
```bash
cd /www/wwwroot/Sub2API
curl -sSL https://raw.githubusercontent.com/Wei-Shaw/sub2api/main/deploy/docker-deploy.sh -o docker-deploy.sh
bash docker-deploy.sh
```

脚本自动完成：
- 下载 `docker-compose.yml`（基于 `docker-compose.local.yml`）
- 下载 `.env.example` 并生成 `.env`
- **自动生成安全密钥**：`POSTGRES_PASSWORD`、`JWT_SECRET`、`TOTP_ENCRYPTION_KEY`
- 创建数据目录：`data/`、`postgres_data/`、`redis_data/`

### 三、启动服务
```bash
cd /www/wwwroot/Sub2API && docker-compose up -d
```

拉取并启动三个容器：
| 服务 | 镜像 | 状态 |
|------|------|------|
| sub2api | `weishaw/sub2api:latest` | 应用服务 |
| sub2api-postgres | `postgres:18-alpine` | 数据库 |
| sub2api-redis | `redis:8-alpine` | 缓存 |

### 四、验证部署
```bash
# 查看容器状态
docker-compose ps

# 查看日志获取管理员密码
docker-compose logs sub2api
```

### 五、访问服务
- **后台地址**：`http://123.207.220.230:8080`
- **管理员账号**：`admin@sub2api.local`
- **初始密码**：从日志中提取（仅显示一次）

### 六、关键配置说明
| 配置项 | 说明 |
|--------|------|
| `JWT_SECRET` / `TOTP_ENCRYPTION_KEY` | 已固定生成，避免容器重启导致会话失效 |
| `SECURITY_URL_ALLOWLIST_ALLOW_INSECURE_HTTP=true` | 允许 HTTP 上游（内网环境） |
| `OPS_ENABLED=true` | 启用运维监控功能 |

### 七、后续可选操作
1. **配置域名访问**：在宝塔面板 Nginx 中添加反向代理到 `localhost:8080`，并开启 `underscores_in_headers on`
2. **启用 Simple Mode**：修改 `.env` 设置 `RUN_MODE=simple` + `SIMPLE_MODE_CONFIRM=true` 隐藏 SaaS 计费功能
3. **修改管理员邮箱**：编辑 `.env` 中的 `ADMIN_EMAIL` 后重启容器

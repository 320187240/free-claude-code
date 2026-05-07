# Docker 部署指南

## 快速开始

### 1. 构建镜像

```bash
cd docker
docker build -t free-claude-code -f Dockerfile ..
```

### 2. 配置环境变量

复制环境变量模板：

```bash
cd ..
cp .env.example .env
```

编辑 `.env` 文件，至少配置以下内容：

```dotenv
# 选择一个提供商并配置
NVIDIA_NIM_API_KEY="nvapi-your-key"
MODEL="nvidia_nim/z-ai/glm4.7"
ANTHROPIC_AUTH_TOKEN="freecc"
```

### 3. 运行容器

**方式一：使用 docker run**

```bash
docker run -d \
  --name free-claude-code \
  -p 8082:8082 \
  --env-file .env \
  free-claude-code
```

**方式二：使用 docker-compose（推荐）**

```bash
cd docker
docker-compose up -d
```

### 4. 验证运行

```bash
# 查看日志
docker logs -f free-claude-code

# 测试接口
curl http://localhost:8082/v1/models
```

## 使用 Claude Code 连接

设置环境变量后启动 Claude Code：

```bash
ANTHROPIC_AUTH_TOKEN="freecc" \
ANTHROPIC_BASE_URL="http://localhost:8082" \
CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1 \
claude
```

## 高级配置

### 包含语音转录功能

如需本地 Whisper 支持，修改 Dockerfile：

```dockerfile
# 将这行
RUN uv sync --frozen --no-dev

# 改为
RUN uv sync --frozen --no-dev --extra voice_local
```

然后在 `.env` 中启用：

```dotenv
VOICE_NOTE_ENABLED=true
WHISPER_DEVICE="cpu"
WHISPER_MODEL="base"
```

### 使用 Discord/Telegram Bot

在 `.env` 中配置：

```dotenv
MESSAGING_PLATFORM="discord"
DISCORD_BOT_TOKEN="your-token"
ALLOWED_DISCORD_CHANNELS="123456789"
CLAUDE_WORKSPACE="/app/agent_workspace"
```

确保 docker-compose.yml 中已挂载工作空间目录。

### 自定义端口

修改 docker-compose.yml：

```yaml
ports:
  - "8080:8082"  # 主机端口:容器端口
```

或在 docker run 中使用 `-p 8080:8082`。

## 管理命令

```bash
# 启动
docker-compose up -d

# 停止
docker-compose down

# 重启
docker-compose restart

# 查看日志
docker-compose logs -f

# 更新镜像
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

## 故障排查

### 容器无法启动

检查日志：
```bash
docker logs free-claude-code
```

常见问题：
- `.env` 文件配置错误
- 端口 8082 已被占用
- API 密钥无效

### 连接被拒绝

确认容器正在运行：
```bash
docker ps | grep free-claude-code
```

检查端口映射：
```bash
docker port free-claude-code
```

### 性能问题

为容器分配更多资源：

```yaml
services:
  free-claude-code:
    # ...
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
```

## 生产部署建议

1. **使用环境变量而非 .env 文件**：在生产环境中通过容器编排工具注入
2. **启用健康检查**：
   ```yaml
   healthcheck:
     test: ["CMD", "curl", "-f", "http://localhost:8082/v1/models"]
     interval: 30s
     timeout: 10s
     retries: 3
   ```
3. **配置日志轮转**：避免日志文件过大
4. **使用反向代理**：通过 Nginx/Traefik 添加 HTTPS 支持
5. **设置资源限制**：防止容器消耗过多系统资源

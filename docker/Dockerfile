# 使用官方 Python 3.14 镜像
FROM python:3.14-slim

# 设置工作目录
WORKDIR /app

# 安装 uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

# 复制项目文件
COPY pyproject.toml uv.lock README.md ./
COPY .env.example ./
COPY server.py ./
COPY api ./api
COPY cli ./cli
COPY config ./config
COPY core ./core
COPY messaging ./messaging
COPY providers ./providers

# 安装依赖（不包含可选的 voice 依赖以减小镜像体积）
RUN uv sync --frozen --no-dev

# 暴露端口
EXPOSE 8082

# 设置环境变量
ENV PYTHONUNBUFFERED=1

# 启动命令
CMD ["uv", "run", "uvicorn", "server:app", "--host", "0.0.0.0", "--port", "8082"]

# 第一阶段：构建阶段
FROM debian:bookworm-slim AS builder

WORKDIR /app

# 安装必要工具
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# 复制 uv（从 docker/env 目录）
COPY docker/env/uv/uv-linux /usr/local/bin/uv
RUN chmod +x /usr/local/bin/uv

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

# 下载 Python 3.14 并安装依赖（所有下载都在构建时完成）
RUN uv python install 3.14 && \
    uv sync --frozen --no-dev --python /root/.local/share/uv/python/cpython-3.14.0a3-linux-aarch64-gnu/bin/python3

# 第二阶段：创建最小运行镜像
FROM debian:bookworm-slim

WORKDIR /app

# 安装运行时依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# 从构建阶段复制 uv、Python 和应用
COPY --from=builder /usr/local/bin/uv /usr/local/bin/uv
COPY --from=builder /root/.local/share/uv/python /root/.local/share/uv/python
COPY --from=builder /app /app

# 设置 PATH 包含 Python 3.14
ENV PATH="/root/.local/share/uv/python/cpython-3.14.0a3-linux-aarch64-gnu/bin:$PATH"

# 暴露端口
EXPOSE 8082

# 设置环境变量
ENV PYTHONUNBUFFERED=1
ENV PYTHONIOENCODING=utf-8
ENV LOG_LEVEL=DEBUG

# 启动命令
CMD ["uv", "run", "uvicorn", "server:app", "--host", "0.0.0.0", "--port", "8082", "--log-level", "debug", "--access-log"]

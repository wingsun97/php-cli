# php-cli

基于 `php:8.3-cli-bookworm` 构建的 PHP CLI Docker 镜像，预装常用扩展，托管于 GitHub Container Registry。

## 镜像列表

| 镜像 Tag | 说明 | 扩展 |
|----------|------|------|
| `ghcr.io/wingsun97/php-cli:8.3-core` | 基础 CLI 镜像 | bcmath, intl, pdo, pdo_mysql, mysqli, zip, gd, opcache, pcntl |
| `ghcr.io/wingsun97/php-cli:8.3-grpc` | Core + gRPC 扩展 | 包含 core 全部扩展 + grpc (1.76.0) |

`8.3-grpc` 基于 `8.3-core` 构建，通过多阶段构建将 gRPC 编译产物复制到运行阶段，无需在最终镜像中保留 gcc 等编译工具。

## 构建镜像

### 构建 Core

```bash
docker build -f 8.3/Dockerfile-Core -t ghcr.io/wingsun97/php-cli:8.3-core .
```

### 构建 Grpc（依赖 Core）

```bash
# 先构建 Core
docker build -f 8.3/Dockerfile-Core -t ghcr.io/wingsun97/php-cli:8.3-core .

# 再构建 Grpc
docker build -f 8.3/Dockerfile-Grpc -t ghcr.io/wingsun97/php-cli:8.3-grpc .
```

### 一次性构建（通过 build arg 复用）

```bash
docker build -f 8.3/Dockerfile-Core -t ghcr.io/wingsun97/php-cli:8.3-core . \
  && docker build -f 8.3/Dockerfile-Grpc -t ghcr.io/wingsun97/php-cli:8.3-grpc .
```

### 构建并打多个 Tag

```bash
docker build -f 8.3/Dockerfile-Core \
  -t ghcr.io/wingsun97/php-cli:8.3-core \
  -t ghcr.io/wingsun97/php-cli:8.3-core-v1.0 \
  -t ghcr.io/wingsun97/php-cli:latest \
  .
```

## 使用镜像

```bash
docker run --rm -v "$(pwd):/workspace" -w /workspace ghcr.io/wingsun97/php-cli:8.3-core php -v

docker run --rm -v "$(pwd):/workspace" -w /workspace ghcr.io/wingsun97/php-cli:8.3-grpc php -m | grep grpc
```

## CI 自动构建

推送至 `main` 分支时，GitHub Actions 自动构建并推送至 GHCR：

- `core` job：构建 `8.3-core`
- `grpc` job：等待 core 镜像就绪后，构建 `8.3-grpc`

### 通过 Git Tag 构建带版本号的镜像

推送 `v*` 格式的 git tag（如 `v1.0`、`v2.1.0`）时，除滚动标签外还会额外推送版本标签：

| 推送 git tag | 生成的 Docker 镜像 |
|--------------|-------------------|
| `v1.0` | `8.3-core`、`8.3-core-v1.0`、`8.3-grpc`、`8.3-grpc-v1.0` |
| `v2.1.0` | `8.3-core`、`8.3-core-v2.1.0`、`8.3-grpc`、`8.3-grpc-v2.1.0` |

用法：

```bash
git tag v1.0
git push origin v1.0
```

滚动标签始终指向最新构建，版本标签指向固定快照，方便生产环境锁定版本。

## 自定义镜像 Tag

修改 `build.yml` 中的 `CORE_TAG` 和 `GRPC_TAG` 环境变量即可控制输出的 Docker 标签名。如需扩展新版本（如 PHP 8.4），复制 `8.3/` 目录并更新 Dockerfile 中的基础镜像即可。

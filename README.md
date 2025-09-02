## 📌 项目简介

本项目旨在通过 **Docker + Nginx 反向代理**，实现对多个公共容器镜像仓库（Docker Hub、NVIDIA NGC、GitHub Container Registry）的统一代理访问。
通过在 `docker-compose` 中配置 Nginx 服务，并结合 **hosts 劫持**，可实现以下目标：

* 🚀 **加速镜像拉取**：解决部分仓库在国内或特定网络环境下访问速度慢、超时等问题。
* 🔒 **统一代理管理**：将 `registry-1.docker.io`、`nvcr.io`、`ghcr.io` 的访问流量，统一转发至后端镜像源。
* 📊 **统一日志格式**：使用 `main` 日志格式，输出访问源 IP、请求路径、对应仓库域名，便于分析与监控。
* ⚙️ **灵活扩展**：可根据需要增减镜像源（如 Google Container Registry、Quay.io 等）。

关于 docker registry 的配置可以参考[官方文档](https://distribution.github.io/distribution/about/configuration/)。

---

## 📖 README.md（示例）

````markdown
# 🐳 Docker Registry Proxy

## 项目介绍
这是一个基于 **Docker Compose + Nginx** 的多仓库容器镜像代理服务，支持：
- Docker Hub (`registry-1.docker.io`)
- NVIDIA NGC (`nvcr.io`)
- GitHub Container Registry (`ghcr.io`)

通过配置 `hosts` 劫持和反向代理，可解决部分镜像仓库拉取速度慢的问题，并统一日志输出。

---

## 功能特性
- 🚀 加速镜像拉取（Hub / NVIDIA / GitHub）
- 🔒 统一代理与访问控制
- 📊 标准化日志（支持 `$host`、`$remote_addr` 等字段）
- ⚙️ 可扩展支持更多 Registry

---

## 使用方法

### 1. 克隆仓库
```bash
git clone https://github.com/Lax/docker-registry-cache.git
cd docker-registry-proxy
````

### 2. 启动服务

```bash
docker compose up -d
```

### 3. 配置 hosts 劫持

编辑 `/etc/hosts`，将目标域名劫持到代理服务器 IP：

```
127.0.0.1 registry-1.docker.io
127.0.0.1 nvcr.io
127.0.0.1 ghcr.io
127.0.0.1 registry.local
```

### 4. 拉取镜像测试

```bash
docker pull nvcr.io/nvidia/pytorch:25.03-py3
docker pull ghcr.io/github/super-linter:slim
docker pull registry-1.docker.io/library/ubuntu:22.04
```

### 5. 修改 Docker 服务启动参数

如果拉取失败，检查一下服务配置。
在文件`/etc/docker/daemon.json`中采用`http`协议。

```
    "registry-mirrors": [
        "http://registry-1.docker.io"
    ],
```
---

## 日志格式

日志统一为 `main`，包含：

```
$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent \
"$http_referer" "$http_user_agent" "$host"
```

示例：

```
172.17.0.1 - - [02/Sep/2025:15:00:21 +0000] "GET /v2/library/ubuntu/manifests/22.04 HTTP/1.1" 200 546 "-" "docker/25.0.3 go/go1.21.0 git-commit/abcdef" "registry-1.docker.io"
```

---

## 文件结构

```
.
├── docker-compose.yml   # 服务编排文件
├── nginx.conf           # Nginx 配置文件
└── README.md            # 项目说明文档
```

---

## TODO

* [ ] 增加 Google Container Registry 支持
* [ ] 增加 Quay.io 支持
* [ ] 自动化 SSL 配置
* [ ] 增加 Prometheus 指标采集

---

## 许可证

MIT License

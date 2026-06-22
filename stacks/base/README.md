# Base Stack

实现整个项目的基础设施层，所有其他 Stack 依赖此 Stack 运行。

## 服务清单

| 服务 | 镜像 | 用途 |
|------|------|------|
| Traefik | `traefik:v3.1.6` | 反向代理 + 自动 HTTPS |
| Portainer CE | `portainer/portainer-ce:2.21.3` | Docker 管理 UI |
| Watchtower | `containrrr/watchtower:1.7.1` | 容器自动更新 |
| Socket Proxy | `tecnativa/docker-socket-proxy:0.2.0` | 安全隔离 Docker socket |

## 网络

本项目使用名为 `proxy` 的外部网络。所有其他 Stack 通过此网络接入 Traefik。启动前必须先创建该网络：
```bash
docker network create proxy
```

## 环境变量配置

复制 `.env.example` 为 `.env` 并填入必要的信息：
```bash
DOMAIN=example.com
ACME_EMAIL=admin@example.com
TRAEFIK_AUTH=         # htpasswd 生成的用户名:密码
TZ=Asia/Shanghai
```

如何生成 `TRAEFIK_AUTH` 的值：
```bash
echo $(htpasswd -nb your_username your_password) | sed -e s/\\$/\\$\\$/g
```

## 启动

```bash
docker compose up -d
```

## 证书与 DNS 配置说明

- Traefik 配置为使用 Let's Encrypt 自动获取 HTTPS 证书。默认使用 HTTP Challenge 挑战方式验证（占用 80 端口自动重定向到 443）。
- 请确保域名的 DNS A 记录已解析到您的服务器 IP：
  - `*.<DOMAIN>` （泛解析） 或
  - `traefik.<DOMAIN>` 和 `portainer.<DOMAIN>`
- 证书会持久化存储在 `../../data/traefik/acme/acme.json` 中。

## CN 适配（镜像源）

由于基础层（Base Stack）全部使用标准的 Docker Hub 镜像，无 `gcr.io` 或 `ghcr.io`，建议您在国内使用时，配置系统的 `daemon.json` 中的 `registry-mirrors` 进行加速：
```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io"
  ]
}
```
对于 Watchtower、Portainer 等，也可选择替换为 `docker.m.daocloud.io/` 前缀的镜像源。
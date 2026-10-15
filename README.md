# Bubble Emby Cover

为 Emby 媒体库生成封面，内置 **41 个模板**，支持多台 Emby、模板预览、静态与动态封面、定时更新和原封面恢复。

本仓库提供使用说明和 Docker 部署配置。源码在私有仓库维护，由 GitHub Actions 构建镜像并发布到 Docker Hub 的 `bubbleemby` 账号。镜像同时支持 Linux AMD64 和 ARM64，Docker 会自动选择服务器对应的架构。

镜像地址：`bubbleemby/bubble-emby-cover:latest`。

## 部署

下载本仓库的 `docker-compose.yml`，放到服务器的 `/opt/bubble-emby-cover` 目录。所有参数直接写在 Compose 中，不需要额外的配置文件。

编辑 `docker-compose.yml`：

- `environment.ADMIN_PASSWORD`：直接填写自己的管理员密码；默认留空，每次启动生成随机密码，在 1Panel 容器日志或 `docker compose logs bubble-emby-cover` 中查看。不要填写「请改成你的密码」，服务会拒绝使用该占位值启动。
- `environment.ADMIN_USERNAME`：管理员用户名，默认 `admin`。
- `ports`：默认 `8088:8080`，需要换访问端口时修改左侧的 `8088`。
- `image`：默认使用 `latest`；需要固定版本时将末尾标签改为对应版本，例如 `v1.0.0`。

启动：

```sh
cd /opt/bubble-emby-cover
docker compose pull
docker compose up -d
```

打开 `http://服务器IP:8088`，使用 `admin` 和设置的密码（留空时使用日志中的随机密码）登录。在「设置」里添加 Emby 地址和 API Key。

Emby 地址需能从容器内访问。填写 Emby 所在服务器的内网地址；容器里的 `127.0.0.1` 指向封面工具自身。

若使用 1Panel 反向代理，可在 `environment` 中增加 `TRUSTED_PROXIES`，填反向代理连接本容器时的源 IP，多个地址用逗号分隔。默认不信任 `X-Real-IP`、`X-Forwarded-For` 等请求头，登录限流按直连来源计算。只信任你控制的代理，并让代理覆盖 `X-Real-IP` 和 `X-Forwarded-Proto`、正确追加 `X-Forwarded-For`；不要填写全部内网网段。

Emby 重定向只允许同协议、主机和端口；如果 Emby 会跳转到另一个地址，请直接填写最终地址。修改已保存的 Emby 地址后，需要重新填写 API Key。

### 使用 1Panel

1. 通过文件管理，把 `docker-compose.yml` 放到上述目录，直接修改其中的用户名、密码等参数。
2. 在「容器 → 编排 → 创建 → 路径选择」中，选择 `/opt/bubble-emby-cover/docker-compose.yml` 并部署。
3. 如需通过 IP 直接访问，在防火墙和安全组中允许所配置的端口。

镜像公开后，拉取无需 GitHub 登录，服务器也不需要安装 Go、Node 或数据库。

## 更新

```sh
cd /opt/bubble-emby-cover
docker compose pull
docker compose up -d
```

数据保存在 `/opt/bubble-emby-cover/data`，包括数据库、原封面备份、上传字体和生成结果。升级和重建容器时请保留该目录。

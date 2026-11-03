# wildstring-lzcapp

[qingmuhy744/wildstring](https://github.com/qingmuhy744/wildstring) 的懒猫微服（LazyCat）打包。

**野弦 WildString** —— AI 原生的自托管私人音乐播放器。从自己的音乐收藏出发，用自然语言
找歌、生成歌单；逐字歌词、多曲库隔离、播客托管。

## 模式

- **多实例**：每个用户点击应用图标启动独立容器，数据天然隔离（`multi_instance: true`）。
- **镜像**：直接使用上游 GHCR 镜像 `ghcr.io/qingmuhy744/wildstring`，由 Action 转存到
  `registry.lazycat.cloud`；版本跟随上游镜像 tag。
- **双商店发布**：官方商店 + 喵喵（私有）商店。

## 数据与目录

| 容器内路径 | 内容 | 宿主位置 |
| --- | --- | --- |
| `/data` | SQLite 数据库、托管曲库、缓存 | `/lzcapp/var/data` |
| `/podcasts` | 播客音频与封面 | `/lzcapp/var/podcasts` |
| `/lzcapp/documents` | 音乐导入源（只读） | 用户文稿目录，多实例下是当前用户的 |

镜像内 `/data` 与 `/podcasts` 已属于 uid/gid `10001`（上游 Dockerfile 里的 `chown`）。
manifest 里用 `run_as: "10001:10001"` 让系统把 `/lzcapp` 下的持久目录按同一身份映射
owner，再把 `/lzcapp/var/data`、`/lzcapp/var/podcasts` 挂到容器内的 `/data`、`/podcasts`，
这样挂载点依然归 uid 10001 所有，容器可以正常读写。

> `run_as` 需要 lzcos v1.6.0+，因此 `package.yml` 的 `min_os_version` 是 `1.6.0`。

管理界面默认账号是 `admin`，密码在安装时设置（`lzc-deploy-params.yml`，默认随机生成），
只在该账号还没有密码时生效。

## 说明

- 音频与元数据获取、AI 功能需要应用能访问公网（`net.internet` 权限）
- 导入本地音乐：把音乐放进网盘或系统媒体目录，应用会从 `/lzcapp/documents` 读取
- 建议使用支持 QWERTY 的物理键盘

## 所需 Secrets

| Secret | 说明 |
| --- | --- |
| `LZC_API_TOKEN` | 懒猫开发者平台 PAT（官方商店发布需要） |
| `APPSTORE_URL` / `APPSTORE_TOKEN` | 喵喵商店 API 地址与发布令牌 |
| `APP_ID` | 喵喵商店应用 ID |
| `PRIVATE_STORE_GROUP_CODES` | 私有分组码 |

## 许可

GPL-3.0（上游）

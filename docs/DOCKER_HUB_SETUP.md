# Docker Hub 认证配置指南

本指南帮助你配置 Docker Hub 认证，使 CI 工作流能够自动将镜像推送到 Docker Hub。

---

## 前置条件

- 一个 [Docker Hub](https://hub.docker.com/) 账号（免费注册）
- 对该 GitHub 仓库的 **Admin** 或 **Write** 权限（用于添加 Secrets）

---

## 第一步：创建 Docker Hub Access Token

> ⚠️ **安全原因**：Docker Hub 已不再支持账号密码登录 CLI/API，必须使用 Personal Access Token。

1. 登录 [Docker Hub](https://hub.docker.com/)
2. 点击右上角头像 → **Account Settings**
3. 左侧导航选择 **Personal access tokens**（或直接访问 https://app.docker.com/settings/personal-access-tokens）
4. 点击 **Generate new token**
5. 填写信息：

   | 字段 | 建议值 |
   |------|--------|
   | Token description | `GitHub Actions — livekit-dashboard` |
   | Access permissions | **Read & Write** |
   | Expiration date | 建议选择 **1 year** 并设置日历提醒续期 |

6. 点击 **Generate**
7. 🔴 **立即复制生成的 token**（格式为 `dckr_pat_xxxxxxxxxxxxxxxx`）—— 关闭页面后将无法再次查看

![Docker Hub Token](https://docs.docker.com/security/for-developers/access-tokens/images/generate-token.png)

---

## 第二步：在 GitHub 仓库中添加 Secrets

1. 打开你的 GitHub 仓库页面
2. 进入 **Settings** → **Secrets and variables** → **Actions**
3. 点击 **New repository secret**，分别添加以下两个 secrets：

### Secret 1：`DOCKERHUB_USERNAME`

| 字段 | 值 |
|------|-----|
| Name | `DOCKERHUB_USERNAME` |
| Value | 你的 Docker Hub **用户名**（不是邮箱） |

> 在 Docker Hub 页面右上角可以看到你的用户名，例如 `skiinder`

### Secret 2：`DOCKERHUB_TOKEN`

| 字段 | 值 |
|------|-----|
| Name | `DOCKERHUB_TOKEN` |
| Value | 第一步生成的 Access Token（`dckr_pat_...`） |

添加完成后，页面应如下所示：

```
Repository secrets:
  DOCKERHUB_TOKEN      *** (updated)
  DOCKERHUB_USERNAME   your-username
```

---

## 第三步：验证镜像名称

最终推送到 Docker Hub 的镜像地址为：

```
docker.io/<DOCKERHUB_USERNAME>/livekit-dashboard:<tag>
```

即常用的短格式：

```
<DOCKERHUB_USERNAME>/livekit-dashboard:latest
```

例如，用户名为 `skiinder`，则镜像地址为：

```bash
docker pull skiinder/livekit-dashboard:latest
```

---

## 第四步：触发构建

配置完成后，以下操作将自动触发 Docker 构建并将镜像同步推送到 **GHCR** 和 **Docker Hub**：

| 触发方式 | 说明 |
|----------|------|
| `git push` 到 `main` 分支 | 构建并推送 `latest` + `sha-xxxxx` 标签 |
| `git push` 一个 `v*` 标签 | 构建并推送语义版本标签（如 `v1.0.0`、`v1.0`） |
| 手动触发 | 在 Actions 页面点击 **Run workflow** |
| Pull Request | 仅构建验证，**不推送** |

---

## 双仓库镜像路径速查

| 仓库 | 路径格式 |
|------|----------|
| **GitHub Container Registry** | `ghcr.io/<owner>/livekit-dashboard:latest` |
| **Docker Hub** | `docker.io/<username>/livekit-dashboard:latest` |

---

## 常见问题

### Q: 为什么使用 Access Token 而不是密码？

Docker Hub 已于 2023 年停止支持密码方式的 API 认证。Personal Access Token 是唯一安全的 CI/CD 认证方式，且支持细粒度权限控制和过期时间设置。

### Q: Token 过期了怎么办？

1. 在 Docker Hub 重新生成一个 Token
2. 更新 GitHub 仓库中的 `DOCKERHUB_TOKEN` secret
3. 重新运行失败的 Action

### Q: 镜像名称可以自定义吗？

可以。修改 `.github/workflows/docker-build.yml` 中 `env.IMAGE_NAME` 的值，例如改为 `livekit-manager`。

---

## 安全提醒

- 🔒 永远不要将 Access Token 硬编码在代码或配置文件中
- 🔒 定期轮换 Token（建议每 90 天）
- 🔒 确保 `.env` 和包含敏感信息的文件已在 `.gitignore` 中

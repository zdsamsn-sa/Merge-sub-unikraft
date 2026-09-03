# Merge-sub → Unikraft Cloud 一键部署

将 [Merge-sub](https://github.com/eooce/Merge-sub)（代理订阅合并管理）部署到 **Unikraft Cloud**。

- 订阅合并（VMess / VLESS / Trojan / Hysteria2 / ss 等）
- Web 管理界面 + Basic Auth
- 支持 `?CFIP=&CFPORT=` 动态替换节点地址
- 数据卷持久化，重新部署不丢订阅/节点
- 手动触发 GitHub Actions 部署，不自动跑

---

## 首次配置（约 3 分钟）

### 1. 建仓库

把本目录推到你的 GitHub 仓库（建议 **Private**）。

### 2. 必填 Secret

仓库 → **Settings → Secrets and variables → Actions → New repository secret**

| Name | Value |
|------|--------|
| `UNIKRAFT_API_TOKEN` | Unikraft Cloud 控制台 → Settings → API Keys |

### 3. 强烈建议的应用凭证（Secret 或 Variables）

| 名称 | 说明 | 默认 |
|------|------|------|
| `USERNAME` | 管理界面用户名 | `admin` |
| `PASSWORD` | 管理界面密码 | `admin` |
| `SUB_TOKEN` | 订阅路径 token，访问 `https://实例域名/SUB_TOKEN` | 自动生成（不固定，**强烈建议固定**） |
| `API_URL` | 订阅转换 API | `https://sublink.eooce.com` |

> 推荐全部放进 **Secrets**，不要写进代码。

### 4. 可选 Variables

| 变量 | 默认 | 说明 |
|------|------|------|
| `DEPLOY_REGIONS` | `sin` | 地区，逗号分隔：`fra,sin,dal,sfo,was` |
| `PROJECT_NAME` | `merge-sub` | 项目名（实例/服务/镜像同名即更新） |
| `MEMORY_MB` | `512` | 每实例内存 |
| `APP_PORT` | `3000` | 应用端口 |
| `DATA_VOLUME` | `merge-sub-data` | 数据卷名前缀（按地区自动加后缀） |
| `VOLUME_MB` | `512` | 卷大小 MB |

---

## 部署

1. 代码 push 到 `main`（仅保存，**不会**自动部署）
2. Actions → **Deploy Merge-sub to Unikraft Cloud** → **Run workflow**
3. 选地区 / 内存 / 端口（可留默认）→ 运行

成功后日志末尾会打印：

```text
sin  running  https://xxxx.sin.unikraft.app
```

### 访问

| 用途 | URL |
|------|-----|
| 管理界面 | `https://实例域名/` （Basic Auth） |
| 订阅链接 | `https://实例域名/你的SUB_TOKEN` |
| CF 优选 | `https://实例域名/你的SUB_TOKEN?CFIP=1.1.1.1&CFPORT=443` |

---

## 更新 / 删除

- **更新**：改代码 push 后，再跑一次 Deploy workflow。同名项目会先删后建（中断约几十秒），**数据卷保留**。
- **删除**：Actions → **Destroy** → `target` 填项目名（如 `merge-sub`）或 `all`，再填 `DELETE` 确认。

---

## 目录结构

```text
├── app/                    ← Merge-sub 应用（已就位）
│   ├── index.js            入口（原 app.js）
│   ├── package.json
│   └── public/             Web 界面静态资源
├── scripts/                Unikraft 构建/部署脚本
├── .github/workflows/
│   ├── deploy.yml          部署到 Unikraft Cloud
│   └── destroy.yml         清理资源
└── README.md
```

应用监听 `PORT`（默认 3000），绑定 `0.0.0.0`，数据写在 `/app/data`（已挂载持久卷）。

---

## 可用地区

| 代码 | 地区 |
|------|------|
| `sin` | 新加坡 |
| `fra` | 法兰克福 |
| `dal` | 达拉斯 |
| `sfo` | 旧金山 |
| `was` | 华盛顿 |

---

## 注意事项

1. **务必修改默认密码**，并固定 `SUB_TOKEN`，否则重启后订阅路径可能变化。
2. Unikraft Cloud 不是完整 VPS：无 Docker-in-Docker / systemd；本应用是纯 Node 服务，可正常运行。
3. 注册表配额约 1GiB，同名镜像反复 push 可能触发 403，可在控制台删无引用的旧镜像。
4. 多地区部署时每个地区会有独立数据卷（`merge-sub-data-sin` 等），数据不跨地区共享。
5. token 只存在 GitHub Secrets / 运行环境中，不会出现在代码或公开日志里。

---

## 本地简单验证（可选）

```bash
cd app
npm install --omit=dev
USERNAME=admin PASSWORD=test SUB_TOKEN=mytoken PORT=3000 node index.js
# 浏览器打开 http://127.0.0.1:3000
```

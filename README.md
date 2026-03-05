# Emby Worker Proxy（Cloudflare Workers + D1）

基于 Cloudflare Workers 的 Emby 前后端分离反代。  
支持后台管理（`/admin`）、节点管理、分流/直连兼容与 D1 持久化存储。

---

## 功能特性

- 管理后台：`/admin`
- 节点增删改查（D1 存储）
- 普通 / 分离模式
- 第三方播放器链路兼容
- 第三方播放器一键导入
- 可选直连模式（按需开启）
- 节点缓存与列表缓存优化

---

## 快速部署

### 1）创建 D1 数据库

在 Cloudflare 控制台创建 D1 数据库（例如：`emby-proxy`）。

### 2）初始化数据表

在 D1 控制台依次执行以下 SQL：

`CREATE TABLE IF NOT EXISTS proxy_kv (k TEXT PRIMARY KEY, v TEXT NOT NULL, updated_at INTEGER NOT NULL);`

`CREATE INDEX IF NOT EXISTS idx_proxy_kv_k ON proxy_kv(k);`

### 3）上传 Worker 代码

将仓库内 `worker.js` 部署到 Cloudflare Workers。

### 4）绑定 D1

在 Worker → 设置 → 绑定 中添加 D1：

- 变量名：`EMBY_D1`
- 数据库：你创建的 D1

### 5）配置环境变量

在 Worker → 设置 → 变量 中添加：

- `ADMIN_TOKEN`（必填）

可选变量：

- `ENABLE_DIRECT_PROXY`：`0` 或 `1`（默认 `0`）
- `RAW_ALLOW_HOSTS`：直连白名单，逗号分隔
- `CAPY_STRIP_EMBY`：兼容性开关（默认 `0`）
- `CORS_ALLOW_ORIGIN`：自定义 CORS 来源（可留空）

### 6）绑定自定义域名（优选域名）

在 Worker → 触发器（Triggers）→ 自定义域（Custom Domains）添加入口域名，例如：

- `emby.yourdomain.com`

建议优先使用 Custom Domain 方式，路由更稳定、维护更简单。

### 7）DNS 配置说明（含 `*` 记录）

如果使用 Custom Domain，DNS 记录通常会自动处理。  
如果你采用手动 DNS / Route 方式，可按需添加：

- `CNAME emby -> 你的 workers.dev 子域（橙云）`
- 如需泛子域访问，再加：`CNAME * -> 你的 workers.dev 子域（橙云）`

并在 Worker Routes 中配置（示例）：

- `emby.yourdomain.com/*`
- `*.yourdomain.com/*`（仅在确实需要泛子域时启用）

### 8）访问后台

打开：`https://你的域名/admin`  
使用 `ADMIN_TOKEN` 登录后即可添加节点。

---

## 更新发布建议

每次更新代码建议按以下流程：

1. 修改 `worker.js`（本地或 GitHub 网页）
2. 提交 commit（例如：`fix: xxx`）
3. 如有文档变化，同步更新 `README.md`
4. 发布新版本 Tag / Release（如 `v1.0.1`）

---

## 安全建议

- 不要把真实 `ADMIN_TOKEN`、数据库 ID、私有域名提交到公开仓库
- 建议定期轮换 `ADMIN_TOKEN`
- 公网服务建议配合访问控制策略（如 WAF / Access）

---

## 免责声明

本项目仅供学习与技术测试使用。请遵守当地法律法规及服务条款，使用风险由使用者自行承担。

---

## License

MIT

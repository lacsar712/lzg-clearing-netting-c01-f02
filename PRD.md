# 项目 01：多币种轧差清算工作台（Clearing Netting Workbench）

## 元信息（提交时对照填写）

| 字段 | 建议值 |
|------|--------|
| 任务类型 | 0-1 代码生成 |
| 任务难度 | 困难 |
| 语言/框架 | Java, Spring Boot 3, JPA, PostgreSQL, Vue 3, Vite, Element Plus, Hexagonal |
| 环境可复现等级 | 已容器化，可一键起环境 |
| Harness | Claude Code |

## 端口分配（禁止与参考项目 3264/8264/33264 冲突）

| 服务 | 宿主机端口 |
|------|------------|
| Frontend | http://localhost:3171 |
| Backend API | http://localhost:8171 |
| PostgreSQL | localhost:54371 |

## 禁止名单核对

- 不是电商订单/购物车；不是「权限 RBAC 后台」产品（仅允许演示用简易登录门禁）
- 不是纯报表看板；页面以**录入义务 → 执行轧差 → 查看净头寸 → 确认 settle** 操作为主
- 核心仍是多边轧差算法与批次状态机

## 首轮 Prompt（可直接粘贴给 Claude）

```text
请从零实现「多币种轧差清算工作台」全栈项目（必须有前端页面 + 后端 API + 数据库），并支持 Docker 一键启动。

## 技术栈（必须遵守）
后端：
1. Java 17+ / Spring Boot 3.x
2. 架构：Hexagonal。domain 不得依赖 Spring/JPA 注解
3. PostgreSQL + Spring Data JPA（仅 adapter 层）
4. REST + JSON；错误 envelope：{ "code", "message", "details" }

前端：
5. Vue 3 + Vite + Element Plus + Vue Router + Pinia
6. 必须有真实可点的页面（不要只给静态说明页）；中文 UI
7. 前端通过相对路径 /api 反代到后端（nginx 或 Vite 生产代理均可）

基础设施：
8. 仓库根目录提供 docker-compose.yml，服务至少含：db / backend / seed / frontend
9. 一键：`docker compose up --build`
10. 首次启动 seed 灌入演示会员与若干 OPEN 义务
11. npm/前端构建若需加速可用 https://registry.npmmirror.com
12. 端口映射必须严格如下（容器内端口可自定，宿主机必须一致）：
    - frontend: 3171
    - backend:  8171
    - postgres: 54371
13. 禁止占用 3264、8264、33264

## 简易登录（门禁，不是 RBAC 产品）
- 仅两个演示账号写入 seed：operator/op123456、viewer/view123456
- operator 可执行轧差与 settle；viewer 只读
- 不要做角色权限配置后台、菜单权限矩阵等 RBAC 模块

## 领域模型（必须）
- Member：memberId, name, status(ACTIVE/SUSPENDED)
- TradeObligation：obligationId, payerMemberId, payeeMemberId, currency, amount(BigDecimal scale=8), tradeDate, settleDate, status(OPEN/NETTED/SETTLED/CANCELLED)
- NettingRun：runId, settleDate, currency, status(CREATED/RUNNING/COMPLETED/FAILED), createdAt
- NetPosition：runId, memberId, currency, netAmount（正应收负应付）
- 首期仅单币种轧差；混币种请求必须拒绝

## 轧差算法（必须正确）
指定 settleDate+currency：取 OPEN 义务 → 拒绝 SUSPENDED 会员与非法金额 → 多边净额 → ΣnetAmount 必须为 0 → 义务改 NETTED → 写 positions → Run COMPLETED；失败整批回滚 FAILED。

## 后端 API（必须）
- POST /api/auth/login
- GET  /api/members
- POST /api/members（operator）
- GET/POST /api/obligations
- POST /api/netting-runs
- GET  /api/netting-runs/{id}
- GET  /api/netting-runs/{id}/positions
- POST /api/netting-runs/{id}/settle
- GET  /api/health

## 前端页面（必须全部可访问）
1. 登录页
2. 工作台首页：今日待轧差义务摘要、最近 Run 列表（表格，不要做成空洞大盘）
3. 会员管理页：列表 + 新建 + 启停
4. 义务录入页：表单录入 + 筛选列表
5. 轧差执行页：选择 settleDate/currency → 执行 → 展示净头寸表与守恒结果
6. 批次详情页：状态、参与义务、settle 按钮

## README 必须包含（参考风格）
- How to Run：`docker compose up --build`
- Services 端口表（3171 / 8171 / 54371）
- 测试账号
- Verification 步骤（登录→录入/使用 seed 数据→轧差→看净头寸→settle）

## 测试
- 后端单元测覆盖轧差守恒与非法会员拒绝
- 完成后自行 compose 起来用 curl/浏览器验证再宣称完成
- .gitignore；密钥不进仓
```

## 建议后续轮次

1. Feature：FAILED run 诊断明细页
2. Feature：同日同币种并发防重
3. Bugfix：前端权限按钮显隐与后端鉴权不一致

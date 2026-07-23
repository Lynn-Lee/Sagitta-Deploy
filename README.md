# Sagitta Control

Sagitta Control（矢准数据库安全管控平台）是面向企业 DBA、数据安全、研发和运维团队的私有化数据库安全管控平台。本仓库是用户下载、部署、升级和验证 Sagitta Control 的公开入口，包含固定版本 Docker Compose、Helm Chart、上线检查脚本、升级脚本、授权校验工具、产品文档和功能截图，不包含后端或前端源码。

当前发布版本：`3.0.0`

## 项目定位

Sagitta Control 聚焦企业数据库访问与变更的“安全控制面”：把数据库实例接入、SQL 工单审批、在线查询授权、敏感数据脱敏、数据字典、SQL 洞察、运行诊断、数据归档、审计追踪、商业授权和上线运维工具统一到一个可交付、可审计、可运维的平台中。

平台不替代企业现有数据库、堡垒机、备份系统、账号体系或网络安全边界，也不自动创建生产数据库账号、授权 SQL、目标库对象或基础设施资源。生产数据库、网络连通、访问账号、备份策略和最小权限仍由客户 DBA、运维或安全团队按内部规范准备；Sagitta Control 负责在已授权连接范围内执行管控流程、记录审计证据、校验权限边界并提供统一操作入口。

部署时需要准备 PostgreSQL、Redis、Sagitta Control 后端与前端服务，以及商业授权所需的客户 ID、部署 ID 和 License 配置。单机部署可直接使用本仓库内置 Docker Compose；Kubernetes 环境可使用 `helm/sagitta-control` Chart，并按客户现场的存储类、Ingress、证书、外部 PostgreSQL / Redis 和 Secret 管理规范覆盖 values。

## 核心能力

- 数据库实例管理：统一维护 MySQL、PostgreSQL、Oracle、SQL Server、ClickHouse、Elasticsearch、OpenSearch、Cassandra、MongoDB 等多引擎连接信息，支持连通性测试、数据库同步、标签和资源组治理。
- SQL 工单治理：覆盖 SQL 检查、提交、审批、执行、风险标记和状态跟踪，支持审批流、工单模板、执行审计和结果回溯。
- 在线查询管控：提供查询工作台、查询权限申请、授权范围校验、导出治理、查询历史和敏感数据脱敏，降低越权查询和数据外泄风险。
- 数据安全能力：提供强密码策略、2FA、HttpOnly Cookie 登录态、CSRF 防护、JWT 黑名单、字段级加密、操作审计和全局异常脱敏。
- 数据字典与权限治理：自动同步实例数据库和 Schema 元数据，结合用户、角色、用户组、资源组和功能权限形成分层访问控制。
- 观测与诊断：聚合实例状态、容量、会话、锁、事务、慢 SQL、SQL 洞察和采集健康，辅助 DBA 做运行巡检和故障定位。
- 数据归档：支持归档任务、批次记录、执行状态和日志留痕，帮助高频业务库按策略做数据生命周期治理。
- 交付与授权：内置商业授权激活、部署指纹绑定、完整性 Manifest 校验、上线检查、升级回滚和诊断包导出工具链。

## 最短部署路径

适合 Docker Compose 单节点试用、PoC 或小规模生产。正式生产部署前请完整阅读 [安装部署手册](docs/installation_deployment.md)，并按客户安全规范配置 HTTPS、反向代理、防火墙和备份策略。

```bash
git clone https://github.com/Lynn-Lee/Sagitta-Deploy.git
cd Sagitta-Deploy
cp .env.example .env
```

编辑 `.env`，至少检查并填妥以下字段：

```text
POSTGRES_PASSWORD
REDIS_PASSWORD
SECRET_KEY
AUTH_COOKIE_SECURE
LICENSE_CUSTOMER_ID
LICENSE_SERVER_URL
LICENSE_DEPLOYMENT_ID
APP_INTEGRITY_REQUIRED
MANIFEST_PUBLIC_KEY
BACKEND_PORT
FRONTEND_PORT
```

可使用交付脚本生成生产随机值并写入客户 ID：

```bash
bash prepare-go-live-env.sh --customer-id <customer_id>
```

启动前确认没有遗留占位值：

```bash
grep -n 'CHANGE_ME\|^LICENSE_CUSTOMER_ID=$' .env || true
```

上面命令没有输出未处理的关键配置后，继续启动：

```bash
docker compose config --quiet
docker compose pull
docker compose up -d postgres redis
docker compose run --rm backend alembic upgrade head
docker compose up -d
docker compose ps
curl -fsS http://localhost:8000/health
curl -fsS http://localhost/health
bash go-live-check.sh
```

默认访问地址：

- 控制台：`http://localhost`
- 后端健康检查：`http://localhost:8000/health`
- 前端健康检查：`http://localhost/health`

如果部署在服务器上，请把 `localhost` 替换成服务器内网地址、域名或反向代理地址。公网生产环境必须通过 HTTPS 访问前端入口，并设置 `AUTH_COOKIE_SECURE=true`。

## 文档入口

- [安装部署手册](docs/installation_deployment.md)：服务器准备、`.env` 配置、Docker Compose / Helm 部署、授权激活和上线检查。
- [产品使用手册](docs/user_manual.md)：从部署成功后的初始化配置开始，指导完成实例接入、权限治理、SQL 工单、在线查询、观测诊断、归档和审计操作。
- [运维升级手册](docs/operations_upgrade.md)：日常巡检、备份、升级、回滚、日志诊断和安全基线。

## 核心功能截图

### 数据看板

![数据看板](screenshots/02-dashboard-query.png)

### SQL 工单列表

![SQL 工单列表](screenshots/06-workflow-list.png)

### SQL 工单提交

![SQL 工单提交](screenshots/07-workflow-submit.png)

### 在线查询工作台

![在线查询工作台](screenshots/09-query-workbench.png)

### 观测中心实例总览

![观测中心实例总览](screenshots/12-monitor.png)

### SQL 洞察

![SQL 洞察](screenshots/12-monitor-sql-insight.png)

### 实例管理

![实例管理](screenshots/15-instance-management.png)

### 数据字典

![数据字典](screenshots/14-data-dictionary.png)

### 权限与用户治理

![用户管理](screenshots/16-user-management.png)

### 交付与支持

![交付与支持](screenshots/23-commercial-support.png)

## 发布包校验

如果下载 `releases/v3.0.0/Sagitta-Control-v3.0.0.zip` 固定版本包，请先校验 sha256：

```bash
shasum -a 256 -c releases/v3.0.0/Sagitta-Control-v3.0.0.zip.sha256
```

发布包、SBOM、镜像标签和文档由 Sagitta Control 源码仓库的发布流程生成。文档或截图单独更新时不应手工伪造发布 manifest、checksum 或签名文件。

## 镜像

- 后端 / Worker / Beat / Flower：`ghcr.io/lynn-lee/sagitta-control-backend:3.0.0`
- 前端：`ghcr.io/lynn-lee/sagitta-control-frontend:3.0.0`

生产环境不要使用 `latest`，请保留 `.env.example`、Docker Compose 和 Helm values 中的明确版本标签。升级时必须复用旧 `.env`，尤其是 `SECRET_KEY`、`LICENSE_CUSTOMER_ID` 和 `LICENSE_DEPLOYMENT_ID`，不要在升级时重新生成稳定部署标识。

## 安全提示

共享部署截图、日志和诊断包前，请先确认没有暴露服务器 IP、`.env`、License 文件、激活码、数据库密码、Token、客户库名、表名、SQL 结果或未脱敏客户数据。

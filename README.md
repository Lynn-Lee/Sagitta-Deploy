# Sagitta Control 用户部署包

Sagitta Control 是面向企业数据库安全管控场景的统一平台。此部署包包含固定版本 Docker Compose、Helm Chart、上线检查脚本、升级脚本、产品截图和标准文档，不包含后端或前端源码。

![交付与支持](screenshots/23-commercial-support.png)

## 文档入口

- [安装部署手册](docs/installation_deployment.md)：服务器准备、Docker Compose / Helm 部署、授权激活和上线检查。
- [产品使用手册](docs/user_manual.md)：平台初始化、权限模型、SQL 工单、在线查询、观测诊断、归档和审计操作。
- [运维升级手册](docs/operations_upgrade.md)：日常巡检、备份、升级、回滚、日志诊断和安全基线。

## 快速部署

```bash
cp .env.example .env
bash prepare-go-live-env.sh
docker compose pull
docker compose up -d postgres redis
docker compose run --rm backend alembic upgrade head
docker compose up -d
bash go-live-check.sh
```

## 运维脚本

- `prepare-go-live-env.sh`：检查 `.env` 中的 `SECRET_KEY`、`LICENSE_DEPLOYMENT_ID`、数据库密码和授权配置。
- `go-live-check.sh`：检查容器、健康接口、授权状态和上线前关键配置。
- `upgrade.sh`：执行固定版本升级、备份、迁移、重启和健康检查。
- `verify-license.sh`：检查授权配置和许可证状态。

共享部署截图、日志和诊断包前，请先确认没有暴露服务器 IP、`.env`、License 文件、激活码、数据库密码、Token 或未脱敏客户数据。

# SagittaDB Enterprise v2.2.0

SagittaDB Enterprise 是面向企业数据库治理场景的统一管控平台，覆盖数据库实例管理、SQL 工单、在线查询、查询权限、数据字典、数据脱敏、SQL 洞察、运行诊断、数据归档、审计追踪和商业交付验收。

本仓库是 SagittaDB Enterprise 的公开交付仓库。这里提供客户部署文件、安装脚本、Helm Chart、产品手册、运维升级指南和版本下载入口；后端源码、前端源码、商业构建脚本、签名私钥和 License 签发工具不在公开仓库中提供。

## 版本与镜像

- 后端：`ghcr.io/lynn-lee/sagittadb-backend:2.2.0`
- 前端：`ghcr.io/lynn-lee/sagittadb-frontend:2.2.0`

生产环境请始终使用明确版本标签。首次部署会自动进入 60 天全功能试用期；试用到期后业务功能将暂停，仅保留登录和授权管理入口。在线授权默认需要至少每 7 天成功联网刷新一次；长期离线部署请使用 challenge-response 离线授权。

## 文档

- [安装部署指南](docs/installation.md)
- [运维升级指南](docs/operations-upgrade.md)
- [产品使用手册](docs/product-manual.md)
- [法律提示](LEGAL-NOTICE.md)

## Docker Compose 快速开始

```bash
cp .env.example .env
./prepare-go-live-env.sh --customer-id <customer_id>
# 按现场信息确认 .env 中的授权、域名、端口和通知配置。
docker compose pull
docker compose up -d postgres redis
docker compose run --rm backend alembic upgrade head
docker compose up -d
docker compose ps
```

前端服务健康后，访问 `http://<server>/`。

正式推广前执行上线门禁：

```bash
./go-live-check.sh \
  --api-base-url http://<server>:8000 \
  --frontend-url http://<server>/ \
  --username <admin> \
  --password '<password>'
```

该脚本要求生产密钥、正式 License、客户 ID、部署指纹、至少一个活跃实例、实施交付向导、验收报告、运行健康和推广就绪度全部通过。若管理员启用了 2FA，请改用 `--token <access_token>`。

## Kubernetes / Helm

仓库内包含 Helm Chart：

```bash
helm dependency update helm/sagittadb
helm upgrade --install sagittadb helm/sagittadb \
  -f helm/sagittadb/values-prod.yaml \
  --set app.secretKey='<random-secret>' \
  --set license.customerId='<customer-id>' \
  --set license.deploymentId='<stable-deployment-id>'
```

## 下载安装包

可以从本仓库的 GitHub Releases 下载完整部署包：

```bash
wget https://github.com/Lynn-Lee/SagittaDB-Enterprise/releases/download/v2.2.0/SagittaDB-Enterprise-v2.2.0.zip
wget https://github.com/Lynn-Lee/SagittaDB-Enterprise/releases/download/v2.2.0/SagittaDB-Enterprise-v2.2.0.zip.sha256
sha256sum -c SagittaDB-Enterprise-v2.2.0.zip.sha256
unzip SagittaDB-Enterprise-v2.2.0.zip
cd SagittaDB-Enterprise-v2.2.0
```

## 升级入口

```bash
./upgrade.sh 2.2.0
```

升级脚本会更新镜像版本、拉取镜像、备份 PostgreSQL、执行 Alembic 迁移并检查前后端健康状态。升级前请阅读 [运维升级指南](docs/operations-upgrade.md)。

## 离线镜像导入

如果服务器无法访问镜像仓库，请导入 SagittaDB 支持团队提供的镜像包：

```bash
docker load < sagittadb-backend-2.2.0.tar
docker load < sagittadb-frontend-2.2.0.tar
docker compose up -d
```

## License 与试用

登录后可在授权管理页面输入在线激活码完成授权，或生成离线 Challenge 后导入商务侧返回的 challenge-response 文件。也可以使用 `verify-license.sh` 验证在线激活、离线 Challenge 生成和刷新流程：

```bash
./verify-license.sh <activation_code> <customer_id>
```

SagittaDB Enterprise 使用统一授权中心 License-Server-Center，客户包默认授权服务地址为 `https://license.loveai.asia`。在线激活和联网刷新会由后端自动提交授权项目码 `sagittadb`，授权管理页应显示 `授权项目：SagittaDB（sagittadb）`。

在线激活前，在授权管理页输入客户 ID，页面会生成“正式激活客户 ID”和“正式激活部署指纹”。请将该正式激活部署指纹录入用户授权中心，再生成并交付激活码。复制指纹时，HTTPS 环境优先使用浏览器剪贴板能力；HTTP 试用部署会自动使用兼容复制方式。

生产环境默认不接受未绑定 Challenge 的裸 License JSON。
在线激活授权默认 `LICENSE_ONLINE_GRACE_DAYS=7`，超过宽限期未成功回源刷新时业务功能会暂停，直到授权刷新成功。

试用期结束或需要正式生产授权时，请联系 SagittaDB 商业支持，并提供授权管理页展示的正式激活部署指纹。

共享日志或配置时，不要打包 License 文件、私钥、激活码或 `.env` 中的敏感值。

## 安全边界

公开仓库和部署包不包含 SagittaDB 源码、签发工具、私钥、真实客户 License、真实激活码、客户数据库连接信息或内部验收记录。Release 同时提供客户包签名文件、前后端镜像 CycloneDX SBOM 及其校验/签名材料，便于客户侧供应链验收。

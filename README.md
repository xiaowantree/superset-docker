# Stock DE Superset 镜像

预装 **Trino SQLAlchemy 驱动** 和 **股票仪表盘 zip** 的 Apache Superset Docker 镜像。

不用 clone Superset 源码，不用 docker-compose，一行命令启动。

---

## 一行启动

```bash
docker run -d --name superset -p 8088:8088 vikiwu/stock-de-superset:latest
```

打开 <http://localhost:8088>，用 `admin / admin` 登录。

---

## 首次配置（两步）

### 1. 添加 Trino 数据库连接

Settings → Database Connections → **+ DATABASE** → 选 **Trino**，SQLAlchemy URI 填：

```
trino://admin@<你的-trino-主机>:8080/s3data/default
```

有密码就用：

```
trino://<用户>:<密码>@<主机>:<端口>/<catalog>/<schema>
```

点 **Test Connection** → 绿勾 → Save。

### 2. 导入股票仪表盘

Settings → **Import Dashboards** → Choose File → 选 `dashboard_export.zip` → Import。

zip 文件就打包在镜像里，用下面命令拿到本地：

```bash
docker cp superset:/app/dashboard_export.zip ./
```

导入时如果问 database 密码，就填你 Trino 的密码。

---

## 环境变量（可选）

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `ADMIN_USER` | `admin` | Superset 管理员用户名 |
| `ADMIN_PASSWORD` | `admin` | Superset 管理员密码 |
| `ADMIN_EMAIL` | `admin@example.com` | 管理员邮箱 |
| `SUPERSET_SECRET_KEY` | *(镜像内置)* | **生产环境强烈建议自己覆盖** |

示例：

```bash
docker run -d --name superset -p 8088:8088 \
  -e ADMIN_PASSWORD=MyStr0ngPass \
  -e SUPERSET_SECRET_KEY=$(openssl rand -base64 42) \
  vikiwu/stock-de-superset:latest
```

---

## 数据持久化

默认情况下 Superset 的元数据（用户、仪表盘、连接）存在容器内部，`docker rm` 会丢。要持久化，挂一个 volume：

```bash
docker run -d --name superset -p 8088:8088 \
  -v superset_home:/app/superset_home \
  vikiwu/stock-de-superset:latest
```

---

## 前置条件

镜像**只含 Superset**。你需要自己准备：

- 一个可访问的 Trino / Starburst 实例
- 表 `current_day_stock_price`（或其它仪表盘依赖的表）已经建好、有数据

镜像不负责部署 Trino、不负责跑数据管道。

---

## 自己 build（可选）

如果不想用 Docker Hub 上的镜像，本仓库包含所有源文件，可以自己构建：

```bash
git clone https://github.com/xiaowantree/superset-docker.git
cd superset-docker

# 需要 dashboard_export.zip 放在当前目录
# （从 vikiwu/stock-de-superset 镜像里拷出来，或从主项目 superset/ 目录复制）

docker build -t my-stock-superset:dev .
docker run -d -p 8088:8088 my-stock-superset:dev
```

---

## 镜像内容

| 组件 | 版本 |
|------|------|
| Apache Superset | 3.1.3 |
| sqlalchemy-trino | 0.5.0 |
| trino (python client) | 0.328.0 |
| Python base | apache/superset 官方镜像 |

---

## 相关链接

- Docker Hub 镜像：<https://hub.docker.com/r/vikiwu/stock-de-superset>
- 完整数据工程项目（含 Trino + Airflow + 数据管道）：<https://github.com/xiaowantree/DE-superset>
- Apache Superset 官方文档：<https://superset.apache.org/>

---

## License

MIT

# Stock DE Superset 镜像

预装 **Trino SQLAlchemy 驱动** 的 Apache Superset Docker 镜像。

不用 clone Superset 源码，不用 docker-compose，一行命令启动。

---

## 为什么做这个镜像

官方的 Apache Superset 想跑起来、再连上 Trino，其实并不轻松：

- **要 clone 整个 Superset 源码仓库**，再 `docker-compose up`，十几个容器（Redis、Postgres、Worker、Beat、Node 等）一起拉，首次启动好几分钟，网络稍差就各种 timeout
- Superset **官方镜像默认不带 Trino 驱动**，得自己改 Dockerfile、`pip install sqlalchemy-trino`，然后重新 build —— 不熟 Docker 的人很容易卡在这里
- 连接 URI 格式、CSRF、SECRET_KEY、metadata 数据库初始化……**一堆环境变量和配置坑**，官方文档又散落在不同页面
**这个镜像把这些都解决了：**

✅ 基于 `apache/superset:3.1.3` 固定版本，**预装 `sqlalchemy-trino` + `trino` 客户端**，Trino 连接开箱即用
✅ 单容器，**一条 `docker run` 起服务**，不需要 docker-compose、不需要额外的 Redis / Postgres
✅ `SECRET_KEY`、metadata DB、admin 账号、gunicorn 启动参数**全部配好**，环境变量可以覆盖

所以用户只要：有一个能访问的 Trino、能跑 Docker，一条命令就能起 Superset。

---

## 一行启动

```bash
docker run -d --name superset -p 8088:8088 vikiwu/stock-de-superset:latest
```

打开 <http://localhost:8088>，用 `admin / admin` 登录。

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

## 配置项

通常一个 web 服务器不止有 web app，还需要数据库、任务队列、消息队列、websocket等，他们中有些配置是重叠的有些是独有的，本文的目的即能在一个地方管理所有变量（配置项）。

将变量抽象为与任何 `app` 无关的单独 `module`，app 可以是 web app、task app、websocket app、DB(Docker) app 等。

## 如何区分 production or development

开发和部署变量显然需要区分，现有的方案有：

- 在一个文件中添加不同前缀
- 不同文件定义相同变量

不同文件:

- 需要一个全局变量 STATE 来控制加载哪个文件。
- 清晰区分两种状态。
- 变量名称不至于过长。

两种前缀：如 PROD_ or DEV_ 这种在一个文件内

- 也需要指定一个全局变量 STATE 确定当前运行状态。
- 不用来回切换文件进行输入，确保变量没有丢失。
- 减少配置文件数量。

最后选择多文件支持，在开发时能够专注开发环境，不至于混乱。

- .env      # Common variables
- .env.dev  # Development variables
- .env.prod # Production variables

TODO: 大多数 app 都有自动加载 .env 的特性，且有些在加载 env 文件时只能加载一个，所以现在实际使用了 `.env.dev` 和 `.env.prod` 两个文件，`.env` 可能后期会变成 dev 专属，从而去掉 `env.dev`。

!!! danger "警告"

    env 文件不要加入公开仓库。

## .env 内容

统一放置环境变量，不同 APP 设置不同前缀。全部大写。

比如：

- CELERY_
- WEB_
- MYSQL_
- REDIS_
- COMPOSE_
- SMS_
- COS_

.env.dev 与 .env.prod 一般成对出现。

## Common and Secret

- .env 等文件要始终保持排除在公开库的版本控制之外。
- .env.prod 中将私密项去掉，在 shell 中指定。（大部分 app 支持 变量优先级 shell 高于 env 文件）

## app 如何使用环境变量

大致可分为两类 Python 相关 app，和非 Python 相关。

- 如果 Python 相关，如 web app、websocket app、Celery app 等，可以直接使用 pydantic（`config.Config`）
- 非 Python ，如 Docker 运行的 Mysql/Redis，则需要直接读取 .env、.env.prod/.env.dev。

### 开发环境

1. Web app (Flask)

```shell
# use flask
flask --env-file .env.dev --debug run

# or use pdm
pdm run dev
# dev = "flask --env-file .env.dev --debug run"
```

2. Mysql 或者其他 Docker 运行的服务

```shell
docker compose --env-file .env.dev up --build <service name>
```

或者直接全部 Docker 启动

```shell
docker compose --env-file .env.dev up --build

# or
pdm run docker
```

### 生产环境

web app 使用 gunicorn

```shell
docker compose --env-file .env.prod -f compose.prod.yaml up --build -d
# or
pdm run prod
```

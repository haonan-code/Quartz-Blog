## 前言
Docker 可以让我们把一个项目的所有依赖环境配置好，我们可以快速的运行起来，而无需处理环境的依赖问题；
而某个项目需要用到诸如数据库、Redis 等其他项目的时候，使用 Docker Compose 可以将所有的项目和项目依赖通过一个 yml 完全配置好，我们只需要通过一行命令就可以快速启动这个项目。
> Compose 使用的三个步骤：
> - 使用 Dockerfile 定义应用程序的环境。
> - 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
> - 最后，执行 docker-compose up 命令来启动并运行整个应用程序。

## 安装 Docker Compose
Ubuntu 系统，执行下列命令：
```bash
sudo apt install docker-compose
```
安装完成后，通过下列命令检查是否安装成功
```bash
docker compose version
```
## 配置文件
配置文件是 Docker Compose 的核心部分，设计两个服务进行讲解，一种为源文件构造，另一种是拉取现有的镜像部署。
```yaml
# compose.yml 
services:        # 定义服务
  web:           # Web 应用
    build: ./app   # 使用 ./app 目录下的 Dockerfile 构建镜像
    command: flask run --host=0.0.0.0 --port=5000   # 启动 Flask 应用
    ports:
      - "8000:5000"   # 本机8000端口映射到容器5000端口
    environment:
      # 数据库连接URL，这里通过服务名 db 连接 Postgres
      DATABASE_URL: postgresql://postgres:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
    depends_on:
      db:
        condition: service_healthy   # 等待数据库健康检查通过再启动

  db:            # 数据库服务（PostgreSQL）
    image: postgres:16   # 使用官方 Postgres 16 镜像
    restart: unless-stopped   # 意外退出时自动重启
    environment:
      POSTGRES_USER: postgres          # 数据库用户名
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}  # 从 .env 文件读取密码
      POSTGRES_DB: ${POSTGRES_DB}      # 默认数据库名
    volumes:
      - db_data:/var/lib/postgresql/data   # 数据持久化，保存到数据卷 db_data 之中
    healthcheck:   # 健康检查，保证数据库可用
      test: ["CMD-SHELL", "pg_isready -U postgres -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 3s
      retries: 10

volumes:        # 定义数据卷
  db_data:      # PostgreSQL 的数据卷

```

### Service - Web 服务（从源文件构造）
以 Web 服务为例，设计一个 Python Flask 项目作为示例：
```yaml
web:           # Web 应用服务（Flask 示例）
  build: ./app   # 使用 ./app 目录下的 Dockerfile 构建镜像
  command: flask run --host=0.0.0.0 --port=5000   # 启动 Flask 应用
  ports:
    - "8000:5000"   # 本机8000端口映射到容器5000端口
  environment:
    # 数据库连接URL，这里通过服务名 db 连接 Postgres
    DATABASE_URL: postgresql://postgres:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
  depends_on:
    db:
      condition: service_healthy   # 等待数据库健康检查通过再启动
```
- **build**：指定构建镜像的目录（./app 里需要有 Dockerfile）。
- **command**：覆盖容器启动命令，这里用 flask 启动了一个允许所有 IP 访问的，端口为 5000 的服务端。
> 注意：这里应该允许所有IP访问（0.0.0.0），而不是只允许本地访问（127.0.0.1），否则会导致无法从外部连接到容器。
- **ports**：端口映射，本机 8000 -> 容器 5000。
- **env_file**：从 .env 文件读取环境变量。
- **environment**：额外定义的环境变量，这里设置为数据库地址。
- **depends_on**：表示依赖关系，这里 web 服务要等到 db 服务通过了健康检查，才会启动。

### Service - db 服务（拉取现有的镜像）
```yaml
db:            # 数据库服务（PostgreSQL）
  image: postgres:16   # 使用官方 Postgres 16 镜像
  restart: unless-stopped   # 意外退出时自动重启
  environment:
    POSTGRES_USER: postgres          # 数据库用户名
    POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}  # 从 .env 文件读取密码
    POSTGRES_DB: ${POSTGRES_DB}      # 默认数据库名
  volumes:
    - db_data:/var/lib/postgresql/data   # 数据持久化，保存到卷 db_data
  healthcheck:   # 健康检查，保证数据库可用
    test: ["CMD-SHELL", "pg_isready -U postgres -d ${POSTGRES_DB}"]
    interval: 5s
    timeout: 3s
    retries: 10
```
- **image**：使用构建好的镜像。这里固定版本为16，如果使用 latest 可能会导致新旧版本的兼容性导致问题。
- **restart**：容器异常退出后会自动重启。
- **environment**：POSTGRES_USER/POSTGRES_PASSWORD/POSTGRES_DB 仅在**第一次**初始化数据目录时生效。之后数据由卷持久化，再改这些变量不会重置已有库。
- **volumes**：用 **命名卷**（db_data）作为文件的存储，后面说映射到容器内的存储目录。
- **healthcheck**：
	- **CMD-SHELL**：表示这条命令会在容器的 **shell** 里执行（相当于 bash -c）。
	- **pg-isready**：这是 PostgreSQL 自带的一个小工具，用来检测数据库是否可以连接。
	- **-U postgres**：指定用 postgres 这个用户去检测。
	- **-d ${POSTGRES_DB}**：指定要检测的数据库名字（这里我们使用的是变量，会从 .env 文件读取同名的环境变量值）。
### 数据卷
在 Docker Compose 中，数据卷的定义通常放在最底部的 `volumes` 字段中。我们可以为每个服务定义一个或多个数据卷，以便在容器之间共享数据或持久化数据。  
数据卷无需我们关心文件在机器的存放位置，Docker 会自动处理，而且数据卷也可以方便的在不同的容器之间共享。
```yaml
volumes:
  db_data:   # PostgreSQL 的数据卷
```
而实际上大多数情况是，一些项目需要大家`git clone`下来，在仓库的路径进行操作，如果我们希望直接把这个目录挂载到容器中，可以在 `volumes` 中进行配置。
```yaml
web:
  volumes:
    - ./app:/app
```
上面的配置将宿主机的 `./app` 目录挂载到容器的 `/app` 目录中，这样我们就可以在宿主机上直接修改代码，而容器内的应用会直接使用这个路径里的内容，同步改变的内容。
## 常用的命令
| **命令**                         | **作用**     | **备注**                                        |
| ------------------------------ | ---------- | --------------------------------------------- |
| docker compose up -d           | 后台启动所有服务   | -d 表示 **detached** 模式，不占用当前终端                 |
| docker compose down            | 停止并清理容器、网络 | 数据卷默认保留，如果需要一起清理，可以使用`docker compose down -v` |
| docker compose ps              | 查看当前服务运行状态 | 类似 docker ps，但只显示 Compose 管理的容器               |
| docker compose logs -f         | 查看日志（实时刷新） | -f 类似 tail -f，适合调试                            |
| docker compose exec <服务名> bash | 进入容器内部     | 比如：docker compose exec web bash 进入 web 容器     |
> 提示：
   如果只想启动单个服务，可以用`docker compose up -d web`来启动。
## 迁移 Docker 项目到 Docker-Compose
把现有的Docker项目迁移到Docker Compose，其实就是把我们使用的**各种参数**转换成**配置文件中的每一个字段**，并且填写进去。
这里附上一份简单的转换表：

|docker run|**Compose 字段**|**示例**|
|---|---|---|
|–name app|container_name|container_name: app|
|-p 8080:80|ports|ports: [“8080:80”]|
|-v host:ctr[:ro]|volumes|volumes: [“app_data:/var/lib/app”] 或 [“./cfg:/etc/app:ro”]|
|-e KEY=VAL|environment / .env|environment: [“KEY=${KEY}”]|
|–env-file .env|env_file|env_file: .env|
|–restart unless-stopped|restart|restart: unless-stopped|
|–network mynet|networks|networks: [“mynet”]|
|–health-cmd …|healthcheck.test|`test: [“CMD-SHELL”,“curl -f [http://localhost/health](http://localhost/health)|
|–cpus/–memory|deploy.resources（本地用 deploy 限制有限）|deploy: { resources: { limits: { cpus: “1.0”, memory: “512M”} } }|
|–dns/–add-host|dns / extra_hosts|extra_hosts: [“db.local:10.0.0.5”]|
|–log-driver|logging.driver|logging: { driver: “json-file” }|
示例如下：
```sh
docker run -d --name db \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=devpassword \
  -e POSTGRES_DB=mydb \
  --restart unless-stopped \
  postgres:16
```
转换成docker compose的配置文件，便得到了如下结果：
```yaml
services:
  db:
    image: postgres:16
    container_name: db
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL","pg_isready -U postgres -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 3s
      retries: 10
    ports:
      - "5432:5432"
    restart: unless-stopped

volumes:
  pgdata:
```
上面我们使用了一些环境变量，在这里我们可以有两种方式使用他们，第一种便是直接写入到配置文件之中，也就是：
```yaml
environment:
  POSTGRES_PASSWORD: devpassword
  POSTGRES_DB: mydb
```
但是这样的话，我们如果把这个配置文件分享给其他人用，或者是上传到Github仓库的时候，那就不是很安全了，所以我们还有第二种方式，也就是上面我使用的方式，我们新建一个`.env`文件来存储这些变量的值。
.env（与 yml 同目录）：
```bash
POSTGRES_PASSWORD=devpassword
POSTGRES_DB=mydb
```

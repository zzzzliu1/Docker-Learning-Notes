
## 3.4 Docker 容器使用

#### 3.4.1 创建并启动容器

在获取镜像并下载完成之后， 可以通过`docker run` 命令运行容器，这个命令的基本格式为：

```shell
$ docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

常用参数说明：

- `-d`：以后台模式运行容器。
- `-p`：端口映射，格式为 `主机端口号:容器端口号`，例如 `80:80`。
- `--name`：指定容器名称。
- `-it`：保持交互式终端连接。
- `-v`：挂载主机目录到容器，格式为 `-v 主机目录:容器目录`。
- `--rm`：容器停止时自动删除容器。
- `--env` 或 `-e`：设置环境变量，格式为 `--env 变量名=变量值`。
- `--network`：指定容器的网络模式。
- `--restart`：容器的重启策略。
- `-u`：指定用户运行，格式为 `-u 用户名`。

以`nginx` 为例，如果我们想要启动一个`nginx` 容器，可以通过以下命令进行启动。

```shell
$ docker run -d -p 80:80 --name nginx-container nginx
```

通过以上命令，Docker会进行以下操作：
1. 使用指定的 `nginx` 镜像构建容器。
2. 允许容器在后台运行。
3. 将容器的 `80` 端口映射到主机的 `80` 端口上。
4. 指定容器名称为 `nginx-container`。
5. 启动容器。

在容器启动后， 通过主机的 IP 地址就能看到 nginx 的默认欢迎页面了。

#### 3.4.2 启动已经终止的容器
如果容器因为某种原因停止运行，可以使用 `docker start容器ID ` 命令重新启动它。

> **注意**：如果每次运行容器使用 `docker run` 命令，则每次都都会创建一个新的容器实例，即使使用相同的镜像和参数也会生成不同的容器。


#### 3.4.3 终止容器
要终止正在运行的Docker 容器，可以使用` docker stop` 命令。这个命令会向容器发送 SIGTERM 信号，让容器有机会执行清理工作并正常关闭。如果容器在指定时间内没有停止，Docker 会强制发送 SIGKILL 信号终止容器。

```shell
$ docker stop nginx-container
nginx-container
```


#### 3.4.4 查看容器
在Docker 使用过程中，经常需要查看容器的运行状态和信息。Docker 提供了多个命令来查看容器。

1. 查看正在运行的容器

使用`docker ps` 命令可以查看当前正在运行的容器。这个命令会显示容器的ID、名称、使用的镜像、创建时间、状态和端口映射等信息。

```shell
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                NAMES
a1b2c3d4e5f6   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:80->80/tcp   nginx-container
```

2. 查看所有容器

`docker ps -a` 命令可以查看所有容器，包括已经停止的容器。这个命令的输出格式与`docker ps` 相同，但会显示更多容器。

```shell
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                      PORTS                NAMES
a1b2c3d4e5f6   nginx          "/docker-entrypoint.…"   5 minutes ago    Up 5 minutes                0.0.0.0:80->80/tcp   nginx-container
b2c3d4e5f6a1   redis:alpine   "docker-entrypoint.s…"   2 days ago       Exited (0) 2 days ago                             redis-test
```


3. 查看容器详细信息

`docker inspect` 命令可以查看容器的详细信息。 这个命令会返回 JSON 格式的数据，包含容器的配置、网络设置、挂载卷等完整信息。

``shell
$ docker inspect nginx-container
```

4. 查看容器日志
`docker logs` 命令可以查看容器的日志输出。 这个命令对于排查容器运行问题很有帮助。 加上 `-f` 参数可以实时查看日志输出。
```shell
$ docker logs nginx-container
$ docker logs -f nginx-container
```

5. 查看容器资源使用情况

`docker stats` 命令可以实时查看容器的资源使用情况，包括 CPU、内存、网络和磁盘 I/O。

```
$ docker stats
CONTAINER ID   NAME             CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
a1b2c3d4e5f6   nginx-container   0.00%     2.5MiB / 1.952GiB   0.13%     1.45kB / 648B     0B / 0B           2
```

#### 3.4.5 进入容器内部

当容器在后台运行时，我们可以通过 `docker exec` 命令来进入运行中的容器内部执行命令或查看状态。
```shell
$ docker exec -it nginx-container /bin/bash
```

以上命令会启动一个 bash shell 让我们能与`nginx-container` 容器进行交互，`-it` 参数会保持终端交互连接。
通过这种方式可以像操作普通 Linux 系统一样在容器内执行命令、查看文件或修改配置。容器启动后，Docker 会为它分配唯一的 ID 和名称，自动设置网络和存储，并根据镜像定义启动指定的进程。



#### 3.4.6 容器的导出和导入

容器的导出和导入功能用于将容器及其当前状态保存为文件，便于迁移或备份。

导出容器可以使用 `docker export` 命令，这个命令会将容器的文件系统打包成 tar 归档文件，但不包含容器的元数据、网络配置或卷信息。

```shell
$ docker export -o nginx-container.tar nginx-container
```

使用上面的容器导出命令会将名为 nginx-container 的容器导出到当前目录下的 nginx-container.tar 文件中。

容器导出后的文件可以通过 `docker import` 命令重新导入为镜像，导入时需要指定镜像名称和标签。

```shell
$ docker import nginx-container.tar my-nginx:v1
```

使用上面的导入命令会创建一个名为 my-nginx、标签为 v1 的新镜像。

#### 3.4.7 删除容器

当容器不再需要时，可以使用 `docker rm` 命令将其删除，这个命令需要指定容器 ID 或容器名称。

```shell
docker rm nginx-container
```

使用上面的删除容器命令，会删除名为 nginx-container 的容器。

> **注意**：如果容器正在运行，直接删除会失败，需要先使用 `docker stop` 停止容器，或者添加 `-f` 参数强制删除运行中的容器。


```总结：
Docker Image（镜像）
        ↓
docker run
        ↓
Docker Container（容器）
```

整个完整的生命周期：
``` text
创建容器
    ↓
启动容器
    ↓
查看容器
    ↓
进入容器
    ↓
停止容器
    ↓
重新启动容器
    ↓
导出容器（可选）
    ↓
删除容器

```

1. 创建容器docker run
```bash
docker run [OPTIONS] IMAGE
```

例如：
```
docker run nginx
```

Docker 会：
1. 下载nginx镜像
2. 基于镜像创建容器
3. 启动容器
4. 执行镜像中的默认程序

python容器
``` bash
docker run -it python:3.13.11-slim
```

发生了什么：
```text
python image
      ↓
创建 container
      ↓
启动 python REPL
      ↓
进入交互模式
```

#### Docker run 常用参数
-d
后台运行
``` bash
docker run -d nginx
```
效果：
容器在后台运行
终端不会被占用


-it
交互模式
```bash
docker run -it ubuntu
```

效果：
保持终端连接
进入容器内部

--name
指定容器名字
``` bash
docker run --name my-nginx nginx
```
否则Docker 自动生成：
hungry_einstein
cool_turing
这种名字

-p
端口映射
```bash
-p 宿主机端口:容器端口
```
例如：
```
docker run -p 8080:80 nginx
```

表示：
``` text
浏览器访问：

localhost:8080
↓
容器内部：
80
```


-v

挂载 Volume
你课程中已经大量使用了。
例如：
```
docker run \
-v $(pwd)/test:/app/test \
python:3.13
```
意思：
```
宿主机目录：

test/

↓

容器目录：

/app/test
```

这样容器就能读取本机文件。


--rm
容器退出自动删除
```bash
docker run --rm hello-world
```
执行结束：
容器自动消失

不留垃圾容器。


-e
环境变量
你启动PostgreSQL 时已经用过。
```bash
docker run \
-e POSTGRES_USER=root \
-e POSTGRES_PASSWORD=root \
postgres
```
相当于： 
给容器内部注入配置





3. 查看容器

查看运行中的容器
``` bash
docker ps
```

例如：
``` text
CONTAINER ID
IMAGE
STATUS
PORTS
NAMES
```

查看所有容器
```bash
docker ps -a
```

4. 停止容器

``` bash
docker stop 容器名
```

例如：
```
docker stop postgres
```


5. 启动已经停止的容器
``` bash
docker start 容器名
```
例如：
```
docker start postgres
```

```text
docker start

启动旧容器


docker run

创建新容器
```


6. 进入容器
``` bash
docker exec -it 容器名 bash
```

进入后：
```
root@container:/#
```
你就在容器里面了

7. 查看日志
```
docker logs container_name
```

例如：
```
docker logs postgres
```

查看：
```
数据库启动日志
```

实时查看
``` bash
docker logs -f postgres
```
类似：
``` bash
tail -f
```


8. 查看容器详情
Inspect
``` bash
docker inspect postgres
```


9. 查看资源使用
stats
``` bash
docker statas
```

输出：
```
CPU

Memory

Network

Disk I/O
```

10. 导出容器
``` bash
docker export postgres > postgres.tar
```

得到：
```
postgres.tar
```

相当于： 把当前容器文件系统打包



11. 导入容器
Import
``` bash
docker import postgres.tar my-postgres:v1
```
变成：
```
my-postgres:v1
```


12. 删除容器

删除单个
``` bash
docker rm 容器名
```

例如：
``` bash
docker rm postgres
```

强制删除
``` bash
docker rm -f postgres
```
即使运行中也删除。


``` text
# Docker Container 常用命令

## 创建容器

docker run IMAGE

## 后台运行

docker run -d IMAGE

## 指定名称

docker run --name my-container IMAGE

## 端口映射

docker run -p 8080:80 IMAGE

## 挂载目录

docker run -v HOST_DIR:CONTAINER_DIR IMAGE

## 环境变量

docker run -e KEY=VALUE IMAGE

## 自动删除

docker run --rm IMAGE

## 查看运行容器

docker ps

## 查看所有容器

docker ps -a

## 停止容器

docker stop CONTAINER

## 启动容器

docker start CONTAINER

## 进入容器

docker exec -it CONTAINER bash

## 查看日志

docker logs CONTAINER

docker logs -f CONTAINER

## 查看详情

docker inspect CONTAINER

## 查看资源使用

docker stats

## 删除容器

docker rm CONTAINER

docker rm -f CONTAINER

## 导出容器

docker export CONTAINER > container.tar

## 导入镜像

docker import container.tar my-image:v1
```









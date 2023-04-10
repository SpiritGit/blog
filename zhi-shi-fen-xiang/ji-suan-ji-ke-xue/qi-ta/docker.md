# Docker

docker是一种轻量级、进程级VM

### docker常用命令

docker 下镜像

```bash
docker pull python
```

拷贝镜像

docker save -o PATH IMAGE

docker save -o ./my-golang-alpine3.7.tar golang:alpine3.7

docker load -i PATH

进入正在运行的容器

docker exec -i <容器名> /bin/bash

守护态运行容器

docker run -d IMAGE:TAG /bin/bash

启动已终止的容器

docker start <容器名>

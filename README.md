# docker
Tips for using docker
# 启动重启Docker服务
> sudo service docker start/stop/restart
# 查看运行的Docker容器
> sudo docker ps

# 查看已创建的容器相关配置
> docker inspect nginx

# 启动/停止/重启/运行 -- start/stop/restart/run
> docker restart nginx
# 创建和运行容器
> docker rm [容器name/id] # 删除容器
> docker rmi [镜像name/id] # 删除镜像
> docker run --privileged=true -p80:80 -v /root/nginx_docker/www:/www -v /root/nginx.conf:/etc/nginx/nginx.conf --name nginx -d docker.io/nginx
# docker清理占用卷
> 如果你的docker目录仍然占据着大量空间，那可能是因为多余的卷占用了你的磁盘。RM命令的-v命令通常会处理这个问题。但有时，如果你关闭容器不会自动删除容器，VFS目录将增长很快。我们可以通过删除不需要的卷来恢复这个空间。要做到这一点，有一个Docker镜像，你可以使用如下命令来运行它：
> docker run -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker:/var/lib/docker --rm martin/docker-cleanup-volumes
# 删除所有已经停止的容器
> sudo docker rm $(sudo docker ps -a -q)
# 杀死所有正在运行的容器
> sudo docker kill $(sudo docker ps -a -q)

# 备份镜像
> Usage
    docker save [OPTIONS] IMAGE [IMAGE...]
# 导出golang镜像
> sudo docker save --output golang.tar golang:1.2
# 从本地导入镜像
> Usage
    docker load [OPTIONS]
# 导入golang镜像
> sudo docker load --input golang.tar
# 导出容器快照
> Usage
     docker export [OPTIONS] CONTAINER
# 导出hello容器快照
> sudo docker export --output hello.tar
# 从容器快照导入镜像
> Usage
    docker import [OPTIONS] URL|- [REPOSITORY[:TAG]]
# 导入hello快照，并制定镜像标签为hello:1.0
> cat hello.tar | sudo docker import - hello:1.0
# 安装docker-compose:
> apt-get install docker-compose 
# 使用docker-compose创建和启动容器
> docker-compose up -d (默认使用当前目录下docker-compose.yml)
> docker-compose --f [file] up -d (制定file)
# 使用docker-compose管理容器
> docker-compose start/stop/restart/run/rm/ps nginx
> docker-compose run nginx bash #运行容器并启动bash
version: '2'

services:
  nginx:
    restart: always
    image: nginx
    container_name: nginx
    hostname: nginx
    stdin_open: true
    tty: true
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /data1/auto_upline/nginx:/data/service
      - /data1/Docker/nginx:/etc/nginx
      - /data1/log/nginx:/data/log/nginx
    command: nginx -g 'daemon off;'
    links:
      - beta_api_qa

  beta_api_qa:
    # 自动重启
    restart: always
    image: jdk8:latest
    container_name: beta_api_qa
    hostname: beta_api_qa
    # 工作目录，表示容器运行后所在目录
    working_dir: /opt/app
    stdin_open: true
    # 分配tty设备，该可以支持终端登录
    tty: true
    # 端口映射
    ports:
      - "8080:8080"
    # 文件映射
    volumes:
      - /data1/auto_upline/common:/opt/common
      - /data1/auto_upline/beta_api:/opt/app
    # 参数
    environment:
      JAVA_OPTS: "-Duser.timezone=GMT+08 -Xmx1G -Xms1G -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=512m"
    # 启动命令 -Dspring.profiles.active=dev表示使用dev配置
    command: java -jar -Dspring.profiles.active=dev app-alone.war

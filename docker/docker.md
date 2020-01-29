# docker
1. 卸载docker
```shell
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```  
2. 安装相关依赖  
```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```  
3. 添加下载镜像  
```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```  
4. 安装docker  
```shell
yum -y install docker-ce
```  
5. 开启docker  
```shell
systemctl enable docker && systemctl start docker
systemctl disable firewalld && systemctl stop firewalld
```  
6. 搜索镜像  
```shell
docker search mysql
```  
7. 拉取镜像  
```shell
docker pull mysql:5.6.43
```  
8. 创建容器
```shell
docker run -t -i mysql:5.6.43 /bin/bash
```  
9. 查看所有容器  
```shell
docker ps -a #查看所有容器，不带 -a 则只显示启动的容器
```  
10. 端口映射与磁盘映射
```shell
docker run -p 8808:3306 --name mysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6.43
```  
11. 查看容器启动日志
```shell
docker logs 7e166dfd27eb
docker logs --tail 100 -f 0491e185ae4e
```  
12. 容器开始停止和删除
```shell
docker start 7e166dfd27eb
docker stop 7e166dfd27eb
docker rm 7e166dfd27eb
```  
13. 进入docker控制台
```shell
docker exec -it 7e166dfd27eb /bin/bash
```  
14. 镜像的删除
```shell
#要将对应的所有容器删除，才能删除镜像
docker rmi <image id>
```
15. 打包一个java应用到docker
```shell
docker pull java:8
vi Dockerfile
```
```text
FROM java:8
MAINTAINER larry
RUN ["mkdir", "app"]
ADD mysql-travel-1.0-SNAPSHOT.jar /app
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/mysql-travel-1.0-SNAPSHOT.jar"]
```

```shell
docker build -t java:springboot .
#手动打标签
docker tag <image id> name:tag

docker run -p 8080:8080 --name springboot -d java:springboot
```  
16. Dockfile 参数

| 指令         |  描述                   |  常用    |  
| ----------- |:-----------------------:| :------: |  
| FROM        |  构建镜像的基础镜像       | <font color=#0099ff>*</font> |  
| MAINTAINER  |  镜像维护者姓名或邮箱地址  | <font color=#0099ff>*</font> |  
| RUN         |  构建镜像时运行的Shell命令 | <font color=#0099ff>*</font> |  
| CMD         |  运行容器时执行的Shell命令 | <font color=#0099ff>*</font> |  
| EXPOSE      |  声明容器运行的服务端口    | <font color=#0099ff>*</font> |  
| ENV         |  设置容器内环境变量        | - |  
| ADD         |  拷贝文件或目录到镜像，可以自动解压缩或者下载 | <font color=#0099ff>*</font> |  
| COPY        |  拷贝文件或目录到镜像，不能自动解压缩 | <font color=#0099ff>*</font> |  
| ENTRYPOINT  |  容器启动后执行的shell命令 | <font color=#0099ff>*</font> |  
| VOLUME      |  指定容器挂载点到宿主机自动生成的目录或其他容器 | - |  
| USER        |  为RUN，CMD和ENTRYPOINT执行命令指定运行用户 | - |  
| WORKDIR     |  为RUN，CMD，ENTRYPOINT，COPY和ADD设置工作目录 | <font color=#0099ff>*</font> |  
| HEALTHCHECK |  健康检查                 | - |  
| ARG         |  在构建镜像时指定一些参数  | - |  

17. 打包镜像  
```shell
docker save springbootdemo:latest > springbootdemo.tar
```  
18. 构建本地镜像库  
```shell
docker pull registry
docker run -d -p 5000:5000 --restart=always --name registry registry
docker push localhost:5000/app:spring-boot
curl http://192.168.127.100:5000/v2/app/tags/list
http://192.168.1.100:5000/v2/_catalog

# docker push error:http: server gave HTTP response to HTTPS client
1.vim  /etc/docker/daemon.json    增加一个daemon.json文件
{ "insecure-registries":["192.168.1.100:5000"] }
保存退出

2.重启docker服务
systemctl daemon-reload
systemctl restart docker

3.重启容器
4.上传镜像
```
19. 安装docker-compose  
```shell
curl -L https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
# 自动补充命令
curl -L https://raw.githubusercontent.com/docker/compose/$(docker-compose version --short)/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```  
20. 配置服务数量  
```shell
docker-compose up --scale spring-cloud-server=2
```
21. 相关链接  
[Docker Compose 入门](http://book.itmuch.com/3%20%E4%BD%BF%E7%94%A8Docker%E6%9E%84%E5%BB%BA%E5%BE%AE%E6%9C%8D%E5%8A%A1/3.8.2%20Docker%20Compose%E5%85%A5%E9%97%A8%E7%A4%BA%E4%BE%8B.html "Docker Compose")  
[Docker Compose 部署 Spring Cloud](https://www.cnblogs.com/sweetchildomine/p/7440262.html)  
[同上](https://medium.com/@madhupathy/simplified-microservices-building-with-spring-cloud-netflix-oss-eureka-zuul-hystrix-ribbon-2faa9046d054)  
[docker网络通信](https://docs.docker.com/network/)  
22. 镜像加速  
```shell
# https://www.docker-cn.com/registry-mirror
docker pull registry.docker-cn.com/library/

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://pc8jr851.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
23. 常用容器  
```shell
docker run -p 6379:6379 --name redis -d redis
docker run -p 3306:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
-- centos7 下上面的mysql安装会提示访问/var/lib/mysql目录没有权限，需要使用--privileged=true
docker run -p 3306:3306 --name mysql --privileged=true -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql/ -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
docker run -p 8529:8529 -e ARANGO_ROOT_PASSWORD=openSesame --name arango arangodb/arangodb:3.4.5
```
24. 常用镜像
```shell
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.0.0
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --name es docker.elastic.co/elasticsearch/elasticsearch:7.0.0

docker pull docker.elastic.co/kibana/kibana:7.0.0
-- docker run -d -p 5601:5601 -e "elasticsearch.hosts=http://127.0.0.1:9200" --name kibana docker.elastic.co/kibana/kibana:7.0.0
docker run -d -p 5601:5601 -e ELASTICSEARCH_HOSTS=http://192.168.127.9:9200 --name kibana docker.elastic.co/kibana/kibana:7.0.0
-- 用其它容器开放的端口可用该参数 network 指定容器共享elasticsearch容器的网络栈 (使用了--network 就不能使用-p 来暴露端口)
kibana.yml
```
25. Dockerfile
```dockerfile
FROM docker.elastic.co/elasticsearch/elasticsearch:7.1.0
ADD ./config/elasticsearch.yml /usr/share/elasticsearch/config/
USER root
RUN chown elasticsearch:elasticsearch config/elasticsearch.yml
USER elasticsearch
```
```yaml
http.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-methods: OPTIONS, HEAD, GET, POST, PUT, DELETE
http.cors.allow-headers: "X-Requested-With, Content-Type, Content-Length, X-User"
xpack.security.enabled: false
# Uncomment the following lines for a production cluster deployment
# #transport.host: 0.0.0.0
# #discovery.zen.minimum_master_nodes: 1
```

```shell
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --name es -d es:7.1.0
```
26. elasticsearch kibana dockerfile
```dockerfile
FROM openjdk:jre-alpine

LABEL maintainer "nshou <nshou@coronocoya.net>"

ARG ek_version=6.5.4

RUN apk add --quiet --no-progress --no-cache nodejs wget \
 && adduser -D elasticsearch

USER elasticsearch

WORKDIR /home/elasticsearch

ENV ES_TMPDIR=/home/elasticsearch/elasticsearch.tmp

RUN wget -q -O - https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss-${ek_version}.tar.gz \
 |  tar -zx \
 && mv elasticsearch-${ek_version} elasticsearch \
 && mkdir -p ${ES_TMPDIR} \
 && wget -q -O - https://artifacts.elastic.co/downloads/kibana/kibana-oss-${ek_version}-linux-x86_64.tar.gz \
 |  tar -zx \
 && mv kibana-${ek_version}-linux-x86_64 kibana \
 && rm -f kibana/node/bin/node kibana/node/bin/npm \
 && ln -s $(which node) kibana/node/bin/node \
 && ln -s $(which npm) kibana/node/bin/npm

CMD sh elasticsearch/bin/elasticsearch -E http.host=0.0.0.0 --quiet & kibana/bin/kibana --host 0.0.0.0 -Q

EXPOSE 9200 5601
```
27. volume
```
docker volume ls
docker volume rm <VOLUME_NAME>
```

Docker 部署
===

快速 打包 测试 部署应用

各种软件 开发环境 打包成Images镜像 运行在Docker上

Docker客户端 通过daemon 和宿主机交互



Docker  pull 拉取 Registry 中通过Docker包装过的软件

docker hub 



docker run 新建启动容器，把镜像运行在容器   ： 运行指定的镜像

docker run -p 91:80  nginx :  

​		小盒子里面运行有nginx镜像 在小盒子的80端口

​		无法通过80直接访问小盒子  但是可以通过映射到的宿主机的91端口进行访问nginx



查看容器信息

​	docker inspect id   查看 盒子 ip

​	在宿主端  进行 crul  http：地址









部署微服务至docker

1新建 Dockerfile  宿主机

2 构建镜像：docker build  -t  nginx:名称 . Dockerfile 路径  .表示Dockerfile路径

3.打了tag 成为新的 镜像  运行






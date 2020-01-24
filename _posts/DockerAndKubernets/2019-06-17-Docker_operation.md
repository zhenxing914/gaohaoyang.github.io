

## 一 Docker file 基本知识

## 1.1 规则

### 1.1.1 Docker file 指令格式

```
指令(命令)	参数列表
```



### 1.1.2 Docker file 常用指令

很多用法都支持shell 格式，按照 shell 命令格式写即可。

常用命令解释：

-  添加文件、目录(源压缩包会解压，源链接会下载)
    ```
	ADD src dest
	```
	
- 拷贝文件内容(文件夹拷贝只拷贝文件夹下内容，不拷贝文件夹)

   ```
   COPY src dest
   ```

- 环境变量设置(支持 shell 设置方法，建议一个变量一行，好修改) 

  ```
  ENV
  ```

- 工作路径(没有会自动创建)

   ```
   WORKDIR 目 录
   ```

- 暴漏端口(默认 TCP 协议)

   ```
   EXPOSE 端口 1<：协议> 端口 2<：协议>
   ```

-   基镜像(可以有多个)

   ```
	FROM image:tag
   启动命令(只有最后一个会生效，建议用json 数组，建议用 ENTRYPOINT)
   ENTRYPOINT ["executable", "param1", "param2"]
	```

   

具体用法参考：

<https://docs.docker.com/v17.09/engine/reference/builder/#usage>

## 1.2 示例

```dockerfile

#用centso : 7 做基镜像

FROM centos:7

ADD jdk-8u201-linux-x64.tar.gz /usr/local


ENV JAVA_HOME /usr/local/jdk1.8.0_201

ENV PATH $JAVA_HOME/bin:$JAVA_HOME/sbin:$PATH

WORKDIR /usr/local/admin

COPY ./es-cluster-admin-0.0.1-SNAPSHOT-dev.jar /usr/local/admin


ENTRYPOINT ["java","-jar","es-cluster-admin-0.0.1-SNAPSHOT-dev.jar"]
EXPOSE 8090

```



# **二**	**Docker** **安装**

按照系统以及发行版安装即可，如果可以联网，复制粘贴就行。

 

详见：<https://docs.docker.com/install/linux/docker-ce/centos/>





# 三	docker 常用命令

-  docker build -t  镜像名字：镜像 tag -f dockerFile  **.**(猜测是docker file 执行目录)

-  docker image –help

-  docker image ls <-a> 查看 docker 镜像(-a 可以查看隐藏镜像)

-  docker container --help

-  docker container ls <-a> 查看容器(-a 可以查看关闭的容器)

-  docker run -d(后台运行) -p 宿主机端口：docker 容器端口 –name 容器名字 镜像名字：镜像tag

-  docker run -it(交互式运行) -p 宿主机端口：docker 容器端口 –name 容器名字 镜像名字：镜像tag

- docker exec -it **contain_ID** bash 进去容器内部,退出用 exit

 

docker 命令用法详见：<https://docs.docker.com/engine/reference/run/>



# 四	概念补充

​	Docker file 把资源打包成镜像image, 运行的 image 叫容器 container，run 命令每执行一次，都会重新起一个容器。对特定容器的操作，通过 docker container 操作去执行。


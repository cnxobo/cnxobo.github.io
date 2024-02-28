---
title: Spring Boot 构建docker镜像
tags:
  - docker
  - dockerfile
id: '879'
categories:
  - - ops
date: 2021-09-10 21:58:20
---

Spring官网提供了一个简明的基于SpringBoot项目构建docker镜像的教程（[Getting Started Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/))。这个教程给我了很大的启发，结合我现在实际项目，又进行一些改造。

## 引入多阶段构建

原始教程都是基于构建好的jar包做操作，这样就要求宿主机配置好JDK和Maven,这样就存在在不同的宿主机下，可能构建出不同的镜像，不够`The Docker Way`。 通过[多阶段构建](https://docs.docker.com/develop/develop-images/multistage-build/)我们一个在一个 Dockerfile 里使用多个`FROM`。每一个`FROM`都可以使用不同的基础镜像,没一个都开启一个构建的新阶段。 我们可以选择性的复制构建物从一个阶段到另一个，这样在最终的镜像里就不会有我们不需要的中间产物了。

```Dockerfile
FROM maven:3.8.2-adoptopenjdk-8 AS builder
WORKDIR /build
COPY . .
RUN mvn clean package

FROM adoptopenjdk:8-jdk-hotspot
COPY --from=builder /build/target/*.jar /app/
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

### 减少文件复制

构建时.git，target等目录是不需要的，创建`.dockerignore`文件:

```
.git/
target/
.idea/
Dockerfile
```

### 减少与Maven服务器交互

如果项目严格按照每次部署jar包时版本号都改变，可以认为pom文件没发生变化时maven不需要向服务器更新依赖. `mvn dependency:go-offline` 可以下载所有的项目依赖，后续的所有mvn操作我们都可以添加`-o`参数，maven就工作的离线模式，不再需要和maven服务器交互，提高构建速度。

```Dockerfile
FROM maven:3.8.2-adoptopenjdk-8 AS builder
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline
COPY . .
RUN mvn  -o clean package

FROM adoptopenjdk:8-jdk-hotspot
ARG JAR_FILE=target/*.jar
COPY --from=builder  ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

### 缓存maven repository

现在每个项目的构建都有一个独立的本地maven repository，极大的浪费磁盘空间，同时也浪费网络带宽。共享maven repository有两个选择： 一个是通过挂载`VOLUME`,或者利用`BuildKit`(>=18.09)的`Cache`. VOLUME:

```
VOLUME "/root/.m2"
RUN mvn xxx
```

BuildKit:

```
RUN --mount=type=cache,id=mvnrepo,target=/root/.m2 mvn xxx
```

#### 启用BuildKit

docker默认是不启用BuildKit的。可以在执行`docker build`之前设置环境变量`DOCKER_BUILDKIT=1`，例如:

```Shell
DOCKER_BUILDKIT=1 docker build .
```

如果想默认启用BuildKit，可以通过配置`/etc/docker/daemon.json`,然后重启daemon。

```JavaScript
{ "features": { "buildkit": true } }
```

## 更精细的镜像分层

原始教程镜像分为`/BOOT-INF/lib`,`/META-INF`,`/BOOT-INF/classes` 三层。对于普通项目来说是够用的，但是我司有一些自有jar包，更新频率高于三方的外部库，同时低于业务代码。为最大程序利用 docker 构建缓存及镜像层，我把`/BOOT-INF/lib`分为两层，一层是外部依赖的jars，一层是公司的jars。 jar包的分层我是通过maven的`dependency:copy-dependencies`并指定`excludeGroupIds`和`includeGroupIds`来实现的。

```Shell
mvn dependency:copy-dependencies -DexcludeGroupIds=org.xobo -DoutputDirectory=./target/lib/3rd
mvn dependency:copy-dependencies -DincludeGroupIds=org.xobo -DoutputDirectory=./target/lib/xobo
```

额外要注意: 1. 如果项目有更精细的分层需求，有一点要注意，COPY 虽然支持模糊匹配，但是在复制目录的时候会把匹配上的_目录内的内容_复制到目的文件夹下。这样文件夹目录少了一层。 2. [Maven 3.8.1禁止了HTTP repository, 必须使用HTTPS协议。](https://maven.apache.org/docs/3.8.1/release-notes.html) 可以升级HTTP为HTTPS,或者添加一个mirror，或者版本降到3.6.3。 3. 不要追求镜像的绝对最小，像alpine这种镜像很多命令、基础库都和我们日常使用的Linux不一样，虽然一共能省了不到100M空间(docker基础层是共享的)但留下很多隐患。

## 最终的Dockerfile:

```Dockerfile
FROM maven:3.8.2-adoptopenjdk-8 AS builder
WORKDIR /build
COPY pom.xml .
RUN  --mount=type=cache,id=mvnrepo,target=/root/.m2 mvn dependency:go-offline
COPY . .
RUN --mount=type=cache,id=mvnrepo,target=/root/.m2 mvn -B -o clean package && \ 
  mvn -o dependency:copy-dependencies -DexcludeGroupIds=org.xobo -DoutputDirectory=./target/lib/3rd && \ 
  mvn -o dependency:copy-dependencies -DincludeGroupIds=org.xobo -DoutputDirectory=./target/lib/my && \ 
  jar xf ./target/demo.jar

FROM adoptopenjdk:8-jdk-hotspot
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN adduser --system --no-create-home --group app
USER app:app
COPY --from=builder --chown app:app /build/target/lib/3rd /app/lib
COPY --from=builder --chown app:app /build/target/lib/my /app/lib
COPY --from=builder --chown app:app /build/BOOT-INF/classes /app
COPY --from=builder --chown app:app /build/META-INF  /app/META-INF 

EXPOSE 8080
ENTRYPOINT ["java","-cp","app:app/lib/*","org.xobo.DemoApplication"]
```
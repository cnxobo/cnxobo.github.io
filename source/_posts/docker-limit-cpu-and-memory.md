---
title: 在docker内设置内存与CPU限制
tags: []
id: '843'
categories:
  - - ops
comments: false
date: 2021-02-02 22:50:03
---

## 0x01 总览

在生产环境中，为了保证服务器不因某一个软件导致服务器资源耗尽，我们会限制软件的资源使用。同样的在使用docker的时候，我们可以对docker镜像限制内存与CPU资源限制。

## 0x02 使用docker run 时设置资源限制

我们在使用`docker run`的时候可以直接设置资源限制。

### 2.1 内存

假如我们希望容器的内存最多只使用512M。 我们通过 `--memory` 或 `-m` 参数设置内存限制。 参数后面跟一个数字，单位可以是b、 k、 m 或者 g, 最小值是 4M。

```Shell
docker run -m 512m nginx
```

我们也可以设置一个`保留内存(memory reservation)`。 这个值要小于`--memory`值。当docker检测到宿主机上内存较少的或存在`内存竞争(memory contention)`时才会启用该限制。但是因为是软限制，所以无法保证docker容器不超出这个限制。

```Shell
docker run -m 512m --memory-reservation=256m nginx
```

### 2.2 CPU

默认情况下，CPU的算力是不受限制的。我们可以通过`cpus`参数设置CPU限制。例如，我们限制容器最多使用两个CPU:

```Shell
docker run --cpus=2 nginx 
```

我们也可以指定CPU分配的优先级。默认优先级是1024, 更高的数值的优先级更高:

```Shell
docker run --cpus=2 --cpu-shares=2000 nginx
```

## 0x03 通过docker-compose 文件设置内存限制

我们也可以通过docker-compose配置文件实现类似的效果。要注意的是，对于不同版本的docker-compose格式会有不同的。

### 3.1 docker-compose API v3+ with docker swarm

给nginx服务限制0.5个CPU和512M内存，并且设置保留CPU为四分之一，保留内存为128M。我们需要在配置文件里创建 `deploy` 和 `resources` 配置段:

```YAML
services:
  service:
    image: nginx
    deploy:
        resources:
          limits:
            cpus: 0.50
            memory: 512M
          reservations:
            cpus: 0.25
            memory: 128M
```

deploy 配置是`API v3`新加的，而且只有通过`docker stack deploy`在使用`Swarm`时环境下使用才生效。

```Shell
docker stack deploy --compose-file docker-compose.yml bael_stack
```

如果希望使用`API v3`又不想使用`Swarm`的话,docker-compose在1.20.0+提供了一个`--compatible`参数。开启该参数Docker Compose就会尝试把`API v3`配置里deploy相关配置转化为等价的非`Swarm`配置`API v2`。

```
docker-compose --compatibility up
```

同时可以通过`config`命令查看转化后的配置文件。(根据我的测试转化deploy配置需要docker-compose <=1.26.0, 新的版本取消了该功能)

```
docker-compose --compatibility config
```

### 3.2 docker-compose API v2

在 docker-compose里，我们可以把资源限制放在service的主属性上.他们名字也略有差异:

```YAML
service:
  image: nginx
  mem_limit: 512M
  mem_reservation: 128M
  cpus: 0.5
  ports:
    - "80:80"
```

这样我们就可以直接通过`docker-compose up`启动了。

## 0x04 验证资源使用

在我们设置过资源限制之后，我们可以通过`docker stats` 命令来验证设置是否生效。

```
$ docker stats
CONTAINER ID    NAME    CPU %    MEM USAGE / LIMIT   MEM %    NET I/O    BLOCK I/O    PIDS
8ad2f2c17078    bael_st   0.00%        2.578MiB / 512MiB   0.50%    936B / 0B          0B / 0B             2
```

## 0x05 总结

docker-compose的两个版本v2和v3并不是替代关系。v3是为了兼容Compose和Swarm设计的，所以v3版本里的`deploy`和容器的配置并不是1-1的匹配的。而且v2并不比v3老(v2.3 和 v3.4 发布时间差不多。)，所以这两个版本的API是基于不同目的来实现的。 如果没有使用Swarm的计划，建议还是使用v2。

### 0x06 参考链接

Runtime options with Memory, CPUs, and GPUs https://docs.docker.com/config/containers/resource\_constraints/ Runtime metrics https://docs.docker.com/config/containers/runmetrics/ Compose file version 3 reference https://docs.docker.com/compose/compose-file/compose-file-v3/#deploy docker-compose Command options overview and help https://docs.docker.com/compose/reference/overview/#command-options-overview-and-help Setting Memory And CPU Limits In Docker https://www.baeldung.com/ops/docker-memory-limit How to specify Memory & CPU limit in version 3 https://github.com/docker/compose/issues/4513 Memory Resource Controller https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt
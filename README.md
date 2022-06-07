# Promtail_Loki_Grafana

此脚本收集部署在k8s中的服务日志

***

  

## 组件介绍



- Promtail



```markdown
Promtail采用Deamonset类型部署在k8s集群中，保证每个node节点都有一个pod在运行，用来收集指定目录中的日志（这里指定为docker的家目录，/home/docker），并且将日志发送给loki

简而言之，Promtail是一个从指定存储位置收集日志并发送给loki的代理程序的 ***日志收集工具***
```



- Loki




```markdown
Promtail获取到日志后，会将日志转换为流，通过HTTP API 将流推送到 Loki

Loki收到日志后，会对日志的元数据的标签进行索引；会对日志进行压缩，以块的形式存储在对象存储或本地目录（这里采用本地目录）

简而言之，Loki是负责日志存储和查询处理的 ***主服务***
```



- Grafana




```markdown
Grafana 用作UI ***展示*** 服务或机器的性能、运行状态、日志（这里仅展示日志）
```




***



## 如何使用一键部署脚本

> 一键脚本是ansible编写，服务以docker方式运行

- 剧本功能流程介绍

```shell
1. 自带服务固定版本image，内网可安装
   
2. 包含预定义grafana界面展示数据包
   
3. 会安装docker-compose
   
4. 会安装nginx-docker，如果不需要请自行注释[docker-compose.yml](./roles/grafana/templates/docker-compose.yml.j2)中有关nginx_grafana的内容
   
5. 自动部署Promtail到k8s集群中
   自动部署Loki、Grafana到宿主机
   
6. 不足之处
   需要自行安装ansible
   此脚本现阶段只能将Grafana和Loki部署在一台服务器上，并未做分离
   
   导入预定义grafana数据后，admin账户密码不生效，可能需要手动执行密码重置操作（偶发）
   执行命令
   docker exec -it grafana bash -c "grafana-cli admin reset-admin-password admin" >/dev/null 2&>1
```


-  怎么使用脚本

> 首先需要修改两个文件
>
> [变量文件](./roles/grafana/vars/main.yaml) 需要修改一些参数例如：\_\_ip\_\_,\_\_port\_\_
>
> [hosts主机文件](./hosts) 表示需要在哪一台服务器上操作此脚本

#### 角色：grafana_loki.yml

```shell
# 使用命令
ansible-playbook -i hosts grafana_loki.yml
```

### 
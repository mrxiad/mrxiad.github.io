---
title: kong
date: 2024-4-1 13:50:55
tags: 微服务
categories: 
- 微服务
- kong
---









# kong & konga

[参考笔记](https://blog.csdn.net/zz18435842675/article/details/122449579?spm=1001.2014.3001.5501)

[中文文档](https://github.com/wanglongsxr/kong-docs-cn?tab=readme-ov-file)

## 下载

### 第1步：创建Docker网络

首先，创建一个Docker网络`kong-net`，这样你的服务可以在隔离的网络空间中通信。

```bash
docker network create kong-net
```

### 第2步：安装并启动PostgreSQL容器

运行以下命令来启动PostgreSQL容器，使用`kong-net`网络，并设置环境变量来配置数据库用户、密码和数据库名。

```bash
docker run -d --name kong-database \
           --network=kong-net \
           -p 5432:5432 \
           -e "POSTGRES_USER=kong" \
           -e "POSTGRES_DB=kong" \
           -e "POSTGRES_PASSWORD=kong" \
     postgres:9.6
```

- `-p 5432:5432`: 将容器的5432端口映射到宿主机的5432端口，使得可以从宿主机访问PostgreSQL服务。
- `-e`: 设置环境变量，用于配置PostgreSQL数据库的**用户名**、**数据库名**和**密码**。



```bash
docker run --rm \
           --network=kong-net \
           -e "KONG_DATABASE=postgres" \
           -e "KONG_PG_HOST=kong-database" \
           -e "KONG_PG_USER=kong" \
           -e "KONG_PG_PASSWORD=kong" \
           kong:latest kong migrations bootstrap
```

- `--rm`: 容器退出后**自动清理容器**文件系统。
- `-e`: 设置环境变量来配置Kong连接到PostgreSQL的细节。
- `kong:latest kong migrations bootstrap`: 使用最新的Kong镜像执行数据库迁移命令，准备数据库。



>  这一步指令的作用

- **创建表**：Kong需要在PostgreSQL数据库中创建多个表，用来存储它的配置数据，比如服务（Services）、路由（Routes）、消费者（Consumers）、插件（Plugins）等的信息。
- **应用数据模式更新**：随着Kong的版本更新，数据库模式（schema）也可能发生变化。迁移确保数据库模式与Kong的当前版本兼容。
- **准备数据库**：此外，迁移还会执行一些检查和准备工作，确保Kong可以顺利地与数据库交互。





### 第3步：安装配置Kong



启动Kong容器

```bash
docker run -d --name kong \
           --network=kong-net \
           -e "KONG_DATABASE=postgres" \
           -e "KONG_PG_HOST=kong-database" \
           -e "KONG_PG_USER=kong" \
           -e "KONG_PG_PASSWORD=kong" \
           -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
           -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
           -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
           -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
           -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
           -p 8000:8000 \
           -p 8443:8443 \
           -p 8001:8001 \
           -p 8444:8444 \
     kong:latest
```

- **8000**: Kong的代理监听端口，用于接收API请求。
- **8443**: Kong的代理SSL监听端口，用于接收API请求（通过HTTPS）。
- **8001**: Kong的Admin API监听端口，用于配置Kong和查看状态信息。
- **8444**: Kong的Admin API SSL监听端口，通过HTTPS提供同8001相同的功能。



> 查看docker网络
>
> ```bash
> docker network inspect kong-net
> ```



### 第4步：安装并启动PostgreSQL容器

```bash
docker run -d --name konga-database \
           --network=kong-net \
           -p 5433:5432 \
           -e "POSTGRES_USER=konga" \
           -e "POSTGRES_DB=konga" \
           -e "POSTGRES_PASSWORD=konga" \
     postgres:9.6
```



```bash
docker run --rm \
           --network=kong-net \
           pantsel/konga:latest \
           -c prepare \
           -a postgres \
           -u postgresql://konga:konga@konga-database:5432/konga
```

### 第5步：启动Konga容器

最后一步是启动Konga容器，它将连接到同一个Docker网络内的PostgreSQL数据库。Konga提供了一个Web界面，使得管理Kong变得更加容易和直观。

```bash
docker run -d --name konga \
           --network=kong-net \
           -e "DB_ADAPTER=postgres" \
           -e "DB_HOST=konga-database" \
           -e "DB_PORT=5432" \
           -e "DB_USER=konga" \
           -e "DB_PASSWORD=konga" \
           -e "DB_DATABASE=konga" \
           -e "DB_PG_SCHEMA=public" \
           -e "NODE_ENV=production" \
           -p 1337:1337 \
     pantsel/konga
```



### 第6步：docker-compose



>  docker-compose

```yaml
version: '3'

services:
  kong-database:
    image: postgres:9.6
    container_name: kong-database
    networks:
      - kong-net
    environment:
      POSTGRES_DB: kong
      POSTGRES_USER: kong
      POSTGRES_PASSWORD: kong
    ports:
      - "5432:5432"

  kong:
    image: kong:latest
    container_name: kong
    networks:
      - kong-net
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: '0.0.0.0:8001, 0.0.0.0:8444 ssl'
    depends_on:
      - kong-database
    ports:
      - "8000:8000"
      - "8443:8443"
      - "8001:8001"
      - "8444:8444"
    command: kong migrations bootstrap && kong start

  konga-database:
    image: postgres:9.6
    container_name: konga-database
    networks:
      - kong-net
    environment:
      POSTGRES_DB: konga
      POSTGRES_USER: konga
      POSTGRES_PASSWORD: konga
    ports:
      - "5433:5432"

  konga:
    image: pantsel/konga:latest
    container_name: konga
    networks:
      - kong-net
    environment:
      DB_ADAPTER: postgres
      DB_HOST: konga-database
      DB_PORT: 5432
      DB_USER: konga
      DB_PASSWORD: konga
      DB_DATABASE: konga
      NODE_ENV: production
    depends_on:
      - konga-database
    ports:
      - "1337:1337"
    command: ["./wait-for-it.sh", "konga-database:5432", "--", "npm", "start"]

networks:
  kong-net:
    driver: bridge
```





## 路由

kong的路由主要涉及到两个组件，`service`和`route`

定义
**service服务**

就是我们自己定义的上游服务,通过Kong匹配到相应的请求要转发的地方, Service 可以与下面的Route进行关联,一个Service可以有很多Route,匹配到的Route就会转发到Service中, 当然中间也会通过Plugin的处理,增加或者减少一些相应的Header或者其他信息。

**route路由**

Route实体定义匹配客户端请求的规则. 每个路由都与一个服务相关联,而服务可能有多个与之相关联的路由. 每一个匹配给定路线的请求都将被提交给它的相关服务。



### example

[参考博客](https://blog.csdn.net/zz18435842675/article/details/120410085)



#### konga方式：

1. 打开konga后台管理画面，打开Service
2. 配置service: 添加`name=baidu-text`,`url=https://www.baidu.com`
3. 配置route路由:    点击刚才配置好的service，然后点击routes，配置`name=baidu-route`和`paths=/myPath`
4. 测试：浏览器访问`http://8.130.144.38:8000/myPath`,或者输入`curl http://localhost:8000/myPath`



#### kong方式:

1. 创建service服务

	```bash
	curl -i -X POST \
	  --url http://localhost:8001/services/ \
	  --data 'name=baidu-test' \
	  --data 'url=http://www.baidu.com'
	```

2. 为service添加route

	```bash
	curl -i -X POST \
	  --url http://localhost:8001/services/baidu-test/routes \
	  --data 'name=baidu-route' \
	  --data 'paths[]=/myPath'
	```

3. 验证是否成功

	```bash
	curl http://localhost:8000/myPath
	```





## 身份认证(oauth2)

### 创建Service

创建一个Kong的Service Object指向上游的服务

```bash
curl -X POST \
  --url "http://localhost:8001/services" \
  --data "name=oauth2-test" \
  --data "url=http://httpbin.org/anything"
```

> response
>
> ```json
> {
>   "host": "httpbin.org",
>   "id": "33459a79-e284-4bb8-aa6f-65dafd456c6f",
>   "protocol": "http",
>   "read_timeout": 60000,
>   "tls_verify_depth": null,
>   "port": 80,
>   "updated_at": 1615001132,
>   "ca_certificates": null,
>   "created_at": 1615001132,
>   "connect_timeout": 60000,
>   "write_timeout": 60000,
>   "name": "oauth2-test",
>   "retries": 5,
>   "path": "/anything",
>   "tls_verify": null,
>   "tags": null,
>   "client_certificate": null
> }
> ```



### 创建Route

```bash
curl -X POST \
  --url "http://localhost:8001/services/oauth2-test/routes" \
  --data 'paths[]=/demo'
```

> response
>
> ```json
> {
>   "strip_path": true,
>   "tags": null,
>   "updated_at": 1615004204,
>   "destinations": null,
>   "headers": null,
>   "protocols": [
>     "http",
>     "https"
>   ],
>   "methods": null,
>   "service": {
>     "id": "33459a79-e284-4bb8-aa6f-65dafd456c6f"
>   },
>   "snis": null,
>   "hosts": null,
>   "name": null,
>   "path_handling": "v0",
>   "paths": [
>     "/demo"
>   ],
>   "preserve_host": false,
>   "regex_priority": 0,
>   "response_buffering": true,
>   "sources": null,
>   "id": "e804fef4-fa42-4f7e-be0c-bbe9b9999027",
>   "https_redirect_status_code": 426,
>   "request_buffering": true,
>   "created_at": 1615004204
> }
> ```







>  在这之后我们可以通过 `curl localhost:8000/demo`来访问服务。





### 启用Oauth2插件

```bash
curl -X POST \
  --url http://localhost:8001/services/oauth2-test/plugins/ \
  --data "name=oauth2" \
  --data "config.scopes[]=email" \
  --data "config.scopes[]=phone" \
  --data "config.scopes[]=address" \
  --data "config.mandatory_scope=true" \
  --data "config.provision_key=oauth2-demo-provision-key" \
  --data "config.enable_authorization_code=true" \
  --data "config.enable_client_credentials=true" \
  --data "config.enable_implicit_grant=true" \
  --data "config.enable_password_grant=true"
```

> response
>
> ```json
> {
>   "created_at": 1615003048,
>   "id": "8bc8ed59-cdb4-4ac4-ab48-7719655cb9f3",
>   "tags": null,
>   "enabled": true,
>   "protocols": [
>     "grpc",
>     "grpcs",
>     "http",
>     "https"
>   ],
>   "name": "oauth2",
>   "consumer": null,
>   "service": {
>     "id": "33459a79-e284-4bb8-aa6f-65dafd456c6f"
>   },
>   "route": null,
>   "config": {
>     "pkce": "lax",
>     "accept_http_if_already_terminated": false,
>     "reuse_refresh_token": false,
>     "token_expiration": 7200,
>     "mandatory_scope": true,
>     "enable_client_credentials": true,
>     "hide_credentials": false,
>     "enable_authorization_code": true,
>     "enable_implicit_grant": true,
>     "global_credentials": false,
>     "refresh_token_ttl": 1209600,
>     "enable_password_grant": true,
>     "scopes": [
>       "email",
>       "phone",
>       "address"
>     ],
>     "anonymous": null,
>     "provision_key": "oauth2-demo-provision-key",
>     "auth_header_name": "authorization"
>   }
> }
> ```



> 我们再次链接 `curl localhost:8000/demo` 的时候，我们会得到 `HTTP/1.1 401 Unauthorized` 已经以下错误信息。这就说明Oauth2插件已经成功开启。



### 创建Consumer

```bash
curl -X POST \
  --url "http://localhost:8001/consumers/" \
  --data "username=oauth2-tester"
```

> response
>
> ```json
> {
>   "custom_id": null,
>   "created_at": 1615003502,
>   "id": "06d53376-8bfd-4bc7-aaaf-05c37316e7ef",
>   "tags": null,
>   "username": "oauth2-tester"
> }
> ```



### 创建Oauth2 Credential (App)

然后我们会在这个consumer下创建Oauth2的身份凭证。在这里我也会使用自定义的 `client_id` 和 `client_secret`。如果留空，Kong会自动生成这两个变量。

> 如果使用Kong来生成身份凭证请切记不要添加 `hash_secret=true` 在您的curl命令里面。

```bash
curl -X POST \
  --url "http://localhost:8001/consumers/oauth2-tester/oauth2/" \
  --data "name=Oauth2 Demo App" \
  --data "client_id=oauth2-demo-client-id" \
  --data "client_secret=oauth2-demo-client-secret" \
  --data "redirect_uris[]=http://localhost:8000/demo" \
  --data "hash_secret=true"
```

> response
>
> ```json
> {
>   "created_at": 1615004674,
>   "id": "f602f09b-7b7e-4326-b236-2fa8d45badff",
>   "tags": null,
>   "name": "Oauth2 Demo App",
>   "client_secret": "$pbkdf2-sha512$i=10000,l=32$e3SNVIWRFt8PuBxjoL1ncQ$/hF26HS30QHopDLMzlZqC+zv0nt3m4YFokuW9eTma6Q",
>   "client_id": "oauth2-demo-client-id",
>   "redirect_uris": [
>     "http://localhost:8000/demo"
>   ],
>   "hash_secret": true,
>   "client_type": "confidential",
>   "consumer": {
>     "id": "06d53376-8bfd-4bc7-aaaf-05c37316e7ef"
>   }
> }
> ```



### 测试【等待完成】





## 身份认证(jwt)



### 创建service

```bash
curl -X POST \
  --url "http://localhost:8001/services" \
  --data "name=jwt-test" \
  --data "url=http://www.baidu.com"
```



### 创建Route

```bash
curl -i -X POST \
  --url http://localhost:8001/services/jwt-test/routes \
  --data 'name=jwt-route' \
  --data 'paths[]=/jwt'
```



### 启用jwt插件

```bash
curl -X POST http://localhost:8001/services/jwt-test/plugins \
	--data "name=jwt"
```



> 尝试访问 `curl http://localhost:8000/jwt`
>
> response:{"message":"Unauthorized"}



### 在 Kong 中创建用户

```bash
curl -X POST http://localhost:8001/consumers \
	--data "username=kirito"
```



### 创建jwt凭证(可以指定jwt的key和secret)

```bash
curl -X POST http://localhost:8001/consumers/kirito/jwt
```



### 查看用户的jwt信息

```bash
curl http://127.0.0.1:8001/consumers/kirito/jwt
```



> 然后生成jwt



### 携带jwt访问

```bash
curl http://localhost:8000/jwt -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MTE5OTA2MTksImlzcyI6ImU1SzI0UHZaNEhVbUI0dm01dlFBbmlHVGZMSm5DdlRWIn0.wKzHKWqkvSleVJFT6Mnhlmj8PAekXDHwZr9Pa5l8CNY'
```





## 限流



## 黑白名单
---
title: 搭建kafka
date: 2024-3-19 20:33:12
tags: kafka
categories: 
- kafka
---





# docker-compose创建



使用Docker Compose可以将一系列创建及映射资源（网络、数据卷等）操作放在配置文件中，并且可以通过depends_on参数指定容器的启动顺序，通过environment参数指定Kafka需要的基本参数信息

## 1.创建**kafka-group.yml**，保存以下信息

```yml
version: '3'

services:
  zookeeper-test:
    image: zookeeper
    ports:
      - "2181:2181"
    volumes:
      - zookeeper_data_vol:/data
      - zookeeper_datalog_vol:/datalog
      - zookeeper_logs_vol:/logs
    container_name: zookeeper-test

  kafka-test:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: "localhost"   #注意这个改成公网ip
      KAFKA_ZOOKEEPER_CONNECT: "zookeeper-test:2181"
      KAFKA_LOG_DIRS: "/kafka/logs"
    volumes:
      - kafka_vol:/kafka
    depends_on:
      - zookeeper-test
    container_name: kafka-test
    
volumes:
  zookeeper_data_vol: {}
  zookeeper_datalog_vol: {}
  zookeeper_logs_vol: {}
  kafka_vol: {}
```



## 2.启动容器组

```bash
# 启动Kafka容器组
docker compose -f kafka-group.yml up -d
docker compose up -d

#停止并移除容器以及数据卷
docker-compose -f kafka-group.yml down -v
docker-compose down -v

# 输出示例
 ✔ Network kafka-group_default         Created 
 ✔ Volume "kafka-group_zookeeper_vol"  Created 
 ✔ Volume "kafka-group_kafka_vol"      Created 
 ✔ Container zookeeper-test            Started 
 ✔ Container kafka-test                Started
```



## 3.进入kafka容器检查所有topic(以及所有消息)

```bash
docker exec -it kafka-test /bin/bash

#检测所有topic
/opt/kafka/bin/kafka-topics.sh --list --zookeeper zookeeper-test:2181

#检测topic为[test]的所有信息
/opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```





# docker-compose搭建集群


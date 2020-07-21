---
layout: post
title:  "aws-dynamodb的基本使用"
date:   2020-07-21 10:28:09 +0800
categories: Note
---


# Dynamodb

# Dynamodb 基本概念
Amazon DynamoDB是一款完全托管的NoSQL数据库服务，可提供快速，可预测的性能和无缝可扩展性。

## DynamoDB 基本结构

DynamoDB 几个概念：tables, items, and attributes。（表，项目和属性）
Table是一个item集合， 每个item又是attributes集合。可以对应理解为mongodb中 集合 文档 属性。

## DynamoDB中主键的概念
DynamoDB不像mongodb默认情况下会生成_id来唯一标识某条数据。所以在DynamoDB中就有了主键的概念。
指定表的主键。主键唯一标识表中的每个项目。（指定某个属性为主键）

DynamoDB支持两种不同的主键：
- 1 分区键
DynamoDB使用分区键的值作为内部散列函数的输入。散列函数的输出确定项目将存储在其中的分区(DynamoDB内部的物理存储)。
- 2 复合主键(分区键和排序键)
DynamoDB使用分区键值作为内部散列函数的输入。散列函数的输出确定项目将存储在其中的分区（DynamoDB内部的物理存储）。所有具有相同分区键的项目都按照排序键值存储在一起。

在具有分区键和排序键的表中，有可能两个项具有相同的分区键值。但是，这两个项目必须具有不同的排序键值。

# 部署DynamoDB到本地服务器

## 下载并解压DynamoDB code
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html

## 在dynamodb 目录下运行（默认为8000端口）
```
java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb

docker run -it -p 8686:8000 amazon/dynamodb-local:latest
```

# Aws Cli

## create table
```
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --endpoint-url http://localhost:8686
```
##  upsert item
```
aws dynamodb put-item \
--table-name Music  \
--item \
    '{"Artist": {"S": "No One You Know2"}, "SongTitle": {"S": "Call Me Today2"}, "AlbumTitle": {"S": "Somewhat Famous2"}}' \
--return-consumed-capacity TOTAL  \
--endpoint-url http://localhost:8686

aws dynamodb put-item \
    --table-name Music \
    --item \
    '{"Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}, "AlbumTitle": {"S": "Songs About Life"}, "Awards": {"N": "10"} }' \
  --return-consumed-capacity TOTAL  \
  --endpoint-url http://localhost:8686
```
## get item
```
aws dynamodb get-item --consistent-read \
    --table-name Music \
    --key '{ "Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}' \
  --endpoint-url http://localhost:8686
```
## update item
```
aws dynamodb update-item \
    --table-name Music \
    --key '{ "Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}' \
    --update-expression "SET AlbumTitle = :newval" \
    --expression-attribute-values '{":newval":{"S":"Updated Album Title"}}' \
    --return-values ALL_NEW \
  --endpoint-url http://localhost:8686
```
## query items
```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :name" \
    --expression-attribute-values  '{":name":{"S":"Acme Band"}}' \
  --endpoint-url http://localhost:8686

aws dynamodb query --table-name Music --key-conditions file://key-conditions.json --endpoint-url http://localhost:8686
```
## list tables
```
aws dynamodb list-tables --endpoint-url http://localhost:8686
```
## desc table
```
aws dynamodb describe-table --table-name Music --endpoint-url http://localhost:8686
```
## resource limit
```
aws dynamodb describe-limits --endpoint-url http://dynamodb-local:8686 --region us-west-2 
```
## Create a Global Secondary Index
```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
    --endpoint-url http://localhost:8686 \
    --global-secondary-index-updates \
    "[{\"Create\":{\"IndexName\": \"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}], \
    \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```
## Query the Global Secondary Index
```
aws dynamodb query \
--table-name Music \
--index-name AlbumTitle-index \
--key-condition-expression "AlbumTitle = :name" \
--expression-attribute-values  '{":name":{"S":"Somewhat Famous"}}' \
--endpoint-url http://localhost:8686 
```
## delete table
```
aws dynamodb delete-table --table-name Music --endpoint-url http://localhost:8686 
```
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GettingStarted.NodeJs.html

[program:jenkins]
directory=/home/git/
command=java -Dmail.smtp.starttls.enable="true" -jar /usr/local/jenkins/jenkins.war --httpPort=8000 --prefix=/jenkins
autostart=true
autorestart=true
stopsignal=QUIT
user=git
stdout_logfile=/var/log/supervisor/stdout_jenkins.log
stderr_logfile=/var/log/supervisor/stder_jenkins.log
environment = HOME="/home/git", USER="git"


-Dmail.smtp.starttls.enable=true

1. SMTP server=smtp.office365.com
2. Default user e-mail suffix=@mycompany.com
3. Use SMTP Authentication (Checked)
4. User Name = my-name@mycompany.com
5. password = ********************
**6. Use SSL (Un-Checked)**
7. SMTP Port = 587
8. Reply-To Address = my-name@mycompany.com
9. Charset = UTF8
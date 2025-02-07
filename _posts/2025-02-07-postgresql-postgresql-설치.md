---
layout: post
title: "[PostgreSQL] 1강 PostgreSQL 설치"
date: 2025-02-07 21:45 +0900
toc: true
comments: true
categories:
- 교과서
- PostgreSQL
tags:
- postgreSQL
- 강의
description: PostgreSQL EZIS IT 1강 정리
---

{% include embed/youtube.html id='QXmKhy3molY' %}

## 가상머신 프로그램과 리눅스 설치
(WSL사용으로 생략)

## PostgreSQL 설치
(Docker Image 사용할 예정)  


#### Docker Desktop 설치
[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)


#### docker-compose 설치
```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \  
sudo chmod +x /usr/local/bin/docker-compose && \  
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose && \  
docker-compose --version  
```


#### compose.yml
postgreSQL 버전 주의 (나는 psql 버전에 맞게 설정했다)  
[https://stackoverflow.com/questions/60060701/postgresql-getting-an-error-when-listing-databases-with-message-error-column](https://stackoverflow.com/questions/60060701/postgresql-getting-an-error-when-listing-databases-with-message-error-column)
```
services:
  postgres:
    image: 'postgres:16.6'
    environment:
      - 'POSTGRES_DB=test'
      - 'POSTGRES_PASSWORD=test'
      - 'POSTGRES_USER=test'
    ports:
      - 5432:5432

```


### docker-compose 실행
```
sudo docker-compose up &
```
Docker Desktop에서 container 실행 가능  


#### WSL에 psql client 설치
```
sudo apt install postgresql-client -y
```


## psql

#### psql로 postgreSQL 접속
```
psql -U test -d test -h localhost -p 5432
```

#### 데이터베이스 크기 확인
```
\l+
```

#### password 변경
```
\password test
```

#### 종료
```
\q
```

#### .bashrc에 환경변수 저장

```
export PATH=$PATH:/usr/lib/postgresql/16/bin
export PGDATA=/var/lib/postgresql/16/main/
export PGCONF=/etc/postgresql/16/main/
export PGHOME=/var/lib/postgresql/
export PGDATABASE=test
export PGPORT=5432
export PGPASSWORD=test
```
---
layout: post
title: "[PostgreSQL] 3강 PostgreSQL Architecture II - Part.1 PostgreSQL 물리적 구조"
date: 2025-04-26 18:54 +0900
toc: true
comments: true
categories:
- 교과서
- PostgreSQL
tags:
- postgreSQL
- 강의
description: PostgreSQL EZIS IT 3강 정리
---

{% include embed/youtube.html id='s7gA4VDCIVc' %}

---

## PostgreSQL 개요 및 물리적 구조

![summary](/assets/image/2025-04-26-summary.png)

- Data 영역
  - base  
  실제 데이터를 저장
  - pg_tblspc  
  table space 정보를 저장
  - global  
  클러스터 내에서 공유하는 영역

- Log 영역
  - log  
  DB에서 이벤트가 발생할 때 기록
  - pg_wal  
  트랜잭션 기록


---



## Data 영역 - base

![summary](/assets/image/2025-04-26-base.png)

데이터베이스의 테이블, 인덱스, 데이터베이스 등 실제 데이터가 저장되는 디렉토리
- database 생성시, OID를 이름으로 하위 디렉토리 생성
- OID(Object Identifieres)  
  PostgreSQL에서 객체(테이블 ,인덱스, 데이터베이스 등)를 고유하게 식별하기 위해 사용하는 식별자

![summary](/assets/image/2025-04-26-base2.png)

#### OID와 database 확인

![summary](/assets/image/2025-04-26-base3.png)

디렉토리의 OID가 실제 database의 OID인지 `pg_database` 테이블 조회로 확인할 수 있음

#### FileNode
database 디렉토리 하위에 Table의 FileNode 확인
- database : 디렉토리 -> OID로 관리
- FileNode : 파일 -> 테이블, 인덱스 등 파일로 관리되는 데이터는 FileNode로 관리


---

## Data 영역 - pg_tblspc

![summary](/assets/image/2025-04-26-pg_tblspc.png)

TableSpace의 링크 정보(실제 데이터 디렉토리 위치)를 저장
- TableSpace를 별도로 생성하지 않을 경우, base 하위에 저장
- TableSpace는 I/O 분산 및 디스크 공간 관리 목적으로 사용

#### TableSpace 실습

![summary](/assets/image/2025-04-26-pg_tblspc2.png)

실습순서
- TableSpace로 사용할 디렉토리 생성
- TableSpace 생성
```sql
CREATE TABLESPACE cust_tbs LOCATION '/cust_tbs';
```
- 이미 존재하는 Table에 TableSpace 설정
```sql
ALTER TABLE customer SET TABLESPACE cust_tbs;
```
- Table 생성시 TableSpace 설정
```sql
CREATE TABLE sales (
    sales_id SERIAL PRIMARY KEY,
    product_id INT,
    total_quantity INT,
    sale_date DATE
) TABLESPACE sales_tbs;
```
- TableSpace 디렉토리 하위에 파일 생성 확인

![summary](/assets/image/2025-04-26-pg_tblspc3.png)
- /sales_tbs 를 TableSpace로 설정
- /sales_tbs 하위에 파일이 생성됨 (PG_15_202209061)
- pg_tblspc에 링크 정보 확인 (24718 -> /sales_tbs)

---



## Data 영역 - global

![summary](/assets/image/2025-04-26-global.png)

데이터베이스 클러스터의 데이터베이스들이 전역적으로 사용하는 공간
데이터베이스 시작시 global 데이터를 참조
- pg_control  
  클러스터 전역적인 상태 정보를 저장
- pg_internal.init  
  시스템 카탈로그 정보를 저장
- pg_database  
  데이터베이스 정보를 저장
- pg_db_role_setting  
  role 관련 정보를 저장

##### pg_control 정보 조회

![summary](/assets/image/2025-04-26-pg_control.png)

`pg_controldata` 명령어로 pg_control 정보를 볼 수 있다
- pg_control의 마지막 수정 날짜
- 클러스터 상태
- checkpoint 관련 정보
- LSN(Log Sequence Number) 정보
- wal 파일 정보

#### 시스템 카탈로그 조회

![summary](/assets/image/2025-04-26-system_catalog.png)

`pg_class` 테이블을 이용한 시스템 카탈로그 조회
- 시스템 카탈로그(System Catalog)  
  RDBMS에서 테이블, 인덱스 등 메타데이터를 저장하는 내부 테이블

![summary](/assets/image/2025-04-26-system_catalog2.png)

`pg_class` 테이블에서 dept 테이블의 시스템 카탈로그 정보 확인
- oid(database ID)
- relfilenode(FileNode ID)

#### oid2name

![summary](/assets/image/2025-04-26-oid2name.png)

`oid2name` tool을 활용하여 global 디렉토리에 구성된 정보 확인
- 우분투에서 기본으로 설치

#### global 참조 실습

![summary](/assets/image/2025-04-26-global2.png)

실습 순서
- 데이터베이스 정지
- global의 파일이름(OID) 변경
- 데이터베이스 시작
- 에러 로그 확인


---


## Log 영역

![summary](/assets/image/2025-04-26-log.png)

- 데이터베이스에서 일어난 다양한 이벤트 로그
- 에러 메세지, 쿼리 실행 정보, 연결 정보 등의 로그를 포함
- postgresql.conf에서 로그 파일 위치 설정
```
log_filename = 'postgresql-%A.log`
log_rotation_size = 10MB
log_truncate_on_rotation=on
```
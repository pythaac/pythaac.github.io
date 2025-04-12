---
layout: post
title: "[PostgreSQL] 2강 PostgreSQL Architecture I - 메모리, 프로세스, 논리적 구성"
date: 2025-04-13 00:19 +0900
toc: true
comments: true
categories:
- 교과서
- PostgreSQL
tags:
- postgreSQL
- 강의
description: PostgreSQL EZIS IT 2강 정리
---

{% include embed/youtube.html id='6f-TqM4HYPY' %}

---
## 메모리

![memory](/assets/image/2025-04-13-memory.png)

- Shared Memory  
  - 모든 세션이 공유해서 사용하는 공간
  - 클라이언트 요청에 의한 데이터를 저장(캐싱)하여 빠르게 처리
  - Transaction, Log, Lock 등의 데이터를 저장
- Local Memory  
   - 클라이언트 별로 할당되는 공간
   - SQL 문장 처리 관련 데이터를 저장
   - connection 수가 다량 발생 시 Local Memory 사용 공간이 증가

#### Shared Memory

![memory](/assets/image/2025-04-13-memory2.png)

- Shared Buffers  
   - 데이터 및 변경사항을 Block 단위로 저장
   - 물리적 I/O 없이 데이터를 빠르게 처리
- WAL(Write Ahead Log) Buffers
   - 데이터 변경에 대한 트랜잭션 정보를 저장
   - WAL file로 저장되어 데이터베이스 복구에 사용
- CLOG(Commit LOG) Buffers
   - 각 트랜잭션의 상태 정보를 저장하는 공간
- Lock Space
   - 트랜잭션을 보호하기 위한 Lock 정보를 저장
   - 접근중인 Table이 삭제나 수정되지 못하도록 함

#### Local Memory

![memory](/assets/image/2025-04-13-memory2.png)

- Maintenance_work_mem
   - 클라이언트 요청 SQL문에 대한 Vacuum, 인덱스, 테이블 변경, FK 추가 등 작업에 사용되는 공간  
- Temp_buffers
   - 클라이언트에서 세션단위로 처리할 수 있는 Temporary 테이블에 사용되는 공간
   - Temp 테이블을 사용하는 경우에만 할당
- work_mem
   - 클라이언트 요청 SQL문에서 GROUP BY와 ORDER BY, SORT와 HASH 작업을 위한 메모리 공간
- Catalog_cache
   - 데이터베이스 정보, 데이터베이스 구조와 상태 정보인 메타 데이터를 조회할 때 사용하는 시스템 카탈로그를 이용하기 위한 공간
- Optimizer /executor
   - 클라이언트 요청 SQL문의 실행계획 및 실행을 위한 공간
   - query에 대한 최적 실행 계획

---
## 프로세스

![memory](/assets/image/2025-04-13-process.png)

- 데이터베이스 기동 시, 환경설정 파일을 읽어 Shared Memory 할당 및 Daemon Process 기동
- Daemon Process에 의해 트랜잭션 및 데이터 처리를 위한 Background Process들이 기동
- 클라이언트 요청시 Daemon Process는 Backend Process에 할당 및 Local Memory 할당

#### Daemon Process / Backend Process

![memory](/assets/image/2025-04-13-daemon.png)

- 데이터베이스 기동시 Daemon Process 기동
- 클라이언트 요청시 Backend Process에 할당 및 Local Memory가 할당됨


#### BG Writer

![memory](/assets/image/2025-04-13-bgwriter.png)

- DML문에 의한 변경 데이터는 Shared Buffer에 저장
- 주기적으로 BG Writer에 의해 Data file에 write

#### Checkpointer

![memory](/assets/image/2025-04-13-checkpointer.png)

- checkpoint란, Data file로 동기화된 변경사항의 위치를 나타냄
- checkpoint 발생시 BG Writer에 의해 Data file이 write됨
- checkpoint 발생시 동기화 정보인 Checkpoint record가 WAL Buffer에 기록
- pg_control에 기록

#### WAL writer / Archiver

![memory](/assets/image/2025-04-13-wal.png)

- DML문이 발행되면 트랜잭션 정보가 WAL Buffer에 순차적으로 저장됨
- WAL Writer에 의해 주기적으로 WAL File에 write
- Archive 모드가 켜져 있을 경우, WAL File은 주기적으로 Archiver에 의해 Archive file로 복사됨

---
## 논리적 구성

![memory](/assets/image/2025-04-13-logical.png)

- Cluster
   - cluster는 여러 개의 Role, Database, Tablespace로 구성 가능
- Role
   - 권한의 집합
- Tablespace
   - 별도의 공간에 테이블, 인덱스 등 Object를 저장
- Database
   - 여러 개의 schema 구성 가능 (보통 1개로 구성)
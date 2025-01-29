---
layout: post
title: DB Table column 이름을 바로 바꿀 수 없다
date: 2025-01-30 01:03 +0900
toc: true
comments: true
categories:
- 수필
tags:
- RDB
- Database
- Table
- Column
description: 무중단 서비스의 column 이름은 절차대로 변경이 필요하다
---

![git](/assets/image/2025-01-30.webp)

## column 이름이 이상하다
신입 개발자로 입사 초기 때 일이다. 담당한 기능 개발 관련하여 RDB Table에 column 1개 추가가 필요했다. column 추가를 위해서는 작성한 쿼리를 DBA분께 전달하며 요청을 드려야 했고, 나는 간단한 쿼리를 작성했다.
```sql
ALTER TABLE tableName ADD columnName boolean NOT NULL DEFAULT 'true';
```
그런데 개발을 하면서 마음에 들지 않는 부분이 발견되었다. DB Table column의 이름이, 이 column을 사용하는 서버의 변수와 API 응답의 key와 일치하지 않은 것이다. 보통 Table에 저장된 데이터를 API를 통해 내려받을 때, column 이름과 받은 데이터의 key 이름이 동일하다. 예를 들면, User라는 Table의 name column은 API로 내려받을 때도 name이라는 key로 내려받는다.

물론 항상 일치하지 않을 수 있고, 그게 당연하다고 생각하지 않는다. 그러나 Table column 이름들은 이미 비슷한 의미를 가진 이름이 즐비했고, 충분히 다른 값과 착각하여 버그를 만들 수 있는 이름이었다. 이를 고치는 것 또한 내가 해야할 일이라 생각했다.

## 왜 column 이름이 상이해졌을까
나는 먼저 이런 현상이 일어난 원인 파악에 나섰다. 두 이름이 상이하게 설정된 이유가 있을 수도 있다고 생각했다. 그리고 나는 그 이유를 알 수 있었다.

> 브라질에서 개발하면서 영어가 부자연스럽다고 마음대로 바꿨어요

결국 우리가 결정한 API key 이름의 어감이 외국인에게 이상하게 느껴졌고, 관련 서비스를 개발하던 브라질에서 해당 Table의 column 이름을 바꿔서 생성한 것이다.

나는 영어의 어감보다 서비스의 일관성이 더 중요하다고 생각했다. Table column 이름을 바꾸는게 좋지 않겠냐는 내 의견에 팀원들로부터 명확한 답변을 듣지 못했다. 그래서 아래와 같이 쿼리를 수정하고, DBA분께 전달했다.
```sql
ALTER TABLE tableName ADD columnName boolean NOT NULL DEFAULT 'true';
ALTER TABLE tableName RENAME COLUMN beforeName TO afterName;
```

## column 이름은 바로 바꾸면 안되는구나
전달된 쿼리를 확인한 파트장님께 연락이 왔고, 해당 부분은 실행될 수 없으니 지워달라고 하셨다. 나는 그 이유가 궁금했고, 파트장님 설명을 듣고서야 내가 무중단 서비스를 개발하고 있다는 사실을 깨달았다.

> column 이름 변경 쿼리와 변경된 column을 읽는 서비스는 에러 없이 동시에 배포될 수 없다

column 이름이 변경되려면 서비스도 함께 변경이 발생하다. column 이름을 변경하는 쿼리와 이 변경된 서비스는 함께 실행 및 배포되어야 하는데, 요청이 밀려들어오는 상황에서 에러 없이 이를 성공하려면 완벽하게 동시에 이루어져야한다. 이는 불가능하다.

그런데 궁금했다. 그렇다면 무중단 서비스에서는 column name을 바꿀 수 없는 걸까? 이 또한 질문을 드렸고, 파트장님은 다시 친절하게 답변해주셨다.

> 새로운 column을 먼저 추가 -> 서비스 배포 -> 기존 column 삭제 같은 방법이 있다

column 이름 하나를 바꾸기 위해서 이 많은 과정을 거쳐야하기에, 이 서비스는 지금까지 이 상태로 유지되어 왔던 것이다. 나는 오늘도 이 과정을 겪지 않기 위해 naming에 많은 시간을 쏟는다.
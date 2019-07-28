---
layout: post
title: "Partitioning"
date: 2019-07-28
excerpt: " Transaction 이란
            DB의 상태를 변환시키는 하나의 논리적인 작업 단위를 구성하는 연산들의 집합"
tags: [database, partitioning, 기술면접]
comments: false
category: database
---
# Partitioning

이번에 D2 Fest mini 프로젝트를 진행하면서, 대용량 아키텍처에 대해 처음으로 생각해보게 되었다. 서비스가 사용자가 많아지고 관리해야 될 데이터의 양이 많아 질 수록, 개발자는 보다 많은 짐을 얻게 된다. 어느 정도 몸짓이 커지면 어플리케이션단을 넘어서, **시스템 아키텍처를 고민하고 설계**해야한다. 학부생때야 직접 서비스를 운영(그것도 아주 잘 나가는 서비스)해보지 않고서는 이런 고민을 할 일이 없다. 때문에 이번 공모전을 계기로 대용량 아키텍처를 공부하고 관련 내용을 기록으로 남겨보고자 한다.  개인적으로 회원 테이블에 대해, 휴면계정과 활성계정으로 파티셔닝을 적용해보면 하는 생각이 들었다. 

아래의 내용은 **Mysql 기준**이다.



### Partitioning

- **Partitioning 이란**
  - **VLDB(Very Large DBMS)가 등장**하면서 하나의 DBMS가  많은 Table을 관리하다 보니 속도가 느려지고, 심지어는 하나의 DBMS에서 관리 할 수 없는 경우도 생겨나기 시작했다.  해서 크기가 **큰 Table을 partiotion이라는 작은 단위로 나누어 관리하는 Partitioning 기법**이 등장하게 되었다. 주의할 점은 물리적인 데이터 분할이 있더라도 **DB에 접근 하는 application 입장에서는 이를 인식하지 못하여야 한다.**
  - **Partitioning은 주로 하나의 데이터베이스 안에서 이루어 지는 경우를 애기한다.**
- 이렇게 Partitioning 을 적용함으로서, 데이터 검색시 필요한 부분만 탐색하고 파티션 단위로 I/O 분산 및 백업/복구가 가능함으로 **성능이 증가**한다. 또한 전체 데이터가 분산관리되어짐으로 전체 데이터가 한번에 손실 될 가능성이 적어 **가용성이 향상**된다.
- **Partitioning 종류**
  - **수평(horizontal) 파티셔닝**
    - **Row 단위로 자른다.** 즉 하나의 큰 테이블을 같은 스키마를 가지는 여러개의 작은 테이블로 나누는 것이다.
    - Schema를 복제한 후 샤드키를 기준으로 데이터를 나눈다.
  - **수직(vertical) 파티셔닝** 
    - **특정 컬럼을 기준으로 쪼개서 저장하는 형태**이다. RDBS에서 제 3정규와 비슷한 개념이나, 파티셔닝은 이미 정규화 된 데이터를 분리하는 과정이다.
    - 자주 사용하는 컬럼 등을 분리시켜 성능을 향샹 시킬 수 있다.

* **Partition 방법**

  - **Range Partiotioning**

    - **Partiotion Key의 연속된 범위로 파티션을 정의**, 범위 기반으로 데이터를 여러 파티션에 균등하게 나눌 수 있는 경우, 파티션 키 위주로 검색이 자주 실행 될 경우 유용하다.

    - ex) 년별 데이터 : 2010년, 2011년 …..

      ```sql
      CREATE TABLE employees (
          id INT NOT NULL,
          name VARCHAR(30),
          hired DATE NOT NULL DEFAULT '1970-01-01',
          job_code INT NOT NULL,
          store_id INT NOT NULL
      ) PARTITION BY RANGE (YEAR(hired)) (PARTITION p0 VALUES LESS THAN (2010) ,
                                                              PARTITION p1 VALUES LESS THAN (2011) ,
                                                              PARTITION p2 VALUES LESS THAN (2012) ,
                                                              PARTITION p3 VALUES LESS THAN MAXVALUE);
      
      ```

      

  - **List Partitioning**

    - **Partiotion Key 값이 코드 값이나 카테고리와 같이 고정값일 경우 사용**, 키 값이 연속적이지 않고 정렬 순서와 관계없이 파티션을 해야 할 경우 유용하다.

    - ex) 대륙별 데이터 : 한국 -> 아시아, 미국 -> 아메리카

      ```sql
      CREATE TABLE employees (
          id INT NOT NULL,
          name VARCHAR(30),
          hired DATE NOT NULL DEFAULT '1970-01-01',
          job_code INT NOT NULL,
          store_id INT NOT NULL
      ) PARTITION BY LIST (job_code) (PARTITION p0 VALUES IN (3) ,
                                                       PARTITION p1 VALUES IN (1 , 9) ,
                                                       PARTITION p2 VALUES IN (2 , 6 , 7) ,
                                                       PARTITION p3 VALUES IN (4 , 5 , 8 , NULL));
      
      ```

  - **Hash Partiotioning**

    - **Hash 함수결과의 n mode 값을 기준으로 파티션**, Range나 List로 데이터를 균등하게 나누는 것이 어려울 때 사용

    - **새롭게 파티션이 추가 될 경우, 파티션에 저장된 모든 레코드는 재배치 되야 함으로 많은 부하가 발생한다**.

    - 파티션 병합, 삭제, 분할 기능을 제공하지 않는다.

    - ex) Hash 함수를 기준으로 4개의 파티션을 나눔

      ```sql
      CREATE TABLE employees (
          id INT NOT NULL,
          name VARCHAR(30),
          hired DATE NOT NULL DEFAULT '1970-01-01',
          job_code INT NOT NULL,
          store_id INT NOT NULL
      ) PARTITION BY HASH (id)
      PARTITIONS 4;
      
      ```

* **파티션 테이블의 Index 스캔과 정렬**

  * Mysql의 파티션 테이블에서 인덱스는 **전부 로컬 인덱스에 해당**, 모든  인덱스는 파티션 단위로 생성된다. 
  * 파티션 된 테이블에서 인덱스는 정렬 되지 않은 상태이다.

  - 실제 MySQL 서버는 여러 파티션에 대해 인덱스 스캔을 수행 할 때, 각 파티션으로부터 조건에 맞는 레코드를 우선순위 큐에 임시저장, 그리고 우선순위 큐에서 인덱스에 따라 정렬한다.

- **Mysqsl 파티션 제약사항**
  - 파티션 테이블에서는 외래키 사용이 불가능
  - 파티션 테이블은 Full Text Index 생성이 불가능
  
  
 # Reference 
 Mysql : https://12bme.tistory.com/48
 개념 : https://gmlwjd9405.github.io/2018/09/24/db-partitioning.html

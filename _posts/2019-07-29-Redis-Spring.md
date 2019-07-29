---
layout: post
title: "SpringBoot - Redis"
date: 2019-07-28
excerpt: "Timline API를 구현하는 중 Redis를 사용해,  효율을 높이고자 했으나, 아직 **Spring AOP와 Redis에 대한 전반적인 이해가 부족**한  "
tags: [database, springboot, redis]
comments: false
category: database
---

# Redis

Timline API를 구현하는 중 Redis를 사용해,  효율을 높이고자 했으나, 아직 **Spring AOP와 Redis에 대한 전반적인 이해가 부족**한 거 같아, 적용하지 않기로 했다. 시간이 촉박하기에 충분한 이해가 없이 진행 될 상황을 우려했고, 이 때문에 로직이 복잡해진다거나 오히려 성능이 나빠지는 경우가 있을 수 있기 때문에 과감하게 보류했다.  이번 포스팅은 Redis를 적용하고자 찾아본 내용들을 정리한 포스팅이다.



### Redis 설치 및 콘솔실행

```bash
brew install redis ## 설치
redis-server # redis 실행
redis-cli # redis 콘솔창 실행
```



### Redis CRUD

```bash
set ${key} ${value}
get ${key}
keys ${pattern}
```



### Redis Hash

+ 해시맵으로, spring code에서 RedisHash("객체이름")을 주게 되면 객체이름을 통해 이 **엔티티만을 보관하는 하나의 집합이 만들어진다.** 

+ 각 **엔티티의 고유 아이디는 ${객체이름}:${id}**가 된다.

  ```bash
  smembers ${객체이름} # 집합조회
  scard ${객체이름} # 집합의 갯수 조회
  hget ${객체이름:id} ${fild 이름} # id에 해당하는 객체의 필드 값
  hkeys ${객체이름:id} # 객체의 키값 조회
  hgetall ${객체이름:id} # select * from ${객체이름} where id = id
  smembers field2_world # field2 = world 조회
  type ~ # ~ 타입확인
  ```

   

### Spring boot data - redis

**기본설정**

1. 스프링부트 의존성 추가 (pom.xml)

   ```xml
   <dependency>
   	<groupId>org.springframework.boot</groupId>
   	<artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```

2. Redis 커넥션 설정 (applications.properties)

   ```properties
   spring.redis.port=6379
   spring.redis.host=127.0.0.1
   spring.cache.type=redis
   ```

3. 캐시 활성화 (Application.java)

   ```java
   @SpringBootApplication
   @EnableCaching // 캐시활성화
   public class Application {
       public static void main(String[] args) {
           SpringApplication.run(TimelineApplication.class, args);
       }
   }
   ```



**캐시 조작**

+  캐시 어노테이션의 key, condition, unless 인자들은 **SpEL**을 지원

+ **@Cacheable** : 조회 시 캐시에 존재하면 캐시 내용을 전달, 없으면 해당 메소드 수행하고 저장
+ **@CachePut** : 무조건 캐시에 저장
+ **@CacheEvict**  : 캐시 삭제



**캐시정책**

+ RedisCacheManager 또는 Config 파일을 별도로 구현



**주의사항**

Spring @Cacheable은 내부적으로 **Spring AOP를 이용**하기 때문에 @Async, @Transactional 등과 마찬가지로 아래와 같은 제약사항을 갖는다.

+ **pulbic method에만 사용가능**
+ **같은 객체내의 method끼리 호출시에는 @Cacheable이 설정되어있어도 캐싱되지 않는다.**





### Reference

##### [Spring @Cacheable Cache 처리](http://dveamer.github.io/backend/SpringCacheable.html)

**[SpringBoot기반 Redis Cache 활용법](https://yonguri.tistory.com/82)**

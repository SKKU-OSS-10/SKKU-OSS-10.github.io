---
layout: post
title: cron을 통한 주기적 프로세스 실행
date: 2019-05-23
excerpt: " 특정 시간에 명령어가 수행 될 수 있도록 예약해주는 리눅스용 작업 Scheduler, Daemon으로 동작하며 Cron table 형식의 예약 스크립트를 작성하도록 되어있다."
tags: [linux, cron, 스케쥴]
comments: false
---
``
#  cron 명령어

+ 특정 시간에 명령어가 수행 될 수 있도록 예약해주는 리눅스용 작업 Scheduler, Daemon으로 동작하며 Cron table 형식의 예약 스크립트를 작성하도록 되어있다. 
+ cron은 기본적으 멀티스레드(task)를 지원하지 않는다. 동시에 다수의 task를 실행시키고 싶다면 아래와 같이 스크립트를 작성하고 이를 cron에 등록한다.
```shell
vi main.sh
./etc/rip_first_radio.sh &
./etc/rip_second_radio.sh &
./etc/rip_third_radio.sh &
./etc/rip_fourth_radio.sh &
```
* & 연산자는 & 앞 명령을 백그라운드로 넘긴다. &&는 앞 명령이 성공적으로 수행된 후에 뒷 명령이 수행된다.

+ 설치

  ```shell
  apt-get install -y cron # 유닉스 계열은 보통 기본적으로 설치 되어있다.
  ```

+ 사용

  + 옵션 

    **-e**  : edit, 예약 스크립트를 수정

    **-l**  : List, 예약된 스크립트 목록을 출력 

    **-r**   : Reset, 예약된 스크립트 초기화

  + 옵션 -e로 실행하면 예약스크립트를 수정할 수 있는 에디터가 실행되는데 형식은 다음과 같다.

    ```shell
    crontab -옵션
    
    # Host Timezone을 기준으로 실행된다.
    [분] [시] [일] [월] [요일] [실행할 명령어]
    
    # 월 ~ 금요일 10시 30분에 test.py 실행
    30 10 * * 1-5 python /home/norr/test.py
    # 매월 15일에 10분마다 scan.py 실행
    */10 * 15 * * python /home/norr/scan.py
    ```

+ 실행 확인

  ```shell
  ps -ef | grep crond
  cat /var/log/syslog # cron log 정보를 볼 수 있다. 우분투 기준 경로이며, os에 따라 다를 수 있다.
  ```

+ 서비스 시작 및 재부팅

  ```shell
  service cron start
  service cron stop
  service cron restart
  ```

  

+ 로그 남기기

  + cron job들은 자동으로 수행되기때문에 별도로 작업로그를 남기지 않으며 해당 작업이 정상적으로 수행되었는지 체크하기가 매우 까다롭다. cron job의 수행 여부 자체는 시스템 로그(우분투는 /var/log/syslog) 남지만, 실제 해당 job이 실행되면서 job에서 발생시키는 로그는 따로 기록이 필요하다. cron job을 등록할 때 아래와 같은 명령어를 사용하면 로그 파일을 남길 수 있다. 

    ```shell
    0 0 * * * /some/job > ~/log/job_`date +\%Y\%m\%d\%H\%M\%S`.log 2>&1
    # date 명령어를 이용하면 지정한 포메터 대로 현재 시간을 출력할 수 있다.
    # 쉘 스크립트에서 `문자를 이용하여 명령어를 감싸면 해당 명령어의 stdout을 return해주는데 이를 이용하여 파일 이름을 정해줄 수 있다.
    # 2>&1 의미 : 1은 stdout, 2는 stderr, >는 리다이렉션을 의미한다
    # 즉 stderr를 stdout으로 리디렉션해서 stdout과 동일하게 처리함을 의미한다.
    ```

  + 리디렉션 기호인 **>** 를 이용하여 stdout을 파일로 기록하려면 다음과 같이하면 된다.

    ```shell
    # 해당 로그파일에 overwrite 
    /some/job > ~/log/job_`date +\%Y\%m\%d\%H\%M\%S`.log  
    
    # 해당 로그파일의 appedn
    /some/job >> ~/log/job_`date +\%Y\%m\%d\%H\%M\%S`.log  
    
    # 로그 파일 버리기
    /some/job > /dev/null
    ```

+ 가상환경 키고 파이썬 파일 실행

  ```shell
  SHELL=/bin/bash
  */1 * * * * source ~/AuctionParsing/venv/bin/activate && python3 ~/AuctionParsing/modify_ver1.py > ~/AuctionParsing/cron.log 2>&1
  ```

 

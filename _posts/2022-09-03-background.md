---
layout: single
title:  "[Spring][AWS EC2] nohup를 이용하여 무중단 서비스 만들기
categories: Spring
tag: [Spring, ec2]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# 과정

1. 로컬에서 프로젝트를 빌드하고 jar 파일을 ec2 로 업로드합니다.
2. java -jar [jar 파일] 을 통해 jar파일을 실행시킵니다.



이 과정으로 실행시키면 정상적으로 잘 실행되지만 터미널을 종료하면 jar 파일도 같이 종료됩니다.

터미널을 종료해도 실행시킨 jar 파일이 중단되지 않게하려면 nohup 를 이용하여 백그라운드에서 실행시켜야 합니다.



# nohup

```
nohup java -jar [jar파일] & > /dev/null
```

앞서 실행했던 명령어 앞에 nohup를 붙이고, 뒤에 &를 붙이면 백그라운드에서 실행되어 터미널을 종료해도 EC2 상에서 계속 돌아갑니다. 뒤에 > /dev/null 은 log를 저장하지 않겠다는 뜻이고, 설정하지 않으면 nohup.out 으로 저장됩니다.



# 프로세스 종료

nohup 로 백그라운드에서 실행중인 프로세스를 종료

**PID를 아는경우**

```
sudo kill -9 [PID]
```



**PID를 모르는 경우**

아래 명령어로 PID를 찾고 kill 명령어로 중단시키면 됩니다.

```
ps -ef | grep [실행시켰던 파일이름]

sudo kill -9 [PID]
```



아래 명령어로 포트랑 PID 확인 가능

```
netstat -tnlp
```


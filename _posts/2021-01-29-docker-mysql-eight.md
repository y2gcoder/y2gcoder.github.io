---
title: "Docker - MySQL 8 컨테이너 이전 인증 방식으로 접속하기"
excerpt: "이전 인증 방식으로 Docker MySQL 8버전 컨테이너를 생성하고 접속하기"

categories:
  - Studies
tags:
  - Docker
  - MySQL
  - caching_sha2_password

last_modified_at: 2021-01-21T19:00:00-12:00
---

**MySQL 8** 버전의 도커 컨테이너를 만들고 root 계정으로 접속하려고 하면 이러한 오류가 뜨는 경우가 있다. 

```
  Authentication plugin 'caching_sha2_password' cannot be loaded:
```

원인을 찾아보니 MySQL 8 버전부터 사용자 암호를 `SHA-256`방식으로 해싱할 수 있는데, `caching_sha2_password`은 인증 시에 서버 캐싱을 이용해 SHA-256 인증을 하는 모듈이라고 한다. 

MySQL 8버전부터 기본 인증 모듈이 `mysql_native_password`에서 `caching_sha2_password`로 바뀌었고  **SSL이나 RSA 보안을 적용하지 않고 접속을 시도**할 때 발생하는 에러다.

이러한 에러가 뜨는 것을 원하지 않고, 기존의 `mysql_native_password` 방식으로 로그인하고 싶다면 docker run 명령어에 추가해서 사용할 수 있다.

다음의 명령어를 이용하면 앞서 말한 문제점을 해결하면서 로컬에 설치한 MySQL DB처럼 이용할 수 있다. 

```bash
docker run -d --name (컨테이너 이름) -p (호스트 포트):3306 -e mysql:8 
MYSQL_ROOT_PASSWORD=(root 암호) 
--default-authentication-plugin=mysql_native_password 
```

대신 이렇게만 하면 기본적으로 테이블의 기본 character-set이 `latin1`로 설정되어 한글로 된 데이터를 넣을 때 한글이 깨질 수 있다. DB 컨테이너를 만들고 나서 수정하기는 귀찮으므로 `--character-set-server`와 `--collation-server`을 `utf8` 계열로 설정하는 부분도 추가해주면 좋다. 

위에서 도커의 **호스트 볼륨**과 데이터에 **이모티콘**을 넣을 것까지 고려해서 컨테이너를 생성하고 싶다면 다음 명령어를 사용하면 된다. 

```bash
docker run -d --name (컨테이너 이름) -p (호스트 포트):3306 
-v /(호스트 볼륨 컴퓨터 경로):/var/lib/mysql mysql:8 
-e MYSQL_ROOT_PASSWORD=(root 암호) 
--default-authentication-plugin=mysql_native_password --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```


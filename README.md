# Docker-log
도커 공부한 것들을 정리하는 곳 입니다.

## 도커란?
1. 컨테이너라는 공통적인 형태로 추상화하여 환경에 상관없이 이 컨테이너를 실행이 가능하다.
2. 독립적으로 실행 다른컨테이너에 영향을 미치지 않는다.
3. 가상머신보다 쉽고 빠르다. 명령어 하나로 컨테이너를 실행한다.
4. 가상머신 처럼 사용하지만 성능저하가 거의 없다.

특징.1 - 확장성 / 이식성
- 도커가 설치되어 있다면 어디서든 컨테이너를 실행할 수 있다.
- 오픈소스이기에 특정 회사나 서비스에 종속적이지 않다.
- 쉽게 개발서버를 만들 수 있고 테스트서버 생성도 간편한다.

특징.2 - 표준성
- 컨테이너라는 표준으로 서버를 배포하므로 모든 서비스들의 배포과정이 동일해진다.
- 도커를 사용하지 않는다면 ruby, nodejs, go, php 로 만든 서비스들의 배포 방식은 제각각 다르다.

특징.3 - 이미지
- 이미지에서 컨테이너를 생성하기 때문에 반드시 이미지를 만드는 과정이 필요
- Dockerfile을 이용하여 이미지를 만들고 처음부터 재현 가능하다.
- 빌드 서버에서 이미지를 만들면 해당 이미지를 이미지 저장소에 저장하고 운영서버에서 이미지를 불러온다.

특징.4 - 설정관리
- 환경변수로 설정을 제어한다.
- 컨테이너를 띄울때 환경변수를 같이 지정
- 하나의 이미지가 환경변수에 따라 동적으로 설정파일을 생성하도록 만들어져야한다.

특징.5 - 자원관리
- 컨테이너는 삭제 후 새로 만들면 모든 데이터가 초기화
- 업로드 파일을 외부 스토리지와 링크하여 사용하거나 별도의 저장소가 필요
- 세션이나 캐시를 memcached나 redis와 같은 외부로 분리

***

## 도커는 다양한 데이터베이스를 손쉽게 생성/삭제가 가능하다.

맥북 m1 버전에서 MySQL 이미지 다운로드 및 컨테이너 실행을 위하여 `docker pull mysql:5.7` 하면 에러가 생긴다.
`--platform` 으로 명시를 해줘야 정상 작동한다.
```dockerfile
docker pull --platform linux/amd64 mysql:5.7
```

MySQL 컨테이너를 생성, 실행 할 때도 플랫폼 명시를 해야한다.
```dockerfile
docker run --platform linux/amd64 --name mysql -d -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7
```

MySQL 데이터베이스 실행하기
```dockerfile
#exec 명령어로 접속
docker exec -it mysql mysql 

#테이블 만들기
create database wp CHARACTER SET utf8;
grant all privileges on wp.* to wp@'%' identified by 'wp';
flush privileges;
```
[참고] 컨테이너 실행할 때 뒤에
- `--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci` 를 입력하면 한글이 깨지지 않는다.

exec 명령어
- exec 명령어는 run 명령어와 달리 실행중인 도커 컨테이너에 접속할 때 사용하며 컨테이너 안에 ssh server 등을 설치하지 않고 exec 명령어로 접속합니다. 

워드프레스 블로그 실행하기
```dockerfile
docker run -d -p 8080:80 \
  -e WORDPRESS_DB_HOST=host.docker.internal \
  -e WORDPRESS_DB_NAME=wp \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_PASSWORD=wp \
  wordpress
```
앞에 만든 MySQL을 실행한 상태에서 생성한 뒤 8080 포트로 접속

---
## 도커 기본 명령어

ps 명령어
- `docker ps`: 실행중인 컨테이너 목록을 확인하는 명령어
- `dokcer ps -a`: 중지된 컨테이너도 확인 -a 옵션 붙이기

stop 명령어
- `docker stop [OPTIONS] CONTAINER [CONTAINER...]`
- 실행중인 컨테이너를 중지하는 명령어. 하나 또는 여러개(띄어쓰기) 중지 가능

rm 명령어
- `docker rm [OPTIONS] CONTAINER [CONTAINER..]`
- 종료된 컨테이너를 완전히 제거하는 명령어

logs 명령어
- `docker logs [OPTIONS] CONTAINER`
- 컨테이너가 정상적으로 동작하는지 로그를 확인 할 수 있다.

images 명령어
- `docker images [OPTIONS] [REPOSITORY[:TAG]]`
- 도커가 다운로드한 이미지 목록을 보는 명령어
- `docker rmi image`: 이미지 삭제, 실행중인 컨테이너는 삭제가 안된다.

network 명령어
- `docker network cerate [OPTIONS] NETWORK`
  - 도커 컨테이너끼리 이름으로 통신할 수 있는 가상 네트워크를 만든다.
- `docker network connect [OPTIONS] NETWORK CONTAINER`
  - 기존에 생성된 컨테이너에 네트워크를 추가합니다.
- `docker network ls` , `docker network rm [xxx]`; 
  - 도커에 생성된 네트워크 리스트를 봅니다. , xxx 네트워크 삭제

volume 명령어
- Docker 컨테이너의 생명주기와 관계없이 데이터를 영속적으로 저장할 수 있도록 하는 옵션이다.
- https://www.daleseo.com/docker-volumes-bind-mounts/ => 자세한 내용 참고

---

## 도커 컴포즈(docker compose)
- 도커 명령어들은 띄어 쓰기나 명령어를 실수하지 않게 조심 쓰럽게 써야한다 이걸 편하게 해주는 것이 docker-compose 이다

설치 확인
- `$ docker-compose version`

docker-compose.yml 설정
```yaml
version: '2'
services:
  db:
    image: mysql:5.7
    volumes:
      - ./mysql:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
  wordpress:
    image: wordpress:latest
    volumes:
      - ./wp:/var/www/html
    ports:
      - "8080:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
```

up 명령어   
- `docker-compose up -d`
- docker compose를 이용하여 mysql과 wordpress를 실행한다.
- docker-compose.yml 파일 위치에 가서 명령어를 입력해야 한다.

down 명령어
- `docker-compose down`
- docker compose를 이용하여 mysql과 wordpress를 조욜한다.

---

## 도커 컴포즈 문법

version
- `version: '3`
- dokcer-compose.yml 파일의 명세 버전, 버전에 따라 지원하는 도커 엔진 버전도 다름

services
```yaml
services:
 postgres:
  ..
 django:
  ..
```
- 실행할 컨테이너 정의
- docker run --name django와 같다고 생각할 수 있음

image
```yaml
services:
 django:
  image: django-sample
```
- 컨테이너에 사용할 이미지 이름과 태그
- 태그를 생략하면 latest
- 이미지가 없으면 자동으로 pull

ports
```yaml
services: 
 django:
  ..
  ports:
   - "8000:8000"
```
- 컨테이너와 연겨랄 포트(들)
- {호스트 포트}:{컨테이너 포트}

environment
```yaml
services:
 mysql:
  ..
   environment:
    - MYSQL_ROOT_PASSWORD=somewordpress: '3'
```
- 컨테이너에서 사용할 환경변수(들)
- {환경변수 이름}:{값}

volumes
```yaml
services:
 django:
  ..
   volumes:
    - ./app:/app
```
- 마운트하려는 디렉터리(들)
- {호스트 디렉터리}:{컨테이너 디렉터리}

restart
```yaml
services:
 django:
  restart: always
```
- 재시작 정책
  - restart: "no"
  - restart: always
  - restart: on-failure
  - restart: unless-stopped

build
```yaml
 django:
  build:
   context:
   dockerfile: ./compose/dajngo/Dockerfile-dev
```
- 이미지를 자체 빌드 후 사용
- image 속성 대신 사용, 별도의 도커 파일 필요

---

## 도커 컴포즈 명령어

### up   
docker-compose.yml 에 정의된 컨테이너를 실행
- docker-compose up
- docker-compose up -d
  - docker run의 -d 옵션과 동일
- docker-compose up --force-recreate
  - 컨테이너를 새로 만들기
- docker-compose up --build
  - 도커 이미지를 다시 빌드(build 로 선언했을 때만)

### start
멈춘 컨테이너를 재개
- docker-compose start
- docker-compose start wordpress
  - wordpress 컨테이너만 재개

### restart
컨테이너를 재시작
- docker-compose restart
- docker-compose restart wordpress

### stop
컨테이너 멈춤
- docker-compose stop
- docker-compose stop wordpress

### down
컨테이너 종료하고 삭제
- docker-compose down

### logs
컨테이너의 로그
- docker-compose logs
- docker-compose logs -f
  - 로그 follow

### ps
컨테이너 목록
- docker-compose ps

### exec
실행 중인 컨테이너 에서 명령어 실행
- docker-compose exec {컨테이너 이름}{명령어}
- docker-compose exec wordpress bash

### build
컨테이너 build 부분에 정의된 내용대로 빌드, build로 선언된 컨테이너만 빌드됨
- docker-compose build
- docker-compose build wordpress

---

### 개인실습
__nginx 컨테이너 만들기 , 포트 50000 연결 , index.html 파일을 만들고 컨테이너 실행(volume 옵션 활용)__
- 이미지 만들기: `docker pull --platform linux/amd64 nginx` 
- 컨테이너 실행: `docker run -d --rm -p 80:80 -v /Users/ddingu/mg:/usr/share/nginx/html nginx`
- 절대경로를 써줘야 굴러간다는 사실을 알았다. 그 이유까지는 잘 모르겠다.


__php cli 컨테이너 실행하기__
- 이미지: php:7
- 브라우저 접속이 아닌 CLI 테스트.

`<?php phpinfo() ?>`
1. hello.php를 php contatiner로 실행 (-v 옵션으로 hello.php를 연결)
2. 실행결과(php 설정 정보)를 확인

답: `docker pull php:7`   
실행: `docker run --rm -v /Users/ddingu/mg/hello.php:/app/hello.php php:7 php /app/hello.php`
- /Users/ddingu/mg/hello.php <<< 해당 경로에 hello.php 가 있다는 뜻 이것도 절대경로 사용
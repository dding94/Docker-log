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

도커는 다양한 데이터베이스를 손쉽게 생성/삭제가 가능하다.

맥북 m1 버전에서 MySQL 이미지 다운로드 및 컨테이너 실행을 위하여 `docker pull mysql:5.7` 하면 에러가 생긴다.
`--platform` 으로 명시를 해줘야 정상 작동한다.
```aidl
docker pull --platform linux/amd64 mysql:5.7
```

MySQL 컨테이너를 생성, 실행 할 때도 플랫폼 명시를 해야한다.
```aidl
docker run --platform linux/amd64 --name mysql -d -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7
```

MySQL 데이터베이스 실행하기
```
docker exec -it mysql mysql  // exec 명령어로 접속
//테이블 만들기
create database wp CHARACTER SET utf8;
grant all privileges on wp.* to wp@'%' identified by 'wp';
flush privileges;
```

exec 명령어
- exec 명령어는 run 명령어와 달리 실행중인 도커 컨테이너에 접속할 때 사용하며 컨테이너 안에 ssh server 등을 설치하지 않고 exec 명령어로 접속합니다. 

워드프레스 블로그 실행하기
```aidl
docker run -d -p 8080:80 \
  -e WORDPRESS_DB_HOST=host.docker.internal \
  -e WORDPRESS_DB_NAME=wp \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_PASSWORD=wp \
  wordpress
```
앞에 만든 MySQL을 실행한 상태에서 생성한 뒤 8080 포트로 접속
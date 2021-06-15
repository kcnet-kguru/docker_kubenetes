# 도커란?

## 1. 가상머신과 도커 컨테이너
<img width="640" src="https://gblobscdn.gitbook.com/assets%2F-M1TxO6lPQE2b5vWOIk9%2F-M1U9Pbm5zKP_maqE7_j%2F-M1UB2No2H2sWxm1YEXI%2Fimage.png?alt=media&token=a19d66a8-c0c2-46fb-a9ea-c51bfbe3e179" />

* 가상 머신은 게스트 운영체제를 사용하기 위한 라이브러리, 커널 등을 전부 포함하기 때문에 가상 머신을 배포하기 위한 이미지로 만들었을 때 이미지의 크기가 크다
* 이에 비해 도커 컨테이너는 가상화된 공간을 생성하기 위해 리눅스 자체 기능을 사용
  * chroot (change root directory)  - 현재 실행중인 프로세스와 자녀 프로세스의 루트 디렉토리를 변경하는 작업 </br>
    <img width="430" src="https://d2uleea4buiacg.cloudfront.net/files/a77/a77b00d3bd5ae17c1fa41357d3fb98d15155426e045819117945cf44ac159698.m.png" />
  * 네임 스페이스 - 프로세스를 실행할 때 시스템의 리소스를 분리해서 실행할 수 있도록 도와주는 기능
    * Cgroup 네임스페이스(cgorup)
    * IPC 네임스페이스(ipc)
    * 네트워크 네임스페이스(network)
    * 마운트 네임스페이스(mnt)
    * PID 네임스페이스(pid)
    * UTS 네임스페이스(user)
    * 사용자 네임스페이스(uts)
    * 시간 네임스페이스(time)
  * cgroup -  프로세스들의 자원의 사용(CPU, 메모리, 디스크 입출력, 네트워크 등)을 제한하고 격리시키는 리눅스 커널 기능
    <img width="430" src="https://www.cloudsigma.com/wp-content/uploads/cgroups-docker.jpg"/>

## 2. 도커엔진
* 도커에서 사용하는 기본 단위는 이미지와 컨테이너 이다.

### 2.1 도커 이미지
* 이미지는 컨테이너를 생성할 때 필요한 요소이며, 여러 개의 계층으로 된 바이너리 파일로 존재하고, 컨테이너를 생성하고 실행할 때 읽기 전용으로 사용된다.<br/>
  <img width="640" src="https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/image-layer.png" />
* 이미지는 도커 명령어로 내려받을수 있다.  

```bash
$ docker ps -a
 
CONTAINER ID  IMAGE   COMMAND   CREATED   STATUS    PORTS   NAMES

```
- docker ps 명령어
  * CONTAINER ID: 컨테이너 고유 ID
  * IMAGE: 컨테이서 생성시 사용되었던 이미지
  * COMMAND: 컨테이너가 시작될 때 실행될 명령어
  * CREATED: 컨테이너가 생성되고 난 뒤 흐른 시간
  * STATUS: 컨테이너의 상태를 나타냄
  * PORTS: 컨테이너가 개방한 포트와 호스트에 연결한 포트를 나열
  * NAMES: 컨테이너의 고유한 이름
  
### 2.2 컨테이너 애플리케이션 구축
```bash
$ docker run -d \
--name wordpressdb \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
mysql:5.7

$ docker run -d \
-e WORDPRESS_DB_HOST=mysql \
-e WORDPRESS_DB_USER=root \
-e WORDPRESS_DB_PASSWORD=password \
--name wordpress \
--link wordpressdb:mysql \
-p 80 \
wordpress
```
#### 옵션
* -e: 컨테이너 내부 환경변수를 설정
* --link: A 컨테이너에서 B 컨테이너로 접근하는 방법 중 하나

### 2.3 도커 볼륨
* 컨테이너의 데이터를 영속적(Persistent) 데이터로 활용할 수 있는 방법 몇가지 중 하나가 도커 볼류을 활용하는 것

#### 2.3.1 호스트 볼륨 공유
```bash
$ docker run -d \
--name wordpressdb_hostvolume \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
-v /home/wordpress_db:/var/lib/mysql \
mysql:5.7

$ docker run -d \
-e WORDPRESS_DB_PASSWORD=password \
--name wordpressdb_hostvolume \
--link wordpressdb_hostvolume:mysql \
-p 80 \
wordpress
```

#### 2.3.2 볼륨 컨테이너
* -v 옵션으로 볼륨을 사용하는 컨테이너를 다른 컨테이너와 공유하는 것
```bash
$ docker run -i -t \
--name volume_overide \
-v /home/wordpress_db:/home/testdir_2 \
alicek106/volume_test


$ docker run -i -t \
--name volumes_from_container \
--volumes-from volume_overide \
ubuntu:14.04

```
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FDUFT5%2FbtqFIHOPt2A%2FJ7kWqsCHp21qcmcOlDo8Nk%2Fimg.png"/>


### 2.3.3 도커 볼륨
* 도커 자체에서 제공하는 볼륨 기능을 활용해 데이터를 보존
* 볼륨은 디렉터리 하나에 상응하는 단위로서 도커 엔진에서 관리
* 도커 볼륨도 호스트 볼륨 공유와 마찬가지로 호스트에 저장함으로써 데이터를 보존
```bash
$ docker volume create --name myvolume

$ docker run -i -t --name myvolumne_1 \
-v myvolume:/root/ \
ubuntu:14.04

```

### 2.3.4 도커 네트워크
<img width="540" src="https://media.vlpt.us/images/brslover/post/10d97dfe-20ae-4b97-b55d-7724882a0ea9/image.png"/>

* docker0 브리지는 각 veth 인터페이스와 바인딩돼 호스트의 eth0 인터페이스에 이어주는 역할을 한다.
* 도커 네트워크의 기능
  * **브리지 네트워크 :** 브리지 네트워크는 docker0이 아닌 사용자 정의 브리지를 새로 생성해 각 컨테이너에 연결하는 네트워크 구조
  * **호스트 네트워크 :** 네트워크를 호스트로 설정하면 호스트의 네트워크 환경을 그대로 쓸 수 있다.
  * **논(none) 네트워크 :** none은 말 그대로 아무런 네트워크를 쓰지 않는 것을 뜻한다.
  * **컨테이너 네트워크 :** --net 옵션으로 container를 입력하면 다른 컨테이너의 네트워크 네임스페이스 환경을 공유 할 수 있다.

## 2.4 도커 이미지
* 모든 컨테이너는 이미지를 기반으로 생성되며, 도커 허브(Docker Hub)라는 중앙 이미지 저장소에서 이미지를 내려 받을 수 있다.
* 도커 이미지 저장소를 직접 구축해 비공개로 사용할 수도 있다.

### 2.4.1 도커 이미지 생성
* 컨테이너를 생성 후 docker commit 명령어로 해당 컨테이너를 이미지화 할 수 있다.
```bash
$ docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```
<img width="540" src="https://sachcode.com/static/3422749f7f459abcf3de835201c77794/c1b63/docker-layers.png" />


## 2.5 Dockerfile
* 개발한 애플리케이션을 컨테이너화 하는 방법
  * 아무것도 존재하지 않는 이미지(우분투, CentOS 등)로 컨테이너를 생성
  * 애플리케이션을 위한 환경을 설치하고 소스코드 등을 복사해 잘 동작하는 것을 확인
  * 컨테이너를 이미지로 커밋(commit)
  
* 도커는 위와 같은 일련의 과정을 손쉽게 기록하고 수행할 수 있는 빌드(build) 명령어를 제공(Dockerfile)

```dockerfile
FROM ubuntu:14.04
MAINTAINER alicek106
LABEL "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
EXPOSE 80
CMD apachectl -DFORGEGROUND
```
* Dockerfile
  * FROM: 생성할 이미지의 베이스
  * MAINTAINER: 이미지를 생성한 개발자의 정보
  * LABEL: 이미지에 메타데이터를 추가
  * RUN: 이미지를 만들기 위해 컨테이너 내부에서 명령어를 실행
  * ADD: 파일을 이미지에 추가, 파일은 Dockerfile이 위치한 디렉터리인 Context에서 가져옴
  * WORKDIR: 명령을 실행할 디렉터리를 나타냄
  * EXPOSE: Dockerfile의 빌드로 생성된 이미지에서 노출할 포트를 설정
  * CMD: 컨테이너가 시작될 때마다 실행할 명령어(커맨드)를 설정하며, Docker에서 한 번만 사용할 수 있다.
  


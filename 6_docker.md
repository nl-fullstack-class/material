# Docker

### Docker란?
다양한 운영체제와 시스템 환경 상에서, 서버 세팅을 위한 작업이 각각 다르고 복잡하다.  
도커는 컨테이너 기반의 가상화 플랫폼으로, 컨테이너 위에 서버를 세팅할 수 있다.  
따라서 기반 환경이 다르더라도 도커를 통해 컨테이너를 실행하면 동일한 서버 세팅이 가능하다.

### Docker 설치

* 리눅스 : 
```
sudo apt update
sudo apt-get install curl  
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```
* 맥 : 
```
brew install --cask docker
```
* 윈도우 : 
아래 웹사이트에 접속하여 설치파일 다운로드 후 설치
https://docs.docker.com/desktop/setup/install/windows-install/ 
혹은, WSL2 위에 Docker 설치


### Docker 설치 확인
```
docker version
```

### Docker의 작동 원리
* LXC(Linux Container)
* 리눅스 운영체제 레벨에서 영역과 자원할당(CPU,메모리 등)을 분리하여 마치 별도의 시스템처럼 사용
* 컨테이너 위에 서버 세팅 가능
* 도커는 리눅스 커널에 LXC를 사용해서 컨테이너를 만들고, 컨테이너 상에 별도로 구성된 파일 시스템에 시스템 설정 및 응용 프로그램을 실행할 수 있도록 하는 기술을 정의한 것이라고 이해하면 됨
* 최근 도커는 별도 컨테이너 기술을 구현하여 사용하고 있음

### Docker의 구성 요소
* Docker Engine : 도커는 서버/클라이언트 구조로 이루어짐  
    * 도커 서버: docker daemon 프로세스
    * 도커 클라이언트: docker 명령어를 통해 도커 서버에 command을 전달하는 인터페이스  
    * 도커 서버와 클라이언트 간에는 REST API를 통해 통신
* Docker Image : docker 컨테이너를 생성하기 위한 명령들을 가진 템플릿.
    * 여러 이미지들을 레이어로 쌓아서 원하는 형태의 이미지를 만든다.
    * 예시: ubuntu 이미지에 apache 웹서버를 설치하여 원하는 형태의 이미지를 만든다.
* Docker Container : 도커 이미지를 실행한 상태 (instance)
    * docker daemon에 있는 커널에서 리눅스 컨테이너를 생성한 후, 해당 컨테이너에 docker image에 포함된 명령을 실행하여, docker container를 만든 후 실행

### Docker image
* Docker Hub에서 다운로드 가능
* https://hub.docker.com/ 에서 회원가입
* Docker for Mac, Docker for Windows에서는 GUI로 로그인 가능
* 리눅스에서는 다음곽 같이 로그인
```
docker login
username, password 입력
docker logout
```

* 도커 이미지 검색
```
docker search [image name]
ex) docker search ubuntu
ex) docker search --limit=5 ubuntu
```

* 도커 이미지 다운로드
```
docker pull [image name]
ex) docker pull ubuntu
ex) docker pull ubuntu:22.04
```

* 다운로드 받은 도커 이미지 목록 확인
```
docker images
docker image ls
docker images -q (image id만 출력)
docker image ls -q (image id만 출력)
```

* 다운로드 받은 도커 이미지 삭제
```
docker image rm [image name]
docker image rm [image id]
docker rmi [image name]
docker rmi [image id]
```

### Docker container
* 컨테이너 생성
    * 각 이미지는 컨테이너로 만들어줘야 실행 가능함
    * 이미지와 컨테이너는 각각 관리해줘야 함
    * 컨테이너 생성시, docker 프로그램에서 이름이 자동으로 부여됨
```
docker create [image name]
ex) docker create ubuntu

docker create --name [내가 원하는 컨테이너 이름] [image name]
```

* 생성된 컨테이너 확인
```
docker ps       (실행중인 컨테이너 확인)
docker ps -a    (모든 컨테이너 확인)
docker ps -a -q  (모든 컨테이너 id 확인)
```

항목 설명
* CONTAINER ID : 컨테이너 ID
* IMAGE : 컨테이너를 생성하기 위해 사용한 이미지
* COMMAND : 컨테이너 실행 시 실행된 명령어
* CREATED : 컨테이너 생성 시간
* STATUS : 컨테이너 상태 (Created: 생성됨, Up: 실행중, Paused: 일시정지, Exited: 종료됨)
* PORTS : 컨테이너 포트 정보
* NAMES : 컨테이너 이름




* 컨테이너 실행
```
docker start [container id]
docker start [container name]
```
* 실행시 바로 중지되는 이유
    * 도커는 컨테이너를 하나의 응용 프로그램처럼 다루고 있음.
    * 따라서 컨테이너에서 실행하게끔 설정된 응용 프로그램의 실행이 끝나면, 해당 컨테이너는 중지됨.
    * 별도의 터미널 및 표준 입력 연결 설정이 없기 때문에 컨테이너기 바로 중지되는 것임.
* unix standard stream에 대해서...
    * 유닉스 계열에서 동작하는 프로그램은 실행 시 세 개의 스트림이 오픈됨.
        * 표준 입력(stdin)
        * 표준 출력(stdout)
        * 표준 에러(stderr)
    * 도커는 컨테이너를 실행하는 시점에 이 스트림을 자동으로 연결해줌.
    * 따라서 컨테이너 내에서 실행되는 프로그램은 표준 입력을 기다리다가 아무것도 없으면 종료됨.
    * 따라서 컨테이너 실행 시 바로 중지되는 것임.
* 컨테이너가 중지되지 않게 하는 방법
    * 쉘 프로그램을 실행해야 함
        * 쉘 프로그램은 fork() 시스템 콜을 사용해서 자식 프로세스를 생성하는데, 이 때 표준 입력을 복사해서 자식 프로세스에 전달함.
        * 따라서 쉘 프로그램을 실행하면 컨테이너는 중지되지 않음.
        * 쉘 프로그램 예시: sh, bash, zsh 등
        * 쉘 프로그램이 아닌 프로그램 예시: vim, telnet, ftp 등

```
반드시 이미지로 실행해야한다.
docker run -it [image name]
ex) docker run -it ubuntu
docker run -it -d ubuntu (컨테이너 백그라운드 실행)
```
여기서 i는 표준 입력을 연결하고, t는 터미널 입출력(tty)을 연결하는 옵션임.


* 데몬으로 실행중인 컨테이너에 명령 실행하기
```
docker exec -it [container id] [command]
ex) docker exec -it 1234567890 /bin/bash
```

* 컨테이너 종료
```
docker stop [container id]
docker stop [container name]
```

* 컨테이너 삭제
```
docker rm [container id]
docker rm [container name]
```

* docker가 사용하고 있는 저장매체 현황 확인
```
docker system df
```

* 실행중인 컨테이너가 사용하는 시스템 자원 확인하기
```
docker container stats
```

* alpine 이미지란...
    * 도커 이미지 중 가장 기본이 되는 작은 이미지
    * 통상 리눅스 사용 시, 다양한 기능을 가진 ubuntu 등의 리눅스 패키지를 사용하지만, 
    * 도커 컨테이너의 경우는 특정 응용 프로그램 실행을 목적으로 하는 경우가 많기 때문에, 경량화된 이미지 사용.

### 웹서버 띄우기 실습
```
docker search httpd --limit=5
docker pull httpd
docker run -d -p 9999:80 --name myweb httpd
docker exec -it myweb /bin/sh
# ls
# htdocs
# apt-get update
# apt-get install vim
# vim index.html
# exit
```

### 도커 청소
```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
docker rmi -f $(docker images -q)
```
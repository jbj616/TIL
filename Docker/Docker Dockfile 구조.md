## Docker Dockfile 구조

### 예제 Dockfile

```dockerfile
From node:0.10

MAINTAINER Anno Doe <anna@example.com>

LABEL "rating"="Five Starts" "class"="First Class"

User root

ENV AP /data/app
ENV SCPATH /etc/supervisor/conf.d

RUN apt-get -y update

# The daemons
RUN apt-get -y install supervisor
RUN mkdir -p /var/log/supervisor

# Supervisor Configuration
ADD ./supervisord/conf.d/* $SCPATH/

# Application Code
ADD *.js* $AP/

WORKDIR $AP

RUN npm install

CDM ["supervisrod", "-n"]
```



### 명령어

`From node:0.10`

- Node.0.10.x를 실행하는 우분투 리눅스 이미지를 제공할 것



`MAINTAINER Anno Doe <anna@example.com>`

- 메타데이터에 Author 필드



`LABEL "rating"="Five Starts" "class"="First Class"`

- key=value 쌍으로 된 메타데이터



`USER root`

- 컨테이너 내에서 모든 프로세스를 root로 실행



```dockerfile
ENV AP /data/app
ENV SCPATH /etc/supervisor/conf.d
```

- 셸 환경 변수를 설정



```dockerfile
RUN apt-get -y update

# The daemons
RUN apt-get -y install supervisor
RUN mkdir -p /var/log/supervisor
```

- apt-get -y update -> 업데이트시 빌드 시간을 길게 할 수 있음
- `RUN` - 필요한 파일/디렉터리 구조를 만들고, 요구되는 소프트웨어를 설치하기 위해 사용됨



```dockerfile
# Supervisor Configuration
ADD ./supervisord/conf.d/* $SCPATH/

# Application Code
ADD *.js* $AP/
```

- `ADD` - 로컬 파일 시스템의 파일을 이미지로 카피하는 데 씀

  - 코드나 필요한 보조 파일들을 카피하는 데 쓰임

  > ADD는 로컬 파일 시스템의 파일을 이미지에 포함할 수 있게 해준다. 하지만 한번 이미지가 빌드되고 나면 파일이 이미 이미지에 카피되어 있으므로 원래 파일에 접근하지 않아도 이미지를 사용할 수 있다



`WORKDIR $AP`

- 이후의 작업 내용들을 위해 작업 디렉터리를 바꿀 수 있도록 함.



#### !실행순서 주의

빌드시 적용되는 변경사항이 마지막 부분에 위치하도록 해야함.

​	why? 이미지를 새로 빌드하면 처음으로 바뀐 부분부터 새로 빌드되기 때문



`CDM ["supervisord", "-n"]`

- 컨테이너 안에서 실행하고 싶은 프로세스를 띄우는 명령어 지정



#### Build 명령어

`docker build -t <tagName>`


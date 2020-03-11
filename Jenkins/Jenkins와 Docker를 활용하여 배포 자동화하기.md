## Jenkins와 Docker를 활용하여 배포 자동화하기

### EC2

1. 시작하기 앞서 `sudo apt update` 명령어를 이용하여 apt 업데이트하기

2. Docker 를 설치한다. `sudo apt install docker.io`

3. systemctl 설정

   ```
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

   ---

4. docker를 통해 jenkins 컨테이너 받아오기

   `sudo docker pull jenkins`

5. jenkins 이미지를 사용한다 

   `sudo docker run -d -p 8080:8080 -v /home/jenkins:/var/jenkins_home jenkins`

6. jenkins 8080포트로 접속하여 설정한다. 

7. jenkins 컨테이너 접속 `docker exec -it c7d2c6d8bb28 /bin/bash`

8. jenkins 내부에 docker build 설치하기

   `curl -s httpS://get.docker.com/ | sudo sh`

--> plugin error 이유 : jenkins 컨테이너로 받을 때는 버전이 낮아서 plugin 설치가 안됨.

---

4. 컨테이너가 아닌 jenkins 직접설치

   ```
   sudo apt update
   sudo apt install openjdk-8-jdk
   
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   
   sudo apt update
   sudo apt install jenkins
   
   systemctl status jenkins
   ```

5. 8080포트로 접속하여 세팅하기

6. jenkins

   - Git repository URL
   - GiHub project URL
   - GIt WebHook 

7. sudo err -> https://gist.github.com/hayderimran7/9246dd195f785cf4783d

8. shell script 작성 



### Docker

#### SprinbBoot DockerFile 만들기

```java
FROM openjdk:8-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]

출처 : spring.io - https://spring.io/guides/gs/spring-boot-docker/
```

```java
# Start with a base image containing Java runtime
FROM java:8

# Add Author info
LABEL maintainer="f.softwareengineer@gmail.com"

# Add a volume to /tmp
VOLUME /tmp

# Make port 8080 available to the world outside this container
EXPOSE 8080

# The application's jar file
ARG JAR_FILE=build/libs/MySpringApp-0.0.1-SNAPSHOT.jar

# Add the application's jar to the container
ADD ${JAR_FILE} to-do-springboot.jar

# Run the jar file
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/to-do-springboot.jar"]

출처: https://imasoftwareengineer.tistory.com/40 [삐멜 소프트웨어 엔지니어]
```

- FROM : 이미지를 만들기위해 기반이 되는 이미지 레이어
  - `From <이미지 이름> : <태그>` 
- LABEL : 이미지를 관리하는 사람이 누구인지 명시
- VOLUME : 컨테이너가 필요한 여러가지 데이터를 저장하는 곳
- EXPOSE : 포트 넘버
- ARG : 어떤 어플리케이션을 실행하는지, 즉 어플리케이션으 실행파일은 연결
- ADD${JAR_FILE} : 이름을 붙여준다.
- ENTRYPOINT : 실행시키기 위한 명령어



### 으쌰 프로젝트에 적용

- 첫번째 DockerFile을 적용

- ```
  docker build --build-arg JAR_FILE=build/libs/*.jar -t sproutt/eussya-eussya-api .
  docker run -p 8080:8080 -t <TagName>
  ```

### 참고

https://imasoftwareengineer.tistory.com/40

https://spring.io/guides/gs/spring-boot-docker/
# nginx-docker-spring
간단한 springboot app을 이용해 간단히 실습합니다~
gradle 기반입니당 :)

## 0. 도커 엔진 시작
- 설치는 https://docs.docker.com/engine/install

## 1. 간단 rest controller 정의
```java
@SpringBootApplication
@RestController
public class DockerApplication {

    @RequestMapping("/")
    public String home() {
        return "Hello Docker World";
    }
    public static void main(String[] args) {
        SpringApplication.run(DockerApplication.class, args);
    }
}
```

## 2. 루트_폴더/Dockerfile 생성
```dockerfile
# 이미지 베이스
FROM openjdk:11-jdk 

# 환경 변수 -> spring jar 이 생성되는 위치
ARG JAR_FILE=build/libs/*.jar

# jar 파일을 app.jar 이라는 이름으로 복사
# 실행할 jar 파일명 통일 위해서 ~ jar 파일명 매번 달라지면 실행 번거로움
COPY ${JAR_FILE} app.jar

# 이미지를 container 로 씌월 때 jar 파일이 실행되도록 (=서버가 구동되도록) 커맨드 설정
# shell 스크립트 직접 작성해서 선언하는 것도 가능
# ex) run.sh : #!/bin/sh exec java -jar /app.jar
# ENTRYPOINT["run.sh"]
ENTRYPOINT ["java","-jar","/app.jar"]
```

## 3. 도커 이미지 생성 위한 jar 파일 생성
```shell
$ ./gradlew bootJar
```
## 4. 도커파일 빌드(= 도커 이미지 생성) 
- 도커 이미지 생성
```shell
$ docker build -t [생성 이미지명]/[artifact] [도커 파일 위치] .
```
- ex)
```shell
$ docker build --build-arg DEPENDENCY=build/dependency -t aa9919/springboot_test .
```

## 5. (로컬)컨테이너 생성 / 실행 테스트
```shell
// -p 태그 : 컨테이너 내부 포트와 호스트의 포트 끼리의 포워딩
// 포트는 맞춘대로 ! (나는 8080)

// 1. run : 생성과 실행까지한다
$ docker run -p [외부 포트:컨테이너 내부 포트] [이미지명]

// 2. create : 컨테이너 생성만 하고 시작하지는 않는다.
$ docker create --name [서버 이름] -p [외부 포트:컨테이너 내부 포트] [이미지명:버전 태그]

// 2-1. 컨테이너 확인 (-a : 실행중이지 않은 컨테이너 포함)
$ docker ps -a

// 2-2. 실행
$ docker start 서버명
```

## 6. 도커 허브 push
- 도커 허브 레포지토리에서 푸쉬 명령어 확인한다.
- 태그명은 일단 떼고 넣는다
- tag? 이미지는 url 방식으로 관리하며 태그를 붙일 수 있습니다.
```shell
$ docker push aa9919/springboot_test
```

# EC2에 배포

## 1. EC2 접 & 도커 설치
```shell
$ sudo apt update

// Https 이용해 repository 사용할 수 있도록 필요 패키지 추가
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common

// 도커 공식 GPG 키 추가
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

// stable repository 설정 위한 명령어
$ udo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   
// 패키지 업데이트
$ sudo apt update

// docker engine, containered 최신버전 설치
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

// 테스트 도커 이미지 구동
$ sudo docker run hello-world

// 해당 이미지 컨테이너 아이디 확인
$ sudo docker ps -a

// 테스트 도커 컨테이너 삭제
$ sudo docker rm [테스트 컨테이너 아이디]
```

## 2. 도커 허브에 올라간 도커 이미지 Pull 받기
```shell
$ sudo docker pull [아이디]/[레포명]
```
## 3. 컨테이너 구동 명령어로 도커 이미지 배포
```shell
$ sudo docker run -p [외부 포트:컨테이너 내부 포트] [이미지명]
```
## 추가
````shell
// 구동 중인 컨테이너 확인
$ sudo docker ps -a

// 컨테이너 삭제 
$ sudo docker rm [컨테이너 아이디]

// 생성된 도커 이미지 목록 확인
$ sudo docker images

// 도커 이미지 삭제
$ sudo docker rmi [이미지명]:[Tag]
````
#  Docker Basic 03 - Dockerfiles

#dev/skill/container/docker

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/42826236#overview) 

[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/43367448#overview)

## 개요
![](Docker%20Basic%2003%20-%20Dockerfiles/image.png)<!-- {"width":870} -->
이번 장에서는 Dockerfile에 대해 알아보자

## Dockerfile 구성
> Docker는 `Dockerfile`을 읽어서 `Docker Image`를 생성한다.
> 이제 우리는 `Docker run` 명령어를 더 깔끔하게 작성할 수 있게 될 것이다.

1. FROM -> docker의 기본 이미지를 지정하는 데 사용한다.
   * pull image와 같으며, Docker 이미지의 첫 번째 Layer로 넣는다 -> 기본 이미지(base)가 된다.
2. WORKDIR -> 작업 디렉토리를 지정한다.
3. COPY -> 복사할 애플리케이션을 지정한다. 
   - (2, 3번은 앞선 Docker run 명령어에서 마운트 과정과 유사하다)
4. CMD -> 수행할 명령어를 작성한다.

## Dockerfile 기본
> 이제 Dockerfile을 작성하고, 컨테이너를 생성하자.

### 작성
강의에서 아래와 같은 샘플을 제공하고 있다.
```dockerfile
## 1. Which base image do you want to use?

## 2. Set the working directory.

## 3. Copy your source code file to the working directory.

## 4. Define the command to run when the container starts.
```

실제 샘플을 작성하자.
```dockerfile
## 1. Which base image do you want to use?
FROM python:3.8-slim

## 2. Set the working directory.
WORKDIR /app

## 3. Copy your source code file to the working directory.
COPY python-app.py /app

## 4. Define the command to run when the container starts.
CMD ["python", "/app/python-app.py"]

```

그리고 다음과 같이 명령한다. Dockerfile을 읽어 빌드하는 과정이다.
-> `docker build -t python-app . `
- docker build
- -t : 태그 이름을 지정한다.
- . : 현재 위치를 의미한다 -> 현재 디렉토리 내에 Dockerfile이 존재한다.

### 빌드
- 먼저 docker build를 진행하고
  ![](Docker%20Basic%2003%20-%20Dockerfiles/image%202.png)
- 이후 docker images를 확인해 보면, 방금 빌드한 `python-app`이 생성되었다.
  ![](Docker%20Basic%2003%20-%20Dockerfiles/image%203.png)

### 실행
이제 앞서 생성한 이미지를 가지고 컨테이너를 실행하자.
- `docker run --rm --name python-app-container python-app`
- 명령어가 깔끔하게 줄어들었다.
- 실행 결과 -> 
  ![](Docker%20Basic%2003%20-%20Dockerfiles/image%204.png)

### 살펴보기
> 추가적으로, 실행된 컨테이너 내부를 살펴보고, 상호작용하기 위한 방법을 알아보자.

기존 명령어에 ==-it==를 추가하고, ==/bin/sh==를 추가한다.
- `docker run -it --rm --name python-app-container python-app /bin/sh`
- 실행 결과 -> 
  ![](Docker%20Basic%2003%20-%20Dockerfiles/image%205.png)
- 보다시피, 우리는 곧장 **작업 디렉토리(WORKDIR)** 위치로 접근한 것을 알 수 있다. (pwd -> /app)
- 당연하게도, 기존 명령어를 실행하면 아래와 같이 정상 동작한다. -> `python python-app.py`
  ![](Docker%20Basic%2003%20-%20Dockerfiles/image%206.png)<!-- {"width":447} -->
## Dockerfile 응용

### COPY 파트
기존 dockerfile에서 COPY target 부분을 `'.'`으로 바꿔보자. 이것은 **작업 디렉토리(WORKDIR)과 같은 위치**를 의미한다.
- dockerfile 수정
```dockerfile
## 1. Which base image do you want to use?
FROM python:3.8-slim

## 2. Set the working directory.
WORKDIR /app

## 3. Copy your source code file to the working directory.
# COPY python-app.py /app
COPY python-app.py .

## 4. Define the command to run when the container starts.
CMD ["python", "/app/python-app.py"]

```
- 이후 dockerfile을 다시 빌드하고, 해당 이미지로 컨테이너를 실행해도 동일한 결과가 나온다
![](Docker%20Basic%2003%20-%20Dockerfiles/image%207.png)

마찬가지로 COPY source 부분 역시 현재 디렉토리로 지정할 수 있다.
```dockerfile
## 1. Which base image do you want to use?
FROM python:3.8-slim

## 2. Set the working directory.
WORKDIR /app

## 3. Copy your source code file to the working directory.
# COPY python-app.py /app
COPY . .

## 4. Define the command to run when the container starts.
CMD ["python", "/app/python-app.py"]

```
다만 이 경우에는 현재 디렉토리의 모든 파일이 마운트되기 때문에 심지어 `dockerfile`까지 같이 전달된다(!).
- 실행 결과 -> 
![](Docker%20Basic%2003%20-%20Dockerfiles/image%208.png)

### .dockerignore 파일 생성
위와 같은 경우를 방지하기 위해서 `.dockerignore` 파일을 작성할 수 있다. 마치 `.gitignore`와 같다.
- 예제
```docker
Dockerfile
```
이후, 다시 빌드하고 이미지를 실행한 결과 다음과 같이 dockerfile은 보이지 않는다.
![](Docker%20Basic%2003%20-%20Dockerfiles/image%209.png)

## Docker Image 태그
> docker 이미지 생성 시 태그는 자동으로 latest가 된다.
> 이것을 선호하지 않는다면 반드시 태그를 지정하자.

다음과 같이 명령하면 태그가 붙게 된다.
```sh
docker build -t python-app:0.0.1 .
docker images
```
```sh
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
python-app   latest    2f33e800016f   12 minutes ago   215MB
python-app   0.0.1     405682fdaff7   12 minutes ago   215MB
```

## Dockerfile 실습
### Java 애플리케이션
이번에는 java 애플리케이션을 dockerfile로 빌드하고 컨테이너를 생성하자.
#### Dockerfile 작성
```dockerfile
## 1. Which base image do you want to use?
FROM openjdk:11-jre-slim

## 2. Set the working directory.
WORKDIR /app/

## 3. Copy your source code file to the working directory.
COPY . .

## 4. Define the command to run when the container starts.
CMD [ "java", "-cp", "JavaApp.jar", "JavaApp" ]
```
#### docker build
```sh
docker build -t java-app:0.0.1 .
docker images
```
```sh
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
java-app     0.0.1     24f22aa039a1   4 seconds ago    306MB
python-app   latest    2f33e800016f   17 minutes ago   215MB
python-app   0.0.1     405682fdaff7   17 minutes ago   215MB
```
#### docker run
```sh
docker run --rm --name java-app-container java-app:0.0.1
```
```sh
     ____.  _________   _________
    |    | /  _  \   \ /   /  _  \
    |    |/  /_\  \   Y   /  /_\  \
/\__|    /    |    \     /    |    \
\________\____|__  /\___/\____|__  /
                 \/              \/
```
![](Docker%20Basic%2003%20-%20Dockerfiles/image%2010.png)
### Ruby 애플리케이션
> 일반적인 ruby 예제
#### dockerfile 작성
```dockerfile
# Dockerfile for Ruby User Input
FROM ruby:3.0

WORKDIR /app/

COPY script.rb . 

CMD ["ruby", "script.rb"] 

```
#### docker build & run
```sh
docker build -t ruby-app:0.0.1 .
...
docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
ruby-app     0.0.1     5d046bf45a39   29 seconds ago   1.25GB
java-app     0.0.1     24f22aa039a1   25 minutes ago   306MB
python-app   0.0.1     405682fdaff7   42 minutes ago   215MB
python-app   latest    2f33e800016f   42 minutes ago   215MB
...
docker run --rm --name ruby-app-container ruby-app:0.0.1
```
![](Docker%20Basic%2003%20-%20Dockerfiles/image%2012.png)
### Python 애플리케이션
> 이번에는 ==파라미터==를 넘기는 예제다.
#### dockerfile 작성
```dockerfile
# Dockerfile for Python Script
FROM python:3.8-slim
WORKDIR /app
COPY script.py . 
CMD ["python", "script.py", "Hello", "Docker", "World!"]
```
#### docker build & run
```sh
docker build -t python-script-app .
...
docker images 
REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
python-script-app   latest    2aae14b90c8a   19 seconds ago   215MB
ruby-app            0.0.1     5d046bf45a39   11 minutes ago   1.25GB
java-app            0.0.1     24f22aa039a1   36 minutes ago   306MB
python-app          0.0.1     405682fdaff7   54 minutes ago   215MB
python-app          latest    2f33e800016f   54 minutes ago   215MB
...
docker run --rm --name python-script-app-container python-script-app
...
```
![](Docker%20Basic%2003%20-%20Dockerfiles/image%2013.png)
### Go 애플리케이션
> 이번에는 환경변수가 존재하나, Dockerfile로 지정하는 설명이 없다. 불가능 한가? 
> -> 결론부터 이야기하면 ==가능==하다.
#### dockerfile 작성 (기본)
```dockerfile
# Dockerfile for Go Environment Variables
FROM golang:1.16
WORKDIR /app
COPY main.go . 
CMD ["go", "run", "main.go"]
```
#### docker build & run
```sh
docker build -t go-app:0.0.1 . 
...
docker images
REPOSITORY          TAG       IMAGE ID       CREATED             SIZE
go-app              0.0.1     d63c4eb0a378   5 seconds ago       1.17GB
python-script-app   latest    2aae14b90c8a   15 minutes ago      215MB
ruby-app            0.0.1     5d046bf45a39   26 minutes ago      1.25GB
java-app            0.0.1     24f22aa039a1   51 minutes ago      306MB
python-app          latest    2f33e800016f   About an hour ago   215MB
python-app          0.0.1     405682fdaff7   About an hour ago   215MB
...
docker run --rm --name go-app-container go-app:0.0.1
Hello, World!
```

그러나 위 버전은 ==ENV==를 지정하지 않았다. 물론 ==Docker run== 명령어 작성 시, ==-e== 속성을 사용해 가능하다.
```sh
docker run --rm -e MESSAGE="Yes this is ENV" --name go-app-container go-app:0.0.1
Yes this is ENV
```

그런데 강의에서 **Dockerfile로 빌드하는 과정**에서 ==ENV==를 넣는 방법은 설명하지 않았다. 관련해 찾아보니 가능한 것을 확인했다. 
#### dockerfile 작성 (ENV 포함)
```dockerfile
# Dockerfile for Go Environment Variables
FROM golang:1.16
WORKDIR /app
COPY main.go . 
ENV MESSAGE="Yes this is ENV in dockerfile"
CMD ["go", "run", "main.go"]
```
#### docker build & run (ENV 포함)
```sh
docker build -t go-app:0.0.2 .
...
docker images
REPOSITORY          TAG       IMAGE ID       CREATED             SIZE
go-app              0.0.1     d63c4eb0a378   5 minutes ago       1.17GB
go-app              0.0.2     b12cd2505313   5 minutes ago       1.17GB
python-script-app   latest    2aae14b90c8a   20 minutes ago      215MB
ruby-app            0.0.1     5d046bf45a39   32 minutes ago      1.25GB
java-app            0.0.1     24f22aa039a1   57 minutes ago      306MB
python-app          0.0.1     405682fdaff7   About an hour ago   215MB
python-app          latest    2f33e800016f   About an hour ago   215MB
...
docker run --rm --name go-app-container go-app:0.0.2
Yes this is ENV in dockerfile
```
의도한 대로 동작하는 것을 확인했다. 

---
## 정리
> Dockerfile을 통한 Docker image 생성에 대해 알아봤다.
- Docker Image 생성 시, ==태그==를 입력할 수 있다. 입력하지 않으면 `latest`가 기본이다.
- Dockerfile은 기본적으로(최소?) 3개의 계층으로 이루어져 있다.
  1. ==FROM== -> 애플리케이션에 필요한 **기본 환경** 정의
  2. ==WORKDIR== -> 작업 디렉토리 지정, 이후 실행될 명령의 기준
  3. ==COPY== -> 애플리케이션 소스 코드 복사 정의
- 그리고 마지막으로 ==CMD==를 통해 컨테이너 시작 시 실행할 명령어를 정의한다.
  - 나중에 학습하는 키워드
    - ==ADD== : ==COPY==와 유사한 목적으로 사용되며 URL 다운로드 및 압축 해제 기능 등이 있으나, ==COPY==를 권장
    - ==ENV== : 환경변수 입력 (==ARG==는 빌드 시점에만 사용되는 환경변수)
    - ==RUN== : 이미지 빌드 과정에서 실행하는 명령어 입력 (==CMD==와 ==ENTRYPOINT==도 있다)
    - ==EXPOSE== : 컨테이너가 사용할 포트를 명시
    - ==VOLUME== : 볼륨 설정
    - ==LABEL== : 이미지에 작성자, 버전 등의 정보를 입력하기 위함
- ==Dockerfile==로 [생성](bear://x-callback-url/open-note?id=4D5740F7-6C9C-4630-BF99-BAC7DD453B0D&header=%EA%B0%9C%EC%9A%94)된 ==Image==, 그리고 그 Image를 이용해 ==Docker Container==를 실행한다.
![](Docker%20Basic%2003%20-%20Dockerfiles/image%2011.png)

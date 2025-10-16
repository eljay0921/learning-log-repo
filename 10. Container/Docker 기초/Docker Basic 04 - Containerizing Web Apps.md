#  Docker Basic 04 - Containerizing Web Apps

#dev/skill/container/docker

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/43364174#overview)

## 개요
> 이번 장은 Web Application을 컨테이너로 생성하는 실습이다.

### Docker 시스템 정리
진행에 앞서, docker를 깔끔하게 정리하자. 아래 명령어는 모든 정지된 컨테이너, 이미지, 지금까지 진행한 빌드를 제거한다.
```sh
docker system prune -a
```

#### 기본 동작 
```sh
docker system prune
```
위 명령어 수행 시, 항목들을 삭제한다.
* **중지된 컨테이너**
* **사용되지 않는 네트워크** (컨테이너가 연결되지 않은 네트워크)
* **dangling 이미지** (태그가 없거나 어떤 컨테이너도 사용하지 않는 이미지 레이어)
* **빌드 캐시**
#### -a 옵션 추가
```sh
docker system prune -a
```
- 위 기본 명령어에서 삭제하는 리소스 + **사용되지 않는 모든 이미지**를 삭제
- **dangling 이미지뿐 아니라, 현재 사용 중이지 않은 이미지까지 전부 삭제**
  * 예: 어떤 컨테이너에서도 참조하지 않는 이미지
  * 다음에 다시 사용하려면 docker pull로 이미지를 새로 받아야 함
* 중지된 컨테이너, 빌드 캐시, 사용되지 않는 네트워크도 모두 삭제
#### 삭제하지 않는 것
* 실행 중인 컨테이너에서 사용 중인 이미지
* 실행 중인 컨테이너 자체
* **볼륨(volume)** 은 기본적으로 삭제하지 않음 (별도 옵션 필요)
-> 볼륨까지 삭제하려면, 
```sh
docker system prune -a --volumes
```



## Flask Web App
> Python flask 웹 애플리케이션을 Docker Container로 생성하자.

### 요구사항 확인
먼저, 아래 경로로 이동해 해당 web application에 어떤 것들이 필요한지 확인하자.
-> /Users/.../sources-lectures/docker-course-remastered-main/lesson-starter-projects/05-starter-code/**flask-demo**
`app.py`
```python
from flask import Flask
from app_routes import configure_routes

app = Flask(__name__)
configure_routes(app)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)


# Developer: Captain DevOps, there are a couple of ways to get this Flask app up and running:
#
# First, install the dependencies with:
#   pip install -r requirements.txt
#
# Then, choose one of the following options to run the app:
#
# Option 1:
#   Run the app directly with:
#     python app.py
#
# Option 2:
#   Set the FLASK_APP shell environment variable and run using the flask command.
#   For Git Bash:
#     export FLASK_APP=app.py
#   For PowerShell:
#     $env:FLASK_APP = "app.py"
#   For Command Prompt:
#     set FLASK_APP=app.py
#   For Mac Terminal:
#     export FLASK_APP=app.py
#   Then run the app with:
#     flask run
```
`requirements.txt`
```text
Flask==2.0.1
Werkzeug==2.0.1
```

### Dockerfile 작성하기 (v1)
```dockerfile
## 1. Which base image do you want to use?
FROM python:3.8-slim

## 2. Set the working directory
WORKDIR /app/

## 3. Copy the project files into the working directory.
COPY . . 

## 4. Install the dependencies
RUN pip install -r flask-demo/requirements.txt

## 5. Document and inform the developer that the application will use PORT 5000 of the container.
EXPOSE 5000

## 6. Define the command to run when the container starts.
CMD ["python", "flask-demo/app.py"]
```
4번의 ==RUN== 명령어의 경우 이미지를 **build**하는 과정에서 실행되는 것으로, 6번의 ==CMD==와 차이가 있다. CMD는 **컨테이너가 시작될 때 실행**된다. RUN 명령어를 통해, 해당 이미지에 필요한 ==종속성==**을 설치**한다. 이때, RUN 명령어에 필요한 내용은 `app.py` 내에서 확인할 수 있었다. 단, `app.py` 문서(주석)와 다르게 `requirements.txt`는 `flask-demo/` 디렉토리 하위에 위치한다. 그 점을 반영하자.
5번에는 이 Web App이 사용할 ==PORT==의 **매핑 정보**를 정의한다.

### docker build & run (0.0.1)
```sh
docker build -t flask-app:0.0.1 .
```
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image.png)
Dockerfile에 명시했던 RUN 명령어가 정상적으로 동작했다.
```sh
docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
flask-app    0.0.1     491a65e60ffd   About a minute ago   234MB
...
docker run --rm --name flask-app-container flask-app:0.0.1
 * Serving Flask app 'app' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.17.0.2:5000/ (Press CTRL+C to quit)
```
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%202.png)
docker run 결과로 flask-app이 정상적으로 올라온 것으로 보였으나, local에서는 해당 경로로 접근되지 않았다.
> http://172.17.0.2:5000/
docker container는 호스트(로컬)와 완전히 격리되어있다. 즉, 호스트와 해당 컨테이너 간에 통신을 위해 포트를 매핑할 필요가 있다. docker run을 다시 실행하자.
```sh
docker run --rm -p 8080:5000 --name flask-app-container flask-app:0.0.1
```
==-p== 속성을 통해 호스트(로컬)의 8080 포트와 Docker container의 5000 포트를 매핑했다. 이제 localhost:8080으로 접근하면 다음과 같이 Flask Web App이 동작하고 있는 것을 확인할 수 있다.
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%203.png)<!-- {"width":496} -->
물론, 이런 상태는 ==Docker Desktop==을 통해 보다 편리하게 확인할 수 있다.
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%204.png)

### Dockerfile 작성하기 (v2)
> 앞선 내용도 충분하나, **특정 폴더만 containerizing**하도록 하자.
```dockerfile
## 1. Which base image do you want to use?
FROM python:3.8-slim

## 2. Set the working directory
WORKDIR /app/

## 3. Copy the project files into the working directory.
COPY flask-demo . 

## 4. Install the dependencies
RUN pip install -r requirements.txt

## 5. Document and inform the developer that the application will use PORT 5000 of the container.
EXPOSE 5000

## 6. Define the command to run when the container starts.
CMD ["python", "app.py"]
```
==COPY== 파트에서 `flask-demo` 폴더만 지정함으로써, ==RUN==이나 ==CMD== 명령에서 해당 폴더를 참조할 필요가 없게 되었으며, 해당 Web Application에 필요한 리소스만 효율적으로 적용할 수 있게 되었다. 

### Docker build & run (0.0.2)
```sh
docker build -t flask-app:0.0.2 .
...
docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
flask-app    0.0.2     9340fb649652   4 seconds ago    234MB
flask-app    0.0.1     491a65e60ffd   14 minutes ago   234MB
...
docker run --rm -p 8081:5000 --name flask-app-container flask-app:0.0.2
 * Serving Flask app 'app' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.17.0.2:5000/ (Press CTRL+C to quit)

```
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%205.png)
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%206.png)<!-- {"width":544} -->
마찬가지로 정상 동작하는 것을 확인했다.

### Dockerfile 작성하기 (v3)
> 또 한 번 개선할 사항이 있다. 종속성 파트다.
```dockerfile
## 1. Which base image do you want to use?
FROM python:3.8-slim

## 2. Set the working directory
WORKDIR /app

## 3. Copy the project files into the working directory.
## -> Copy the requirements.txt into the working directory.
COPY flask-demo/requirements.txt . 

## 4. Install the dependencies
RUN pip install -r requirements.txt

## 5. (Added) Copy th remaining project files into the working directory.
COPY flask-demo/ .

## 6. Document and inform the developer that the application will use PORT 5000 of the container.
EXPOSE 5000

## 7. Define the command to run when the container starts.
CMD ["python", "app.py"]
```
첫 번째 ==COPY== 파트에서 폴더 내 모든 파일을 옮기지 않았고, `pip install`을 위한 `requirements.txt` 파일만 옮겼다. 이후 ==RUN== 명령어를 수행하고, 그 이후 나머지 파일을 옮기기 위한 ==COPY==를 다시 정의했다. (여기서 flask-demo/*.py로 작성해도 가능하다) 
**이렇게 Dockerfile을 수정하는 이유**에 대해서 궁금했다. 강의에서 설명하는데 번역이 개판이라 무슨 뜻인지 이해하기 쉽지 않다. 따라서 `chatGPT`에게 변경사항에 대해 설명을 요청했다.
---
> 이렇게 Dockerfile을 수정하는 가장 큰 이유는 **빌드 캐싱 최적화와 빌드 속도 개선**입니다.
#### as-is 버전의 문제점
```dockerfile
COPY flask-demo . 
RUN pip install -r requirements.txt
```
- flask-demo 폴더 전체를 복사 → requirements.txt 변경 여부와 상관없이 매번 모든 소스 변경이 COPY **단계에서 캐시를 무효화**함.
- 결과적으로 코드 한 줄만 바꿔도 pip install이 다시 실행됨 → 빌드 속도가 느려짐.

#### to-be 버전의 개선 포인트
```dockerfile
COPY flask-demo/requirements.txt .
RUN pip install -r requirements.txt

COPY flask-demo/ .
```
(1) requirements.txt만 먼저 복사
* 의존성 파일만 따로 복사하고, 먼저 pip install을 실행.
* **코드가 변경되지 않고 requirements.txt가 그대로라면, 캐시가 유지되어** pip install**을 다시 실행하지 않음.**
* requirements.txt가 바뀌었을 때만 의존성을 다시 설치.
(2) 나머지 소스 코드 마지막에 복사
* 의존성 설치가 끝난 후에만 전체 소스를 복사.
* 코드 변경이 잦아도 pip install 단계는 건너뛸 수 있어 **빌드 속도가 빨라짐.**

#### 부가적인 개선점
* 빌드 캐싱 최적화 외에도 이미지 레이어 구조가 더 깔끔해짐.
* 소스코드 변경(자주 바뀜)과 의존성 설치(덜 바뀜)를 분리 → 변경 가능성이 낮은 레이어를 최대한 위로 올려 재사용성 극대화.
---
### Docker build (0.0.3)
위 설명대로 빌드 캐싱 최적화가 이루어졌다. 아래와 같이 docker build를 진행하면, ==CACHED==로 출력되고 있는 단계들이 보인다.
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%207.png)
물론 ==WORKDIR==과 ==COPY== 단계는 이전에도 ==CACHED==로 진행되었다. 다만 이번 개선으로 `RUN pip ~ ` 부분에서까지 최적화가 이루어진 것이다. 

### Dockerfile 작성하기 (v4)
> 이번에는 환경 쉘 변수를 설정하는 방법이다.

앞선 [요구사항](bear://x-callback-url/open-note?id=89395A71-E1BD-4B78-944A-EA245CAEE4E6&header=%EC%9A%94%EA%B5%AC%EC%82%AC%ED%95%AD%20%ED%99%95%EC%9D%B8)을 다시 확인해보자. Option 2라고 작성된 부분이 있다. 
```python
# Option 2:
#   Set the FLASK_APP shell environment variable and run using the flask command.
#   For Git Bash:
#     export FLASK_APP=app.py
#   For PowerShell:
#     $env:FLASK_APP = "app.py"
#   For Command Prompt:
#     set FLASK_APP=app.py
#   For Mac Terminal:
#     export FLASK_APP=app.py
#   Then run the app with:
#     flask run
```
이는 기존에 Flask App을 실행하는 방법인 Option 1과 다른 방법이다. -> `python app.py`
`flask run `명령어는 ==어떤 파일을 애플리케이션 엔트리 포인트==로 사용할지 알아야 한다. 이를 지정하기 위한 방법으로 `FLASK_APP` 환경변수를 설정하는 것이며, 각 환경에 따른 가이드를 하고 있는 것이다.

아래는 Option 2로 실행하기 위한 Dockerfile이다. **7번**을 다시 작성했다. 
```dockerfile
## 1. Which base image do you want to use?
FROM python:3.8-slim

## 2. Set the working directory
WORKDIR /app

## 3. Copy the project files into the working directory.
## -> Copy the requirements.txt into the working directory.
COPY flask-demo/requirements.txt . 

## 4. Install the dependencies
RUN pip install -r requirements.txt

## 5. (Added) Copy th remaining project files into the working directory.
COPY flask-demo/ .

## 6. Document and inform the developer that the application will use PORT 5000 of the container.
EXPOSE 5000

## 7. Set an environment shell variable 
## and Define the command to run when the container starts.
ENV FLASK_APP=app.py 
CMD ["flask", "run"]

# (as-is)
## 7. Define the command to run when the container starts.
# CMD ["python", "app.py"]
```

물론 이 방법은 개인적으로 먼저 [실습](bear://x-callback-url/open-note?id=4D5740F7-6C9C-4630-BF99-BAC7DD453B0D&header=dockerfile%20%EC%9E%91%EC%84%B1%20%28ENV%20%ED%8F%AC%ED%95%A8%29)했던 내용이다.

### Docker build & run (0.0.4)
```sh
docker build -t flask-app:0.0.4 .
...
docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
flask-app    0.0.4     bfb998eb6157   About a minute ago   234MB
flask-app    0.0.2     9340fb649652   2 hours ago          234MB
flask-app    0.0.3     1c8c1d9e3be7   2 hours ago          234MB
flask-app    0.0.1     491a65e60ffd   2 hours ago          234MB
...
docker run --rm -p 8082:5000 --name flask-app-container flask-app:0.0.4
 * Serving Flask app 'app.py' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
...
```
그러나 http://localhost:8082/으로 접근 해도, `0.0.4` 이미지로 실행한 컨테이너에 접근이 되지 않았는데, 이는 `flask`를 실행하는 방식에 관한 것으로 보인다. 당황했지만 다행히(?) 강사도 똑같이 접근이 안 되어서 설명을 들을 수 있었다. 문제의 부분은 다음과 같다.
in Dockerfile : 
```dockerfile
CMD ["flask", "run"]
CMD ["flask", "run", "--host=0.0.0.0"]
```
---
#### chatGPT 보충 설명 
이는 Flask 개발 서버의 기본 바인딩 주소(host) 설정 때문이라고 한다. 아무리 `-p 8080:5000`으로 포트 매핑을 하더라도 컨테이너 내부의 flask 서버는 **5000번 포트에 바인딩 되어있지 않기 때문**에 접근이 안 된다. `flask run`의 기본 동작은 `127.0.0.1:5000` (localhost)에만 바인딩 된다. 즉, `--host=0.0.0.0`를 추가함으로써, 모든 네트워크 인터페이스에 바인딩하라는 의미이다. 
이는 ==개발용 서버에서만== 사용하는 형태로, ==운영 환경==에서는 `gunicorn`, `uwsgi` 같은 ==WSGI 서버==를 사용하는 게 일반적이라고 한다. 자세히 보면 위 `docker run` 실행 명령 결과로 찍힌 로그에 `WARNING`과 함께 "Use a production WSGI server instead."라는 메시지가 보인다. 
---

아무튼, 위의 2번째 줄과 같이 수정 후 다시 build 했고, 결과적으로 성공했다. 
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%208.png)<!-- {"width":588} -->

---

## SpringBoot Web App
> Java SpringBoot 웹 애플리케이션을 Docker Container로 생성하자.

### 요구사항 확인하기
이번에도 마찬가지로 java 애플리케이션을 구동하기 위한 요구사항을 확인하자. 지금까지 해왔던 것처럼 Developer의 의견을 보자. 아래 내용을 보면 알 수 있겠지만, `maven`에 대한 ==종속성==이 있다. 
-> lesson-starter-projects/06-starter-code/springboot-demo/src/main/java/com/example/ltp/demo/==AppController.java==
```java
package com.example.ltp.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class AppController {
    @GetMapping("/")
    @ResponseBody
    public String home() {
        String response = "<pre>" 
        + "     ____.  _________   _________   \n"
        + "    |    | /  _  \\   \\ /   /  _  \\  \n"
        + "    |    |/  /_\\  \\   Y   /  /_\\  \\ \n"
        + "/\\__|    /    |    \\     /    |    \\\n"
        + "\\________\\____|__  /\\___/\\____|__  /\n"
        + "                 \\/              \\/ \n"
        + "             -- Java --             "
        + "</pre>";
        return response;
    }


}

// Developer: Ahoy, Captain DevOps! Here are a few ways to get this Spring Boot app sailing:
//
// Option 1: 
//   Use Maven to clean, install dependencies, and run the app in one command:
//     mvn clean spring-boot:run
//   This single command takes care of everything, including installing the required dependencies and starting the application.
//
// Option 2:
//   Use Maven to install the dependencies first, then run the app:
//     mvn install
//     mvn spring-boot:run
//   The `mvn install` command installs the necessary dependencies, so the subsequent `mvn spring-boot:run` doesn't need to handle that.
//
// Option 3:
//   Use Maven to package the application and its dependencies into a self-contained JAR file, then run it:
//     mvn package
//     java -jar target/demo-0.0.1-SNAPSHOT.jar
//   The `mvn package` command bundles the application and its dependencies into a single, executable JAR file, which you can then run using the `java -jar` command.
//
// I trust you to choose the best route for your journey. May your application have a smooth voyage!
```
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%209.png)

### Dockerfile 작성하기
#### Docker hub
먼저 Docker hub에서 maven 이미지를 확인하자.
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%2010.png)

#### Dockerfile
```dockerfile
## 1. Which base image do you want to use?
FROM maven:3.8-openjdk-17-slim

## 2. Set the working directory.
WORKDIR /app/

## 3. Copy the application's project files into the working directory.
COPY /springboot-demo . 

## 4. Document and inform the developer that the application will use the container port: 8080.
EXPOSE 8080

## 5. Define the command to run when the container starts.
CMD ["maven", "clean", "spring-boot:run"]

```
`Maven` 이미지와 태그, 명령어까지 그럴듯 하게 작성된 것처럼 보인다. 하지만 이렇게 이미지를 만들면 `Maven`에 대한 종속성은 지켰으나, 매번 컨테이너를 실행할 때마다 이 `java application`의 `dependencies`를 내려받게 될 것이다. 그러니까 이 이미지가 가지게 될 종속성(`maven`)은 지켰지만, `java application`에서 가질 `maven`의 종속성은 컨테이너 실행 이후로 미뤄져 있다. 일단은 이 상태에서 진행해 보자.

### Docker build & run
```sh
docker build -t spring-boot-demo:0.0.1 .
...
docker images
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
spring-boot-demo   0.0.1     6efe526f45f8   37 seconds ago   661MB
...
docker run --rm -p 8080:8080 --name spring-boot-demo-container spring-boot-demo:0.0.1
/usr/local/bin/mvn-entrypoint.sh: 50: exec: maven: not found
```
docker run 결과 실패했다. 메시지를 봐서는 maven이 설치되지 않은 것으로 보인다. 다시 요구사항을 확인하니 명령어가 "maven"이 아니라, "mvn"이었다. 😅
```dockerfile
## 5. Define the command to run when the container starts.
CMD ["mvn", "clean", "spring-boot:run"]
```
docker build를 다시 진행하고, docker run을 재시도했다. (위와 동일)

그 결과 pom.xml에 선언된 ==수많은 dependencies의 다운로드가 진행==되었다. 이 작업은 수 초 이상 진행되었고, 어쨌든 SpringBoot Application은 성공적으로 올라왔다. 하지만 매번 이렇게 필요한 것들을 내려받는 방식은 옳지 않다. 이를 개선하자.
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%2011.png)
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%2012.png)<!-- {"width":488} -->

### Dockerfile 작성하기 (v2)
앞선 [요구사항](bear://x-callback-url/open-note?id=89395A71-E1BD-4B78-944A-EA245CAEE4E6&header=%EC%9A%94%EA%B5%AC%EC%82%AC%ED%95%AD%20%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0)을 보면, option 2가 명시되어있다. 이를 이용해 Dockerfile을 개선하고, ==이미지 빌드 과정에 필요한 종속성 대상들을 모두 설치==하자.
```dockerfile
## 1. Which base image do you want to use?
FROM maven:3.8-openjdk-17-slim

## 2. Set the working directory.
WORKDIR /app/

## 3. Copy the application's project files into the working directory.
COPY /springboot-demo . 

## (added) 4. Install the dependencies listed inside of pom.xml
RUN mvn install

## 5. Document and inform the developer that the application will use the container port: 8080.
EXPOSE 8080

## 6. Define the command to run when the container starts.
CMD ["mvn", "spring-boot:run"]

```
4번에 ==RUN== 명령어에 `mvn install`을 추가했고, ==CMD== 명령어에서 `"clean"`을 제거했다.

### Docker build & run (v2)
```sh
docker build -t spring-boot-demo:0.0.2 .
...
```
docker build 명령을 실행하니, 이미지 생성 과정에서 수 많은 dependencies를 내려받는 것을 확인할 수 있었다. 아래 캡쳐가 그 상황이다. 이전에 우리가 docker run 명령을 통해 컨테이너가 시작될 때 진행했던 과정이 이제는 이미지 빌드(생성) 과정으로 옮겨진 것을 이해할 수 있다. 해당 과정(`RUN mvn install`)은 총 `12.9초`가 소요되었다. 이제 이미지 생성 단계에서 종속성을 모두 갖췄으니, **컨테이너 실행 속도는 더욱 빨라질 것**이다.
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%2013.png)
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%2014.png)

```sh
docker images
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
spring-boot-demo   0.0.2     4c87f6c68326   3 minutes ago    848MB
spring-boot-demo   0.0.1     a6825582cc1d   19 minutes ago   661MB
...
docker run --rm -p 8080:8080 --name spring-boot-demo-container spring-boot-demo:0.0.2
...
```
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%2015.png)
이전과 달리 ==download 과정없이 바로 SpringBoot Application이 실행된 것을 확인==할 수 있다.

### Dockerfile 작성하기 (v3)
`v2` 버전으로도 충분한 개선이 이루어졌지만, 조금 더 개선할 방법이 있다. `Maven`을 이용해 application을 미리 ==패키징==하는 방법이다. 이를 통해 `dependencies`는 모두 `.JAR` 파일에 종속된다. 그리고 해당 `.JAR` 파일을 실행하는 것으로 변경하자.

Dockerfile을 수정하기 위해 [요구사항](bear://x-callback-url/open-note?id=89395A71-E1BD-4B78-944A-EA245CAEE4E6&header=%EC%9A%94%EA%B5%AC%EC%82%AC%ED%95%AD%20%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0)에서 option 3을 확인하자. 설명대로 ==RUN== 명령을 `mvn install`에서 `mvn package`로 변경했고, ==CMD== 명령을 `java -jar target/demo-0.0.1-SNAPSHOT.jar`로 바꿨다. 
```dockerfile
## 1. Which base image do you want to use?
FROM maven:3.8-openjdk-17-slim

## 2. Set the working directory.
WORKDIR /app/

## 3. Copy the application's project files into the working directory.
COPY /springboot-demo . 

## (added) 4. Install the dependencies listed inside of pom.xml
RUN mvn package

## 5. Document and inform the developer that the application will use the container port: 8080.
EXPOSE 8080

## 6. Define the command to run when the container starts.
CMD ["java", "-jar", "target/demo-0.0.1-SNAPSHOT.jar"]

```

### Docker build & run (v3)
```sh
docker build -t spring-boot-demo:0.0.3 . 
...
docker images
REPOSITORY         TAG       IMAGE ID       CREATED              SIZE
spring-boot-demo   0.0.3     5c4bd65e64c7   About a minute ago   810MB
spring-boot-demo   0.0.2     4c87f6c68326   18 minutes ago       848MB
spring-boot-demo   0.0.1     a6825582cc1d   34 minutes ago       661MB
...
```
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%2016.png)
`docker build` 명령 실행 결과, `v2`에서와 마찬가지로 ==RUN== 명령어가 실행되며 `dependencies`를 내려받아 패키징하는 과정이 진행되었다. 

```sh
docker run --rm -p 8080:8080 --name spring-boot-demo-container spring-boot-demo:0.0.3
...
```
![](assets/Docker%20Basic%2004%20-%20Containerizing%20Web%20Apps/image%2017.png)
이후 `docker run` 명령 실행 결과 SpringBoot Application이 바로 실행된 것을 볼 수 있다. 앞서 이미지 빌드 단계에서 maven package(jar 파일 생성) 과정까지 이미 완료했기 때문이다. 이렇게 `v3`에서는 빌드와 실행이 분리되어 이제 이 컨테이너는 독립 실행 가능한 JAR 파일을 운영하고 있다.


## 정리
> Flask와 SpringBoot 애플리케이션을 Dockerize하는 방법을 익혔다.
- Dockerize하기 위해 해당 애플리케이션에 대한 요구사항 및 설명(주석, 개발자 지침 등)을 확인한다. 
  - 과연 실무에서 개발자 지침이 항상 존재할까? 그렇지 않을 가능성이 더 높다.
- 해당 애플리케이션의 기반이 되는 이미지를 Docker Hub에서 확인한다.
- 이후 Dockerfile을 작성하는데, 앞선 개발자 지침에서 확인한 내용을 반영한다.
  - 이미지
  - 복사할 리소스
  - dependencies 설치 명령어
  - 포트
  - 최종 CMD 명령어 등...
- Dockerfile을 잘 구성하는 것만으로도 보다 효율적인 작업이 가능하다.


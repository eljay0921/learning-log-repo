# Udemy Docker Basic 01 - Get started
#dev/skill/container/docker 

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요

## 개요
### Docker를 사용하는 근본적인 이유?
* *It works on my machine!* → ==Bare metal== 문제
* 응용 프로그램이 개발자의 환경에서는 정상적으로 동작하지만, 다른 시스템에서는 제대로 작동하지 않는 경우가 많다.
* 종속성 충돌, 다양한 소프트웨어·라이브러리 버전 차이, 운영체제 환경 문제 등으로 인해 응용 프로그램이 **교란**될 수 있다.
* 이러한 문제를 해결하기 위해 응용 프로그램을 분리하고 **환경 차이로부터 보호**할 필요가 있었다.
* 2000년대 초 등장한 첫 번째 대안은 ==virtual machine==이었지만, VM은 하이퍼바이저 위에 **게스트 OS 전체를 구동**해야 하므로 리소스를 많이 사용하고 상대적으로 느리며, 관리 비용도 높았다.
* 반면 ==Docker==는 호스트 OS의 커널을 공유하고, 응용 프로그램에 필요한 라이브러리와 의존성만 포함하므로 훨씬 가볍고 빠르다.

> Docker는 현재 컨테이너 기술의 사실상 표준이다.

## 시작하기
### in local
- local 환경에 있는 어떤 애플리케이션(*.py)를 실행하고 싶다고 가정하면, 
  - `/Users/.../docker-course-remastered-main/lesson-starter-projects/01-starter-code/python-app.py`
  - 위 파일을 실행시킬 것이고, 다음과 같은 결과를 얻을 수 있다.
    ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image.png)<!-- {"width":660} -->

### in Docker
- 이를 ==Docker==에서 실행하려면, 
  - Docker에 **마운트(-v)**해야하고, Docker를 **실행(Docker run)**하면 된다.
  - 마운트 : **작업(working) 디렉토리** -> /app/python-app.py
- 그렇다면 Python 이미지는 어디서 가져올 수 있을까?
  - -> ==Docker Hub==는 Docker에서 제공하는 이미지 저장소다. like github
  - 이미지는? **스냅샷**을 생각하라.
  - -> 그 환경을 스냅샷으로 패키징하고, 그것을 Docker Hub에 업로드 한다.
  ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%202.png)<!-- {"width":488} -->![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%203.png)<!-- {"width":501} -->
- Docker Hub에서 우리가 사용할 Python 3.8 slim 버전을 tag에서 검색해 찾아보자. 다음과 같이 확인할 수 있다.
  -> `docker pull python:3.8.20`![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%204.png)<!-- {"width":956} -->

## Docker로 시작하기
> 자 그럼 docker로 어떻게 시작하면 될까?

1. 먼저 다음과 같이 명령어를 작성하자.
   - `docker run -v`
   - docker run -> docker를 실행하며,
   - ==-v== -> 볼륨을 마운트 한다
2. 이어, 어떤 디렉토리를 마운트할 것인지 작성한다.
   - docker run -v "/Users/.../sources-lectures/docker-course-remastered-main/lesson-starter-projects/01-starter-code:/app/"
   - “{로컬경로}:{마운트경로}” -> ==콜론(:)==으로 분리한다.
3. 어떤 Python 이미지를 가져올 것인지 명시한다.
   - docker run -v "/Users/.../sources-lectures/docker-course-remastered-main/lesson-starter-projects/01-starter-code:/app/" **python:3.8.19-slim**
   - 경량화 버전인 **slim** 버전을 작성해봤다. 
4. 이제 마운트 이후 수행할 명령어를 작성한다.
   - docker run -v "/Users/.../sources-lectures/docker-course-remastered-main/lesson-starter-projects/01-starter-code:/app/" python:3.8.19-slim python **/app/python-app.py** 
5. 이제 명령어를 실행하자!
   - 물론, 그 전에 Docker Desktop이 실행되어있는지 확인하자. (colima 등)![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%205.png)<!-- {"width":684} -->
   - 실행 결과는 다음과 같다.
     - `docker run -v "/Users/.../sources-lectures/docker-course-remastered-main/lesson-starter-projects/01-starter-code:/app/" python:3.8.19-slim python /app/python-app.py` 
       ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%206.png)<!-- {"width":716} -->
     - bear에서 작성한 따옴표(“)가 터미널에서 다른 이슈가 있어 중간에 한 번 실패함 (편집 -> 대체 -> **스마트 인용** 체크 해제)
6. 이제 실행된 docker process를 확인하자
   - 위 명령어를 몇 차례 더 실행하고, docker ps를 확인했다.
   - `docker ps -a`
     ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%207.png)<!-- {"width":883} -->
   - 각각의 이름은 자동으로 생성되었다.
   - 물론 우리는 ==Docker Desktop==을 사용하고 있으므로 GUI 화면으로도 볼 수 있다.
     ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%208.png)<!-- {"width":890} -->

## Docker 제어하기
1. 이제 불필요한 컨테이너들을 ==삭제==하자.
- docker rm {컨테이너 이름 or 컨테이너 ID}
  ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%209.png)<!-- {"width":404} -->
2. 이번에는 ==이름==을 지정해서 생성하자
   - docker run -v "/Users/.../sources-lectures/docker-course-remastered-main/lesson-starter-projects/01-starter-code:/app/" ==--name python-app-container== python:3.8.19-slim python /app/python-app.py
   - --name 옵션을 추가하고 이름을 작성하면 된다.
   - 그리고 다시 동일한 이름으로 컨테이너를 생성하려고 하면 오류와 함께 충돌되었다는 메시지를 준다.
     ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%2010.png)<!-- {"width":1037} -->
   - 즉, 우리는 이미 컨테이너의 이름을 알고 있으므로, 해당 컨테이너를 삭제하거나, 다른 이름으로 생성하면 된다. 
   - -> **컨테이너의 이름을 작성해야 하는 이유**(책임)라고 할 수 있다.
3. 그런데, Docker를 생성하고 수행이 끝나면 **바로 삭제하는 방법**도 있다.
   - ==--rm== 옵션을 추가하자 -> `docker run --rm -v …` 
     ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%2011.png)
   - docker가 실행되고 python 애플리케이션이 정상 동작했지만 이후 남아있는 컨테이너는 없다.
   - 덕분에 반드시 컨테이너 이름을 지정할 필요는 없겠지만, 대부분의 경우 **컨테이너 이름을 지정하는 것을 권장**한다. 
4. 이번엔 Docker Image를 확인하자
   - `docker images` 명령어를 실행하면 설치된 이미지 목록이 나타난다.
     ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%2012.png)<!-- {"width":748} -->
   - 삭제는 `docker image rm {repository:tag}`로 가능하고, **Image ID**로도 가능하다.

## Docker로 "다시" 시작하기
> 이번에는 python이 아니라, java 애플리케이션이다.
> 주목할 점은 python 애플리케이션을 실행하기 위해 진행했던 절차와 동일하다는 것.

1. 마찬가지로 어떤 파일(애플리케이션)을 마운트 할 것인지 디렉토리를 확인하자.
   - /Users/…/sources-lectures/docker-course-remastered-main/lesson-starter-projects/02-starter-code/**JavaApp.jar**
     ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%2013.png)<!-- {"width":643} -->
     ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%2014.png)
     (openjdk:11-jre-slim)
2. 명령어를 작성하자
   - `docker run --rm -v "/Users/.../sources-lectures/docker-course-remastered-main/lesson-starter-projects/02-starter-code/:/app/src"  --name java-app-container openjdk:11-jre-slim java -cp /app/src/JavaApp.jar JavaApp`
     - --rm -> 실행 후 바로 삭제하기 위함
     - -v -> 마운트, 이전과 동일하나 java의 경우 /app/src/ 하위에 위치시켰다
     - --name -> 컨테이너 이름 지정
     - openjdk:11-jre-slim -> jre 11 slim 이미지 지정
     - java 실행 명령어 -> 강의에서 빠뜨린 게 있는데 `java -cp /app/src/JavaApp.jar JavaApp`이라고 명령해야 동작한다. 
       ![](Udemy%20Docker%20Basic%2001%20-%20Get%20started/image%2015.png)
       (앞서 실행 테스트 과정에서 이미 java 이미지를 내려받아서 바로 동작하는 결과만 보인다)

---
## 정리
- Docker 등장 배경, Docker를 사용하는 이유/목적
- Docker 실행 구성 -> 작업 디렉토리, 도커 이미지(Docker Hub), 시작 명령어
- 명령어
  - `Docker run -v "{마운트 원본 위치}:{도커 내 대상 위치}" {이미지}:{태그} {명령어 입력}`
    - --rm 실행 후 바로 삭제
    - --name {컨테이너 이름}
  - `Docker ps -a` 
  - `Docker rm {컨테이너 이름|컨테이너 ID}`
  - `Docker images`
  - `Docker images rm {이미지}:{태그}`

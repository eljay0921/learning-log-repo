#  Docker Basic 08 - Docker Compose

#dev/skill/container/docker

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/45162163#overview)

## 개요
> 지금까지 우리는 다양한 Docker Image를 빌드하고, 이를 컨테이너로 실행했다.
> 그러나, 여러 개의 application들을 각각 배포하는 경험은 좋지 않았다. 이를 ==Docker Compose==로 통합하자.

## Docker Compose 실습
(먼저 `08-starter-code`에 포함된 2개의 애플리케이션을 `docker hub`으로 **push**하자, 그 방법은 [이전 챕터](bear://x-callback-url/open-note?id=6D956EEC-1743-410F-A1FE-6C934A539206)에서 배웠다)

각각 애라 2개의 애플리케이션이다.
- lesson-starter-projects/08-starter-code/grade-submission-api
- lesson-starter-projects/08-starter-code/grade-submission-portal
```sh
docker build -t lj7812/grade-submission-api .
[+] Building 17.0s (11/11) FINISHED
docker build -t lj7812/grade-submission-portal .
[+] Building 7.6s (11/11) FINISHED
...
docker images
REPOSITORY                       TAG       IMAGE ID       CREATED         SIZE
lj7812/grade-submission-portal   latest    43f3cd82955f   7 seconds ago   239MB
lj7812/grade-submission-api      latest    9ac2d2c12c4f   2 minutes ago   1.3GB
...
docker push lj7812/grade-submission-api:latest
...
docker push lj7812/grade-submission-portal:latest
The push refers to repository [docker.io/lj7812/grade-submission-portal]
3ac2d9063b04: Pushed
2da73249dbde: Pushed
d847ad1879f2: Pushed
73530e7f56ac: Pushed
5f595df0fd95: Pushed
f04bf7efd2bf: Pushed
56d45b8e1404: Pushed
0cd45dda32d5: Pushed
14c9d9d19932: Pushed
latest: digest: sha256:43f3cd82955f1b02ea40bd42c07d5bf7a0cbf90723f6dfef1d3ebcfadd078a19 size: 856
...
```

### docker compose로 컨테이너 정의하기
> 자, 이제 `docker-compose.yml` 파일을 작성하자, 참고로 이 yaml 파일은 들여쓰기(indentation)를 잘 지켜야한다.
```yml
version: '3'
services: 
  flask-app: 
    image: lj7812/grade-submission-portal:latest
    container_name: flask-app
    ports: 
      - "5001:5001"
    environment:
      - GRADE_SERVICE_HOST=node-server
    depends_on:
      - node-server
  node-server:
    image: lj7812/grade-submission-api:latest
    container_name: node-server
    ports:
      - "3000:3000"
    # environment:
    #   -
    # depends_on:
    #   - 
```
`docker-compose.yml` 파일의 기본적인 구성이다. 개략적인 설명을 알아보자.
- `version: '3'`
  - **Docker Compose** 파일 형식을 정의하는 버전이며, **Docker Engine 1.13** 이상에서 사용 가능하다. (최신 기능을 지원)
- `services: `
  - 여러 **Docker Container**를 정의하는 블록이다. 위 문서에서는 `flask-app`과 `node-server` 2개의 서비스를 명시했다.
- `image: `
  - dockerfile에 작성하는 `FROM`과 유사한 정의로, 이미지가 없으면 **Docker Hub**에서 내려받기도 한다.
- `container_name:`
  - 말 그대로 컨테이너 이름을 명시한다. `docker run --name`의 역할이라고 보면 되겠다.
- `ports:`
  - 해당 컨테이너가 사용할 포트를 매핑한다. `docker run -p`의 역할로 볼 수 있으며, 배열로 작성할 수 있다. ('-' 행을 추가)
- `environment:`
  - 해당 컨테이너의 환경변수를 정의한다. `docker run -e` 또는 `dockerfile ENV`와 같은 역할이다. 마찬가지로 배열로 작성할 수 있다.
- `depends_on:`
  - 각 application(container) 간의 **의존 관계를 명시**한다. (**실행 순서**라고 보면 되겠다)
  - 즉, `flask-app`은 `node-server`에 의존하므로, `node-server`로 명시된 컨테이너를 먼저 실행한다.
  - (주의) 단, `node-server`를 실행한 직후 `flask-app` 컨테이너를 시작하며, `node-server`의 **healthcheck**를 보장하진 않기 때문에 `node-server`가 완전히 올라온 뒤 `flask-app`이 실행되어야 하는 이유가 있다면, 별도의 healthcheck를 구성할 필요가 있다.

### docker compose로 컨테이너 실행하기
> 드디어 컨테이너를 실행할 시간이다. docker compose up을 통해 실습하자.

```sh
docker compose up
WARN[0000] /Users/trongs/dev/sources-lectures/docker-course-remastered-main/lesson-starter-projects/08-starter-code/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 3/3
 ✔ Network 08-starter-code_default  Created                                                                                                                                                                  0.0s
 ✔ Container node-server            Created                                                                                                                                                                  0.0s
 ✔ Container flask-app              Created                                                                                                                                                                  0.0s
Attaching to flask-app, node-server
node-server  | Grade service is running on port 3000
flask-app    |  * Serving Flask app 'app' (lazy loading)
flask-app    |  * Environment: production
flask-app    |    WARNING: This is a development server. Do not use it in a production deployment.
flask-app    |    Use a production WSGI server instead.
flask-app    |  * Debug mode: on
flask-app    |  * Running on all addresses.
flask-app    |    WARNING: This is a development server. Do not use it in a production deployment.
flask-app    |  * Running on http://172.18.0.3:5001/ (Press CTRL+C to quit)
flask-app    |  * Restarting with stat
flask-app    |  * Debugger is active!
flask-app    |  * Debugger PIN: 926-138-631
...
```
실행된 로그를 보면, `08-starter-code_default`이라는 network를 자동으로 생성했으며, node-server와 flask-app 컨테이너가 차례로 생성된 것을 볼 수 있다. 그리고 현재 해당 컨테이너들의 상태와 로그를 다음과 같이 docker desktop에서도 볼 수 있다.
![](assets/Docker%20Basic%2008%20-%20Docker%20Compose/image.png)
앞서 정의한 localhost:5001로 접근하면 정상적으로 서버가 올라온 것도 볼 수 있다. 
![](assets/Docker%20Basic%2008%20-%20Docker%20Compose/image%202.png)<!-- {"width":442} --> ![](assets/Docker%20Basic%2008%20-%20Docker%20Compose/image%203.png)<!-- {"width":506} -->
docker desktop을 통해 container들의 로그도 모니터링 할 수 있다.
![](assets/Docker%20Basic%2008%20-%20Docker%20Compose/image%204.png)

### docker compose로 컨테이너 중지하기
```sh
docker compose down
[+] Running 3/3
 ✔ Container flask-app              Removed                                                                                                                                                                  
 ✔ Container node-server            Removed                                                                                                                                                                 
 ✔ Network 08-starter-code_default  Removed
...
```
`docker compose down` 명령을 통해 모든 컨테이너를 중지 및 삭제하고, network까지 정리할 수 있다. 
![](assets/Docker%20Basic%2008%20-%20Docker%20Compose/image%205.png)

## 정리
> docker compose를 이용해 여러 개의 서비스를 실행하는 방법을 학습했다.
- docker-compose.yml 구성 요소와 작성 방법
- 실행 : docker compose up
- 중지 : docker compose down

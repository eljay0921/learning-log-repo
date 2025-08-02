#  Docker Basic 09 - Docker Compose: Microservices

#dev/skill/container

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/43367524#overview)<!-- {"preview":"true"} -->

## 개요
> 이제 docker compose를 이용해, [[Docker Basic 06 - Microservices]]에서 학습했던 e-commerce 서비스를 Containerize하자

## 사전 준비
강의를 시작하는데 강사가 이미 해당 서비스들을 모두 푸시했다고 이야기한다. (...) 나도 학습 준비를 해보자 😅

먼저 깔끔한 시작을 위해 `prune`을 명령하고, 
```sh
docker system prune -a
```

### Push to Docker Hub
각각의 application을 빌드 및 태깅, 그리고 push 한다. 아래는 대상 applications이며, 각 명령은 각각의 폴더 하위에서 진행한다.
![](assets/Docker%20Basic%2009%20-%20Docker%20Compose%20Microservices/image.png)<!-- {"width":963} -->
```sh
docker build -t lj7812/ecommerce-ui:0.0.1 . 
[+] Building 53.6s (17/17) FINISHED
...
docker build -t lj7812/contact-support-team:0.0.1 .
[+] Building 7.9s (11/11) FINISHED
...
docker build -t lj7812/order-management:0.0.1 .
[+] Building 31.5s (10/10) FINISHED
...
docker build -t lj7812/product-catalog:0.0.1 .
[+] Building 2.7s (10/10) FINISHED
...
docker build -t lj7812/product-inventory:0.0.1 .
[+] Building 1.1s (10/10) FINISHED
...
docker build -t lj7812/profile-management:0.0.1 .
[+] Building 2.8s (10/10) FINISHED
...
docker build -t lj7812/shipping-and-handling:0.0.1 .
[+] Building 13.8s (9/9) FINISHED
...
docker images
REPOSITORY                     TAG       IMAGE ID       CREATED              SIZE
lj7812/shipping-and-handling   0.0.1     d40235d661fa   52 seconds ago       1.26GB
lj7812/profile-management      0.0.1     14b60f80b1d2   About a minute ago   1.3GB
lj7812/product-inventory       0.0.1     f2c582bfb1e5   2 minutes ago        234MB
lj7812/order-management        0.0.1     4daf0a63a305   3 minutes ago        822MB
lj7812/product-catalog         0.0.1     6b91741aa951   3 minutes ago        1.3GB
lj7812/contact-support-team    0.0.1     3c5b6d9271b7   4 minutes ago        234MB
lj7812/ecommerce-ui            0.0.1     9dc99c7bf585   5 minutes ago        2.12GB
...
```
태그가 `1.0.0`이 아니라, `0.0.1`로 작성되었다는 것을 중간에 알았다. 뭐 상관없으니... 그냥 `0.0.1`로 진행한다. ~~귀찮다~~

![](assets/Docker%20Basic%2009%20-%20Docker%20Compose%20Microservices/image%202.png)
그리고 각각의 `image`를 `docker push`했다. 내용은 생략하겠다. 아래는 업로드 완료한 모습.
![](assets/Docker%20Basic%2009%20-%20Docker%20Compose%20Microservices/image%203.png)

### Clear
그리고 다시 한번 시스템을 정리하자. 우리는 ==docker-compose==를 이용해 Microservice e-commerce를 구축할 것이다. 
```sh
docker system prune -a
```

## Docker Compose!
> 이제 docker-compose.yml 파일을 작성하자.

우리는 e-commerce 서비스의 몇 가지 요구사항을 알고 있다.
- `ecommerce-ui` application은 ==다른 6개의 application에 의존==한다. -> 즉, 다른 application이 먼저 실행될 필요가 있다.
  - 물론 [[Docker Basic 06 - Microservices]] 실습 과정에서는 별로 상관 없었다...
- 각각의 application과 통신하기 위해, `ecommerce-ui`와 `order-management` application은 각각 환경변수를 가지고 있었다.
  - 각 application의 호스트명이 필요하므로 작성할 때 참고하자.
- 각각의 application이 어떤 포트를 사용했는지 작성할 때 참고하자.

자, 이제 위 요구사항을 잘 기억하고 `docker-compose.yml`을 작성하자.
> workbook-starter-projects/04-starter-projects/docker-compose.yml
```yml
version: '3'
services: 
  ecommerce-ui:
    image: lj7812/ecommerce-ui:0.0.1
    container_name: ecommerce-ui-container
    ports:
      - "4000:4000"
    environment:
      - REACT_APP_PROFILE_API_HOST=http://profile-management
      - REACT_APP_PRODUCT_API_HOST=http://product-catalog
      - REACT_APP_INVENTORY_API_HOST=http://product-inventory
      - REACT_APP_ORDER_API_HOST=http://order-management
      - REACT_APP_SHIPPING_API_HOST=http://shipping-and-handling
      - REACT_APP_CONTACT_API_HOST=http://contact-support-team
    depends_on:
      - contact-support-team
      - order-management
      - product-catalog
      - product-inventory
      - profile-management
      - shipping-and-handling

  contact-support-team:
    image: lj7812/contact-support-team:0.0.1

  order-management:
    image: lj7812/order-management:0.0.1
    environment: 
      - REACT_APP_PRODUCT_API_HOST=http://product-catalog
      - REACT_APP_INVENTORY_API_HOST=http://product-inventory
    depends_on:
      - product-catalog
      - product-inventory

  product-catalog:
    image: lj7812/product-catalog:0.0.1

  product-inventory:
    image: lj7812/product-inventory:0.0.1

  profile-management:
    image: lj7812/profile-management:0.0.1

  shipping-and-handling:
    image: lj7812/shipping-and-handling:0.0.1
```
결과를 보면 알겠지만, 앞에서 배운 것과 조금 다르게 작성했다. 2가지를 생략했다. (강의에서는 모두 작성하고 있다)

### (1) container_name 생략
기본인 `ecommerce-ui`를 제외한 나머지 서비스는 `container_name`을 **생략**했다. 그 이유는 **Scale-out, 확장성**에 있다. (ecommerce-ui도 생략해도 된다)
1. Docker Compose는 자동으로 서비스명 자체를 DNS 이름으로 사용한다.
2. container_name을 생략하면 자동으로 `{프로젝트명}_{서비스명}_{숫자}`의 형태로 컨테이너 이름을 생성한다.
-> 즉, 통신은 서비스 이름을 기준으로 하기에 문제가 없고, 컨테이너 이름의 중복을 피할 수 있다.

### (2) ports 생략
단순히 로컬 접근이 불필요할 것 같아서 생략했다. 우리는 `ecommerce-ui`로만 접속해 테스트하면 된다. 

### (3) 이름의 통일
**`container_name`을 생략**하면서 자동 이름을 적용하기 때문에 `environment`에서도, `depends_on`에서도 각각의 서비스명에서도 모두 이름을 간편하게 통일했다. `docker-compose.yml` 파일을 보다 간결하게 작성할 수 있게 되었으며, 오탈자 가능성이 낮아졌다.
-> 모든 것들 서비스 이름을 이용해 작성하면 된다.

### docker compose up
이제 해당 docker-compose.yml 파일이 위치하는 경로로 이동해 명령을 실행하자.
```sh
docker compose up
[+] Running 48/56
 ✔ ecommerce-ui Pulled                                                                                                                                                                                     
 ✔ product-catalog Pulled                                                                                                                                                                                  
 ✔ profile-management Pulled                                                                                                                                                                               
 ✔ contact-support-team Pulled                                                                                                                                                                             
 ✔ shipping-and-handling Pulled                                                                                                                                                                            
 ✔ product-inventory Pulled                                                                                                                                                                               
 ⠇ order-management [⣶⣿⣿⣿⣿⣿⣿⣿⣿⣿] 291.9MB / 306.6MB Pulling
```
![](assets/Docker%20Basic%2009%20-%20Docker%20Compose%20Microservices/image%204.png)
진행 중, order-management만 한참 동안 pulling 상태였다. 뭔가 네트워크 오류인듯 하여 취소 후 다시 진행하니 정상적으로 컨테이너들이 올라왔다.

보다시피 각각의 서비스가 모두 올라왔다.
-> log
```sh
docker compose up
[+] Running 8/8
 ✔ Network 04-starter-projects_default                    Created                                                                                                                                           
 ✔ Container 04-starter-projects-contact-support-team-1   Created                                                                                                                                           
 ✔ Container 04-starter-projects-profile-management-1     Created                                                                                                                                           
 ✔ Container 04-starter-projects-product-catalog-1        Created                                                                                                                                           
 ✔ Container 04-starter-projects-product-inventory-1      Created                                                                                                                                           
 ✔ Container 04-starter-projects-shipping-and-handling-1  Created                                                                                                                                           
 ✔ Container 04-starter-projects-order-management-1       Created                                                                                                                                           
 ✔ Container ecommerce-ui-container                       Created                                                                                                                                           
Attaching to contact-support-team-1, order-management-1, product-catalog-1, product-inventory-1, profile-management-1, shipping-and-handling-1, ecommerce-ui-container
product-catalog-1        | Product Catalog microservice is running on port 3001
profile-management-1     | Authentication API is running on port 3003
contact-support-team-1   |  * Serving Flask app 'server' (lazy loading)
contact-support-team-1   |  * Environment: production
product-inventory-1      |  * Running on all addresses.
contact-support-team-1   |    WARNING: This is a development server. Do not use it in a production deployment.
product-inventory-1      |    WARNING: This is a development server. Do not use it in a production deployment.
contact-support-team-1   |    Use a production WSGI server instead.
contact-support-team-1   |  * Debug mode: off
product-inventory-1      |  * Running on http://172.18.0.6:3002/ (Press CTRL+C to quit)
product-inventory-1      |  * Serving Flask app 'inventory_api' (lazy loading)
product-inventory-1      |  * Environment: production
product-inventory-1      |    WARNING: This is a development server. Do not use it in a production deployment.
product-inventory-1      |    Use a production WSGI server instead.
contact-support-team-1   |  * Running on all addresses.
product-inventory-1      |  * Debug mode: off
contact-support-team-1   |    WARNING: This is a development server. Do not use it in a production deployment.
contact-support-team-1   |  * Running on http://172.18.0.4:8000/ (Press CTRL+C to quit)
ecommerce-ui-container   | Server is running on port 4000
order-management-1       | [INFO] Scanning for projects...
order-management-1       | [INFO]
order-management-1       | [INFO] ----------------------< com.ltp:order-management >----------------------
order-management-1       | [INFO] Building order-management 0.0.1-SNAPSHOT
order-management-1       | [INFO] --------------------------------[ jar ]---------------------------------
order-management-1       | [INFO]
order-management-1       | [INFO] >>> spring-boot-maven-plugin:3.2.3:run (default-cli) > test-compile @ order-management >>>
order-management-1       | [INFO]
order-management-1       | [INFO] --- maven-resources-plugin:3.3.1:resources (default-resources) @ order-management ---
order-management-1       | [INFO] Copying 1 resource from src/main/resources to target/classes
order-management-1       | [INFO] Copying 0 resource from src/main/resources to target/classes
...
```

-> docker desktop
![](assets/Docker%20Basic%2009%20-%20Docker%20Compose%20Microservices/image%205.png)
![](assets/Docker%20Basic%2009%20-%20Docker%20Compose%20Microservices/image%206.png)

### 오류 발생
docker desktop으로 로그를 보면서 `localhost:4000`에 접속해 테스트를 하고 있었다. 모두 정상적으로 동작했는데, 갑자기 오류 메시지가 길게 찍혔다. 특정 상품을 **ADD TO CART**하는 버튼에서 발생했다. 
![](assets/Docker%20Basic%2009%20-%20Docker%20Compose%20Microservices/image%207.png)<!-- {"width":567} -->

아래 로그를 보면 `inventory-api-container`라는 host를 찾을 수 없는 것처럼 보인다. 그런데 이상하다 난 `docker-compose.yml` 파일에서 컨테이너 이름을 이렇게 지정하지 않았는데? 어딘가 캐싱이 되어있는 것인가...
```log
http://inventory-api-container:3002/api/inventory/7⁠
2025-07-31T15:04:59.144Z ERROR 62 --- [order-management] [0.0-9090-exec-8] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed: org.springframework.web.client.ResourceAccessException: I/O error on GET request for "http://inventory-api-container:3002/api/inventory/7": inventory-api-container] with root cause
                     
java.net.UnknownHostException: inventory-api-container
	at java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:567) ~[na:na]
	at java.base/java.net.Socket.connect(Socket.java:630) ~[na:na]
	at java.base/java.net.Socket.connect(Socket.java:581) ~[na:na]
...
```

### 오류 해결하기 : TroubleShooting
이 문제를 해결하기 위해 컨테이너를 다 내리고 상황을 초기화 해보자
```sh
docker compose down -v
[+] Running 8/8
 ✔ Container ecommerce-ui-container                       Removed                                                                                                                                            
 ✔ Container 04-starter-projects-order-management-1       Removed                                                                                                                                             
 ✔ Container 04-starter-projects-profile-management-1     Removed                                                                                                                                            
 ✔ Container 04-starter-projects-contact-support-team-1   Removed                                                                                                                                            
 ✔ Container 04-starter-projects-shipping-and-handling-1  Removed                                                                                                                                             
 ✔ Container 04-starter-projects-product-catalog-1        Removed                                                                                                                                            
 ✔ Container 04-starter-projects-product-inventory-1      Removed                                                                                                                                            
 ✔ Network 04-starter-projects_default                    Removed
...
docker system prune -a
Are you sure you want to continue? [y/N] y
Deleted Images:
untagged: lj7812/order-management:0.0.1
deleted: sha256:4daf0a63a30548a7074d665e894bf2855a280e626a4271a03415f298a5ff8219
deleted: sha256:d8160b5213e5bc7272fc6f3864e798a8b85598a572e392498a0e861bbda9fff1
deleted: sha256:63f2023a62b109c468619971631597dd4a5a233b6a0934b5f5fbe3e2c9087dd0
...
```
그리고 `docker desktop`을 완전히 종료한 뒤 다시 실행했다. 
```sh
docker compose up --build -d
```
이후, 다시 docker compose up을 실행해봤다.
![](assets/Docker%20Basic%2009%20-%20Docker%20Compose%20Microservices/image%208.png)
최종적으로 완료된 로그를 보면, 문제가 되었던 `inventory-api-container`라는 컨테이너 이름은 없다. 물론 아까부터 없었다. 과연 이번에는...
```sh
[+] Running 8/8
 ✔ Network 04-starter-projects_default                    Created                                                                                                                                             
 ✔ Container 04-starter-projects-product-inventory-1      Started                                                                                                                                             
 ✔ Container 04-starter-projects-contact-support-team-1   Started                                                                                                                                             
 ✔ Container 04-starter-projects-profile-management-1     Started                                                                                                                                             
 ✔ Container 04-starter-projects-shipping-and-handling-1  Started                                                                                                                                             
 ✔ Container 04-starter-projects-product-catalog-1        Started                                                                                                                                             
 ✔ Container 04-starter-projects-order-management-1       Started                                                                                                                                             
 ✔ Container ecommerce-ui-container                       Started
```

하지만 ==여전히 오류가 발생==하고 있다. 문제는 동일하다. `order-management`가 계속해서 http://inventory-api-container:3002 API를 호출한다. 하지만 [docker-compose.yml 파일](bear://x-callback-url/open-note?id=A4D85492-3A4D-4064-90B5-EE0B2833D160&header=Docker%20Compose!)을 보면 그런 단어 자체가 없고, environment도 문제 없다. 

그렇다면 이제 어쩔 수 없다. `order-management`에 직접 접속해 하나씩 확인해봐야겠다. 먼저 `Dockerfile`이 보인다. 내용을 보니 환경변수가 예전에 실습했던 내용이 들어가있다. 하지만 이건 Dockerfile로 이미지를 빌드할때 사용했던 파일일 뿐이다. ==실제로 현재 이미지에 환경변수가 어떻게 적용되어 있는지== 보자.
```sh
# ls
Dockerfile  README.md  pom.xml  src  target
# pwd
/app
# cat Dockerfile
# Use the official Maven runtime with JDK 17 as a base image
FROM maven:3.6.3-openjdk-17-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the project files
COPY . .
RUN mvn install

# Expose the port that the application listens on
EXPOSE 9090

# (Added) set ENV
ENV PRODUCT_INVENTORY_API_HOST=http://inventory-api-container
ENV PRODUCT_CATALOG_API_HOST=http://product-api-container
ENV SHIPPING_HANDLING_API_HOST=http://ship-handle-app-container

## 5. Define the command to run when the container starts.
CMD ["mvn", "spring-boot:run"]
```
아래는 해당 **컨테이너의 환경변수 중에서 _HOST를 포함하는 것들만 확인**한 결과다. 드디어 ==문제를 찾았다==. 내가 `docker-compose.yml`에서 `environment`를 작성할 때 `PRODUCT_INVENTORY_API_HOST`가 아닌, `REACT_APP_INVENTORY_API_HOST`를 사용해서 key와 value가 덮어쓰여지지 않은 것이다. 해당 이미지를 생성(build)했을 때는 Dockerfile을 참조했을테니, 기존 환경변수가 적용되어있는 상태인 것이다. 그렇다. 오타다. (...)
```sh
# env |grep _HOST
PRODUCT_INVENTORY_API_HOST=http://inventory-api-container
REACT_APP_PRODUCT_API_HOST=http://product-catalog
SHIPPING_HANDLING_API_HOST=http://ship-handle-app-container
REACT_APP_INVENTORY_API_HOST=http://product-inventory
PRODUCT_CATALOG_API_HOST=http://product-api-container
```
역시 휴먼에러가 가장 무서운 법이다. ~~하...~~

자, 다시 docker 컨테이너들을 모두 내리고, docker-compose.yml 파일을 수정한 뒤 다시 시작해보자.
```sh
docker compose down -v
[+] Running 8/8
 ✔ Container ecommerce-ui-container                       Removed                                                                                                                                            
 ✔ Container 04-starter-projects-profile-management-1     Removed                                                                                                                                            
 ✔ Container 04-starter-projects-shipping-and-handling-1  Removed                                                                                                                                             
 ✔ Container 04-starter-projects-order-management-1       Removed                                                                                                                                             
 ✔ Container 04-starter-projects-contact-support-team-1   Removed                                                                                                                                            
 ✔ Container 04-starter-projects-product-inventory-1      Removed                                                                                                                                            
 ✔ Container 04-starter-projects-product-catalog-1        Removed                                                                                                                                            
 ✔ Network 04-starter-projects_default                    Removed
...
```
```yml
version: '3'
services: 
  ecommerce-ui:
    image: lj7812/ecommerce-ui:0.0.1
    container_name: ecommerce-ui-container
    ports:
      - "4000:4000"
    environment:
      - REACT_APP_PROFILE_API_HOST=http://profile-management
      - REACT_APP_PRODUCT_API_HOST=http://product-catalog
      - REACT_APP_INVENTORY_API_HOST=http://product-inventory
      - REACT_APP_ORDER_API_HOST=http://order-management
      - REACT_APP_SHIPPING_API_HOST=http://shipping-and-handling
      - REACT_APP_CONTACT_API_HOST=http://contact-support-team
    depends_on:
      - contact-support-team
      - order-management
      - product-catalog
      - product-inventory
      - profile-management
      - shipping-and-handling

  contact-support-team:
    image: lj7812/contact-support-team:0.0.1

  order-management:
    image: lj7812/order-management:0.0.1
    environment: 
      # - REACT_APP_PRODUCT_API_HOST=http://product-catalog
      # - REACT_APP_INVENTORY_API_HOST=http://product-inventory
      - PRODUCT_INVENTORY_API_HOST=http://product-inventory
      - PRODUCT_CATALOG_API_HOST=http://product-catalog
      - SHIPPING_HANDLING_API_HOST=http://shipping-and-handling
    depends_on:
      - product-catalog
      - product-inventory

  product-catalog:
    image: lj7812/product-catalog:0.0.1

  product-inventory:
    image: lj7812/product-inventory:0.0.1

  profile-management:
    image: lj7812/profile-management:0.0.1

  shipping-and-handling:
    image: lj7812/shipping-and-handling:0.0.1

```
```sh
docker compose up
[+] Running 8/8
```

됐다. 드디어 성공이다. order-management도 정상적으로 http://product-inventory:3002 api를 호출하고 있다. 이제 상품 화면에서 ADD TO CART를 하더라도 오류가 발생하지 않는다.
![](assets/Docker%20Basic%2009%20-%20Docker%20Compose%20Microservices/image%209.png)
## 정리
> 여러 서비스를 정의하고, 각각에 필요한 환경변수와 의존성 등을 정의하는 방법을 익혔다.
- docker-compose.yml 작성 방법
  - 여러 service를 정의하는 방법
  - container_name를 생략한 이유
- 잘못된 서비스 이름 호출에 대한 troubleshooting

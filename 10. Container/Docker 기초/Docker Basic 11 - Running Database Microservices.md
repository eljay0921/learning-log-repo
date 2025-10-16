# Docker Basic 11 - Running Database: Microservices

#dev/skill/container/docker

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/43367526?start=0#overview)

## 개요
> 이번 장에서도 Database가 포함된 서비스를 다룬다. 앞서 실습했던 e-commerce 시스템이다.

## Push to Docker Hub
현재 시스템을 모두 정리하고, 프로젝트에서 아래 경로로 이동한다. 해당 경로의 애플리케이션들을 모두 docker image로 빌드하고 hub로 푸시하자. 모든 애플리케이션이 Mongo DB를 바라보도록 변경되었기 때문에 이 작업을 수행한다.
> workbook-starter-projects/05-starter-projects

![](assets/Docker%20Basic%2011%20-%20Running%20Database%20Microservices/image.png)

### system 정리
```sh
docker system prune -a
```
![](assets/Docker%20Basic%2011%20-%20Running%20Database%20Microservices/image%202.png)

### image build & push
```sh
docker build -t lj7812/ecommerce-ui:latest . 
docker build -t lj7812/order-management:latest .
...
docker images
REPOSITORY                     TAG       IMAGE ID       CREATED              SIZE
lj7812/shipping-and-handling   latest    92ccaf1a8b27   12 seconds ago       1.45GB
lj7812/profile-management      latest    39c222a4f16a   57 seconds ago       1.3GB
lj7812/product-inventory       latest    ba1de7ae44a1   About a minute ago   1.47GB
lj7812/product-catalog         latest    7498825b837c   About a minute ago   1.36GB
lj7812/order-management        latest    6af461791913   2 minutes ago        636MB
lj7812/contact-support-team    latest    cc36f890dbf4   3 minutes ago        1.47GB
lj7812/ecommerce-ui            latest    14dc8e560493   3 minutes ago        2.78GB
...
docker push lj7812/shipping-and-handling 
docker push lj7812/profile-management
...
```
![](assets/Docker%20Basic%2011%20-%20Running%20Database%20Microservices/image%203.png)
최종적으로 docker hub로 업로드 완료된 모습이다.


## Docker Compose 작성하기
> 자, 이제 e-commerce 시스템 구축을 위한 docker-compose.yml 파일을 작성해보자.

### docker-compose.yml
```yml
version: '3'
services:

  ecommerce-ui:
    image: lj7812/ecommerce-ui:latest
    # container_name: ecommerce-ui-container
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
    image: lj7812/contact-support-team:latest
    environment:
      - MONGODB_HOST=mongodb-contact-support
      - MONGODB_PORT=27017
      - MONGODB_DATABASE=contact-support-db

  mongodb-contact-support:
    image: mongo
    # container_name: mongodb-contact-support
    # environment: # Not necessary
    volumes:
      - mongodb-contact-support:/data/db

  order-management:
    image: lj7812/order-management:latest
    environment: 
      - PRODUCT_INVENTORY_API_HOST=http://product-inventory
      - PRODUCT_CATALOG_API_HOST=http://product-catalog
      - SHIPPING_HANDLING_API_HOST=http://shipping-and-handling
      - SPRING_DATA_MONGODB_DATABASE=order-mgmt-db
      - SPRING_DATA_MONGODB_URI=mongodb://mongodb-order-management:27017/order-mgmt-db
    depends_on:
      - product-catalog
      - product-inventory

  mongodb-order-management: # port 27017
    image: mongo
    # container_name: mongodb-order-management
    # environment:
    #   - MONGO_INITDB_DATABASE=order-mgmt-db
    #   - MONGO_INITDB_ROOT_USERNAME=root_user_name
    #   - MONGO_INITDB_ROOT_PASSWORD=root_user_password
    volumes:
      - mongodb-order-management:/data/db

  product-catalog:
    image: lj7812/product-catalog:latest
    environment:
      - MONGODB_HOST=mongodb-product-catalog
      - MONGODB_PORT=27017
      - MONGODB_DATABASE=product-catalog-db

  mongodb-product-catalog:  # port 27017
    image: mongo
    # container_name: mongodb-product-catalog
    # environment:
    #   - MONGO_INITDB_DATABASE=product-catalog-db
    #   - MONGO_INITDB_ROOT_USERNAME=root_user_name
    #   - MONGO_INITDB_ROOT_PASSWORD=root_user_password
    volumes:
      - mongodb-product-catalog:/data/db

  product-inventory:
    image: lj7812/product-inventory:latest
    environment:
      - POSTGRES_HOST=postgres-product-inventory
      - POSTGRES_PORT=5432
      - POSTGRES_DB=product-inventory-db
      - POSTGRES_USER=prd_inven_user
      - POSTGRES_PASSWORD=prd_inven_password
  
  postgres-product-inventory: # port 5432
    image: postgres
    # container_name: postgres-product-inventory
    environment:
      - POSTGRES_DB=product-inventory-db
      - POSTGRES_USER=prd_inven_user
      - POSTGRES_PASSWORD=prd_inven_password
    volumes:
      - postgres-product-inventory:/var/lib/postgresql/data

  profile-management:
    image: lj7812/profile-management:latest
    environment:
      - MYSQL_HOST=mysql-profile-management
      - MYSQL_PORT=3306
      - MYSQL_DATABASE=profile-mgmt-db
      - MYSQL_USER=profile_user
      - MYSQL_PASSWORD=profile_password
  
  mysql-profile-management: # port 3306
    image: mysql:8.0
    # container_name: mysql-profile-management
    environment:
      - MYSQL_DATABASE=profile-mgmt-db
      - MYSQL_USER=profile_user
      - MYSQL_PASSWORD=profile_password
      - MYSQL_ROOT_PASSWORD=super_power_password
    volumes:
      - mysql-profile-management:/var/lib/mysql

  shipping-and-handling:
    image: lj7812/shipping-and-handling:latest
    environment:
      - MONGO_URI=mongodb://mongodb-shipping:27017/shipping-db

  mongodb-shipping:
    image: mongo
    # container_name: mongodb-shipping
    # environment: # Not necessary
    volumes:
      - mongodb-shipping:/data/db

volumes:
  mongodb-product-catalog:
  mongodb-contact-support:
  mongodb-shipping:
  mongodb-order-management:
  mysql-profile-management:
  postgres-product-inventory:

```
몇 가지 작성 포인트를 알아보자
- database(mongo, mysql)를 사용하기 위한 volumes를 명시했다.
- profile-management 애플리케이션은 mysql을 사용한다.
  > workbook-starter-projects/05-starter-projects/profile-management/auth_api.js
- product-inventory 애플리케이션은 postgresql을 사용한다.
  > workbook-starter-projects/05-starter-projects/product-inventory/inventory_api.py
- 각 database 마다 필요한 환경 변수를 확인하자.
  - 공식 이미지와 애플리케이션에서 필요한 환경 변수를 확인 하자.
  - [mongo - Official Image](https://hub.docker.com/_/mongo#environment-variables)
  - [mysql - Official Image](https://hub.docker.com/_/mysql#environment-variables)
  - [postgres - Official Image](https://hub.docker.com/_/postgres#environment-variables)
- 각 database 마다 기본적인 volume 위치를 확인하자.
  - [mongo - Official Image](https://hub.docker.com/_/mongo#where-to-store-data)
  - [mysql - Official Image](https://hub.docker.com/_/mysql#where-to-store-data)
  - [postgres - Official Image](https://hub.docker.com/_/postgres#pgdata)
- Mongo DB의 환경변수 중, `MONGO_INITDB_DATABASE`는 반드시 작성할 필요는 없다.
  - `MONGO_INITDB_ROOT_USERNAME`, `MONGO_INITDB_ROOT_PASSWORD`도 마찬가지다.
  - 가 아니라... **지금은 작성하면 안 된다.** 이 환경변수가 있는 경우 인증 모드로 작동해 접속 uri에 계정 정보가 필요하다.
    - 그러나 현재 application 소스 코드를 보면 비인증 모드로 작성되어 있다.
    - 그러므로 위 환경변수는 제거해야 현재 실습이 가능하다. 
    - 실제 운영이라면 인증 모드 + 계정 정보를 입력하는 것이 맞다.
- 파일 내용이 길어지기 때문에 작성 중간 중간 `docker compose up`을 진행해 정상적으로 컨테이너가 실행되는지 확인하자
  -> `profile-management `애플리케이션에서 `mysql` 컨테이너 연결될 때
  ![](assets/Docker%20Basic%2011%20-%20Running%20Database%20Microservices/image%204.png)
- 참고로 Spring의 환경변수 포맷은 다음과 같은데, yml 파일에서 작성한 형태와 알아서 호환된다.
  - Spring Boot는 `Relaxed Binding`이라는 유연한 프로퍼티 매핑 시스템을 사용
```properties
...
spring.data.mongodb.uri=mongodb://host:27017/name
spring.data.mongodb.database=order_management
```
```yml
  order-management:
	image: lj7812/order-management:latest
	environment: 
  	- SPRING_DATA_MONGODB_DATABASE=order-mgmt-db
  	- SPRING_DATA_MONGODB_URI=mongodb://mongodb-order-management:27017/order-mgmt-db
```

### troubleshooting (1)
참고로 `database` 컨테이너가 보통은 더 늦게 올라오다 보니, `node.js`와 같은 다른 애플리케이션에서 해당 컨테이너에 `depens`할 때, 일시적으로 연결 실패 오류 메시지가 출력되는 것을 확인할 수 있다. 
```log
Error connecting to MySQL (attempt 1): Error: connect ECONNREFUSED 172.18.0.3:3306
    at Object.createConnectionPromise [as createConnection] (/app/node_modules/mysql2/promise.js:19:31)
    at connectToMySQL (/app/auth_api.js:30:32)
    at Object.<anonymous> (/app/auth_api.js:70:1)
    at Module._compile (internal/modules/cjs/loader.js:1114:14)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1143:10)
    at Module.load (internal/modules/cjs/loader.js:979:32)
    at Function.Module._load (internal/modules/cjs/loader.js:819:12)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:75:12)
    at internal/main/run_main_module.js:17:47 {
  code: 'ECONNREFUSED',
  errno: -111,
  sqlState: undefined
}
```
이를 방지하기 위해선 애플리케이션 레벨에서 재시도 로직을 적절히 구현하는 것도 가능하지만, `docker` 레벨에서 `healthcheck`를 하는 것이 좋다. (더 좋다는 건 그냥 내 생각)

아래는 그냥 샘플이고, 
```yml
version: '3.9'

services:
  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: test
    ports:
      - "3306:3306"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      retries: 10
      timeout: 3s

  auth-api:
    build: .
    depends_on:
      mysql:
        condition: service_healthy
    ports:
      - "3003:3003"

```

#### health-check
아래는 현재 실습하던 파일에 적용한 모습이다.
```yml

```

### troubleshooting (2)
자꾸만 **Mongo DB** 컨테이너에 의존하는 애플리케이션 컨테이너들이 제대로 붙지 않는다. 자세히 보니 프로토콜은 `mongodb`로 명시하지 않았다. (...) 휴먼 에러 ㅠㅠ
> as-is)
> SPRING_DATA_MONGODB_URI=//mongodb-order-management:27017/order-mgmt-db
> to-be)
> SPRING_DATA_MONGODB_URI=mongodb://mongodb-order-management:27017/order-mgmt-db

### troubleshooting (3)
일부 mongo db에 연결이 안 되었다. 강의에선 작성하지 않았는데, 내가 굳이 굳이 작성한 것 때문에 발생한 문제였다. 
```yml
  mongodb-order-management: # port 27017
    image: mongo
    # container_name: mongodb-order-management
    environment:
      - MONGO_INITDB_DATABASE=order-mgmt-db
      - MONGO_INITDB_ROOT_USERNAME=root_user_name
      - MONGO_INITDB_ROOT_PASSWORD=root_user_password
    volumes:
      - mongodb-order-management:/data/db
```
여기서 보면 `environment:`에서 `MONGO_INITDB_...` 환경변수를 3가지 작성했다. 이것을 입력할 경우 mongo db가 ==인증 모드==로 동작해 **연결 시 계정 정보**를 요구한다. 그러나 현재 실습 중인 `application`들은 모두 ==비인증 모드==로 uri를 호출해 연결하고 있다. 그렇기에 연결에서 계속 실패가 발생하는 것이었다. 본 실습에선 지우고 하자. (...)

### troubleshooting (4)
자꾸만 `shipping-and-handling application`이 뜨지 않아서 확인해 보니, 다른 서비스 `environment`를 그대로 복사해서 사용한 것이 원인이었다. 이 `application`은 `MONGODB_HOST`, `MONGODB_PORT`, `MONGODB_DATABASE`를 요구하지 않았고, `uri`를 통째로 요구했다. 심지어 `database name`은 소스 코드에 하드 코딩 되어있다. 항상 `application`에서 뭘 필요로 하는지 잘 확인하자.

### troubleshooting 모두 해결
이 모든 **troubleshooting**이 해결된 최종본이 [위에 작성된 docker-compose.yml](bear://x-callback-url/open-note?id=8D1B1081-9AEA-4A35-B21D-C57781EED2A0&header=Docker%20Compose%20%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0) 파일이다. (1번만 미포함)

## Docker Compose Up
> 작성된 docker-compose.yml 파일을 이용해 서비스를 실행하자.

```sh
docker compose up
[+] Running 14/14
 ✔ Network 05-starter-projects_default                         Created                                                                                                                                      
 ✔ Container 05-starter-projects-mongodb-contact-support-1     Started                                                                                                                                      
 ✔ Container 05-starter-projects-mongodb-order-management-1    Started                                                                                                                                      
 ✔ Container 05-starter-projects-postgres-product-inventory-1  Started                                                                                                                                      
 ✔ Container 05-starter-projects-mysql-profile-management-1    Started                                                                                                                                      
 ✔ Container 05-starter-projects-mongodb-shipping-1            Started                                                                                                                                      
 ✔ Container 05-starter-projects-shipping-and-handling-1       Started                                                                                                                                      
 ✔ Container 05-starter-projects-profile-management-1          Started                                                                                                                                      
 ✔ Container 05-starter-projects-mongodb-product-catalog-1     Started                                                                                                                                      
 ✔ Container 05-starter-projects-contact-support-team-1        Started                                                                                                                                      
 ✔ Container 05-starter-projects-product-catalog-1             Started                                                                                                                                      
 ✔ Container 05-starter-projects-product-inventory-1           Started                                                                                                                                      
 ✔ Container 05-starter-projects-order-management-1            Started                                                                                                                                      
 ✔ Container 05-starter-projects-ecommerce-ui-1                Started
```

이후 **localhost:4000**으로 접속해, 모든 페이지과 기능이 정상적으로 동작하는 것을 확인했으며, `Database`를 적용했으므로 서버를 내렸다가(`docker compose down`) 다시 올려도 모든 데이터가 유지되는 점도 확인했다.

## 정리
> docker compose를 이용해 e-commerce Microservices를 구축했다.
- docker-compose.yml 작성 방법 (복습)
  - database 리소스 작성 방법 (복습)


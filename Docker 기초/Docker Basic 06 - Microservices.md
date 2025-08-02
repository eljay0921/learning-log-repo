#  Docker Basic 06 - Microservices

#dev/skill/container/docker

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/43367484#overview)<!-- {"preview":"true"} -->

## 개요 
> 이번 장에서는 보다 많은 Docker container를 실행하고, 그들 사이의 네트워크를 구성하면서 Microservice e-commerce를 구축한다.

## 시스템 구성 확인하기
먼저, 우리가 구축할 시스템에 어떤 것들이 있는지 보자.
> workbook-starter-projects/03-starter-projects/...

![](Docker%20Basic%2006%20-%20Microservices/image.png)

`ecmmerce-ui`부터 해서, `주문 관리`, `상품 재고` 등 다양한 애플리케이션이 존재한다. 우리는 `ecommerce-ui`를 먼저 실행할 계획이다. 

## (요구사항) ecommerce-ui
![](Docker%20Basic%2006%20-%20Microservices/image%202.png)<!-- {"width":412} -->
`ecommerce-ui`에는 `client`와 `server`가 있는데, `server`의 `server.js`에서 개발자의 지침을 읽어보자. 이 애플리케이션이 `bare metal`에서는 어떻게 실행되는지 설명하고 있다.
```javascript
import express from 'express';
import bodyParser from 'body-parser';
import cors from 'cors';
import path from 'path';
import { fileURLToPath } from 'url';

const app = express();
app.use(bodyParser.json());
app.use(cors());

// (생략)
...

// # Developer: Captain DevOps, here's how to run the app:

// # 1. Clone the repo and navigate to the server directory
// # 2. Run 'npm install' to install server dependencies
// # 3. Navigate to the client directory and run 'npm install' for React app dependencies
// # 4. In the client directory, run 'npm run build' to build the React app
// # 5. Navigate back to the server directory and run 'node app.js'

// # This starts the Node.js server, serving the React app and providing API routes.

// # I've been running it bare metal, but I need you to containerize it.

// # Let me know if you need anything else.

// # Cheers,
// # Master NodeJS
```

## (Dockerize) ecommerce-ui
### Dockerfile
```dockerfile
# Use the official Node.js image as the base image
FROM node:14

# Set the working directory inside the container
WORKDIR /app

# Copy the server code to the working directory
COPY server /app/server

# Change to the server directory
WORKDIR /app/server

# Install the server dependencies
RUN npm install

# Change back to the app directory
WORKDIR /app

# Copy the client code to the client directory
COPY client /app/client

# Change to the client directory
WORKDIR /app/client

# Install the client dependencies
RUN npm install

# Build the React application
RUN npm run build

# Change back to the server directory
WORKDIR /app/server

# Expose the port that the server listens on
EXPOSE 4000

# Run the Node.js server when the container starts
CMD ["node", "server.js"]
```
위 `dockerfile`은 앞선 개발자의 지침을 참고해 작성한 것이다. 또한 지침에서는 `node app.js`를 실행하라고 했는데, 파일명을 보니 `server.js`이다. 

### Docker build & run
```sh
docker build -t ecommerce-ui . 
[+] Building 52.7s (17/17) FINISHED
...
docker images
REPOSITORY     TAG       IMAGE ID       CREATED              SIZE
ecommerce-ui   latest    7f8f6941a912   About a minute ago   2.12GB
...
docker network create ecommerce-network
0f7e93c8a283df25ea7b4292128a028c82574331f2264ca8a9c7f0f76d062835
...
docker network ls
NETWORK ID     NAME                DRIVER    SCOPE
26c64483f1ad   bridge              bridge    local
0f7e93c8a283   ecommerce-network   bridge    local
255a0299a3d4   host                host      local
a2cd7e6dcbf7   none                null      local
```
docker image를 빌드하고, 이번 `ecommerce` 시스템 구축을 위한 `network`도 미리 만들었다. ==ecommerce-network==.
```sh
docker run --rm -p 4000:4000 --network ecommerce-network --name ecommerce-ui-container ecommerce-ui
Server is running on port 4000
```
docker run으로 `ecommerce-ui`를 실행시켰다. 이제 `localhost:4000`으로 접근하면 준비된 `ecommerce-ui` 서버가 확인된다.
![](Docker%20Basic%2006%20-%20Microservices/image%203.png)
CREATE AN ACCOUNT 페이지로 접근 해, SIGN UP을 시도하면 다음과 같이 오류가 발생하며 실패한다.
![](Docker%20Basic%2006%20-%20Microservices/image%204.png)
당연하게도, 아직 다른 서비스가 준비되지 않았기 때문이다. 현재 ecommerce-ui-container에서 출력되는 로그는 다음과 같다. 
```log
Error signing up: TypeError: node-fetch cannot load undefined:3003/api/signup. URL scheme "undefined" is not supported.
    at file:///app/server/node_modules/node-fetch/src/index.js:54:10
    at new Promise (<anonymous>)
    at fetch (file:///app/server/node_modules/node-fetch/src/index.js:49:9)
    at file:///app/server/routes/auth.js:8:30
    at Layer.handle [as handle_request] (/app/server/node_modules/express/lib/router/layer.js:95:5)
    at next (/app/server/node_modules/express/lib/router/route.js:149:13)
    at Route.dispatch (/app/server/node_modules/express/lib/router/route.js:119:3)
    at Layer.handle [as handle_request] (/app/server/node_modules/express/lib/router/layer.js:95:5)
    at /app/server/node_modules/express/lib/router/index.js:284:15
    at Function.process_params (/app/server/node_modules/express/lib/router/index.js:346:12)
```

## (요구사항) profile-management
자 이번에는 회원 관리를 위한 profile 시스템을 구축하자.
> workbook-starter-projects/03-starter-projects/profile-management/
> workbook-starter-projects/03-starter-projects/profile-management/auth_api.js
```js
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const jwt = require('jsonwebtoken');

const app = express();
app.use(bodyParser.json());
app.use(cors());

// (중략)
// ...
  
// Start the server
const port = 3003;
app.listen(port, () => {
  console.log(`Authentication API is running on port ${port}`);
});

```
이번에는 개발자의 지침 따위(?)는 없다. 소스코드와 포트 정보만 확인할 수 있을 뿐이다. 
![](Docker%20Basic%2006%20-%20Microservices/image%205.png)<!-- {"width":409} -->
profile-management의 파일 구조를 보고 dockerfile을 작성해보자.
## (Dockerize) profile-management
### Dockerfile
```dockerfile
# Use the official Node.js image as the base image
FROM node:14

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json . 

# Install the dependencies
RUN npm install

# Copy the rest of the application source code
COPY . . 

# Expose the port that the API listens on
EXPOSE 3003

# Run the Node.js application when the container starts
CMD ["node", "auth_api.js"]
```
개발자의 지침은 없었지만, package.json에 명시된 dependencies를 설치해야 함은 알고 있다. COPY 이후 RUN을 통해 npm install을 진행하자. 나머지 항목은 다른 애플리케이션과 유사하다. 
### Docker build & run
```sh
docker build -t profile-mgmt .
[+] Building 4.2s (11/11) FINISHED
...
docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
profile-mgmt   latest    15f2c874c63d   59 seconds ago   1.3GB
ecommerce-ui   latest    7f8f6941a912   29 minutes ago   2.12GB
...
docker run --rm -p 3003:3003 --network ecommerce-network --name profile-mgmt-container profile-mgmt 
Authentication API is running on port 3003
```

### Test (1)
docker build 이후 run까지 별다른 이슈 없이 정상적으로 완수했다. localhost:3003으로 접근되는 것도 확인했다.
![](Docker%20Basic%2006%20-%20Microservices/image%206.png)
자, 이제 다시 `ecommerce-ui`(localhost:4000)으로 접근해 Account를 다시 생성해보...려고 했는데, **여전히 실패**했다. 
원인은 내가 생성한 `docker container`의 이름 때문이다. `profile-mgmt-container`로 생성했는데 이는 `ecommerce-ui-container`에서 접근할 **profile-management API**의 호스트명과 달랐던 것이다. 다시 강의와 동일하게 작성해보자.

### Test (2)
```sh
docker run --rm -p 3003:3003 --network ecommerce-network --name profile-management profile-mgmt
Authentication API is running on port 3003
```
but, 여전히 실패한다. 강사의 말대로 따라하는데 뭔가 문제가 있다. `ecommerce-ui`에서 `profile-management`로 어떻게 접근하는지 확인했더니 다음과 같았다. 첫 번째는 `auth.js` 소스 코드이고, 두 번째는 `.env` 파일이다. 그렇다. ==환경변수==를 적용하면 해결될 것으로 보인다. (강의를 계속 보니 강사도 이걸 적용한다)
```js
import express from 'express';
const router = express.Router();
import('node-fetch').then(({ default: fetch }) => {
  const PROFILE_API_HOST = process.env.REACT_APP_PROFILE_API_HOST;

  router.post('/signup', async (req, res) => {

// (생략)
```
```
REACT_APP_PROFILE_API_HOST=http://localhost
REACT_APP_PRODUCT_API_HOST=http://localhost
REACT_APP_INVENTORY_API_HOST=http://localhost
REACT_APP_ORDER_API_HOST=http://localhost
REACT_APP_SHIPPING_API_HOST=http://localhost
REACT_APP_CONTACT_API_HOST=http://localhost
```

### Test (3)
자, 환경변수를 적용하고 다시 docker run을 해보자. 

먼저 `profile-management` 애플리케이션은 내가 하고싶은 이름으로 작성했다. -> `profile-mgmt-container`
```sh
docker run --rm -p 3003:3003 --network ecommerce-network --name profile-mgmt-container profile-mgmt
Authentication API is running on port 3003
```
그리고 다시 `ecommerce-ui`에 환경변수를 적용하고 실행해보자
```sh
docker run --rm -p 4000:4000 --network ecommerce-network -e REACT_APP_PROFILE_API_HOST=http://profile-mgmt-container --name ecommerce-ui-container ecommerce-ui
Server is running on port 4000
```

이번에는 드디어 성공했다. 다음과 같이 입력하고 sign up에 완료했다.
![](Docker%20Basic%2006%20-%20Microservices/image%207.png)<!-- {"width":605} --> ![](Docker%20Basic%2006%20-%20Microservices/image%208.png)<!-- {"width":307} -->
이후 가입했던 계정 정보를 입력하니 로그인까지 가능했다.
![](Docker%20Basic%2006%20-%20Microservices/image%209.png)<!-- {"width":605} -->
아직 다른 메뉴에 접근하면 오류가 발생했지만, Manage Profile 메뉴는 접근이 가능했다. 
![](Docker%20Basic%2006%20-%20Microservices/image%2010.png)<!-- {"width":605} -->

## (요구사항) shipping-and-handling
> 이번에는 배송 및 취급 애플리케이션을 구축하자

> workbook-starter-projects/03-starter-projects/shipping-and-handling/main.go
```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "time"
    "net/http"
)

// (생략)
// ...

func main() {
    http.HandleFunc("/shipping-fee", corsMiddleware(handleShippingFee))
    http.HandleFunc("/shipping-explanation", corsMiddleware(handleShippingExplanation))
    http.HandleFunc("/all-shipping-fees", corsMiddleware(handleAllShippingFees))

    fmt.Println("Server is running on port 8080...")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```
이번에도 개발자 지침은 없다. 소스 코드를 봤을 때, 8080 포트를 사용하겠다는 정도만 확인된다. 바로 dockerfile 작성으로 넘어가자. 

## (Dockerize) shipping-and-handling

### Dockerfile
```dockerfile
# Use the official Golang image as the base image
FROM golang:1.20

# Set the working directory inside the container
WORKDIR /app

# Copy the main.go file to the working directory
COPY main.go . 

# Expose the port that the API listens on
EXPOSE 8080

# Run the code when the container starts
CMD ["go", "run", "main.go"]
```

### Docker build & run 
```sh
docker build -t ship-handle-app .
[+] Building 14.1s (9/9) FINISHED
...
docker images
REPOSITORY        TAG       IMAGE ID       CREATED         SIZE
ship-handle-app   latest    1ab84c183859   4 seconds ago   1.26GB
profile-mgmt      latest    15f2c874c63d   3 hours ago     1.3GB
ecommerce-ui      latest    7f8f6941a912   3 hours ago     2.12GB
...
docker run --rm -p 8080:8080 --network ecommerce-network --name ship-handle-app-container ship-handle-app
Server is running on port 8080...
...
```

마찬가지로 별다른 이슈없이 docker container가 올라왔다. 이제 `ecommerce-ui-container`에 환경 변수를 추가하고, 다시 실행해야 할 것 같은데, 매번 변수를 추가하고 재실행 하려니 너무 비효율적이다. `dockerfile`을 아예 수정하고 다시 빌드하자. 강의에선 그냥 `docker run`에 `-e` 옵션을 하나 더 추가했다. (...) 아마 조금 더 뒤에 설명하려는 것 같다.

### Dockerfile re-write (ecommerce-ui 개선)
```dockerfile
# Use the official Node.js image as the base image
FROM node:14

# Set the working directory inside the container
WORKDIR /app

# Copy the server code to the working directory
COPY server /app/server

# Change to the server directory
WORKDIR /app/server

# Install the server dependencies
RUN npm install

# Change back to the app directory
WORKDIR /app

# Copy the client code to the client directory
COPY client /app/client

# Change to the client directory
WORKDIR /app/client

# Install the client dependencies
RUN npm install

# Build the React application
RUN npm run build

# Change back to the server directory
WORKDIR /app/server

# Expose the port that the server listens on
EXPOSE 4000

# (Added) Set ENV
ENV REACT_APP_PROFILE_API_HOST=http://profile-mgmt-container
ENV REACT_APP_PRODUCT_API_HOST=http://product-api-container
ENV REACT_APP_INVENTORY_API_HOST=http://inventory-api-container
ENV REACT_APP_ORDER_API_HOST=http://order-api-container
ENV REACT_APP_SHIPPING_API_HOST=http://ship-handle-app-container
ENV REACT_APP_CONTACT_API_HOST=http://contact-api-container

# Run the Node.js server when the container starts
CMD ["node", "server.js"]
```
기존에 환경변수로 추가했던 `profile-mgmt-container`는 물론이고, 이번에 추가할 `ship-handle-app-container`와 더불어 나머지 host 이름도 그냥 지금 정해버렸다. 앞으로 위 이름에 맞게 컨테이너를 실행하면 될 것이다.

### Docker re-build & re-run (ecommerce-ui 개선)
```sh
docker build -t ecommerce-ui . 
[+] Building 1.0s (16/16) FINISHED
...
docker images
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
ship-handle-app   latest    1ab84c183859   12 minutes ago   1.26GB
profile-mgmt      latest    15f2c874c63d   3 hours ago      1.3GB
ecommerce-ui      latest    24516bdcf614   3 hours ago      2.12GB
...
docker run --rm -p 4000:4000 --network ecommerce-network --name ecommerce-ui-container ecommerce-ui
Server is running on port 4000
...
```
앞서 작성한 dockerfile로 다시 build를 진행했고, 서버까지 정상적으로 띄웠다. 기존에 잘 동작했던 profile-mgmt-container에도 이상 없다.
![](Docker%20Basic%2006%20-%20Microservices/image%2011.png)
![](Docker%20Basic%2006%20-%20Microservices/image%2012.png)

그리고 이제 [앞서 실행](bear://x-callback-url/open-note?id=2D1BAF1E-AC29-4191-B472-CD3DF9D0BC0B&header=%28Dockerize%29%20shipping-and-handling)했던 `ship-handle-app-container`를 확인해보면(Shipping Options 클릭) 다음과 같이 정상적인 페이지가 호출된다.
![](Docker%20Basic%2006%20-%20Microservices/image%2013.png)

## (요구사항) contact-support-team
> 이번에는 contact-support-team 애플리케이션이다. 계속하자.
![](Docker%20Basic%2006%20-%20Microservices/image%2014.png)<!-- {"width":353} -->
> workbook-starter-projects/03-starter-projects/contact-support-team/server.py
> workbook-starter-projects/03-starter-projects/contact-support-team/requirements.txt
이번에는 2개의 파일을 참조하자.
```python
from flask import Flask, jsonify, request
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

@app.route('/api/contact-message', methods=['GET'])
def get_contact_message():
    response = {
        'message': "We're here to help! If you have any questions, concerns, or feedback, please don't hesitate to reach out to us. Our dedicated support team is ready to assist you."
    }
    return jsonify(response)

@app.route('/api/contact-submit', methods=['POST'])
def submit_contact_form():
    post_data = request.get_json()
    print("Received submission:", post_data)
    response = {'status': 'success', 'message': 'Your message has been successfully submitted.'}
    return jsonify(response)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```
```txt
Flask==2.0.1
flask-cors==3.0.10
Werkzeug==2.0.1
```
개발자 지침은 없지만 요구사항은 명확하다. 포트는 `8000`번이고, `requirements.txt`에 명시된 `dependencies`를 설치하면 된다.

## (Dockerize) contact-support-team
### Dockerfile
```dockerfile
# Use the official Python image as the base image
FROM python:3.8-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the requirements file to the working directory
COPY requirements.txt . 

# Install the dependencies
RUN pip install -r requirements.txt

# Copy the Flask application to the working directory
COPY server.py . 

# Expose the port that the Flask application listens on
EXPOSE 8000

# Run the Flask application when the container starts
CMD ["python", "server.py"]
```

### Docker build & run
```sh
docker build -t contact-support-app .
[+] Building 9.3s (11/11) FINISHED
...
docker images
contact-support-app   latest    1c2e678a2458   28 seconds ago   234MB
ship-handle-app       latest    1ab84c183859   27 minutes ago   1.26GB
profile-mgmt          latest    15f2c874c63d   3 hours ago      1.3GB
ecommerce-ui          latest    24516bdcf614   4 hours ago      2.12GB
...
docker run --rm -p 8000:8000 --network ecommerce-network --name contact-api-container contact-support-app 
 * Serving Flask app 'server' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.18.0.5:8000/ (Press CTRL+C to quit)
...
```
참고로 로컬에서 직접 접근하고 싶은 게 아니라면 굳이 `-p` 옵션으로 포트를 매핑할 필요는 없다. 아무튼 이전과 같은 패턴으로 `contact-support-app`을 컨테이너로 실행시켰다. 이름은 [앞서 작성](bear://x-callback-url/open-note?id=2D1BAF1E-AC29-4191-B472-CD3DF9D0BC0B&header=Dockerfile%20re-write%20%28ecommerce-ui%20%EA%B0%9C%EC%84%A0%29)했던 `ecommerce-ui`의 `dockerfile ENV`를 참고했다. -> `contact-api-container`

테스트 결과, http://localhost:8000/은 `Not Found`를 응답하지만 서버는 정상 실행 중인 상태다. `ecommerce-ui-container`에서도 정상적으로 접근되고 있다. (메인 화면에서 Contact Support 클릭) 
![](Docker%20Basic%2006%20-%20Microservices/image%2015.png)<!-- {"width":963} -->![](Docker%20Basic%2006%20-%20Microservices/image%2016.png)<!-- {"width":482} --> ![](Docker%20Basic%2006%20-%20Microservices/image%2017.png)<!-- {"width":462} -->

다음은 contact-api-container에 찍히고 있는 로그이다.
```log
 * Serving Flask app 'server' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.18.0.5:8000/ (Press CTRL+C to quit)
192.168.65.1 - - [30/Jul/2025 06:21:50] "GET / HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:21:50] "GET /favicon.ico HTTP/1.1" 404 -
172.18.0.2 - - [30/Jul/2025 06:21:57] "GET /api/contact-message HTTP/1.1" 200 -
192.168.65.1 - - [30/Jul/2025 06:22:03] "GET /apple-touch-icon-precomposed.png HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:03] "GET /apple-touch-icon.png HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:03] "GET /favicon.ico HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:04] "GET / HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:21] "GET /apple-touch-icon-precomposed.png HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:21] "GET /apple-touch-icon.png HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:21] "GET /favicon.ico HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:21] "GET / HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:41] "GET /apple-touch-icon-precomposed.png HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:41] "GET /apple-touch-icon.png HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:41] "GET /favicon.ico HTTP/1.1" 404 -
192.168.65.1 - - [30/Jul/2025 06:22:41] "GET / HTTP/1.1" 404 -
172.18.0.2 - - [30/Jul/2025 06:23:11] "GET /api/contact-message HTTP/1.1" 200 -
172.18.0.2 - - [30/Jul/2025 06:23:45] "POST /api/contact-submit HTTP/1.1" 200 -
```

강의에선 계속해서 docker run으로 ecommerce-ui를 중지-재실행하면서 -e 옵션으로 각 컨테이너의 호스트 이름을 넣고 있다. 이를 따르지 않고 바로 dockerfile에서 ENV로 적용했는데 다행히 잘 동작한다. ~~강사는 언제 설명하려는 걸까?~~

## (요구사항) product-inventory
> 계속해서 앞선 내용의 반복이므로 설명은 생략한다.

![](Docker%20Basic%2006%20-%20Microservices/image%2018.png)<!-- {"width":423} -->
> workbook-starter-projects/03-starter-projects/product-inventory/inventory_api.py
> workbook-starter-projects/03-starter-projects/product-inventory/requirements.txt
```python
from flask import Flask, jsonify, request
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

# (생략)
# ...

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3002)
```
```txt
Flask==2.0.1
flask-cors==3.0.10
Werkzeug==2.0.1
```

## (Dockerize) product-inventory
### Dockerfile
```dockerfile
# Use the official Python image as the base image
FROM python:3.8-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the requirements file to the working directory
COPY requirements.txt . 

# Install the dependencies
RUN pip install -r requirements.txt

# Copy the Flask application to the working directory
COPY inventory_api.py . 

# Expose the port that the Flask application listens on
EXPOSE 3002

# Run the Flask application when the container starts
CMD ["python", "inventory_api.py"]
```

### Docker build & run
```sh
docker build -t product-inventory-app .
[+] Building 2.2s (11/11) FINISHED
...
docker images
REPOSITORY              TAG       IMAGE ID       CREATED          SIZE
product-inventory-app   latest    1f43b5e44c10   14 seconds ago   234MB
contact-support-app     latest    1c2e678a2458   14 minutes ago   234MB
ship-handle-app         latest    1ab84c183859   42 minutes ago   1.26GB
profile-mgmt            latest    15f2c874c63d   3 hours ago      1.3GB
ecommerce-ui            latest    24516bdcf614   4 hours ago      2.12GB
...
docker run --rm -p 3002:3002 --network ecommerce-network --name inventory-api-container product-inventory-app
 * Serving Flask app 'inventory_api' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.18.0.6:3002/ (Press CTRL+C to quit)
...
```
이렇게 `product-inventory` 애플리케이션까지 컨테이너로 잘 띄웠으나, 해당 `Inventory Management` 페이지에서 내부적으로 `product api`까지 호출하고 있기 때문에 아직 정상적으로 동작하진 않는다. 바로 이어서 `product api(product-catalog)`까지 진행하자.
![](Docker%20Basic%2006%20-%20Microservices/image%2019.png)

## (요구사항) product-catalog

![](Docker%20Basic%2006%20-%20Microservices/image%2020.png)<!-- {"width":360} -->
> workbook-starter-projects/03-starter-projects/product-catalog/index.js
```js
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');

const app = express();
app.use(bodyParser.json());
app.use(cors());

// In-memory storage for products
let products = [
  {
    id: 1,
    name: 'Wireless Bluetooth Headphones',
    description: 'High-quality sound and comfortable fit',
    price: 59.99,
    category: 'Electronics',
  },
  {
    id: 2,
    name: 'Vintage Leather Backpack',
    description: 'Stylish and durable backpack for everyday use',
    price: 89.99,
    category: 'Accessories',
  },

// (생략)
// ...

  {
    id: 11,
    name: 'Insulated Camping Tent',
    description: 'A durable and insulated tent for your outdoor adventures',
    price: 349.99,
    category: 'Outdoor',
  },
  {
    id: 12,
    name: 'Bluetooth Speaker',
    description: 'Portable speaker with exceptional sound quality',
    price: 99.99,
    category: 'Electronics',
  }
];
// Get all products
app.get('/api/products', (req, res) => {
  res.json(products);
});

// Get a single product by ID
app.get('/api/products/:id', (req, res) => {
  const productId = parseInt(req.params.id);
  const product = products.find((p) => p.id === productId);

  if (product) {
    res.json(product);
  } else {
    res.status(404).json({ error: 'Product not found' });
  }
});

// Start the server
const port = process.env.PORT || 3001;
app.listen(port, () => {
  console.log(`Product Catalog microservice is running on port ${port}`);
});
```

## (Dockerize) product-catalog
### Dockerfile
```dockerfile
# Use the official Node.js image as the base image
FROM node:14

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json . 

# Install the dependencies
RUN npm install

# Copy the rest of the application source code
COPY index.js . 

# Expose the port that the API listens on
EXPOSE 3001

# Run the Node.js application when the container starts
CMD ["node", "index.js"]
```

### Docker build & run
```sh
docker build -t product-catalog-app .
[+] Building 1.1s (10/10) FINISHED
...
docker images
REPOSITORY              TAG       IMAGE ID       CREATED          SIZE
product-catalog-app     latest    f5f9310a2477   37 seconds ago   1.3GB
product-inventory-app   latest    1f43b5e44c10   12 minutes ago   234MB
contact-support-app     latest    1c2e678a2458   26 minutes ago   234MB
ship-handle-app         latest    1ab84c183859   54 minutes ago   1.26GB
profile-mgmt            latest    15f2c874c63d   4 hours ago      1.3GB
ecommerce-ui            latest    24516bdcf614   4 hours ago      2.12GB
...
docker run --rm -p 3001:3001 --network ecommerce-network --name product-api-container product-catalog-app
Product Catalog microservice is running on port 3001
...
```

이후 `ecommerce-ui-container`로 띄운 서버(localhost:4000)의 메인 화면에서 `Inventory Management` 메뉴로 접근하면 정상적인 화면을 볼 수 있다. 이로써 `product-inventory`와 `product-catalog` 시스템을 모두 테스트했다. 
![](Docker%20Basic%2006%20-%20Microservices/image%2021.png)

물론 Product Catalog 메뉴로 접근해도 정상적인 화면을 볼 수 있다.
![](Docker%20Basic%2006%20-%20Microservices/image%2022.png)<!-- {"width":444} --> ![](Docker%20Basic%2006%20-%20Microservices/image%2023.png)<!-- {"width":507} -->
하지만 아직 ADD TO CART 기능은 동작하지 않는다.

## (요구사항) order-management
> 이제 마지막 애플리케이션이다.
![](Docker%20Basic%2006%20-%20Microservices/image%2024.png)<!-- {"width":379} -->
> workbook-starter-projects/03-starter-projects/order-management/src/main/resources/application.properties
> workbook-starter-projects/03-starter-projects/order-management/pom.xml
```properties
spring.application.name=order-management
server.port=9090
server.address=0.0.0.0
PRODUCT_INVENTORY_API_HOST=http://localhost
PRODUCT_CATALOG_API_HOST=http://localhost
SHIPPING_HANDLING_API_HOST=http://localhost
```
이 SpringBoot 애플리케이션은 `pom.xml` 파일 내에 `dependencies`가 명시되어 있으며, `application.properties`에서 포트를 포함한 환경변수를 확인할 수 있다. 그렇다면, 앞서 `ecommerce-ui`의 Dockerfile에 작성했던 것과 **같은 HOST 이름**이 필요할 것이다. 

## (Dockerize) order-management
### Dockerfile
```dockerfile
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
`ecommerce-ui`의 dockerfile을 참조해, `ENV`를 세팅했다. key 값은 `application.properties`에서 가져왔다.

### Docker build & run
```sh
docker build -t order-mgmt .
[+] Building 39.5s (10/10) FINISHED
...
docker images
REPOSITORY              TAG       IMAGE ID       CREATED             SIZE
order-mgmt              latest    7843e10ab2f7   2 minutes ago       822MB
product-catalog-app     latest    f5f9310a2477   22 minutes ago      1.3GB
product-inventory-app   latest    1f43b5e44c10   34 minutes ago      234MB
contact-support-app     latest    1c2e678a2458   49 minutes ago      234MB
ship-handle-app         latest    1ab84c183859   About an hour ago   1.26GB
profile-mgmt            latest    15f2c874c63d   4 hours ago         1.3GB
ecommerce-ui            latest    24516bdcf614   4 hours ago         2.12GB
...
docker run --rm -p 9090:9090 --network ecommerce-network --name order-api-container order-mgmt
[INFO] Scanning for projects...
[INFO]
[INFO] ----------------------< com.ltp:order-management >----------------------
[INFO] Building order-management 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] >>> spring-boot-maven-plugin:3.2.3:run (default-cli) > test-compile @ order-management >>>
[INFO]
[INFO] --- maven-resources-plugin:3.3.1:resources (default-resources) @ order-management ---
[INFO] Copying 1 resource from src/main/resources to target/classes
[INFO] Copying 0 resource from src/main/resources to target/classes

...

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.2.3)

2025-07-30T07:08:49.598Z  INFO 62 --- [order-management] [           main] c.l.o.OrderManagementApplication         : Starting OrderManagementApplication using Java 17-ea with PID 62 (/app/target/classes started by root in /app)
...
```

이제 order-management가 정상적으로 동작하는지 테스트하자. 아래 화면에서 ADD TO CART를 누르면 오류 없이 정상적으로 처리된다. 아래 order-api-container의 로그에서도 확인할 수 있다.
![](Docker%20Basic%2006%20-%20Microservices/image%2025.png)
```log
http://inventory-api-container:3002/api/inventory/1
http://product-api-container:3001/api/products/1
Items in user 2's cart:
Product ID: 1, Quantity: 1, Name: Wireless Bluetooth Headphones, Price: 59.99
```

이후 몇 차례 상품을 더 담고, 메인 화면에서 Order Management 메뉴로 접근하면 My Cart 항목이 제대로 출력된다. 
![](Docker%20Basic%2006%20-%20Microservices/image%2026.png)
```log
2
Electronics High-quality sound and comfortable fit Wireless Bluetooth Headphones 59.99 10.0
Electronics Track your fitness and stay connected on the go Smartwatch Fitness Tracker 199.99 10.0
Electronics Portable speaker with exceptional sound quality Bluetooth Speaker 99.99 10.0
30.0
```

## 정리
> 이상으로, 작지만 완벽한(?) E-Commerce 시스템을 Microservices 형태로 구축해봤다. 
- 각각의 application의 기능은 이미 작성된 코드를 활용한 것이고, 나는 dockerize와 각 container 간의 연결에만 집중했다. 
- 물론 지금과 같은 시스템도 좋지만, **구축하는 과정이 너무 비효율적**이었다. 
  - 하나, 하나, 이미지를 빌드하고 컨테이너로 실행하는 경험은 그리 달갑진 않았다. 
- 당연히 이를 개선하기 위한 방법이 있다. 
  - 다음 챕터에서는 ==Docker Compose==에 대해 학습한다. (그 전에 Docker Hub 관련 강의가 먼저 있다)
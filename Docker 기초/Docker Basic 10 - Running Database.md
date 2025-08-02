#  Docker Basic 10 - Running Database

#dev/skill/container/docker

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/45161619#overview)

## 개요
> 이번 장에서는 docker container 내부에서 실행할 database를 실습하자.

![](Docker%20Basic%2010%20-%20Running%20Database/image.png)

가장 앞 단에는 `flask-app` container가 있으며, 해당 container는 `node-app`을 호출한다. 그리고 다시 `node-app`은 `Mongo DB`를 참조한다.

## 요구사항 확인
> lesson-starter-projects/09-starter-code/grade-submission-api/app.js
```js
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const mongoose = require('mongoose');

const app = express();
app.use(bodyParser.json());
app.use(cors());

// Connect to MongoDB
mongoose.connect(`mongodb://${process.env.DB_HOST}:${process.env.DB_PORT}/${process.env.DB_NAME}`, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Define Grade schema
const gradeSchema = new mongoose.Schema({
  name: String,
  subject: String,
  score: Number,
});

// Create Grade model
const Grade = mongoose.model('Grade', gradeSchema);

app.get('/grades', async (req, res) => {
  console.log('Received GET request for grades');
  const grades = await Grade.find();
  res.json(grades);
});

app.post('/grades', async (req, res) => {
  const { name, subject, score } = req.body;
  const newGrade = new Grade({ name, subject, score });
  await newGrade.save();
  console.log('Received POST request, added new grade:', newGrade);
  res.json(newGrade);
});

const port = 3000;
app.listen(port, () => {
  console.log(`Grade service is running on port ${port}`);
});

// Developer: Captain DevOps, I'm counting on you to deploy a MongoDB instance alongside this app.
//            The app uses environment variables (DB_HOST, DB_PORT, DB_NAME) to connect to the database.
//            I trust your deployment skills to ensure a smooth sailing for this app!
```
위 소스 코드에서 알 수 있듯, 해당 `node-app`은 환경변수에 정의된 `DB_HOST`, `DB_PORT`, `DB_NAME`을 참조하고 있다. 

## Docker build & push
> lesson-starter-projects/09-starter-code/grade-submission-api/Dockerfile
-> 그리고 dockerfile 이미 작성되어 있기 때문에, 해당 위치로 이동해 바로 docker build를 진행한다. 
```dockerfile
FROM node:14

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]
```
```sh
docker build -t lj7812/grade-submission-api:latest .
[+] Building 22.4s (11/11) FINISHED
...
docker push lj7812/grade-submission-api:latest 
The push refers to repository [docker.io/lj7812/grade-submission-api]
...
latest: digest: sha256:81302ee41064fc383bdf8506434612057a6822fa6be369c41c9a63c17bba480d size: 856
...
```

> lesson-starter-projects/09-starter-code/grade-submission-portal/Dockerfile
```dockerfile
FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5001

CMD ["python", "app.py"]
```
```sh
docker build -t lj7812/grade-submission-portal:latest .
[+] Building 8.0s (11/11) FINISHED
...
docker push lj7812/grade-submission-portal:latest
The push refers to repository [docker.io/lj7812/grade-submission-portal]
...
latest: digest: sha256:18dc76ce2b876784da4ca2b2a9edf8e62580777aeebbf404012c053a2b9885cc size: 856
```
그리고 난 학습이 끝난 repository는 docker hub에서 모두 삭제했기 때문에, grade-submission-portal도 다시 push를 진행한다.

### docker-compose.yml 확인
```yml
version: '3'
services:
  flask-app:
    image: rslim087/grade-submission-portal
    container_name: flask-app
    ports:
      - "5001:5001"
    environment:
      - GRADE_SERVICE_HOST=node-server
    depends_on:
      - node-server

  node-server:
    image: rslim087/grade-submission-api:2.0.0
    container_name: node-server
    ports:
      - "3000:3000"
```

## 데이터베이스 정의하기
이제 `docker-compose`를 실행하기 전에, `node-app`에서 사용할 ==Mongo DB==를 컨테이너로 실행할 필요가 있다. 기존의 docker-compose.yml 파일을 수정하자! (`container_name` 등을 생략하고 싶지만, 일단 강의를 따라서 그대로 진행한다)
```yml
version: '3'
services:
  flask-app:
    image: rslim087/grade-submission-portal
    container_name: flask-app
    ports:
      - "5001:5001"
    environment:
      - GRADE_SERVICE_HOST=node-server
    depends_on:
      - node-server

  node-server:
    image: rslim087/grade-submission-api:2.0.0
    container_name: node-server
    environment: 
      - DB_HOST=mongo-db
      - DB_PORT=27017
      - DB_NAME=grade_db
    ports:
      - "3000:3000"

  mongo-db:
    image: mongo
    container_name: mongo-db
    environment: 
      - MONGO_INITDB_DATABASE=grade_db
    volumes:
      - mongo-db-data:/data/db

volumes:
  mongo-db-data:
```
수정한 항목은 다음과 같다.
- mongo-db 서비스 추가
  - 환경변수 설정
  - 볼륨 마운트
- volumes 추가
- node-server 수정
  - 환경변수 설정 -> DB 정보

**Database service**를 정의할 때 중요한 것 중 하나는 `volumes`이다. `container`는 언제든 중지 또는 삭제, 재실행될 수 있지만 우리는 `data`의 영속성(**persistence**)을 지킬 필요가 있다. 그렇기에 `data`가 어디에 저장되어야 하는지 명시해야 한다. 
![](Docker%20Basic%2010%20-%20Running%20Database/image%202.png)<!-- {"width":616} -->

## Docker Compose UP!
자 이제 전체 서비스를 실행해보자.
```sh
docker compose up -d
[+] Running 27/27
 ✔ flask-app Pulled
...
 ✔ node-server Pulled
...
 ✔ mongo-db Pulled
...
[+] Running 5/5
 ✔ Network 09-starter-code_default         Created                                                                                                                                                             
 ✔ Volume "09-starter-code_mongo-db-data"  Created                                                                                                                                                             
 ✔ Container mongo-db                      Started                                                                                                                                                             
 ✔ Container node-server                   Started                                                                                                                                                             
 ✔ Container flask-app                     Started
```
![](Docker%20Basic%2010%20-%20Running%20Database/image%203.png)
![](Docker%20Basic%2010%20-%20Running%20Database/image%204.png)
이렇게 로그를 보기엔 서비스가 정상적으로 올라온 것처럼 보인다. 자 이제 실제로 데이터를 생성하고 조회해보자.

실수로 `score`에 과목 이름을 적고, `subject`에 점수(숫자)를 적었더니 `ValidationError`가 발생하고 `application`이 응답하지 않았다. 일단 application의 기능을 검증하는 시간이 아니니까 넘어가자. 😅
```log
(node:1) UnhandledPromiseRejectionWarning: ValidationError: Grade validation failed: score: Cast to Number failed for value "cs" (type string) at path "score"
    at model.Document.invalidate (/app/node_modules/mongoose/lib/document.js:3197:32)
    at model.$set (/app/node_modules/mongoose/lib/document.js:1473:12)
   at model.$set (/app/node_modules/mongoose/lib/document.js:1151:16)
```

`form`에서 몇 가지 입력 테스트를 완료했다. 이제 서버를 내렸다가 다시 올려도 데이터가 살아있는지 확인하면 되겠다.
![](Docker%20Basic%2010%20-%20Running%20Database/image%205.png)<!-- {"width":703} -->

### docker compose down...!
```sh
docker compose down
[+] Running 4/4
 ✔ Container flask-app              Removed                                                                                                                                                                   
 ✔ Container mongo-db               Removed                                                                                                                                                                   
 ✔ Container node-server            Removed                                                                                                                                                                  
 ✔ Network 09-starter-code_default  Removed
...
docker volume ls
DRIVER    VOLUME NAME
local     8e49bf475365cebf8a3671eec143ab8d546c250dbf5af6722ce26f66c7d17d66
local     09-starter-code_mongo-db-data
...
```
이전 챕터에서 오류가 있어서 초기화 할때는 `-v` 옵션까지 넣어서 볼륨도 초기화했다. 지금은 그러면 **안 된다**! 자, 결과를 보면 container와 network까지 모두 제거되었다. 하지만 volume은 살아있는 것을 볼 수 있다. 좋다, 이제 다시 컨테이너들을 실행하자.

### docker compose up!!!
```sh
docker compose up -d
[+] Running 4/4
 ✔ Network 09-starter-code_default  Created                                                                                                                                                                   
 ✔ Container node-server            Started                                                                                                                                                                   
 ✔ Container mongo-db               Started                                                                                                                                                                   
 ✔ Container flask-app              Started
...
```
`-d` 옵션은 `detached mode`라고 해서 백그라운드로 실행하는 것이다. 자꾸 터미널 입력이 불편해져서 사용하고 있다.

그리고 다시 `Flask-app`(localhost:5001/grades)으로 접근하면 서버를 다시 올렸음에도 불구하고 `data`가 모두 살아있는 것을 볼 수 있다. 
![](Docker%20Basic%2010%20-%20Running%20Database/image%206.png)

## 정리
> 이번 챕터에서는 database를 컨테이너로 실행하는 것을 학습했다.
- Mongo DB containerize
- docker-compose.yml 데이터베이스 명시하는 방법
  - 데이터 영속성을 위한 volumes
  - 데이터베이스 설정을 위한 environment
  - db를 사용하는 다른 서비스에서 db 정보에 대한 environment


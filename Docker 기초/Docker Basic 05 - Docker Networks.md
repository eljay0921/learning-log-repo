#  Docker Basic 05 - Docker Networks

#dev/skill/container/docker

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/45162531#overview)<!-- {"preview":"true"} -->

## 개요
이번 장에서는 Docker 컨테이너 간 네트워크에 대해서 알아보자. Flask Application(portal)과 Node.js Application(api)을 각각 도커 컨테이너로 실행하고, 이들 간에 통신이 가능하도록 실습할 예정이다.
![](Docker%20Basic%2005%20-%20Docker%20Networks/image.png)

## 요구사항 확인하기 (node.js, api)
flask와 node 애플리케이션이 각각 어떤 요구사항을 가지고 있는지 확인하자.
> lesson-starter-projects/07-starter-code/grade-submission-api/app.js
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');

const app = express();
app.use(bodyParser.json());
app.use(cors());

let grades = [];

app.get('/grades', (req, res) => {
  console.log('Received GET request for grades');
  res.json(grades);
});

app.post('/grades', (req, res) => {
  const { name, subject, score } = req.body;
  const id = Date.now().toString();
  const newGrade = { id, name, subject, score };
  grades.push(newGrade);
  console.log('Received POST request, added new grade:', newGrade);
  res.json(newGrade);
});

const port = 3000;
app.listen(port, () => {
  console.log(`Grade service is running on port ${port}`);
});


// Developer: Ahoy, Captain DevOps! To get this Node.js API up and running:
//
// First, install the dependencies listed in package.json:
//   npm install
//
// Then, start the API server with:
//   node app.js
//
// The dependencies in package.json are crucial for this API to function properly.
// Once they're installed, you can set sail with the API!
```
위 내용을 살펴보면 `package.json`에 명시된 `dependencies`를 설치할 필요가 있으며, 이는 `npm install`로 실행할 수 있다고 한다. 그리고 application을 실행하는 명령어는 `node app.js`이다.

## Dockerfile (node.js, api) 작성하기
위 요구사항을 참고해 dockerfile을 작성하자. 참고로 이 application의 프로젝트 구조는 다음과 같다.
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%202.png)<!-- {"width":396} -->
```dockerfile
## Base Image
FROM node:14

## Working Directory
WORKDIR /app

## Install Dependencies
COPY package*.json . 
RUN npm install

## Copy Source code
COPY . . 

## Expose container port
EXPOSE 3000

## Commands to execute application
CMD ["node", "app.js"]
```

[앞선 학습](bear://x-callback-url/open-note?id=89395A71-E1BD-4B78-944A-EA245CAEE4E6)에서 **이미지 빌드를 최적화**하는 방법을 익혔다. 이번에도 종속성을 "먼저" 설치할 수 있도록 `package.json`을 옮긴 후 **RUN**을 통해 `npm install`을 진행하도록 정의했다. 이후 다시 **COPY** 명령을 통해 application의 소스 코드를 모두 복사하는데, 이때 불필요한 것들을 제외하기 위해 `.dockerignore` 파일을 생성했다. 특히 `node_modules`에 있는 `dependencies`는 모두 이미지 빌드 과정에서 미리 설치할 예정이므로 **반드시 제외**할 필요가 있다.
```dockerfile
node_modules
Dockerfile
README.md
```

## Docker build & run (node.js, api)
이제 node.js application을 위한 이미지를 생성하고 컨테이너를 실행해 보자.
```sh
docker build -t node-app-api .
...
docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
node-app-api   latest    2e20718db5b3   46 seconds ago   1.3GB
...
docker run --rm -p 3001:3000 --name node-app-api-container node-app-api
Grade service is running on port 3000
```
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%203.png)<!-- {"width":544} -->
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%204.png)<!-- {"width":609} -->
node.js 서버가 정상적으로 실행되었으며, `localhost:3001`로 접근되는 것까지 확인했다. 이제 이 컨테이너를 유지한 상태에서 **Flask application** 서버 이미지를 생성하고 컨테이너로 실행하자.

## 요구사항 확인하기 (flask, portal)
이번에는 flask 애플리케이션이 어떤 요구사항을 가지고 있는지 확인하자.
> lesson-starter-projects/07-starter-code/grade-submission-portal/app.py
> lesson-starter-projects/07-starter-code/grade-submission-portal/requirements.txt
```python
import requests
import os
from flask import Flask, render_template, request, redirect, url_for, jsonify

app = Flask(__name__, template_folder='templates', static_folder='static')

GRADE_SERVICE_HOST = os.environ.get('GRADE_SERVICE_HOST', 'localhost')
GRADE_SERVICE_URL = f'http://{GRADE_SERVICE_HOST}:3000/grades'

@app.route('/', methods=['GET'])
def get_form():
    return render_template('form.html')

@app.route('/handleSubmit', methods=['POST'])
def submit_form():
    form_data = request.form
    grade_data = {
        'name': form_data['name'],
        'subject': form_data['subject'],
        'score': form_data['score'],
    }
    requests.post(GRADE_SERVICE_URL, json=grade_data)
    return redirect(url_for('get_grades'))

@app.route('/grades', methods=['GET'])
def get_grades():
    return render_template('grades.html')

@app.route('/api/grades', methods=['GET'])
def api_get_grades():
    response = requests.get(GRADE_SERVICE_URL)
    grades = response.json()
    print(f"Received grades from Node.js server: {grades}")
    return jsonify(grades)

if __name__ ** '__main__':
    app.run(debug=True, host='0.0.0.0', port=5001)

# Developer: Ahoy, Captain DevOps! This Flask app is quite straightforward, but there's a catch:
#
# I typically run it with the Node.js API on my local machine, so I set the GRADE_SERVICE_HOST 
# environment variable to 'localhost'. But I'm just a programmer, and I wouldn't know the first 
# thing about setting it up in containers, the cloud, or any of those fancy environments.
#
# That's where you come in! I trust you to work your devops magic and ensure this app can talk 
# to the Node.js API, no matter where they're running. You'll need to set the GRADE_SERVICE_HOST 
# variable to whatever it needs to be for the app to find its way.
#
# I'll leave it in your capable hands, Captain DevOps. Make this app sail smoothly!
```
```txt
Flask**2.0.1
Werkzeug**2.0.1
requests**2.26.0
```

다소 현실적인(?) 요구사항이 작성되어있다. 개발자가 자신은 `GRADE_SERVICE_HOST` 환경변수에 `localhost`를 박아서 사용한다고 언급한다. 이걸 알아서 수정 후 배포해달란 요구사항이다. 그리고 종속성 설치에 관한 명령어도 명시되어있지 않다. (...)
## Dockerfile 작성하기 (flask, portal)
일단 dockerfile을 작성해보자
```dockerfile
## Base Image
FROM python:3.8-slim

## Work Directory
WORKDIR /app/

## Install requirements
COPY requirements.txt . 
RUN pip install -r requirements.txt 

## Copy source code
COPY . . 

## Expose container port
EXPOSE 5001

## Execute Application
CMD ["python", "app.py"]
```

## Docker build & run (flask, portal)
```sh
docker build -t flask-app-portal .
...
docker images
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
flask-app-portal   latest    be04cb9e2485   38 seconds ago   239MB
node-app-api       latest    2e20718db5b3   29 minutes ago   1.3GB
...
docker run --rm -p 5002:5001 --name flask-app-portal-container flask-app-portal
 * Serving Flask app 'app' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.17.0.3:5001/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 114-167-061
```
다행히 의도한 대로 build 및 run이 정상 동작했다. localhost:5002로 접근했더니 다음과 같이 Flask Application으로 서비스하는 페이지를 확인할 수 있었다.
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%205.png)

지금까지 진행한 결과로 2개의 컨테이너가 실행 중인 상태이다.
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%206.png)

이후 Flask Portal 페이지를 다음과 같이 테스트 해봤는데, 역시나 예상대로 오류가 발생한다.
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%207.png)<!-- {"width":283} --> ![](Docker%20Basic%2005%20-%20Docker%20Networks/image%208.png)<!-- {"width":615} -->

## 오류 해결하기 (flask, portal)
자, 오류를 해결하자. 앞선 [요구사항](bear://x-callback-url/open-note?id=FE52FF97-6E91-41EC-BE3A-F52E82406606&header=%EC%9A%94%EA%B5%AC%EC%82%AC%ED%95%AD%20%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0%20%28flask,%20portal%29)을 다시 확인하면, 환경 변수 `GRADE_SERVICE_HOST`에 대한 언급이 있다. 이 점을 고려해 docker run을 다시 실행해 보자. 
```sh
docker run --rm -p 5002:5001 --name flask-app-portal-container -e GRADE_SERVICE_HOST=node-app-api-container flask-app-portal
...
```

서버가 정상적으로 올라오지만 여전히 같은 동작은 실패하고 있다. 환경 변수 `GRADE_SERVICE_HOST`는 반영되었지만 통신에 실패한다. 
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%209.png)<!-- {"width":845} -->
-> 이는 사실 각 **Docker 컨테이너가 서로 격리된 환경**이기 때문이다. Flask application이 실행 중인 컨테이너에는 node.js application이 실행되고 있지 않다. 서로 다른 컨테이너다.

### Docker Network 만들기
이를 위해 각 컨테이너가 서로 통신할 수 있는 환경을 구성해야 한다. 그게 바로 `docker network` 명령어다. 우선, 실행 중인 컨테이너를 모두 종료하고 진행하자.
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%2010.png)

이후, `docker network`를 생성하자. 이번에 network를 처음 생성했지만, 기본적으로 생성되어있는 것으로 보이는 네트워크들이 보인다. 
```sh
docker network create my-network
...
docker network ls
NETWORK ID     NAME         DRIVER    SCOPE
46a4f2babf8b   bridge       bridge    local
255a0299a3d4   host         host      local
d400de4322a8   my-network   bridge    local
a2cd7e6dcbf7   none         null      local
```

이제,`docker run` 과정에서 `network`를 명시한다.
```sh
docker run --rm -p 5002:5001 --network my-network -e GRADE_SERVICE_HOST=node-app-api-container --name flask-app-portal-container flask-app-portal
Grade service is running on port 3000
```
```sh
docker run --rm -p 5002:5001 --network my-network --name flask-app-portal-container flask-app-portal
 * Serving Flask app 'app' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.18.0.3:5001/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 120-391-424
```

이후 다시 flask app을 테스트하기 위해 `localhost:5002`로 접근해, 다음과 같이 값을 입력하고 `submit`을 진행했고, 결과는 **성공**이다.
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%2012.png)<!-- {"width":351} --> ![](Docker%20Basic%2005%20-%20Docker%20Networks/image%2011.png)<!-- {"width":567} -->
또한 node app을 확인하기 위해 localhost:3001로 접근해, /grades API를 호출하면 다음과 같이 나타난다.
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%2016.png)<!-- {"width":639} -->

다음은 각각 flask app과 node app에 찍히고 있는 로그 상황이다.
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%2013.png)
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%2014.png)

#### Docker network 최종 모습
같은 docker network에 속한 2개의 컨테이너의 모습이다.
![](Docker%20Basic%2005%20-%20Docker%20Networks/image%2015.png)

---

## 정리
> Dockerize와 함께 Docker container 간의 네트워크 통신에 대해 학습했다.
- 기본적으로 `Docker container`는 개별로 격리된 환경이다.
- 그러므로 **같은 네트워크 안**에 두어야 다른 컨테이너를 바라볼 수 있다.
- 이를 위해 `docker network create {네트워크 이름}`으로 네트워크를 생성하고, 
  - `docker run` 명령 시, `--network {네트워크 이름}`을 입력해 해당 네트워크 내부에서 컨테이너가 실행될 수 있게 한다.


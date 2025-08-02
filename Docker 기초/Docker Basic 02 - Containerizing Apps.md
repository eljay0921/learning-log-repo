# Udemy Docker Basic 02 - Containerizing Apps

#dev/skill/container/docker 

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/43367418#overview)

## 개요
앞선 내용과 비슷하다. 각각의 애플리케이션을 `Docker`를 통해 `컨테이너화(Containerize)` 해보자.

## 명령 인자 전달하기
> 지금까지 단순히 애플리케이션을 실행했다면, 이번에는 파라미터를 같이 넘겨보자.

### Python
1. 먼저 어떤 애플리케이션이 대상인지 확인하자. 다음과 같은 `Python` 코드가 있다.
   -> 소스 코드를 보면 입력받은 파라미터를 출력하고 있다.
   ```python
   import sys
   
   if __name__ == '__main__':
       if len(sys.argv) < 2:
           print("Please provide at least one command-line argument.")
       else:
           print("Command-line arguments:")
           for arg in sys.argv[1:]:
               print(arg)
   
   ```
2. 이를 Docker로 실행하기 위한 명령어를 작성해 보자
   - `docker run --rm -v "/Users/.../sources-lectures/docker-course-remastered-main/workbook-starter-projects/01-starter-projects/python-script:/app/" --name python-script-container python:3.8-slim python /app/script.py Hello Docker Python!`
   - 실행 결과 -> `python 3.8 slim` 이미지를 내려받고, 애플리케이션 실행에 성공했다.
     ![](Docker%20Basic%2002%20-%20Containerizing%20Apps/image.png)

### Go
1. 자 이번에는 `Go`로 작성된 파일이다. 
   -> 이 애플리케이션을 위해 우리는 `MESSAGE`라는 환경변수를 전달할 필요가 있다.
   ```go
   package main
   
   import (
       "fmt"
       "os"
   )
   
   func main() {
       message := os.Getenv("MESSAGE")
       if message == "" {
           message = "Hello, World!"
       }
       fmt.Println(message)
   }
   ```
2. 마찬가지로 Docker 명령어를 작성하자
   - `docker run --rm -v "/Users/.../sources-lectures/docker-course-remastered-main/workbook-starter-projects/01-starter-projects/go-environment-variables:/app/" -e MESSAGE="Hi there\!" --name go-application golang:1.16 go run /app/main.go`
   - 옵션으로 `-e`를 사용했으며, 환경변수의 `name`에 해당하는 =="MESSAGE"==에 값을 입력했다.
   - 강의와 동일하게 작성했는데, 따옴표(")에 대한 이스케이프에 문제가 있는지 엉뚱한 동작을 한다. (dquote) 
     - 원인은 느낌표(!) 때문에 따옴표가 제대로 닫히지 않은 것. 느낌표에 대해 이스케이프('\\') 처리 후 성공했다.
   - 실행 결과
     ![](Docker%20Basic%2002%20-%20Containerizing%20Apps/image%202.png)

## 정리
- Docker 명령 인자에 대해 알아보았다.
- docker run
  - --rm : run 이후, 도커 컨테이너가 종료되면 바로 삭제
  - -v : 작업 공간 마운트
  - -e : 환경변수 입력
  - --name : 컨테이너 이름 지정
  - *(아래는 이후 챕터에서 배우게 된다)*
  - -p : 포트 매핑
  - --network : 도커 네트워크 지정

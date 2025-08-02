#  Docker Basic 07 - Docker Hub

#dev/skill/container/docker

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요](https://www.udemy.com/course/docker-training-learn-docker-from-zero-to-cloud/learn/lecture/42827656#overview)

## 개요
![](Docker%20Basic%2007%20-%20Docker%20Hub/image.png)<!-- {"width":460} -->
> 이번 챕터에서는 Docker Hub를 관리하는 기초를 학습한다.

## Docker Hub Push
> github에 소스 코드를 업로드 하듯이, dockerhub repository에 이미지를 올려보자.
강의에서는 갑자기 엉뚱한 이미지를 가지고 Docker Hub에 push하는 설명을 하고 있다. 아마 강의가 중간에 수정된 것 같다. 일단 지금까지 진행했던 image 중 하나로 실습해보자. -> Docker Compose 챕터를 하면서 다시 보니 `08-starter-code`를 이용하기 위해서 미리 push했던 것으로 보인다. 
```sh
docker images
REPOSITORY              TAG       IMAGE ID       CREATED        SIZE
order-mgmt              latest    8584635a80ac   6 hours ago    822MB
product-catalog-app     latest    f5f9310a2477   6 hours ago    1.3GB
product-inventory-app   latest    1f43b5e44c10   6 hours ago    234MB
contact-support-app     latest    1c2e678a2458   7 hours ago    234MB
ship-handle-app         latest    1ab84c183859   7 hours ago    1.26GB
profile-mgmt            latest    15f2c874c63d   10 hours ago   1.3GB
ecommerce-ui            latest    24516bdcf614   10 hours ago   2.12GB
```
비교적 용량이 작은 `product-inventory-app`을 가지고 진행하겠다.

### Docker Hub 확인
그 전에 내 Docker Hub 계정 상태는 어떤지 확인해보자. 이미 4년 전에 난 비슷한 실습을 했던 것 같다. ~~그때 끝내놓을 것을 지금 왜 다시 하고 있는가... 어휴.~~ 아무튼 경고 메시지를 보니 너무 오래된 이미지라서 archiving할 것을 권유하고 있다. 너무 오래되었으니 삭제하자.
![](Docker%20Basic%2007%20-%20Docker%20Hub/image%202.png)
간단하다. 해당 이미지의 Settings로 진입한 뒤 **Delete repository**를 수행하면 된다. (`DELETING` 상태가 되며 일정 시간 뒤 삭제된다)

### Push image
자 이제 다시 본 학습으로 돌아와 로컬에 있던 이미지를 아무거나 올려보자. `product-inventory-app`로 진행하겠다. 그런데 강의에서 다짜고짜 docker push를 진행하는 바람에 제대로 따라할 수 없었다. 다음 오류와 함께 push할 수 없었는데, **태깅**을 먼저 해야하는 것으로 확인된다. 
```sh
docker push lj7812/product-inventory-app
Using default tag: latest
The push refers to repository [docker.io/lj7812/product-inventory-app]
tag does not exist: lj7812/product-inventory-app:latest
```

## Docker Hub Push (feat. chatGPT)
> 지금부터는 chatGPT를 통해 학습한 내용을 남긴다.

### docker login
먼저, `gpt`는 `docker login`부터 설명하고 있다. 그런데 나는 현재 docker desktop을 이용하고 있어 이미 해당 애플리케이션을 통해 로그인이 되어있다. 하지만 그냥 명령어를 따라 해보자. 
```sh
docker login
Authenticating with existing credentials... [Username: lj7812]
...
Login Succeeded
```
잠시 기다리니 `Login Succeeded` 메시지가 나왔다.

### docker tag
이번에는 **이미지 태깅**이다. 이미지를 업로드하기 전 저장소 경로를 지정하는 과정이라 볼 수 있다. 나는 현재 로컬의 이미지가 `latest`라서 생략했는데, 이것도 지정할 수 있다.
```sh
docker tag product-inventory-app lj7812/my-product-inventory-app:1.0.0
...
docker images
product-inventory-app             latest    1f43b5e44c10   7 hours ago    234MB
lj7812/my-product-inventory-app   1.0.0     1f43b5e44c10   7 hours ago    234MB
...
```
`tag `명령 이후에는 아무런 메시지가 없는데, `docker images`를 확인하니 docker 계정 경로가 지정된 이미지가 확인되었다. 단순히 태깅만 된 것으로, 이미지 생성 시간은 기존과 동일한 것을 볼 수 있다.

### docker build
태깅하지 않고 빌드 과정에서도 저장소 경로를 지정할 수 있다. 앞선 예제에서 `1.0.0`으로 적용했으니, 이 작업은 `1.0.1` 태그를 적용해보자.
```sh
docker build -t lj7812/my-product-inventory-app:1.0.1 .
[+] Building 2.5s (11/11) FINISHED
...
docker images
lj7812/my-product-inventory-app   1.0.1     e269172a6b7a   7 hours ago    234MB
product-inventory-app             latest    1f43b5e44c10   7 hours ago    234MB
lj7812/my-product-inventory-app   1.0.0     1f43b5e44c10   7 hours ago    234MB
...
```
다시 빌드한 것임에도 불구하고 이미지 생성 시간은 여전히 기존 이미지와 동일하다. 아무래도 캐싱되어서 그런 것으로 추정된다.

### docker push
자, 드디어 push 단계다. 이전에 강사가 했던 것처럼 push를 진행해보자.
```sh
docker push lj7812/my-product-inventory-app:1.0.0
The push refers to repository [docker.io/lj7812/my-product-inventory-app]
d847ad1879f2: Pushed
56d45b8e1404: Pushed
5415c047373c: Pushed
c55628893f94: Pushed
0cd45dda32d5: Pushed
f27eb4094907: Pushed
c6481a180440: Pushed
f8497603fb88: Pushed
14c9d9d19932: Pushed
1.0.0: digest: sha256:1f43b5e44c10f53ee1fff7fee806c57c682b633e3910d93d0c305ff0c81f82e1 size: 856
...
```
`1.0.0` 태그를 푸시했으며, 약 25초 정도 소요된 후 완료되었다. Docker Hub에서도 확인된다.
![](Docker%20Basic%2007%20-%20Docker%20Hub/image%203.png)

두 번째 태그인 `1.0.1` 버전을 push 해봤다. 이미 존재하는 Layer라는 메시지와 함께 약 6초만에 완료되었다.
```sh
docker push lj7812/my-product-inventory-app:1.0.1
The push refers to repository [docker.io/lj7812/my-product-inventory-app]
c55628893f94: Layer already exists
d847ad1879f2: Layer already exists
fea11848d8da: Pushed
f27eb4094907: Layer already exists
56d45b8e1404: Layer already exists
c6481a180440: Layer already exists
0cd45dda32d5: Layer already exists
14c9d9d19932: Layer already exists
5415c047373c: Layer already exists
1.0.1: digest: sha256:e269172a6b7a3ca9afb0097f10d9ea27d537f0fe1170439e7671ef736b5cb4e5 size: 856
```
![](Docker%20Basic%2007%20-%20Docker%20Hub/image%204.png)

## 정리
> Docker Hub로 저장소와 이미지를 관리하는 방법을 학습했다.
- 먼저 docker login이 필요하다.
- docker tag로 이미지를 태깅한다.  -> `docker tag {이미지명}:{태그} {계정명}/{저장소명}:{태그}`
  - 아니면, 빌드 과정에서 바로 태깅할 수도 있다. -> `docker build -t {계정명}/{저장소명}:{태그}`
- docker push로 Docker Hub에 업로드한다. -> `docker push {계정명}/{저장소명}:{태그}`


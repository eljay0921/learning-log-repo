# Docker Basic 12 - Build for Multi Architectures

#dev/skill/container/docker

## 강의
강의에서도 짧게 설명하는 부분이긴 하나, 설명이 조금 부족하기 때문에 본 내용은 내가 따로 서치한 것과 chatGPT를 이용해 학습한 것들을 기반으로 작성한다.

## 개요
요즘 세상엔 다양한 하드웨어 아키텍처가 있다. 그리고 우린 그런 환경에 대응할 필요가 있다.

## 아키텍처(플랫폼)
### x86:
> 인텔이 1978년에 개발한 8086 마이크로프로세서에서 시작된 아키텍처로, 32비트 프로세서입니다. x86 아키텍처는 8086 이후의 후속 모델들 (80286, 80386, 80486 등)과 호환성을 유지하며 발전해 왔습니다.
### amd64 (x86-64, x64):
> AMD에서 개발한 64비트 아키텍처로, x86 아키텍처를 확장한 것입니다. 인텔에서도 이 아키텍처를 라이센스하여 사용하고 있으며, 현재 대부분의 데스크톱 및 서버에서 사용되고 있습니다.
### arm64 (AArch64):
> ARM 아키텍처 기반의 64비트 프로세서입니다. ARM은 주로 모바일 기기나 임베디드 시스템에서 사용되었지만, 최근에는 데스크톱 및 서버 시장에서도 점차 사용량이 늘고 있습니다.

이 외에 다양한 특징과 차이점이 존재하지만 이번 챕터는 아키텍처를 알아보는 내용은 아니니 여기까지만 정리한다. 어쨌거나 이 정도의 아키텍처 종류가 있는 만큼 **우리는 다양한 아키텍처를 지원할 수 있는 docker image를 build해서 사용할 필요**가 있다. 

### 왜?
단적인 예를 들어, 어떤 application의 `docker image`를 자신의 `local` 환경에서 작업하던 개발자가 `AWS EC2`와 같은 클라우드 서버에 해당 이미지를 배포했을 때 서로 아키텍처가 달라 호환되지 않을 수도 있는 것이다. 

그럼, 현재 내 시스템인 `Macbook Air M4`는 어떤 아키텍처일까?
```sh
uname -m
arm64
```
보다시디 `arm64`인 것을 알 수 있다. **M 시리즈**로 불리는 Apple Silicon 칩은 대부분 `arm64`인 것으로 알고 있다. 

그럼 `AWS`의 `EC2`는 어떤 아키텍처일까? **당연하게도 이미지마다 다르다.** `x86_64`일 수도, `arm64`일 수도 있다. 그렇다고 개발자의 환경과 동일한 서버만 골라 배포할 수도 없는 것이고, 설사 그게 가능하다 한들 사내 모든 개발자의 환경(아키텍처)이 같을 리도 없다. 그러므로 우리는 어떤 아키텍처에서도 호환될 수 있는 이미지를 빌드하는 것이 필요하다. 

## 테스트
자, 그럼 내 환경(arm64)에서 build한 이미지는 어떤 아키텍처 타입일까? 테스트를 해보자. 
```sh
docker build -t lj7812/build-test . --push
[+] Building 32.7s (12/12) FINISHED
...
```
그리고 docker hub에서 위 이미지를 확인해보면 다음과 같이 OS/ARCH 항목을 확인할 수 있다. 역시나 당연하게도, 결국 **개발자(작업자)의 local 환경에 따른 이미지가 생성된 것**이다. 
![](assets/Docker%20Basic%2012%20-%20Build%20for%20Multi%20Architectures/image%202.png)

그리고 arm64로 빌드된 이 이미지를, amd64 아키텍처 환경에서 실행(docker run)할 경우 다음과 같은 오류가 발생하며 실행되지 않는다. 
```sh
WARNING: The requested image's platform (arm64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested ...
```
이건 내가 직접 해본 것은 아니고 인터넷에서 서치한 결과다. (~~귀찮...~~)

## docker buildx
자, 그래서 어떻게 다양한 아키텍처를 지원할 수 있을까? 
그건 `docker cli`에 내장된 `docker buildx`를 이용하면 가능하다. `docker buildx`는 `docker 19`버전부터 포함된 기능으로, 하나의 `Dockerfile`로 **여러 아키텍처(및 플랫폼)를 지원하는 이미지를 만들 수 있게** 도와주는 기능이다. 

### docker buildx 확인 및 생성
먼저 `docker buildx ls`로 현재 어떤 빌더가 있는지 확인할 수 있다.
```sh
docker buildx ls
NAME/NODE           DRIVER/ENDPOINT     STATUS    BUILDKIT   PLATFORMS
default             docker
 \_ default          \_ default         running   v0.23.2    linux/amd64 (+2), linux/arm64, linux/ppc64le, linux/s390x, (2 more)
desktop-linux*      docker
 \_ desktop-linux    \_ desktop-linux   running   v0.23.2    linux/amd64 (+2), linux/arm64, linux/ppc64le, linux/s390x, (2 more)
```

또한 `docker build inspect`로 어떤 아키텍처를 지원하는지 확인할 수 있다.
```sh
docker buildx inspect
Name:          desktop-linux
Driver:        docker
Last Activity: 2025-08-05 01:01:14 +0000 UTC

Nodes:
Name:             desktop-linux
Endpoint:         desktop-linux
Status:           running
BuildKit version: v0.23.2
Platforms:        linux/arm64, linux/amd64, linux/amd64/v2, linux/riscv64, linux/ppc64le, linux/s390x, linux/386
Labels:
 org.mobyproject.buildkit.worker.containerd.namespace: moby
 org.mobyproject.buildkit.worker.containerd.uuid: 
...
```

그리고, 우리의 목적인 멀티 플랫폼(아키텍처)을 지원하기 위한 커스텀 빌더를 생성할 수 있다.
```sh
docker buildx create --name multi-builder --use
multi-builder
...
docker buildx ls
NAME/NODE            DRIVER/ENDPOINT     STATUS     BUILDKIT   PLATFORMS
multi-builder*       docker-container
 \_ multi-builder0    \_ desktop-linux   inactive
default              docker
 \_ default           \_ default         running    v0.23.2    linux/amd64 (+2), linux/arm64, linux/ppc64le, linux/s390x, (2 more)
desktop-linux        docker
 \_ desktop-linux     \_ desktop-linux   running    v0.23.2    linux/amd64 (+2), linux/arm64, linux/ppc64le, linux/s390x, (2 more)
```
![](assets/Docker%20Basic%2012%20-%20Build%20for%20Multi%20Architectures/image.png)
그런데 다른 `builder`들과 달리 status는 inactive이며, platforms 항목들이 비어있다. 이를 해결하기 위해 `--bootstrap`으로 활성화할 필요가 있다.
```sh
docker buildx inspect multi-builder --bootstrap
[+] Building 6.6s (1/1) FINISHED
 => [internal] booting buildkit                                                                                                                                                                           
 => => pulling image moby/buildkit:buildx-stable-1                                                                                                                                                        
 => => creating container buildx_buildkit_multi-builder0                                                                                                                                                  
Name:          multi-builder
Driver:        docker-container
Last Activity: 2025-08-05 01:38:53 +0000 UTC

Nodes:
Name:                  multi-builder0
Endpoint:              desktop-linux
Status:                running
BuildKit daemon flags: --allow-insecure-entitlement=network.host
BuildKit version:      v0.22.0
Platforms:             linux/arm64, linux/amd64, linux/amd64/v2, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
Labels:
...
```
이제야 다른 빌더들과 같이 여러 가지 Platforms 항목들이 확인된다. 이제 이 커스텀 빌더인 `multi-builder`를 사용하면 `--platform` 옵션으로 여러 아키텍처를 지원할 수 있다.

### docker buildx 빌드하기
자, 이제 여러 아키텍처를 지원하는 이미지를 빌드해보자.
```sh
docker buildx build \
-t lj7812/buildx-test \
--platform linux/arm64,linux/amd64 \
--push \
. 
[+] Building 40.5s (18/18) FINISHED
...
```
빌드 과정을 보면 다음과 같이 amd64, arm64에 대한 로그가 확인된다.
![](assets/Docker%20Basic%2012%20-%20Build%20for%20Multi%20Architectures/image%203.png)
그리고 docker hub에서 보면 다음과 같이 2개의 OS/ARCH가 확인된다. (`MULTI-PLATFORM`이라는 라벨도 보인다)
![](assets/Docker%20Basic%2012%20-%20Build%20for%20Multi%20Architectures/image%204.png)

## 정리
> 멀티 플랫폼(아키텍처) 지원을 위한 docker buildx에 대해 학습했다.
- 우리의 소프트웨어는 다양한 하드웨어와 플랫폼에서 지원될 필요가 있다.
  - 개발하는 환경과 배포될 환경이 다를 수도 있고, 애플리케이션에 따라서는 클라이언트마다 환경이 다를 수도 있다.
- 멀티 플랫폼 지원을 위한 `docker buildx` 기능이 있다.
  - docker buildx create
  - docker buildx inspect
  - docker buildx build

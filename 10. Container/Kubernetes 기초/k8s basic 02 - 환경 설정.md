# k8s basic 02 - 환경 설정
#dev/skill/container/k8s

## 강의
[온라인 강의 - 자신의 일정에 맞춰 뭐든지 배워보세요

## 내용 정리
### docker desktop 설치
`docker` 강의를 수료했기에 이미 `docker desktop`은 설치가 되어있다. 본인의 시스템에 맡게 설치하자.
[download docker desktop](https://docs.docker.com/get-started/get-docker/?_gl=1*1owntno*_gcl_au*MTQ1ODU0Mjc1LjE3NTM2MDgyMTI.*_ga*MTc0ODY3MDk4My4xNzUzNjA4MjEy*_ga_XJWPQMJYHQ*czE3NTY3OTM2MzYkbzE1JGcxJHQxNzU2NzkzNjgzJGoxMyRsMCRoMA..)

### kubernetes 설정하기
`docker desktop` 애플리케이션을 실행하고, Settings - Kubernetes로 이동하자. 이후 Enable Kubernetes를 활성화한 뒤 restart.
![](assets/image.png)

![image-20250902154011128](assets/image-20250902154011128.png)

이후 `kubectl` 명령어를 실행해 k8s와 상호작용하는 `CLI`가 정상 동작하는지 확인한다.
```sh
kubectl                                                                                                                                                        ✔  docker-desktop 󱃾  15:25:05  ▓▒░
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/

Basic Commands (Beginner):
  create          Create a resource from a file or from stdin
  expose          Take a replication controller, service, deployment or pod and expose it as a new Kubernetes service
  run             Run a particular image on the cluster
  set             Set specific features on objects

Basic Commands (Intermediate):
  explain         Get documentation for a resource
  get             Display one or many resources
  edit            Edit a resource on the server
  delete          Delete resources by file names, stdin, resources and names, or by resources and label selector
...
```

### 싱글 노드 vs 멀티 노드 클러스터
* **싱글 노드 클러스터**:
  * 로컬 환경(노트북, PC)에서 **마스터 노드와 워커 노드 역할을 동시에 수행**.
  * 주로 학습·프로토타입 용도.
* **멀티 노드 클러스터**:
  * AWS, Azure 등 클라우드 환경에서 다수의 워커/마스터 노드를 운영.
  * **실제 서비스 운영(프로덕션)** 환경에 사용.
### kubectl 기본 설정
* **kubectl**: 쿠버네티스 클러스터와 상호작용하는 CLI 도구.
* 현재 어떤 클러스터와 연결되어 있는지 확인:
```sh
kubectl config current-context
docker-desktop
```
* Docker Desktop에서 활성화된 **로컬 쿠버네티스 클러스터**와 연결된 상태.
* 이후 실행하는 모든 kubectl 명령은 로컬 쿠버네티스 환경에 적용됨.




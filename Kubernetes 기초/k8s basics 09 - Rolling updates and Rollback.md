# k8s basics 09 - Rolling Updates and Rollback

\#dev/skill/container/k8s

---

## 스크립트 요약

### Rolling Update의 목적
* **다운타임 제거**: 애플리케이션 업그레이드 시 서비스 중단 없이 배포
* 기존 방식의 문제: 모든 구 버전 파드 삭제 → 신 버전 파드 생성 → 이 사이에 다운타임 발생

### 동작 방식
1. 점진적 교체
   * 구 버전 파드를 점진적으로 제거
   * 신 버전 파드를 점진적으로 추가
   * 교체 중에도 항상 일부 파드는 트래픽 처리 가능
2. ReplicaSet 활용
   * v1 ReplicaSet (구 버전): 3 → 0으로 점진적 스케일다운
   * v2 ReplicaSet (신 버전): 0 → 3으로 점진적 스케일업
   * Deployment Controller가 자동으로 관리
### Rollback 기능
* **이전 ReplicaSet 보존**: 구 버전 ReplicaSet을 삭제하지 않고 유지
* **빠른 롤백**: kubectl rollout undo deployment [name] 명령으로 즉시 이전 버전으로 복구
* 신 버전에 문제가 있을 때 구 버전으로 안전하게 되돌리기 가능

### Rolling Update 전략 설정
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 50%  # 업데이트 중 사용 불가능한 최대 파드 비율
      maxSurge: 1          # desired 수량을 초과할 수 있는 최대 파드 수
```
* **maxUnavailable**: 최소한의 파드는 항상 실행 보장
* **maxSurge**: 리소스 제약이 있을 때 동시 실행 파드 수 제한

### 실습 예시
* stateless 버전 → stateful 버전으로 업데이트
* 신 버전에 문제 발생 시 rollout undo로 즉시 복구
* 업데이트 중에도 서비스는 계속 응답 가능

**핵심: Deployment는 자동으로 Rolling Update를 제공하여 무중단 배포를 가능하게 함**

---

## 실습 내용

이번 실습을 위해 section-05 폴더를 만들고 기존 04 파일을 복사한다. 

### 이미지 변경
그리고 기존에 사용하던 grade-submission-api의 버전을 stateful에서 stateless로 바꾼다. 물론 강의에선 반대로 하고 있다. (상관X)
- rslim087/kubernetes-course-grade-submission-api:**stateful**
  - -> rslim087/kubernetes-course-grade-submission-api:**stateless**

> /section-05/grade-submission-api-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grade-submission-api
  namespace: grade-submission
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/instance: grade-submission-api
  template:
    metadata:
      labels:
        app.kubernetes.io/name: grade-submission
        app.kubernetes.io/component: backend
        app.kubernetes.io/instance: grade-submission-api
    spec:
      restartPolicy: Always
      containers:
      - name: grade-submission-api
        image: rslim087/kubernetes-course-grade-submission-api:stateless
        resources:
          requests:
            memory: "128Mi"
            cpu: "128m"
          limits:
            memory: "128Mi"
        ports:
          - containerPort: 3000
```

### 배포 실행
먼저 기존 section-04의 deployments를 활용해서 다음과 같이 4개의 pod를 띄운다. api pod는 2개가 동작하고 있는 걸 볼 수 있다.
```sh
cd section-04
kubectl apply -f .
deployment.apps/grade-submission-api created
service/grade-submission-api created
deployment.apps/grade-submission-portal created
service/grade-submission-portal created
...

kubectl get pods,services -n grade-submission
NAME                                           READY   STATUS    RESTARTS   AGE
pod/grade-submission-api-7d97846d8-fwvlv       1/1     Running   0          19s
pod/grade-submission-api-7d97846d8-zlbfl       1/1     Running   0          19s
pod/grade-submission-portal-86c8798559-d9cjw   1/1     Running   0          19s
pod/grade-submission-portal-86c8798559-lm7bc   1/1     Running   0          19s

NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/grade-submission-api      ClusterIP   10.104.91.68   <none>        3000/TCP         19s
service/grade-submission-portal   NodePort    10.96.189.0    <none>        5001:32000/TCP   19s
...
```

이제 다시 section-05로 가서 pod를 띄워보자
```sh
cd section-05
kubectl apply -f grade-submission-api-deployment.yaml
deployment.apps/grade-submission-api configured
...

kubectl get pods,services -n grade-submission
NAME                                           READY   STATUS              RESTARTS   AGE
pod/grade-submission-api-7d97846d8-fwvlv       1/1     Running             0          3m48s
pod/grade-submission-api-7d97846d8-zlbfl       1/1     Running             0          3m48s
pod/grade-submission-api-7ffc7d4cbb-ncsdd      0/1     ContainerCreating   0          3s
pod/grade-submission-portal-86c8798559-d9cjw   1/1     Running             0          3m48s
pod/grade-submission-portal-86c8798559-lm7bc   1/1     Running             0          3m48s
...

kubectl get pods,services -n grade-submission
NAME                                           READY   STATUS              RESTARTS   AGE
pod/grade-submission-api-7d97846d8-fwvlv       1/1     Running             0          3m53s
pod/grade-submission-api-7d97846d8-zlbfl       1/1     Terminating         0          3m53s
pod/grade-submission-api-7ffc7d4cbb-fws2r      0/1     ContainerCreating   0          1s
pod/grade-submission-api-7ffc7d4cbb-ncsdd      1/1     Running             0          8s
pod/grade-submission-portal-86c8798559-d9cjw   1/1     Running             0          3m53s
pod/grade-submission-portal-86c8798559-lm7bc   1/1     Running             0          3m53s
...

kubectl get pods -n grade-submission
NAME                                       READY   STATUS        RESTARTS   AGE
grade-submission-api-7d97846d8-fwvlv       1/1     Terminating   0          4m2s
grade-submission-api-7d97846d8-zlbfl       1/1     Terminating   0          4m2s
grade-submission-api-7ffc7d4cbb-fws2r      1/1     Running       0          10s
grade-submission-api-7ffc7d4cbb-ncsdd      1/1     Running       0          17s
grade-submission-portal-86c8798559-d9cjw   1/1     Running       0          4m2s
grade-submission-portal-86c8798559-lm7bc   1/1     Running       0          4m2s
...

kubectl get pods -n grade-submission
NAME                                       READY   STATUS    RESTARTS   AGE
grade-submission-api-7ffc7d4cbb-fws2r      1/1     Running   0          3m40s
grade-submission-api-7ffc7d4cbb-ncsdd      1/1     Running   0          3m47s
grade-submission-portal-86c8798559-d9cjw   1/1     Running   0          7m32s
grade-submission-portal-86c8798559-lm7bc   1/1     Running   0          7m32s
...
```
이렇게 보다시피, 새로운 pod가 생성(`ContainerCreating`)되면서, 기존 pod는 삭제(`Terminating`)가 진행된다. 이후 최종적으로 2개의 pod가 새로 생성되어 `Running` 상태로 바뀌었고, 기존 pod는 완전히 사라졌다.

### 롤백 실행
이번에는 다시 롤백하는 과정을 실습해보자. `kubectl rollout undo`라는 명령으로 진행한다.
```sh
kubectl rollout undo deployment/grade-submission-api -n grade-submission
deployment.apps/grade-submission-api rolled back
...

kubectl get pods -n grade-submission
NAME                                       READY   STATUS        RESTARTS   AGE
grade-submission-api-7d97846d8-47bdf       1/1     Running       0          2s
grade-submission-api-7d97846d8-kl85h       1/1     Running       0          3s
grade-submission-api-7ffc7d4cbb-bgllr      1/1     Terminating   0          71s
grade-submission-api-7ffc7d4cbb-trbmp      1/1     Terminating   0          72s
grade-submission-portal-86c8798559-d9cjw   1/1     Running       0          14m
grade-submission-portal-86c8798559-lm7bc   1/1     Running       0          14m
...

kubectl get pods -n grade-submission
NAME                                       READY   STATUS    RESTARTS   AGE
grade-submission-api-7d97846d8-47bdf       1/1     Running   0          42s
grade-submission-api-7d97846d8-kl85h       1/1     Running   0          43s
grade-submission-portal-86c8798559-d9cjw   1/1     Running   0          15m
grade-submission-portal-86c8798559-lm7bc   1/1     Running   0          15m
...
```
명령을 실행하고, 거의 3초도 되지 않는 잠깐의 시간 뒤에 get pods를 조회했는데 이미 기존(stateful) pod가 새로 올라와 `Running` 상태였고, 업데이트했던 버전(stateless)이 `Terminating` 상태가 진행되고 있었다. 굉장히 빠르네?

#### (번외)
이처럼 빠르게 롤백이 가능한 것이 어떤 이유인지 궁금했다. 다음은 claude.ai의 답변이다.
##### 1. 이미지 캐싱
* 이전 버전의 이미지가 이미 각 노드의 로컬 스토리지에 저장되어 있음
* Docker/containerd가 이미지를 다시 다운로드할 필요가 없음
* 특히 실습 환경에서는 같은 노드에서 계속 작업하므로 캐싱 효과가 극대화됨
##### 2. ReplicaSet 재사용
```yaml
# 쿠버네티스는 이전 ReplicaSet을 삭제하지 않고 보관
deployment-v1 (ReplicaSet) - replicas: 0 (보관됨)
deployment-v2 (ReplicaSet) - replicas: 3 (현재 실행중)

# rollout undo 시
deployment-v1 (ReplicaSet) - replicas: 3 (즉시 스케일업)
deployment-v2 (ReplicaSet) - replicas: 0 (스케일다운)
```
##### 3. Pod Template 재활용
* 이전 버전의 Pod 스펙이 이미 ReplicaSet에 정의되어 있음
* 새로 생성할 필요 없이 기존 템플릿을 그대로 사용
##### 4. 빠른 컨테이너 시작
* 이미지가 로컬에 있으면 컨테이너 생성은 수초 내에 완료
* 네트워크 I/O가 없어서 지연 시간이 거의 없음
##### 실제 동작 과정:
```sh
# 1. 현재 상태 확인
kubectl get rs
# NAME                DESIRED   CURRENT   READY
# myapp-5b7d8f9f6b   3         3         3  (v2 - 현재)
# myapp-7d4b5c8d9    0         0         0  (v1 - 이전)

# 2. rollout undo 실행
kubectl rollout undo deployment/myapp

# 3. 즉시 ReplicaSet 교체
# myapp-5b7d8f9f6b   0         0         0  (v2 - 종료중)
# myapp-7d4b5c8d9    3         3         3  (v1 - 실행중)
```

위 내용을 보고 `kubectl get rs -n grade-submission`으로 확인해봤더니 실제로 replica set이 다음과 같이 유지되고 있었다.
```sh
kubectl get rs -n grade-submission
NAME                                 DESIRED   CURRENT   READY   AGE
grade-submission-api-7d97846d8       2         2         2       21m
grade-submission-api-7ffc7d4cbb      0         0         0       17m
grade-submission-portal-86c8798559   2         2         2       21m
```

### 배포 설정
rolling update에 대해서 보다 구체적으로 설정할 수 있다. 다음과 같이 `strategy`와 `type`, `rollingUpdate` 등의 속성을 추가하자.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grade-submission-api
  namespace: grade-submission
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/instance: grade-submission-api
  strategy:
    type: RollingUpdate
    rollingUpdate: 
      maxUnavailable: 50%
      maxSurge: 1 # 최대 3개의 replicas 실행할 것을 보장
  template:
    metadata:
      labels:
        app.kubernetes.io/name: grade-submission
        app.kubernetes.io/component: backend
        app.kubernetes.io/instance: grade-submission-api
    spec:
      restartPolicy: Always
      containers:
      - name: grade-submission-api
        image: rslim087/kubernetes-course-grade-submission-api:stateless
        resources:
          requests:
            memory: "128Mi"
            cpu: "128m"
          limits:
            memory: "128Mi"
        ports:
          - containerPort: 3000
```
- `rollingUpdate.maxUnavailable: 50%` 설정으로 가용 pod가 50% 미만으로 내려가지 않게 하고,
- `rollingUpdate.maxSurge: 1` 설정으로 최대 운용 pod를 3개로 제한했다. (기존 Running 2개에 + 1개만 rolling update)

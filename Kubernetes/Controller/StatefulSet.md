# Kubernetes Stateful Set

## 설명
Stateful Set 은 Controller 의 한 종류로 Pod 를 관리한다.

## Feature
다른 Controller 와는 다르게 Pod 의 상태를 유지해준다.   
다만 Node 당 하나의 Pod 를 보장해주진 않는다.

### 유지되는 항목
- Pod 이름
- Pod 볼륨

### Example File
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sf-nginx
spec:
  replicas: 3 # Pod 의 개수
  serviceName: sf-service # service 이름과 pod 이름을 조합하여 coreDNS 의 Domain 으로 사용
  podManagementPolicy: Parallel # 동시에 실행 / Default 는 OrderedReady 로 순차적으로 Pod 를 실행한다.
  selector:
    matchLabels:
      app: web-ui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web-ui
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14
```

### Excution Result
```shell
ubuntu@master:~/deploy$ kubectl get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running   0          7s    192.168.171.73    worker   <none>           <none>
sf-nginx-1   1/1     Running   0          7s    192.168.219.104   master   <none>           <none>
sf-nginx-2   1/1     Running   0          7s    192.168.171.78    worker   <none>           <none>
```
Stateful Set 이름에 0 부터 순차적으로 숫자가 할당되어 Pod 이름으로 부여된다.

## Scale Out / In
```shell
ubuntu@master:~/deploy$ kubectl scale statefulset sf-nginx --replicas=4
statefulset.apps/sf-nginx scaled
ubuntu@master:~/deploy$ kubectl get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running   0          12m   192.168.171.73    worker   <none>           <none>
sf-nginx-1   1/1     Running   0          12m   192.168.219.104   master   <none>           <none>
sf-nginx-2   1/1     Running   0          12m   192.168.171.78    worker   <none>           <none>
sf-nginx-3   1/1     Running   0          3s    192.168.219.105   master   <none>           <none>
```
Scale Out 을 수행하면 마지막 번호에 +1 되어 새로운 Pod 가 생성된다.   

```shell
ubuntu@master:~/deploy$ kubectl scale statefulset sf-nginx --replicas=2
statefulset.apps/sf-nginx scaled
ubuntu@master:~/deploy$ kubectl get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running   0          15m   192.168.171.73    worker   <none>           <none>
sf-nginx-1   1/1     Running   0          15m   192.168.219.104   master   <none>           <none>
```
Scale In 을 수행하면 마지막 번호부터 순서대로 삭제된다.

## Rolling Update
```shell
ubuntu@master:~/deploy$ kubectl edit statefulsets.apps sf-nginx
```
edit 명령어를 이용하여 생성된 Stateful Set 의 Container 들을 업데이트할 수 있다.

```shell
kubectl apply -f statefulset.yaml
```
또한 yaml 파일을 수정하고 적용하여 업데이트할 수 있다.

## Rollback
```shell
ubuntu@master:~/deploy$ kubectl rollout undo statefulset sf-nginx
statefulset.apps/sf-nginx rolled back
```
업데이트 내용을 Rollback 하고 싶다면 rollout undo 명령어로 바로 이전의 업데이트를 취소할 수 있다.

## Delete
```shell
ubuntu@master:~/deploy$ kubectl delete -f statefulset.yaml
statefulset.apps "sf-nginx" deleted
```
yaml 파일로 정의된 Stateful Set 을 삭제한다.

### Reference.
- https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/
- https://youtu.be/Mx3y9un1KeI
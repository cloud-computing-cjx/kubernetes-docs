# 暴露应用
## 创建 pod
```
# vim run-my-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 1
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80

```
启动 pod
```
# sudo kubectl apply -f ./run-my-nginx.yaml
deployment.apps/my-nginx created
# sudo kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-5b56ccd65f-wgd7b   1/1     Running   0          12s
```
## 创建 Service
```
# sudo kubectl expose deployment/my-nginx --type="NodePort" --port 80
service/my-nginx exposed
```
## 查看 Service
```
# sudo kubectl get svc                                               
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP        4h17m
my-nginx     NodePort    10.43.65.117   <none>        80:31932/TCP   2s

# sudo kubectl describe service/my-nginx
Name:                     my-nginx
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 run=my-nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.65.117
IPs:                      10.43.65.117
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31932/TCP
Endpoints:                10.42.0.9:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
## 测试
```
# curl localhost:31932
```
# 使用标签
## 查看标签
```
# sudo kubectl describe deployment
Name:                   my-nginx
Namespace:              default
CreationTimestamp:      Wed, 01 Sep 2021 15:15:45 +0800
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=my-nginx
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=my-nginx
  Containers:
   my-nginx:
    Image:        nginx
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   my-nginx-5b56ccd65f (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  30m   deployment-controller  Scaled up replica set my-nginx-5b56ccd65f to 1

# sudo kubectl get pods -l run=my-nginx
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-5b56ccd65f-wgd7b   1/1     Running   0          32m

# sudo kubectl get services -l run=my-nginx
No resources found in default namespace.
```
## 打标签
### pod
```
# sudo kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-5b56ccd65f-wgd7b   1/1     Running   0          39m

# sudo kubectl label pods my-nginx-5b56ccd65f-wgd7b version=v1
pod/my-nginx-5b56ccd65f-wgd7b labeled

# sudo kubectl describe pods my-nginx-5b56ccd65f-wgd7b
Name:         my-nginx-5b56ccd65f-wgd7b
Namespace:    default
Priority:     0
Node:         cjx/192.168.50.2
Start Time:   Wed, 01 Sep 2021 15:15:45 +0800
Labels:       pod-template-hash=5b56ccd65f
              run=my-nginx
              version=v1
Annotations:  <none>
Status:       Running
IP:           10.42.0.9
...

# sudo kubectl get pods -l version=v1
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-5b56ccd65f-wgd7b   1/1     Running   0          44m
```
### service
```
# sudo kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP        4h37m
my-nginx     NodePort    10.43.65.117   <none>        80:31932/TCP   19m

# sudo kubectl label service my-nginx app=nginx
service/my-nginx labeled

# sudo kubectl describe service my-nginx                      
Name:                     my-nginx
Namespace:                default
Labels:                   app=nginx
Annotations:              <none>
Selector:                 run=my-nginx
Type:                     NodePort

# sudo kubectl get svc -l app=nginx
NAME       TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
my-nginx   NodePort   10.43.65.117   <none>        80:31932/TCP   22m
```
# 删除service
```
# sudo kubectl delete service -l app=nginx
service "my-nginx" deleted

# curl localhost:31932
curl: (7) Failed to connect to localhost port 31932: 拒绝连接

# sudo kubectl exec -ti my-nginx-5b56ccd65f-wgd7b -- curl localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```
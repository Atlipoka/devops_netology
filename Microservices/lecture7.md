***
Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod
***
1. Создаем deployment состоящее из двух приложений nginx и multitool c кол-вом реплик 1
```
vagrant@vagrant:~/kubernetes/kuber_RunApp$ nano deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool:latest
        ports:
        - containerPort: 80

vagrant@vagrant:~/kubernetes/kuber_RunApp$ microk8s kubectl create -f deployment.yaml

```
* ![task-7-1](https://github.com/Atlipoka/devops_netology/blob/main/Microservices/lecture7-1.png)

2. Увеличиваем кол-во реплик до 2х

```
vagrant@vagrant:~$ microk8s kubectl scale deployment nginx-deployment --replicas=2
deployment.apps/nginx-deployment scaled
....
vagrant@vagrant:~/kubernetes/kuber_RunApp$ microk8s kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           28h
...
vagrant@vagrant:~/kubernetes/kuber_RunApp$ microk8s kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-57565745f6   2         2         2       28h
...
```
3. Создаем сервис для доступа до реплик из п.1
```
vagrant@vagrant:~/kubernetes/kuber_RunApp$ nano svc.yaml
...
piVersion: v1
kind: Service
metadata:
  name: netology-svc
spec:
  ports:
  - name: web
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
     app: nginx
...
vagrant@vagrant:~/kubernetes/kuber_RunApp$ microk8s kubectl create -f svc.yaml
service/netology-svc created
```
4. Создаем отдельный Pod с приложением multitool и проверяем с помощью curl, что из пода есть доступ до приложений из п.1.
```
vagrant@vagrant:~/kubernetes/kuber_RunApp$ nano pod.yaml
...
apiVersion: v1
kind: Pod
metadata:
  name: netology-web
  labels:
     app: nginx
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool:latest
    ports:
    - containerPort: 80
...
vagrant@vagrant:~/kubernetes/kuber_RunApp$ microk8s kubectl create -f pod.yaml
pod/netology-web created
...
vagrant@vagrant:~/kubernetes/kuber_RunApp$ microk8s kubectl get po -o wide
NAME                                READY   STATUS              RESTARTS        AGE    IP            NODE      NOMINATED NODE   READINESS GATES
nginx-deployment-57565745f6-rj9m4   2/2     Running             0               23h    10.1.52.175   vagrant   <none>           <none>
nginx-deployment-57565745f6-wlpm5   2/2     Running             4 (3h58m ago)   29h    10.1.52.181   vagrant   <none>           <none>
netology-web                        1/1     Running             0               175m   10.1.52.180   vagrant   <none>           <none>
...
vagrant@vagrant:~/kubernetes/kuber_RunApp$ microk8s kubectl exec pod/netology-web -it -- /bin/bash
bash-5.1# curl http://10.1.52.181
WBITT Network MultiTool (with NGINX) - nginx-deployment-57565745f6-wlpm5 - 10.1.52.181 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
```
***
Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий
***
1. Создание Deployment приложения nginx и с init контейнером
```
vagrant@vagrant:~/kubernetes/kuber_RunApp$ nano deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
      initContainers:
      - name: wait-service
        image: busybox:1.28
        command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]>
vagrant@vagrant:~/kubernetes/kuber_RunApp$ kubectl create -f deployment.yaml
vagrant@vagrant:~$ kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/1     1            0           19h
vagrant@vagrant:~$ kubectl get po
NAME                                READY   STATUS     RESTARTS   AGE
nginx-deployment-6db745689f-7vrpb   0/1     Init:0/1   0          18h
vagrant@vagrant:~$ kubectl describe po/nginx-deployment-6db745689f-7vrpb
Name:             nginx-deployment-6db745689f-7vrpb
Namespace:        default
Priority:         0
Service Account:  default
Node:             vagrant/10.0.2.15
Start Time:       Mon, 17 Jul 2023 14:05:08 +0000
Labels:           app=nginx
                  pod-template-hash=6db745689f
Annotations:      cni.projectcalico.org/containerID: 879d82639a2cd32de0e2d3d7e59fb698c8e8f0796d35737a4158842370fbb9e2
                  cni.projectcalico.org/podIP: 10.1.52.133/32
                  cni.projectcalico.org/podIPs: 10.1.52.133/32
Status:           Pending
IP:               10.1.52.133
IPs:
  IP:           10.1.52.133
Controlled By:  ReplicaSet/nginx-deployment-6db745689f
Init Containers:
  wait-service:
    Container ID:  containerd://51c87367cf08b95f3407f6ed81be1cdd37ff01d13222100e4e269980649a779e
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done
    State:          Running
      Started:      Tue, 18 Jul 2023 08:53:34 +0000
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-v4l6w (ro)
Containers:
  nginx:
    Container ID:
    Image:          nginx:1.14.2
    Image ID:
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-v4l6w (ro)
Conditions:
  Type              Status
  Initialized       False
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  kube-api-access-v4l6w:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason   Age    From     Message
  ----    ------   ----   ----     -------
  Normal  Created  7m14s  kubelet  Created container wait-service
  Normal  Started  7m     kubelet  Started container wait-service
---

```
2. Создание Service и проверка, что Pod и Deployment успешно запустились.
```
vagrant@vagrant:~/kubernetes/kuber_RunApp$ nano svc.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - name: web
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
     app: nginx
---
vagrant@vagrant:~/kubernetes/kuber_RunApp$ kubectl create -f svc.yaml
service/myservice created
vagrant@vagrant:~/kubernetes/kuber_RunApp$ kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6db745689f-7vrpb   1/1     Running   0          19h
vagrant@vagrant:~/kubernetes/kuber_RunApp$ kubectl describe po/nginx-deployment-6db745689f-7vrpb
Name:             nginx-deployment-6db745689f-7vrpb
Namespace:        default
Priority:         0
Service Account:  default
Node:             vagrant/10.0.2.15
Start Time:       Mon, 17 Jul 2023 14:05:08 +0000
Labels:           app=nginx
                  pod-template-hash=6db745689f
Annotations:      cni.projectcalico.org/containerID: 879d82639a2cd32de0e2d3d7e59fb698c8e8f0796d35737a4158842370fbb9e2
                  cni.projectcalico.org/podIP: 10.1.52.133/32
                  cni.projectcalico.org/podIPs: 10.1.52.133/32
Status:           Running
IP:               10.1.52.133
IPs:
  IP:           10.1.52.133
Controlled By:  ReplicaSet/nginx-deployment-6db745689f
Init Containers:
  wait-service:
    Container ID:  containerd://51c87367cf08b95f3407f6ed81be1cdd37ff01d13222100e4e269980649a779e
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 18 Jul 2023 08:53:34 +0000
      Finished:     Tue, 18 Jul 2023 09:04:24 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-v4l6w (ro)
Containers:
  nginx:
    Container ID:   containerd://c16678c04ee3be5067921fba74d61b52e0dba854d98b3b86317c5c511ce37eaa
    Image:          nginx:1.14.2
    Image ID:       docker.io/library/nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 18 Jul 2023 09:06:25 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-v4l6w (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-v4l6w:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```

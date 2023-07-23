***
Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера
***
1. Создаем Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
```
---
piVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
          name: http-port
      restartPolicy: Always
---
```
2. Создаем Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
```
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: nginx
  ports:
  - name: nginx
    protocol: TCP
    port: 9001
    targetPort: 80
  - name: multitool
    protocol: TCP
    port: 9002
    targetPort: 8080
---
```
3. Создаем отдельный Pod с приложением multitool и промеряем с помощью curl, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
```
---
apiVersion: v1
kind: Pod
metadata:
  name: multitool
  labels:
     app: nginx
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool:latest
    ports:
    - containerPort: 80
    - containerPort: 443
---
vagrant@vagrant:~/kubernetes/kuber_Network1$ kubectl get svc/myservice -o wide
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE   SELECTOR
myservice   ClusterIP   10.152.183.180   <none>        9001/TCP,9002/TCP   2d    app=nginx
vagrant@vagrant:~/kubernetes/kuber_Network1$ kubectl exec pod/multitool -it -- /bin/bash
bash-5.1# curl http://10.152.183.180:9001
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
bash-5.1# curl http://10.152.183.180:9002
WBITT Network MultiTool (with NGINX) - nginx-deployment-67fc44988b-w8vrl - 10.1.52.129 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```
4. Проверяем доступ с помощью curl по доменному имени сервиса.
```
---
vagrant@vagrant:~/kubernetes/kuber_Network1$ kubectl exec -ti multitool -- nslookup myservice
Server:         10.152.183.10
Address:        10.152.183.10#53

Name:   myservice.default.svc.cluster.local
Address: 10.152.183.180
---
bash-5.1# curl http://myservice.default.svc.cluster.local:9001
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
bash-5.1# curl http://myservice.default.svc.cluster.local:9002
WBITT Network MultiTool (with NGINX) - nginx-deployment-67fc44988b-w9f25 - 10.1.52.188 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool) 
---
```
***
Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера
***
1. Создаем отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
```
---
apiVersion: v1
kind: Service
metadata:
  name: myservice-1
spec:
  selector:
    app: nginx
  ports:
  - name: nginx
    protocol: TCP
    port: 80
    nodePort: 30080
  - name: multitool
    protocol: TCP
    port: 8080
    nodePort: 31080
  type: NodePort
---
vagrant@vagrant:~/kubernetes/kuber_Network1$ kubectl describe svc/myservice-1
Name:                     myservice-1
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.152.183.156
IPs:                      10.152.183.156
Port:                     nginx  80/TCP
TargetPort:               80/TCP
NodePort:                 nginx  30080/TCP
Endpoints:                10.1.52.129:80,10.1.52.132:80,10.1.52.157:80 + 1 more...
Port:                     multitool  8080/TCP
TargetPort:               8080/TCP
NodePort:                 multitool  31080/TCP

Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

vagrant@vagrant:~/kubernetes/kuber_Network1$ kubectl get nodes -o wide
NAME      STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
vagrant   Ready    <none>   45d   v1.27.2   192.168.88.199   <none>        Ubuntu 20.04.5 LTS   5.4.0-80-generic   containerd://1.6.15

vagrant@vagrant:~/kubernetes/kuber_Network1$ curl http://192.168.88.199:30080
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

vagrant@vagrant:~/kubernetes/kuber_Network1$ curl http://192.168.88.199:31080
WBITT Network MultiTool (with NGINX) - nginx-deployment-67fc44988b-tzs8q - 10.1.52.157 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)

vagrant@vagrant:~/kubernetes/kuber_Network1$ curl http://192.168.88.199:30080
WBITT Network MultiTool (with NGINX) - multitool - 10.1.52.132 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)

vagrant@vagrant:~/kubernetes/kuber_Network1$ curl http://localhost:30080
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

vagrant@vagrant:~/kubernetes/kuber_Network1$ curl http://localhost:31080
WBITT Network MultiTool (with NGINX) - nginx-deployment-67fc44988b-tzs8q - 10.1.52.157 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```

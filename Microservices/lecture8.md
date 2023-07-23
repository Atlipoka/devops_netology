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
3. Создаем Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
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
5. Создаем отдельный Pod с приложением multitool и промеряем с помощью curl, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
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
7. Проверяем доступ с помощью curl по доменному имени сервиса.
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

***
Задание 1. Создать Deployment приложений backend и frontend
***
1. Создаем Deployment приложения frontend из образа nginx с количеством реплик 3 шт, Deployment приложения backend из образа multitool и добавляем Service, для доступа приложениям внутри кластера.
```
vagrant@vagrant:~/kubernetes/kuber_Network2$ nano frontend.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
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
      restartPolicy: Always
---

vagrant@vagrant:~/kubernetes/kuber_Network2$ nano backend.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: multitool
spec:
  replicas: 2
  selector:
    matchLabels:
      app: multitool
  template:
    metadata:
      labels:
        app: multitool
    spec:
      containers:
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

vagrant@vagrant:~/kubernetes/kuber_Network2$ nano servicefront.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: svc-front
spec:
  selector:
    app: nginx
  ports:
  - name: nginx
    protocol: TCP
    port: 80
  type: ClusterIP
---

vagrant@vagrant:~/kubernetes/kuber_Network2$ nano serviceback.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: svc-back
spec:
  selector:
    app: multitool
  ports:
  - name: multitool
    protocol: TCP
    port: 8080
  type: ClusterIP
---



vagrant@vagrant:~/kubernetes/kuber_Network2$ kubectl get po -o wide
NAME                       READY   STATUS    RESTARTS      AGE     IP            NODE      NOMINATED NODE   READINESS GATES
frontend-cbdccf466-lz8bh   1/1     Running   1 (34m ago)   18h     10.1.52.181   vagrant   <none>           <none>
frontend-cbdccf466-l5fkd   1/1     Running   1 (35m ago)   18h     10.1.52.166   vagrant   <none>           <none>
frontend-cbdccf466-qnhx6   1/1     Running   1 (34m ago)   18h     10.1.52.137   vagrant   <none>           <none>
backend-7d86cbbc89-jqsbq   1/1     Running   0             6m45s   10.1.52.139   vagrant   <none>           <none>
backend-7d86cbbc89-mngv4   1/1     Running   0             6m45s   10.1.52.159   vagrant   <none>           <none>

vagrant@vagrant:~/kubernetes/kuber_Network2$ kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE     SELECTOR
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP    37d     <none>
svc-back     ClusterIP   10.152.183.209   <none>        8080/TCP   3m42s   app=multitool
svc-front    ClusterIP   10.152.183.74    <none>        80/TCP     3m32s   app=nginx

vagrant@vagrant:~/kubernetes/kuber_Network2$ curl http://10.152.183.209:8080
WBITT Network MultiTool (with NGINX) - backend-7d86cbbc89-mngv4 - 10.1.52.159 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)

vagrant@vagrant:~/kubernetes/kuber_Network2$ curl http://10.152.183.74:80
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

vagrant@vagrant:~/kubernetes/kuber_Network2$ kubectl exec pod/backend-7d86cbbc89-jqsbq -it -- /bin/bash
bash-5.1# curl http://10.152.183.74:80
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
***
Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера
*** 
1. Создаю deploy frontend и backend
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
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
      restartPolicy: Always
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: multitool
spec:
  replicas: 2
  selector:
    matchLabels:
      app: multitool
  template:
    metadata:
      labels:
        app: multitool
    spec:
      containers:
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
2. После создания deploy frontend и backend создаю service для каждого с типом ClusterIP
```
---
apiVersion: v1
kind: Service
metadata:
  name: front
spec:
  selector:
    app: nginx
  ports:
  - name: nginx
    protocol: TCP
    port: 80
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: back
spec:
  selector:
    app: multitool
  ports:
  - name: multitool
    protocol: TCP
    port: 8080
  type: ClusterIP
---
```
3. После включаю ingress в microk8s коммандой ```microk8s enable ingress``` и metallb ```microk8s enable metallb```.
4. Потом проверяю аргументы контроллера nginx и вижу, что в аргументах указано ````--ingress-class=public```` и ````--publish-status-address=127.0.0.1````, заатем поискав в интернете инфу редактирую аргументы, удаляю ````--publish-status-address=127.0.0.1```` и меняю класс с ````--ingress-class=public```` на ````--ingress-class=nginx````.
5. Затем создаю ingress и проверяю, что он создался с корректным адресом.
````
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
# - host: default.ingress.ru
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: front
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: back
            port:
              number: 8080
---

vagrant@vagrant:~/kubernetes/kuber_Network2$ kubectl get ing
NAME        CLASS   HOSTS   ADDRESS         PORTS   AGE
myingress   nginx   *       192.168.88.84   80      14m

vagrant@vagrant:~/kubernetes/kuber_Network2$ curl http://192.168.88.84/
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

vagrant@vagrant:~/kubernetes/kuber_Network2$ curl http://192.168.88.84/api
WBITT Network MultiTool (with NGINX) - backend-7d86cbbc89-47sbx - 10.1.52.163 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```` 

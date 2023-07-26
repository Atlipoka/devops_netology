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

vagrant@vagrant:~/kubernetes/kuber_Network2$ nano svc.yaml
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
    port: 80
  - name: multitool
    protocol: TCP
    port: 8080
  type: ClusterIP
---

vagrant@vagrant:~/kubernetes/kuber_Network2$ kubectl get po -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP            NODE      NOMINATED NODE   READINESS GATES

frontend-cbdccf466-l5fkd   1/1     Running   0          29m   10.1.52.171   vagrant   <none>           <none>
frontend-cbdccf466-lz8bh   1/1     Running   0          29m   10.1.52.165   vagrant   <none>           <none>
backend-5769f77c6b-n6vx4   1/1     Running   0          28m   10.1.52.180   vagrant   <none>           <none>
backend-5769f77c6b-g42v4   1/1     Running   0          28m   10.1.52.190   vagrant   <none>           <none>
backend-5769f77c6b-6cttk   1/1     Running   0          28m   10.1.52.144   vagrant   <none>           <none>

vagrant@vagrant:~/kubernetes/kuber_Network2$ kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE   SELECTOR
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP           36d   <none>
myservice    ClusterIP   10.152.183.146   <none>        80/TCP,8080/TCP   31m   app=nginx

vagrant@vagrant:~/kubernetes/kuber_Network2$ kubectl exec pod/backend-5769f77c6b-n6vx4 -it -- /bin/bash
bash-5.1# curl http://10.1.52.168:80
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

bash-5.1# curl http://10.152.183.146:80
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
2.ппрпрпарпарпарпар 

# Домашнее задание к занятию Troubleshooting

## Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить
1. Установить приложение по команде:
```` kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml ````
2. Выявить проблему и описать.
3. Исправить проблему, описать, что сделано.
4. Продемонстрировать, что проблема решена.

***
Выполнение
***

Все выявленные проблемы по порядку:
1. Первое, с чем столкнулся, это отсутствующие неймспесы web и data, создаем их.
2. Приложение из контейнера в неймспейсе web не может подключиться к другому приложению в неймспейсе data по запросу ``curl auth-db`` из за ошибки ``curl: (6) Couldn't resolve host 'auth-db'``, что можно сделать, на мой взгляд:
  * Пересоздать deploy с приложениями с неймспейсом data, чтоб все приложения были в одном неймспейсе. Тогда внутри неймспейса оно будет резолвиться и запросы будут проходить
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-consumer
  namespace: data
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-consumer
  template:
    metadata:
      labels:
        app: web-consumer
    spec:
      containers:
      - command:
        - sh
        - -c
        - while true; do curl auth-db; sleep 5; done
        image: radial/busyboxplus:curl
        name: busybox
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-db
  namespace: data
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth-db
  template:
    metadata:
      labels:
        app: auth-db
    spec:
      containers:
      - image: nginx:1.19.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: auth-db
  namespace: data
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: auth-db

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_trouble$ kubectl get deploy,po,svc -n data
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/auth-db        1/1     1            1           16m
deployment.apps/web-consumer   2/2     2            2           16m

NAME                                READY   STATUS    RESTARTS   AGE
pod/auth-db-864ff9854c-8lv98        1/1     Running   0          16m
pod/web-consumer-84fc79d94d-589ln   1/1     Running   0          16m
pod/web-consumer-84fc79d94d-qsj9x   1/1     Running   0          16m

NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/auth-db   ClusterIP   10.152.183.150   <none>        80/TCP    16m

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_trouble$ kubectl logs pod/web-consumer-84fc79d94d-589ln -n data
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
...
````
  * Если нам все-же нужно разделить приложения по разным неймспейсам, тогда нужно узнать полное DNS имя сервиса и изменить его в запросе, выбор как сделать изменения решать нам, или вязть исправить в исходнике, или прям на горячую изменить используюя edit, я сделал с edit
````
kubectl edit deploy web-consumer -n web
...
- command:
        - sh
        - -c
        - while true; do curl auth-db.data.svc.cluster.local; sleep 5; done
...
deployment.apps/web-consumer edited

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_trouble$ kubectl logs pod/web-consumer-746567899c-h8z8z -n web

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0  38956      0 --:--:-- --:--:-- --:--:--   99k
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
````

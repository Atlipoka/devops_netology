***
Создание pod с именем hello-world
1. yaml манифест для pod'a
```
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
  - name: hello-world
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 8080
```
2. Проброс порта для ```kubectl port-forward --address 127.0.0.1 pod/hello-world :8080``` и использования curl, для проверки работоспособности (локальный порт не прописывал специально, чтоб определилися любой свободный):
```
vagrant@vagrant:~$ curl http://127.0.0.1:39743

Hostname: hello-world

Pod Information:
        -no pod information available-

Server values:
        server_version=nginx: 1.12.2 - lua: 10010

Request Information:
        client_address=127.0.0.1
        method=GET
        real path=/
        query=
        request_version=1.1
        request_scheme=http
        request_uri=http://127.0.0.1:8080/

Request Headers:
        accept=*/*
        host=127.0.0.1:39743
        user-agent=curl/7.68.0

Request Body:
        -no body in request-

vagrant@vagrant:~$ curl http://127.0.0.1:39743


Hostname: hello-world

Pod Information:
        -no pod information available-

Server values:
        server_version=nginx: 1.12.2 - lua: 10010
```
***
Создание сервиса и подключение к Pod
1. Манифесты Service и Pod
```
apiVersion: v1
kind: Service
metadata:
  name: netology-web-service
spec:
  ports:
  - name: web
    port: 80
    protocol: TCP
    targetPort: 8080
  selector:
     app: test-app
```

```
apiVersion: v1
kind: Pod
metadata:
  name: netology-web
  labels:
     app: test-app
spec:
  containers:
  - name: netology-web
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 8080
```
2. Проброс порта для Service ```vagrant@vagrant:~/kubernetes$ kubectl port-forward --address 127.0.0.1 svc/netology-web-service :80``` и использования curl, для проверки работоспособности (локальный порт не прописывал специально, чтоб определилися любой свободный):
```
vagrant@vagrant:~$ curl http://127.0.0.1:33887

Hostname: netology-web

Pod Information:
        -no pod information available-

Server values:
        server_version=nginx: 1.12.2 - lua: 10010

Request Information:
        client_address=127.0.0.1
        method=GET
        real path=/
        query=
        request_version=1.1
        request_scheme=http
        request_uri=http://127.0.0.1:8080/

Request Headers:
        accept=*/*
        host=127.0.0.1:33887
        user-agent=curl/7.68.0

Request Body:
        -no body in request-

```

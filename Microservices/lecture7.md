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
![task-7-1](https://github.com/Atlipoka/devops_netology/blob/main/Microservices/lecture7-1.png)
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

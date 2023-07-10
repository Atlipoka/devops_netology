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

***
Задание 1. Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.
***
1. Создаем deployment состоящий из двух контейнеров multitool и busybox, шарим между ними общий файл с логом и проверяем, что контейнеры видят общий файл.
````
vagrant@vagrant:~/kubernetes/kuber_Storing1$ nano deploy.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: volumes
  labels:
    app: vol
spec:
  replicas: 2
  selector:
    matchLabels:
      app: vol
  template:
    metadata:
      labels:
        app: vol
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
        volumeMounts:
        - name: vol
          mountPath: /output
      - name: busybox
        image: busybox:1.28
        command: ['sh', '-c','while true; do echo "Текущая дата и время" `date` >> /input/log.txt; sleep 5; done']
        volumeMounts:
        - name: vol
          mountPath: /input
      volumes:
      - name: vol
        hostPath:
          path: /var/data
      restartPolicy: Always
---

vagrant@vagrant:~/kubernetes/kuber_Storing1$ kubectl get po
NAME                       READY   STATUS    RESTARTS   AGE
volumes-79b769449c-7fn6f   2/2     Running   0          19m
volumes-79b769449c-tzs2l   2/2     Running   0          19m

vagrant@vagrant:~/kubernetes/kuber_Storing1$ kubectl exec -i -t volumes-79b769449c-7fn6f --container multitool -- /bin/bash
bash-5.1# ls output/
log.txt

bash-5.1# cat output/log.txt
...
Текущая дата и время Tue Aug 1 20:03:55 UTC 2023
Текущая дата и время Tue Aug 1 20:03:58 UTC 2023
Текущая дата и время Tue Aug 1 20:04:00 UTC 2023
Текущая дата и время Tue Aug 1 20:04:03 UTC 2023
Текущая дата и время Tue Aug 1 20:04:05 UTC 2023
Текущая дата и время Tue Aug 1 20:04:08 UTC 2023
Текущая дата и время Tue Aug 1 20:04:10 UTC 2023
Текущая дата и время Tue Aug 1 20:04:13 UTC 2023
Текущая дата и время Tue Aug 1 20:04:15 UTC 2023
Текущая дата и время Tue Aug 1 20:04:18 UTC 2023
Текущая дата и время Tue Aug 1 20:04:20 UTC 2023
...
````

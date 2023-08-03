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

***
Задание 2. Создать DaemonSet приложения, которое может прочитать логи ноды.
***
1. Создаем DaemonSet c подом multitool и шарим для него логи ````/var/log/syslog````

````
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemon
spec:
  selector:
    matchLabels:
      name: daemon
  template:
    metadata:
      labels:
        name: daemon
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
        - name: syslog
          mountPath: /var/log/syslog
      volumes:
      - name: syslog
        hostPath:
          path: /var/log/syslog
       restartPolicy: Always
---

vagrant@vagrant:~/kubernetes/kuber_Storing1$ kubectl exec -i -t daemon-zft5t --container multitool -- /bin/bash
bash-5.1# vi var/log/syslog
Aug  3 11:55:39 vagrant microk8s.daemon-containerd[759]: ++ [[ /snap/microk8s/5372/run-containerd-with-args == \/\s\n\a\p\/\m\i\c\r\o\k\8\s\/\5\3\7\2\/\a\c\t\i\o\n\s\/\c\o\m\m\o\n\Aug  3 11:55:39 vagrant microk8s.daemon-containerd[759]: + is_strict
Aug  3 11:55:39 vagrant systemd[956]: Reached target Timers.
Aug  3 11:55:40 vagrant microk8s.daemon-apiserver-proxy[1128]: ++ /snap/microk8s/5372/bin/uname -m
Aug  3 11:55:40 vagrant microk8s.daemon-k8s-dqlite[1124]: ++ /snap/microk8s/5372/bin/uname -m
Aug  3 11:55:40 vagrant microk8s.daemon-containerd[1131]: + cat /snap/microk8s/5372/meta/snap.yaml
Aug  3 11:55:40 vagrant systemd[956]: Starting D-Bus User Message Bus Socket.
Aug  3 11:55:40 vagrant microk8s.daemon-apiserver-proxy[757]: + ARCH=x86_64
Aug  3 11:55:40 vagrant microk8s.daemon-apiserver-proxy[757]: + export LD_LIBRARY_PATH=/var/lib/snapd/lib/gl:/var/lib/snapd/lib/gl32:/var/lib/snapd/void::/snap/microk8s/5372/lib:/sAug  3 11:55:40 vagrant microk8s.daemon-apiserver-proxy[757]: + LD_LIBRARY_PATH=/var/lib/snapd/lib/gl:/var/lib/snapd/lib/gl32:/var/lib/snapd/void::/snap/microk8s/5372/lib:/snap/micAug
...
````

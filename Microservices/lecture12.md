### Задание 1. Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap. Добавить веб-страницу
---
1. Создаем Deployment приложения, состоящего из контейнеров busybox и multitool. Создаем ConfgMap и берем необходимые значения для Deployment оттуда
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: volumes
  labels:
    app: vol
  namespace: default
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
          valueFrom:
            configMapKeyRef:
              name: env
              key: http_port
        - name: HTTPS_PORT
          valueFrom:
            configMapKeyRef:
              name: env
              key: https_port
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
        persistentVolumeClaim:
          claimName: my-pvc
      restartPolicy: Always
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: env
data:
  http_port: "8080"
  https_port: "1443"

vagrant@vagrant:~/kubernetes/kuber_Config$ kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
volumes-669cbbd5b6-rrrg7            2/2     Running   0          109m
volumes-669cbbd5b6-px48q            2/2     Running   0          109m
````
2. Сделать простую веб-страницу и подключить её к Nginx с помощью ConfigMap. Подключить Service и показать вывод curl или в браузере.
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
            - name: nginx-index
              mountPath: /usr/share/nginx/html/
      volumes:
      - name: nginx-index
        configMap:
          name: nginx-index
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-index
data:
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>Hi! We are successfully change previous web page </h1>
    </html
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  type: ClusterIP
  ports:
    - name: ngins
      port: 8080
      protocol: TCP
      targetPort: 80

vagrant@vagrant:~/kubernetes/kuber_Config$ kubectl get svc
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.152.183.1     <none>        443/TCP    15d
nginx-service   ClusterIP   10.152.183.218   <none>        8080/TCP   5s

vagrant@vagrant:~/kubernetes/kuber_Config$ curl http://10.152.183.218:8080
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! We are successfully change previous web page </h1>
</html
````

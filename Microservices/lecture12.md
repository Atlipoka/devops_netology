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

### Задание 2. Создать приложение с вашей веб-страницей, доступной по HTTPS
---
1. Создать Deployment приложения, состоящего из Nginx.
````
---
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
        - containerPort: 443
        volumeMounts:
            - name: nginx-index
              mountPath: /usr/share/nginx/html/
      volumes:
      - name: nginx-index
        configMap:
          name: nginx-index
````
2. Создать собственную веб-страницу и подключить её как ConfigMap к приложению.
````
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
````
3. Выпустить самоподписной сертификат SSL. Создать Secret для использования сертификата.
 - Сертификат выпустид по инструкции https://devopscube.com/create-self-signed-certificates-openssl/, после настройки ингреса покажу параметры сертификата
4. Создать Ingress и необходимый Service, подключить к нему SSL в вид. Продемонстировать доступ к приложению по HTTPS.
````
kubectl create secret tls test-tls --key="server.key" --cert="server.crt"

vagrant@vagrant:~/kubernetes/kuber_Config$ kubectl get secret
NAME       TYPE                DATA   AGE
test-tls   kubernetes.io/tls   2      24h

--
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  type: ClusterIP
  ports:
    - name: http
      protocol: TCP
      port: 80
    - name: https
      protocol: TCP
      port: 443

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress-2
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - kabaev.test.com
      secretName: test-tls
  rules:
  - host: kabaev.test.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80

vagrant@vagrant:~/kubernetes/kuber_Config$ sudo cat /etc/hosts
# Add adress for Kuber ingress
192.168.88.246 kabaev.test.com

vagrant@vagrant:~/kubernetes/kuber_Config$ curl http://kabaev.test.com/
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! We are successfully change previous web page </h1>
</html

vagrant@vagrant:~/kubernetes/kuber_Config$ curl -v https://kabaev.test.com/ --cacert /etc/ssl/certs/server.crt
*   Trying 192.168.88.246:443...
* TCP_NODELAY set
* Connected to kabaev.test.com (192.168.88.246) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/server.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=RU; ST=Moscow; L=Moscow; O=Kabaev; OU=Kabaev; CN=kabaev.test.com
*  start date: Aug 17 13:38:17 2023 GMT
*  expire date: Aug 16 13:38:17 2024 GMT
*  subjectAltName: host "kabaev.test.com" matched cert's "kabaev.test.com"
*  issuer: CN=kabaev.test.com; C=RU; L=Moscow
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x560be92a8300)
> GET / HTTP/2
> Host: kabaev.test.com
> user-agent: curl/7.68.0
> accept: */*
>
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 200
< date: Fri, 18 Aug 2023 14:35:06 GMT
< content-type: text/html
< content-length: 96
< last-modified: Thu, 17 Aug 2023 14:43:42 GMT
< etag: "64de321e-60"
< accept-ranges: bytes
< strict-transport-security: max-age=15724800; includeSubDomains
<
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! We are successfully change previous web page </h1>
</html
* Connection #0 to host kabaev.test.com left intact
````

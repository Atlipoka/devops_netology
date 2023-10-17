# Домашнее задание к занятию «Обновление приложений»

***
ТРЕБОВАНИЯ
***

## Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор
1. Имеется приложение, состоящее из нескольких реплик, которое требуется обновить.
2. Ресурсы, выделенные для приложения, ограничены, и нет возможности их увеличить.
3. Запас по ресурсам в менее загруженный момент времени составляет 20%.
4. Обновление мажорное, новые версии приложения не умеют работать со старыми.
5. Вам нужно объяснить свой выбор стратегии обновления приложения.

## Задание 2. Обновить приложение
1. Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Количество реплик — 5.
2. Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.
3. Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.
4. Откатиться после неудачного обновления.

## Задание 3*. Создать Canary deploymen
1. Создать два deployment'а приложения nginx.
2. При помощи разных ConfigMap сделать две версии приложения — веб-страницы.
3. С помощью ingress создать канареечный деплоймент, чтобы можно было часть трафика перебросить на разные версии приложения.
***
ВЫПОЛНЕНИЕ
***
## Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор
1. Recreate - самая банальная и простая стратегия, которая тупо убъет старые реплики, а потом создаст новые. Подходит для наших требований, но не оставляет нам никаких вариантов для устранения возможных проблем с работоспсобность новыйх версий и гарантирует простой в момент обновления. К применению этой стратегии следут прибегать в самый крайних случаях.
2. RollingUpdate - стандартная стратегия в кубере, она идеально подходит под те требавания, которые описаны в задании, только параметр maxSurge необходимо у становить в 20%-15%. Используюя эту стратегию мы гаратнируем, что работа будет проведена без простоя, сначала поднимутся новые реплики в 20%-15% соотношении от текущего кол-ва, они пройдут стандартную проверку Rediness, Liveless и если все хорошо, постепенно переедем на новую версию. Считаю эту стратегию наиболее походящей под требования.
3. A/B, Canary - неплохие стратегии, но они не подхоят под требования, т.к. ресурсы ограничены и мы не можем использовать сверх 20% от текущих рабочих ресурсов.

## Задание 2. Обновить приложение
1. Использование стратегии RollingUpdate
````
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 5
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
       maxSurge: 1
       maxUnavailable: 1
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
        image: wbitt/network-multitool:latest
        ports:
        - containerPort: 8080
        - containerPort: 6443
        env:
        - name: HTTP_PORT
          value: "8080"
        - name: HTTPS_PORT
          value: "6443"
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
        - containerPort: 443

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl apply -f deploy.yaml
deployment.apps/nginx created

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl get deploy,po
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   5/5     5            5           2m54s

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-586cfbc9d5-2jbjk   2/2     Running   0          2m53s
pod/nginx-586cfbc9d5-h5lzd   2/2     Running   0          2m53s
pod/nginx-586cfbc9d5-lh22c   2/2     Running   0          2m53s
pod/nginx-586cfbc9d5-lj4sx   2/2     Running   0          2m53s
pod/nginx-586cfbc9d5-s2vpv   2/2     Running   0          2m53s

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl annotate deployment/nginx kubernetes.io/change-cause="First create with nginx image 1.19" --overwrite=true
deployment.apps/nginx annotated

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl rollout history deploy/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         First create with nginx image 1.19

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl set image deploy/nginx nginx=nginx:1.20
deployment.apps/nginx image updated

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl get deploy,po
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   5/5     5            5           6m5s

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-7475b965c7-4pmdn   2/2     Running   0          45s
pod/nginx-7475b965c7-4tnm7   2/2     Running   0          44s
pod/nginx-7475b965c7-cgxvz   2/2     Running   0          36s
pod/nginx-7475b965c7-nj9v5   2/2     Running   0          28s
pod/nginx-7475b965c7-cmg2k   2/2     Running   0          19s

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl annotate deployment/nginx kubernetes.io/change-cause="Update nginx image to version 1.20"
deployment.apps/nginx annotated

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl rollout history deploy/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         First create with nginx image 1.19
2         Update nginx image to version 1.20

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl set image deploy/nginx nginx=nginx:1.28
deployment.apps/nginx image updated

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl get deploy,po
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   4/5     2            4           9m37s

NAME                         READY   STATUS             RESTARTS   AGE
pod/nginx-7475b965c7-4pmdn   2/2     Running            0          4m17s
pod/nginx-7475b965c7-4tnm7   2/2     Running            0          4m16s
pod/nginx-7475b965c7-cgxvz   2/2     Running            0          4m8s
pod/nginx-7475b965c7-nj9v5   2/2     Running            0          4m
pod/nginx-5557d8bffc-tfb7g   1/2     ImagePullBackOff   0          86s
pod/nginx-5557d8bffc-hzg67   1/2     ImagePullBackOff   0          87s

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl annotate deployment/nginx kubernetes.io/change-cause="Update nginx image to version 1.28"
deployment.apps/nginx annotated

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl rollout history deploy/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         First create with nginx image 1.19
2         Update nginx image to version 1.20
3         Update nginx image to version 1.28

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl rollout undo deploy/nginx --to-revision=2
deployment.apps/nginx rolled back

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl get deploy,po
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   5/5     5            5           13m

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-7475b965c7-4pmdn   2/2     Running   0          8m36s
pod/nginx-7475b965c7-4tnm7   2/2     Running   0          8m35s
pod/nginx-7475b965c7-cgxvz   2/2     Running   0          8m27s
pod/nginx-7475b965c7-nj9v5   2/2     Running   0          8m19s
pod/nginx-7475b965c7-kfnxh   2/2     Running   0          48s


vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl annotate deployment/nginx kubernetes.io/change-cause="Return to nginx image version 1.20"
deployment.apps/nginx annotated

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl rollout history deploy/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         First create with nginx image 1.19
3         Update nginx image to version 1.28
4         Return to nginx image version 1.20
````
2. Если ресурсы позволяют, то еще более безопасный вариант, это Blue/Green
````
vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ cat AB-deploy.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-blue
  labels:
    version: blue
spec:
  replicas: 2
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
       maxSurge: 1
       maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        version: blue
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
        volumeMounts:
          - name: blue-nginx
            mountPath: /usr/share/nginx/html/
      volumes:
      - name: blue-nginx
        configMap:
          name: blue-nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-green
  labels:
    version: green
spec:
  replicas: 2
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
       maxSurge: 1
       maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        version: green
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
        volumeMounts:
          - name: green-nginx
            mountPath: /usr/share/nginx/html/
      volumes:
      - name: green-nginx
        configMap:
          name: green-nginx

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ cat AB-istio.yaml
---
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: nginx-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx
spec:
  hosts:
  - "*"
  gateways:
  - nginx-gateway
  http:
  - route:
    - destination:
        host: nginx-svc
        subset: green
      weight: 100
    - destination:
        host: nginx-svc
        subset: blue
      weight: 0
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: blue-green-dest
spec:
  host: nginx-svc
  subsets:
  - name: blue
    labels:
      version: blue
  - name: green
    labels:
      version: green

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl get po,svc,deploy
NAME                               READY   STATUS    RESTARTS   AGE
pod/nginx-green-68d4448c57-7dmsv   1/1     Running   0          32m
pod/nginx-green-68d4448c57-957x2   1/1     Running   0          32m
pod/nginx-blue-69cd65f789-l5rrr    1/1     Running   0          32m
pod/nginx-blue-69cd65f789-68d2d    1/1     Running   0          32m

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP   7d8h
service/nginx-svc    ClusterIP   10.152.183.125   <none>        80/TCP    32m

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-green   2/2     2            2           32m
deployment.apps/nginx-blue    2/2     2            2           32m

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl get gw,dr,vs
NAME                                        AGE
gateway.networking.istio.io/nginx-gateway   4h8m

NAME                                                  HOST        AGE
destinationrule.networking.istio.io/nginx             nginx       8h
destinationrule.networking.istio.io/blue-green-dest   nginx-svc   141m

NAME                                       GATEWAYS            HOSTS   AGE
virtualservice.networking.istio.io/nginx   ["nginx-gateway"]   ["*"]   4h7m

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl get svc/istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                                                      AGE
istio-ingressgateway   LoadBalancer   10.152.183.184   192.168.0.105   15021:30631/TCP,80:30928/TCP,443:32218/TCP,31400:31590/TCP,15443:32259/TCP   9h
vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ curl 192.168.0.105:80
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! It is green version </h1>
</html

Меняем в VirtualService вес трафика, переключаем с green на blue

  - route:
    - destination:
        host: nginx-svc
        subset: green
      weight: 0
    - destination:
        host: nginx-svc
        subset: blue
      weight: 100

Применяем кофниг и отправляем запрос еще раз

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ kubectl apply -f AB-istio.yaml
gateway.networking.istio.io/nginx-gateway unchanged
virtualservice.networking.istio.io/nginx configured
destinationrule.networking.istio.io/blue-green-dest unchanged

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_depapp$ curl 192.168.0.105:80
<html>
<h1>Welcome</h1>
</br>
<h1>Hi! It is blue version </h1>
</html
````

# Домашнее задание к занятию «Как работает сеть в K8s»

## Задание 1. Создать сетевую политику или несколько политик для обеспечения доступа

1. Создать deployment'ы приложений frontend, backend и cache и соответсвующие сервисы.
2. В качестве образа использовать network-multitool.
3. Разместить поды в namespace App.
4. Создать политики, чтобы обеспечить доступ frontend -> backend -> cache. Другие виды подключений должны быть запрещены.
5. Продемонстрировать, что трафик разрешён и запрещён.

***

## Результат выполнения задания

1. Создаем  неймспейс app, deployment'ы и сервисы
````
vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_netpolicy$ kubectl get deploy,po,svc -n app
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/frontend   2/2     2            2           8h
deployment.apps/backend    2/2     2            2           80m
deployment.apps/cache      2/2     2            2           79m

NAME                           READY   STATUS    RESTARTS       AGE
pod/frontend-cfc4d9cbc-pmr48   1/1     Running   1 (112m ago)   8h
pod/frontend-cfc4d9cbc-8b8s9   1/1     Running   1 (112m ago)   8h
pod/backend-76ffff8c9-5pvdb    1/1     Running   0              80m
pod/backend-76ffff8c9-pnjtm    1/1     Running   0              80m
pod/cache-845dc896-84l89       1/1     Running   0              79m
pod/cache-845dc896-dh8qm       1/1     Running   0              79m

NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/frontend   ClusterIP   10.152.183.221   <none>        80/TCP    67m
service/backend    ClusterIP   10.152.183.198   <none>        80/TCP    65m
service/cache      ClusterIP   10.152.183.128   <none>        80/TCP    65m
````
2. Создаем политики для доступа
````
vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_netpolicy$ kubectl get networkpolicies -n app
NAME               POD-SELECTOR   AGE
deny-all-ingress   <none>         47m
back-cache         app=cache      11m
front-back         app=back       19m

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: app
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: front-back
  namespace: app
spec:
  podSelector:
    matchLabels:
      app: back
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: front
  policyTypes:
  - Ingress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: back-cache
  namespace: app
spec:
  podSelector:
    matchLabels:
      app: cache
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: back
  policyTypes:
  - Ingress
````
3. Проверяем доступы
````
vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_netpolicy$ kubectl exec -n app po/frontend-cfc4d9cbc-pmr48  -it -- /bin/sh
/ # curl backend
WBITT Network MultiTool (with NGINX) - backend-76ffff8c9-pnjtm - 10.1.52.138 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
/ # curl cache
^C

vagrant@vagrant:~/Netology_homeworks/kubernetes/kuber_netpolicy$ kubectl exec -n app po/backend-76ffff8c9-5pvdb  -it -- /bin/sh
/ # curl frontend
^C
/ # curl cache
WBITT Network MultiTool (with NGINX) - cache-845dc896-dh8qm - 10.1.52.140 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
```` 

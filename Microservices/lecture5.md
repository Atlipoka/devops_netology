Установил microk8s и cubectl используя инструкции из домашнего задания, произвел начальную настройку.
***
Microk8s
1. Установить на ВМ
```
vagrant@vagrant:~$ microk8s version
MicroK8s v1.27.2 revision 5372
```
2. Установить dasboard
```
vagrant@vagrant:~$ microk8s enable dashboard
```
3. Для генерации сертификата сначала активируем адон и и далее следуем подсказакам из вывода команды
```
vagrant@vagrant:~$ microk8s enable cert-manager
...
microk8s kubectl apply -f - <<EOF
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: fisrtkey
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: kabaev.maksin@yandex.ru
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: letsencrypt-account-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: public
EOF
```
***


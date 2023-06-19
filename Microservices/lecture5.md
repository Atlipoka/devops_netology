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
vagrant@vagrant:~$ microk8s kubectl apply -f - <<EOF
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
Cubectl

1. Установить на ВМ
```
vagrant@vagrant:~$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.2", GitCommit:"7f6f68fdabc4df88cfea2dcf9a19b2b830f1e647", GitTreeState:"clean", BuildDate:"2023-05-17T14:20:07Z", GoVersion:"go1.20.4", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1
```
2. Для корретной настройки подключения к кластеру необходимо задать значение для переменной KUBECONFIG. Я сделал так, нашел где лешит кофниг кластера mickrok8s и копировал конфигурационный файл в папку .kube в домашней директорииЮ получилось так - cp /var/snap/microk8s/current/credentials/client.config ~/.kube/config. После этого нзначил переменную - export KUBECONFIG=~/.kube/config
3. Для проброса порта использовал комнду
```
vagrant@vagrant:~$ microk8s kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443
Forwarding from [::1]:10443 -> 8443
```
Но проброс порта не может заверншиться кооретно и висит в статусе выполнения до тех пор, покая не заверншу команды вучную.
Не подскажете, с чем это может быть связано и как поправить?

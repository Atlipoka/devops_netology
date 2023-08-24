### Задание 1. Создайте конфигурацию для подключения пользователя
---

1. Создайте и подпишите SSL-сертификат для подключения к кластеру.
````
vagrant@vagrant:~/kubernetes/kuber_Enter$ openssl genrsa -out kabaev.key 2048
vagrant@vagrant:~/kubernetes/kuber_Enter$ openssl req -new -key kabaev.key -out kabaev.csr
vagrant@vagrant:~/kubernetes/kuber_Enter$ openssl x509 -req -in kabaev.csr -CA /var/snap/microk8s/current/certs/ca.crt -CAkey /var/snap/microk8s/current/certs/ca.key -CAcreateserial -out kabaev.crt -days 500
````
2. Настройте конфигурационный файл kubectl для подключения.
 - Последовав примеру из лекции, создал две переменные $AUTORITU_DATA и $SERVER для упрощения работы, конечно, можно было создать переменные для всех необходимых значений, но я пока ограничился двумя.
````
AUTORITU_DATA=$(kubectl get secret/kabaev-secret -n netology-test -o=jsonpath="{.data.ca\.crt}")
SERVER=$(kubectl config view -o=jsonpath="{.clusters[0].cluster.server}")

cat << EOF > sa-config-kube.conf
---
apiVersion: v1
kind: Config
clusters:
  - cluster:
      certificate-authority-data: $AUTORITU_DATA
      server: $SERVER
    name: microk8s-cluster
contexts:
  - context:
      cluster: microk8s-cluster
      user: kabaev
    name: kabaev-context
current-context: kabaev-context
preferences: {}
users:
  - name: kabaev
    user:
      client-certificate: /home/vagrant/kubernetes/kuber_Enter/kabaev.crt
      client-key: /home/vagrant/kubernetes/kuber_Enter/kabaev.key
EOF

vagrant@vagrant:~/kubernetes/kuber_Enter$ cat sa-config-kube.conf
---
apiVersion: v1
kind: Config
clusters:
  - cluster:
      certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUREekNDQWZlZ0F3SUJBZ0lVYmRKSWFmRnhIdENTOWFwVU14S2hWdml0OVdZd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0Z6RVZNQk1HQTFVRUF3d01NVEF1TVRVeUxqRTRNeTR4TUI0WERUSXpNRGN6TVRBNU5EWXdPVm9YRFRNegpNRGN5T0RBNU5EWXdPVm93RnpFVk1CTUdBMVVFQXd3TU1UQXVNVFV5TGpFNE15NHhNSUlCSWpBTkJna3Foa2lHCjl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF2blJuaEJZWU1rVitpYXNFdGhrcjdqb205QndJQmd2SmoraHMKVE5paDI4MlNiSzBMbG1Zbm1rVFFMSEhhMzJ3a0xsbWdSSndESmhEb0JPVGJxWDlDTWdjb080bFlYU2VFNG9RWQovUXY2UlQ3YmdPdm5GVXRCYVByckkzb3RtQ2thVGtTU3NJaEVuNk5TTTFDRzhWS04xU1JXTTRaVllvN2d2SHZOCmo1T0NWWUN4TW4xVm12dHNnNUlFWk5RTzRtZFoxWEpKYVk2MUY0elRpYUtiKzNaWmsxL085aGNiaDdoM2VmdFAKMjJGNFNUQTBqNi9EeVNUbFVQSU9FcXVmcGg0QmlvTmZaYzQ0bWViMWk4QzJtVVlRQmpET1o4VUQyRXVMblVpVQpsTi82YkM4SlZuQTR2c09Hend3OWlERFF2TEdXRGx3cDZuck8zblFJK0hYRjNHSUpyd0lEQVFBQm8xTXdVVEFkCkJnTlZIUTRFRmdRVVUrWWhzenVJU2lYUElXSjNNeWJSWEVLMGttQXdId1lEVlIwakJCZ3dGb0FVVStZaHN6dUkKU2lYUElXSjNNeWJSWEVLMGttQXdEd1lEVlIwVEFRSC9CQVV3QXdFQi96QU5CZ2txaGtpRzl3MEJBUXNGQUFPQwpBUUVBQUp4VHJqdTVJQjZHSHpVK0ZBUUMzMGdUdVVKc3ZNeGU3RlR4Rk40NENjZ0VWMjg2eGQzSDAwa0hPWEcyCjczYTZPVzc3czEzSmk5OUl3bWl1YnJDRWZTMU9QNG1rSUoxTGdrM1ZUS0kzQlRLQkthQkI2aXRONFRxR0RPUXIKMUtKeE5nL2dCbEJsdVNmam5mcGhZMGkvS25yS0Rlb09JcW8vQmdoTUFQZ0RVQUtXbzlqVW9QR2hmQ2FDWmlnNQpXWldFalJud1BDNWw5MDNudEd0ZzBRZU9DbWRaZ0N4K3QrMXVkMVJsNElJWGNuQ1FCWjl2VEFZZDdxNVB5N0V3ClAxMFp3WUgvL1ZMVnlpYzFuZzg5L2crMEFiUU16NUxCZDYzSGFPRVhuWkgxNm9WWTV5cloxa1FVTndFRGt5ZVEKNFhobUtGWTgvMVFPYzlmUDlRVG1vVU5UMlE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
      server: https://127.0.0.1:16443
    name: microk8s-cluster
contexts:
  - context:
      cluster: microk8s-cluster
      user: kabaev
    name: kabaev-context
current-context: kabaev-context
preferences: {}
users:
  - name: kabaev
    user:
      client-certificate: /home/vagrant/kubernetes/kuber_Enter/kabaev.crt
      client-key: /home/vagrant/kubernetes/kuber_Enter/kabaev.key
````
3. Создайте роли и все необходимые настройки для пользователя. Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию (kubectl logs pod <pod_id>, kubectl describe pod <pod_id>).
 - ВОПРОС!
Не знаю почему, пока не разобрался, при создании сервисного аккаунта, конфигурационного файла и сертификатов я успользовал имя Юзера с маленькой буквы kabaev, но когда пытался получить информацию о подах из под УЗ kabaev, получил ошибку, что у УЗ Kanaev(почему-то тут УЗ с большой буквы) нет доступа. При добавлении роли и биндинга я знатно намучался, пока не прописаль УЗ с большой буквы. Не сталкивались с таким?
Error from server (Forbidden): pods "hello-world" is forbidden: User "Kabaev" cannot get resource "pods/log" in API group "" in the namespace "netology-test"
````
vagrant@vagrant:~/kubernetes/kuber_Enter$ nano role.yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: viewer
  namespace: netology-test
rules:
  - apiGroups: ["", "extensions", "apps"]
    resources: ["pods","pods/log"]
    verbs: ["get", "list", "watch"]

vagrant@vagrant:~/kubernetes/kuber_Enter$ nano binding.yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: binding
  namespace: netology-test
subjects:
- kind: User
  name: Kabaev
  namespace: netology-test
  apiGroup: ""
roleRef:
  kind: Role
  name: viewer
  apiGroup: ""

Поднял Pod под админской УЗ и проверил все полученные доступы.

vagrant@vagrant:~/kubernetes/kuber_Enter$ kubectl --kubeconfig=sa-config-kube.conf get po/hello-world -n netology-test
NAME          READY   STATUS    RESTARTS   AGE
hello-world   1/1     Running   0          26m

vagrant@vagrant:~/kubernetes/kuber_Enter$ kubectl --kubeconfig=sa-config-kube.conf logs po/hello-world -n netology-test
Generating self-signed cert
Generating a 2048 bit RSA private key
..........+++
..................................................................................................................................................................+++
writing new private key to '/certs/privateKey.key'
-----
Starting nginx

vagrant@vagrant:~/kubernetes/kuber_Enter$ kubectl --kubeconfig=sa-config-kube.conf describe po/hello-world -n netology-test
Name:             hello-world
Namespace:        netology-test
Priority:         0
Service Account:  default
Node:             vagrant/10.0.2.15
Start Time:       Thu, 24 Aug 2023 08:37:47 +0000
Labels:           <none>
Annotations:      cni.projectcalico.org/containerID: dc46e4dc4db5f8a11184ac257086067ea0ec7030d0c1489be95fa2a8e3c69244
                  cni.projectcalico.org/podIP: 10.1.52.151/32
                  cni.projectcalico.org/podIPs: 10.1.52.151/32
Status:           Running
IP:               10.1.52.151
IPs:
  IP:  10.1.52.151
Containers:
  hello-world:
````

## Задание 1. Установить кластер k8s с 1 master node

Требования:
 * Подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды.
 * В качестве CRI — containerd.
 * Запуск etcd производить на мастере.
 * Способ установки выбрать самостоятельно.
***
Инфарстурктуру развернул с помощью терраформа [infa](https://github.com/Atlipoka/devops_netology/tree/main/Kuber_and_Azure/vm.tf), а виртуалки засетапил используя ансибл [setup](https://github.com/Atlipoka/devops_netology/tree/main/Kuber_and_Azure/site.yml). После уже пробовал выполнить первое задание с 1 матсером и 4 рабочими нодами.
1. Инициируем мастер ноду
````
ubuntu@master-1:~$ sudo kubeadm init \
> --apiserver-advertise-address=10.244.0.28 \
> --pod-network-cidr 10.244.0.0/16 \
> --apiserver-cert-extra-sans=130.193.37.176
[init] Using Kubernetes version: v1.28.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.244.0.28:6443 --token 0w8p4r.cyssw2br407z6etg \
        --discovery-token-ca-cert-hash sha256:ee8df28ea92cb500f16cbabb63d9e141405d9194bf363b5ac5443c9aa86fdde0
````
2. Идем по воркер нодам и подключаем их к мастеру
````
ubuntu@worker-1:~$ sudo kubeadm join 10.244.0.28:6443 --token 0w8p4r.cyssw2br407z6etg \
> --discovery-token-ca-cert-hash sha256:ee8df28ea92cb500f16cbabb63d9e141405d9194bf363b5ac5443c9aa86fdde0
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
...
ubuntu@worker-2:~$ sudo kubeadm join 10.244.0.28:6443 --token 0w8p4r.cyssw2br407z6etg \
> --discovery-token-ca-cert-hash sha256:ee8df28ea92cb500f16cbabb63d9e141405d9194bf363b5ac5443c9aa86fdde0
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
...
ubuntu@worker-3:~$ sudo kubeadm join 10.244.0.28:6443 --token 0w8p4r.cyssw2br407z6etg \
> --discovery-token-ca-cert-hash sha256:ee8df28ea92cb500f16cbabb63d9e141405d9194bf363b5ac5443c9aa86fdde0
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
...
ubuntu@worker-4:~$ sudo kubeadm join 10.244.0.28:6443 --token 0w8p4r.cyssw2br407z6etg --discovery-token-ca-cert-hash sha256:ee8df28ea92cb500f16cbabb63d9e141405d9194bf363b5ac5443c9aa86fdde0
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
````
3. Возвращаемся на масте, ставим сетевой драйвер и радуемся
````
ubuntu@master-1:~$ kubectl get nodes
NAME       STATUS     ROLES           AGE     VERSION
master-1   NotReady   control-plane   5m1s    v1.28.2
worker-1   NotReady   <none>          3m39s   v1.28.2
worker-2   NotReady   <none>          2m54s   v1.28.2
worker-3   NotReady   <none>          2m26s   v1.28.2
worker-4   NotReady   <none>          22s     v1.28.2
ubuntu@master-1:~$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
namespace/kube-flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created

ubuntu@master-1:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
master-1   Ready    control-plane   5m42s   v1.28.2
worker-1   Ready    <none>          4m20s   v1.28.2
worker-2   Ready    <none>          3m35s   v1.28.2
worker-3   Ready    <none>          3m7s    v1.28.2
worker-4   Ready    <none>          63s     v1.28.2
````
## Задание 2. Установить HA кластер

Требования:
 * Установить кластер в режиме HA.
 * Использовать нечётное количество Master-node.
 * Для cluster ip использовать keepalived или другой способ.
***
И вот тут я застрял. Щас покажу где. Настройки keepalived [master](https://github.com/Atlipoka/devops_netology/edit/main/Kuber_and_Azure/keepalived-master.conf) and [slave](https://github.com/Atlipoka/devops_netology/edit/main/Kuber_and_Azure/keepalived-slave.conf), настройки хапроски [haproxy.cfg](https://github.com/Atlipoka/devops_netology/edit/main/Kuber_and_Azure/haproxy.cfg)

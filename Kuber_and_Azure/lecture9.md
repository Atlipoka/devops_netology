# Домашнее задание к занятию «Кластеры. Ресурсы под управлением облачных провайдеров»

### Цели задания
1. Организация кластера Kubernetes и кластера баз данных MySQL в отказоустойчивой архитектуре.
2. Размещение в private подсетях кластера БД, а в public — кластера Kubernetes.
***

## Задание 1. Yandex Cloud
1. Настроить с помощью Terraform кластер баз данных MySQL.
 * Используя настройки VPC из предыдущих домашних заданий, добавить дополнительно подсеть private в разных зонах, чтобы обеспечить отказоустойчивость.
 * Разместить ноды кластера MySQL в разных подсетях.
 * Необходимо предусмотреть репликацию с произвольным временем технического обслуживания.
 * Использовать окружение Prestable, платформу Intel Broadwell с производительностью 50% CPU и размером диска 20 Гб.
 * Задать время начала резервного копирования — 23:59.
 * Включить защиту кластера от непреднамеренного удаления.
 * Создать БД с именем netology_db, логином и паролем.
2. Настроить с помощью Terraform кластер Kubernetes.
 * Используя настройки VPC из предыдущих домашних заданий, добавить дополнительно две подсети public в разных зонах, чтобы обеспечить отказоустойчивость.
 * Создать отдельный сервис-аккаунт с необходимыми правами.
 * Создать региональный мастер Kubernetes с размещением нод в трёх разных подсетях.
 * Добавить возможность шифрования ключом из KMS, созданным в предыдущем домашнем задании.
 * Создать группу узлов, состояющую из трёх машин с автомасштабированием до шести.
 * Подключиться к кластеру с помощью kubectl.
 * *Запустить микросервис phpmyadmin и подключиться к ранее созданной БД.
 * *Создать сервис-типы Load Balancer и подключиться к phpmyadmin. Предоставить скриншот с публичным адресом и подключением к БД.
 ***
 ## Выполнение

1. Создаем с помощью Terraform кластер баз данных MySQ с необходимыми параметрами
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$ cat network.tf
resource "yandex_vpc_network" "network" {
  name = "network"
}

resource "yandex_vpc_subnet" "private1" {
  name           = "private1"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}

resource "yandex_vpc_subnet" "private2" {
  name           = "private2"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.network.id
  v4_cidr_blocks = ["192.168.20.0/24"]
}

resource "yandex_vpc_subnet" "private3" {
  name           = "private3"
  zone           = "ru-central1-c"
  network_id     = yandex_vpc_network.network.id
  v4_cidr_blocks = ["192.168.30.0/24"]
}

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$ cat mysql.tf
resource "yandex_mdb_mysql_cluster" "netology_cluster" {
  name                = "netology_cluster"
  environment         = "PRESTABLE"
  network_id          = yandex_vpc_network.network.id
  version             = "8.0"
  deletion_protection = "true"

  resources {
    resource_preset_id = "b1.medium"
    disk_type_id       = "network-ssd"
    disk_size          = 20
  }

  maintenance_window {
    type = "WEEKLY"
    day  = "TUE"
    hour = 23
  }

  backup_window_start {
    hours   = 23
    minutes = 59
  }

  host {
    name      = "db-1"
    zone      = "ru-central1-a"
    subnet_id = yandex_vpc_subnet.private1.id
    # assign_public_ip        = true
  }

  host {
    name                    = "db-2"
    zone                    = "ru-central1-b"
    replication_source_name = "db-1"
    subnet_id               = yandex_vpc_subnet.private2.id
    # assign_public_ip        = true
  }

  host {
    name                    = "db-3"
    zone                    = "ru-central1-c"
    replication_source_name = "db-2"
    subnet_id               = yandex_vpc_subnet.private3.id
    #assign_public_ip        = true
  }
}

resource "yandex_mdb_mysql_database" "netology-db" {
  cluster_id = yandex_mdb_mysql_cluster.netology_cluster.id
  name       = "netology_db"
}

resource "yandex_mdb_mysql_user" "netology" {
  cluster_id = yandex_mdb_mysql_cluster.netology_cluster.id
  name       = "netology"
  password   = "123456789"
  permission {
    database_name = "netology_db"
    roles         = ["ALL"]
  }
}
````
2. Создаем с помощью Terraform региональный мастер Kubernetes c группой узлов и проверяем работоспособность.
````
resource "yandex_vpc_network" "network" {
  name = "network"
}
...
resource "yandex_vpc_subnet" "public1" {
  name           = "public1"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network.id
  v4_cidr_blocks = ["10.5.0.0/16"]
}

resource "yandex_vpc_subnet" "public2" {
  name           = "public2"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.network.id
  v4_cidr_blocks = ["10.6.0.0/16"]
}

resource "yandex_vpc_subnet" "public3" {
  name           = "public3"
  zone           = "ru-central1-c"
  network_id     = yandex_vpc_network.network.id
  v4_cidr_blocks = ["10.7.0.0/16"]
}

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$ cat kms.tf 
resource "yandex_kms_symmetric_key" "kms-key" {
  name              = "kms-key"
  default_algorithm = "AES_128"
  rotation_period   = "1440h" # 1 год.
}

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$ cat k8-cluster.tf
resource "yandex_kubernetes_cluster" "k8s-regional" {
  name       = "k8s-regional"
  network_id = yandex_vpc_network.network.id
  master {
    version   = "1.27"
    public_ip = true

    regional {
      region = "ru-central1"
      location {
        zone      = yandex_vpc_subnet.public1.zone
        subnet_id = yandex_vpc_subnet.public1.id
      }

      location {
        zone      = yandex_vpc_subnet.public2.zone
        subnet_id = yandex_vpc_subnet.public2.id
      }

      location {
        zone      = yandex_vpc_subnet.public3.zone
        subnet_id = yandex_vpc_subnet.public3.id
      }
    }

    maintenance_policy {
      auto_upgrade = true

      maintenance_window {
        day        = "Wednesday"
        start_time = "23:00"
        duration   = "3h"
      }
    }
  }

  service_account_id      = "ajereg9535ct9lmk2cg2"
  node_service_account_id = "ajereg9535ct9lmk2cg2"

  kms_provider {
    key_id = yandex_kms_symmetric_key.kms-key.id
  }
}

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$ cat pods.tf
resource "yandex_kubernetes_node_group" "my_node_group" {
  cluster_id = yandex_kubernetes_cluster.k8s-regional.id
  name       = "netology"
  version    = "1.27"


  instance_template {
    platform_id = "standard-v2"

    metadata = {
      ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
    }

    network_interface {
      nat        = true
      subnet_ids = ["${yandex_vpc_subnet.public1.id}"]
    }

    resources {
      memory        = 2
      cores         = 2
      core_fraction = 20
    }

    boot_disk {
      type = "network-hdd"
      size = 64
    }

    scheduling_policy {
      preemptible = false
    }

    container_runtime {
      type = "containerd"
    }
  }

  scale_policy {
    auto_scale {
      min     = 2
      max     = 6
      initial = 3
    }
  }

  allocation_policy {
    location {
      zone = "ru-central1-a"
    }
  }

  maintenance_policy {
    auto_upgrade = true
    auto_repair  = true

    maintenance_window {
      day        = "Wednesday"
      start_time = "15:00"
      duration   = "3h"
    }
  }
}

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$ yc managed-kubernetes cluster get-credentials --id catq58mtrd4m1eo3jb58 --external


Context 'yc-k8s-regional' was added as default to kubeconfig '/home/vagrant/.kube/config'.
Check connection to cluster using 'kubectl cluster-info --kubeconfig /home/vagrant/.kube/config'.

Note, that authentication depends on 'yc' and its config profile 'netology'.
To access clusters using the Kubernetes API, please use Kubernetes Service Account.
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$ kubectl get pods

No resources found in default namespace.

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$ cat phpmyadmin.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: phpmyadmin-deployment
  labels:
    app: phpmyadmin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: phpmyadmin
  template:
    metadata:
      labels:
        app: phpmyadmin
    spec:
      containers:
        - name: phpmyadmin
          image: phpmyadmin/phpmyadmin
          ports:
            - containerPort: 80
          env:
            - name: PMA_HOST
              value: "rc1a-gldmr6mef5xs1pe9.mdb.yandexcloud.net"
            - name: PMA_PORT
              value: "3306"
            - name: MYSQL_ROOT_PASSWORD
              value: "123456789"

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$ cat service.yml
apiVersion: v1
kind: Service
metadata:
  name: phpmyadmin-service
spec:
  type: LoadBalancer
  selector:
    app: phpmyadmin
  ports:
  - name: http
    port: 80
    targetPort: 80

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture4$ kubectl get deploy,svc
NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/phpmyadmin-deployment   1/1     1            1           80s

NAME                         TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
service/kubernetes           ClusterIP      10.96.128.1    <none>          443/TCP        30m
service/phpmyadmin-service   LoadBalancer   10.96.191.67   158.160.129.6   80:30824/TCP   73s
````
 * ![screen1](https://github.com/Atlipoka/devops_netology/blob/main/Kuber_and_Azure/phpmysql.png)

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

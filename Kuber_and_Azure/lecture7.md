  # Домашнее задание к занятию «Вычислительные мощности. Балансировщики нагрузки»

## Задание 1. Yandex Cloud

#### Что нужно сделать

1. Создать бакет Object Storage и разместить в нём файл с картинкой:
 * Создать бакет в Object Storage с произвольным именем (например, имя_студента_дата).
 * Положить в бакет файл с картинкой.
 * Сделать файл доступным из интернета.
2. Создать группу ВМ в public подсети фиксированного размера с шаблоном LAMP и веб-страницей, содержащей ссылку на картинку из бакета:
 * Создать Instance Group с тремя ВМ и шаблоном LAMP. Для LAMP рекомендуется использовать image_id = fd827b91d99psvq5fjit.
 * Для создания стартовой веб-страницы рекомендуется использовать раздел user_data в meta_data.
 * Разместить в стартовой веб-странице шаблонной ВМ ссылку на картинку из бакета.
 * Настроить проверку состояния ВМ.
3. Подключить группу к сетевому балансировщику:
 * Создать сетевой балансировщик.
 * Проверить работоспособность, удалив одну или несколько ВМ.
4. (дополнительно)* Создать Application Load Balancer с использованием Instance group и проверкой состояния.

***

#### Выполнение

1. Распишу по порядку все операции с корзиной и результаты работы
 * Создаем Корзину
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ cat sb.tf
resource "yandex_iam_service_account_static_access_key" "sa-static-key" {
  service_account_id = "ajebtiua2igaepkna5i8"
  description        = "static access key for object storage"
}

resource "yandex_storage_bucket" "bucket-kabaev" {
  bucket     = "bucket-kabaev"
  access_key = yandex_iam_service_account_static_access_key.sa-static-key.access_key
  secret_key = yandex_iam_service_account_static_access_key.sa-static-key.secret_key
  acl        = "public-read-write"
  max_size   = 500000000
}
````
 * Проверяем параметры корзины, файлик уже загрузил через веб, получаем ссылку и проверяем, что объект доступен
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ yc storage bucket list
+---------------+----------------------+-----------+-----------------------+---------------------+
|     NAME      |      FOLDER ID       | MAX SIZE  | DEFAULT STORAGE CLASS |     CREATED AT      |
+---------------+----------------------+-----------+-----------------------+---------------------+
| bucket-kabaev | b1gjjlp1h6jc8jeaclal | 500000000 | STANDARD              | 2023-11-01 10:08:54 |
+---------------+----------------------+-----------+-----------------------+---------------------+

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ wget https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg
--2023-11-01 16:47:01--  https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg
Resolving storage.yandexcloud.net (storage.yandexcloud.net)... 213.180.193.243, 2a02:6b8::1d9
Connecting to storage.yandexcloud.net (storage.yandexcloud.net)|213.180.193.243|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 102817 (100K) [image/jpeg]
Saving to: ‘devops2.jpg’

devops2.jpg                                   100%[===============================================================================================>] 100.41K  --.-KB/s    in 0.003s

2023-11-01 16:47:01 (38.6 MB/s) - ‘devops2.jpg’ saved [102817/102817]

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ ll
total 184
drwxrwxr-x 3 vagrant vagrant   4096 Nov  1 16:47 ./
drwxrwxr-x 4 vagrant vagrant   4096 Oct 28 09:04 ../
-rw-rw-r-- 1 vagrant vagrant    971 Nov  1 13:32 cloud-init.yaml
-rw-rw-r-- 1 vagrant vagrant 102817 Nov  1 16:44 devops2.jpg
-rw-rw-r-- 1 vagrant vagrant   1250 Nov  1 15:02 ig.tf
-rw-rw-r-- 1 vagrant vagrant    259 Nov  1 14:18 network.tf
-rw-rw-r-- 1 vagrant vagrant    418 Nov  1 13:05 nlb.tf
-rw-rw-r-- 1 vagrant vagrant    178 Oct 28 09:29 provider.tf
-rw-rw-r-- 1 vagrant vagrant    499 Nov  1 14:23 sb.tf
drwxr-xr-x 3 vagrant vagrant   4096 Oct 28 09:29 .terraform/
-rw-r--r-- 1 vagrant vagrant    314 Oct 28 09:30 .terraform.lock.hcl
-rw-rw-r-- 1 vagrant vagrant  16413 Nov  1 15:37 terraform.tfstate
-rw-rw-r-- 1 vagrant vagrant  16553 Nov  1 15:37 terraform.tfstate.backup
````
2. Распишу по порядку все операции по созданию группы ВМ и результаты работы
 * Создаем группу объектов состоящую из 3х ВМ и указываем все необходимы параметры
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ cat ig.tf
resource "yandex_compute_instance_group" "group1" {
  name                = "group1"
  folder_id           = "b1gjjlp1h6jc8jeaclal"
  service_account_id  = "ajebtiua2igaepkna5i8"
  deletion_protection = false
  instance_template {
    platform_id = "standard-v1"
    name        = "vm-{instance.index}"
    resources {
      core_fraction = 20
      memory        = 1
      cores         = 2
    }
    boot_disk {
      initialize_params {
        image_id = "fd827b91d99psvq5fjit"
        size     = 4
      }
    }
    network_interface {
      network_id = "${yandex_vpc_network.network.id}"
      subnet_ids = ["${yandex_vpc_subnet.public1.id}"]
      nat        = true
    }

    metadata = {
      ssh-keys  = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
      user_data = "${file("cloud-init.yaml")}"
    }
  }

  scale_policy {
    fixed_scale {
      size = 3
    }
  }

  allocation_policy {
    zones = ["ru-central1-a"]
  }

  deploy_policy {
    max_unavailable = 1
    max_creating    = 1
    max_expansion   = 1
    max_deleting    = 1
  }

  health_check {
    interval = 30
    timeout  = 10
    tcp_options {
      port = 80
    }
  }

  load_balancer {
    target_group_name            = "tg1"
    max_opening_traffic_duration = 10
  }
}
````
 * Параметры сети и конфигурационны файл формата cloud-init
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ cat network.tf
resource "yandex_vpc_network" "network" {
  name = "network"
}

resource "yandex_vpc_subnet" "public1" {
  name           = "public1"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ cat cloud-init.yaml
#cloud-config
runcmd:
  - sudo su
  - hostn=$(cat /etc/hostname)
  - echo "<html><h1> Hi! it is a $hostn <p><a href="https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg">Click here</a> for download picture </p></h1></html>" > /var/www/html/index.html
users:
  - default
  - name: ubuntu
    groups: sudo
    shell: /bin/bash
    sudo: 'ALL=(ALL) NOPASSWD:ALL'
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc...

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ yc compute instance-group list
+----------------------+--------+--------+------+
|          ID          |  NAME  | STATUS | SIZE |
+----------------------+--------+--------+------+
| cl1po5vu3bdt9gchvqsv | group1 | ACTIVE |    3 |
+----------------------+--------+--------+------+

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ yc compute instance-group list-instances --id cl1po5vu3bdt9gchvqsv
+----------------------+------+---------------+---------------+------------------------+----------------+
|     INSTANCE ID      | NAME |  EXTERNAL IP  |  INTERNAL IP  |         STATUS         | STATUS MESSAGE |
+----------------------+------+---------------+---------------+------------------------+----------------+
| fhm5um51g8otluksc3nk | vm-1 | 51.250.84.196 | 192.168.10.37 | RUNNING_ACTUAL [1h57m] |                |
| fhmebk8nhsg1qa2j4b1k | vm-2 | 84.201.158.12 | 192.168.10.17 | RUNNING_ACTUAL [2h3m]  |                |
| fhmlo8m8vkv6cu16rltn | vm-3 | 62.84.113.29  | 192.168.10.8  | RUNNING_ACTUAL [1h47m] |                |
+----------------------+------+---------------+---------------+------------------------+----------------+
````
 * У меня возникли проблемы при передаче в метадате комманд для настройки стартовой станицы, runcmd вообще не запускаеться на ВМ, поэтому, пришлось вручую настраивать машины, чтоб отображалась корректная стартовая веб-страница
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ curl 51.250.84.196
<html><h1> Hi! it is a vm-1 <p><a href=https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg>Click here</a> for download picture </p></h1></html>
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ curl 84.201.158.12
<html><h1> Hi! it is a vm-2 <p><a href=https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg>Click here</a> for download picture </p></h1></html>
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ curl 62.84.113.29
<html><h1> Hi! it is a vm-3 <p><a href=https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg>Click here</a> for download picture </p></h1></html>
````
3. Распишу по порядку все операции по созданию сетевого балансировщика и результаты работы
 * Целевую группу создали в п.2, когда инициализировали группу ВМ
````
...
load_balancer {
    target_group_name            = "tg1"
    max_opening_traffic_duration = 10
  }
...
````
 * Теперь создаем сетевой балансировщик
````
resource "yandex_lb_network_load_balancer" "nlb" {
  name = "network-load-balancer"

  listener {
    name = "listener"
    port = 80
    external_address_spec {
      ip_version = "ipv4"
    }
  }

  attached_target_group {
    target_group_id = "${yandex_compute_instance_group.group1.load_balancer[0].target_group_id}"

    healthcheck {
      name = "tcp"
      tcp_options {
        port = 80
      }
    }
  }
}

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ yc load-balancer nlb get --name=network-load-balancer
id: enprir8hqdti53isfdqp
folder_id: b1gjjlp1h6jc8jeaclal
created_at: "2023-11-01T13:11:24Z"
name: network-load-balancer
region_id: ru-central1
status: ACTIVE
type: EXTERNAL
listeners:
  - name: listener
    address: 51.250.6.216
    port: "80"
    protocol: TCP
    target_port: "80"
    ip_version: IPV4
attached_target_groups:
  - target_group_id: enpa3s7u1ads951um37n
    health_checks:
      - name: tcp
        interval: 2s
        timeout: 1s
        unhealthy_threshold: "2"
        healthy_threshold: "2"
        tcp_options:
          port: "80"
````
 * Проверяем как рабоает балансировщик, тущим пару виртуалок и проверяем еще раз
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ curl 51.250.6.216
<html><h1> Hi! it is a vm-4 <p><a href=https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg>Click here</a> for download picture </p></h1></html>
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ curl 51.250.6.216
<html><h1> Hi! it is a vm-2 <p><a href=https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg>Click here</a> for download picture </p></h1></html>
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ curl 51.250.6.216
<html><h1> Hi! it is a vm-2 <p><a href=https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg>Click here</a> for download picture </p></h1></html>

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ yc compute instance list
+----------------------+------+---------------+---------+---------------+---------------+
|          ID          | NAME |    ZONE ID    | STATUS  |  EXTERNAL IP  |  INTERNAL IP  |
+----------------------+------+---------------+---------+---------------+---------------+
| fhm5um51g8otluksc3nk | vm-1 | ru-central1-a | RUNNING | 51.250.84.196 | 192.168.10.37 |
| fhmebk8nhsg1qa2j4b1k | vm-2 | ru-central1-a | RUNNING | 84.201.158.12 | 192.168.10.17 |
| fhmlo8m8vkv6cu16rltn | vm-4 | ru-central1-a | RUNNING | 62.84.113.29  | 192.168.10.8  |
+----------------------+------+---------------+---------+---------------+---------------+

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ yc compute instance stop fhmebk8nhsg1qa2j4b1k
done (17s)
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ yc compute instance list
+----------------------+------+---------------+---------+---------------+---------------+
|          ID          | NAME |    ZONE ID    | STATUS  |  EXTERNAL IP  |  INTERNAL IP  |
+----------------------+------+---------------+---------+---------------+---------------+
| fhm5um51g8otluksc3nk | vm-1 | ru-central1-a | RUNNING | 51.250.84.196 | 192.168.10.37 |
| fhmebk8nhsg1qa2j4b1k | vm-2 | ru-central1-a | STOPPED |               | 192.168.10.17 |
| fhmlo8m8vkv6cu16rltn | vm-4 | ru-central1-a | RUNNING | 62.84.113.29  | 192.168.10.8  |
+----------------------+------+---------------+---------+---------------+---------------+

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ curl 51.250.6.216
<html><h1> Hi! it is a vm-1 <p><a href=https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg>Click here</a> for download picture </p></h1></html>
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ curl 51.250.6.216
<html><h1> Hi! it is a vm-1 <p><a href=https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg>Click here</a> for download picture </p></h1></html>

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ yc compute instance stop --id fhm5um51g8otluksc3nk
done (18s)
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ yc compute instance list
+----------------------+------+---------------+---------+--------------+---------------+
|          ID          | NAME |    ZONE ID    | STATUS  | EXTERNAL IP  |  INTERNAL IP  |
+----------------------+------+---------------+---------+--------------+---------------+
| fhm5um51g8otluksc3nk | vm-1 | ru-central1-a | STOPPED |              | 192.168.10.37 |
| fhmebk8nhsg1qa2j4b1k | vm-2 | ru-central1-a | STOPPED |              | 192.168.10.17 |
| fhmlo8m8vkv6cu16rltn | vm-4 | ru-central1-a | RUNNING | 62.84.113.29 | 192.168.10.8  |
+----------------------+------+---------------+---------+--------------+---------------+

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture2$ curl 51.250.6.216
<html><h1> Hi! it is a vm-4 <p><a href=https://storage.yandexcloud.net/bucket-kabaev/devops2.jpg>Click here</a> for download picture </p></h1></html>
````

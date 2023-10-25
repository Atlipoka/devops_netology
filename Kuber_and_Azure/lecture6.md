# Домашнее задание к занятию «Организация сети»

## Задание 1. Yandex Cloud
Что нужно сделать

1. Создать пустую VPC. Выбрать зону.
2. Публичная подсеть.
 * Создать в VPC subnet с названием public, сетью 192.168.10.0/24.
 * Создать в этой подсети NAT-инстанс, присвоив ему адрес 192.168.10.254. В качестве image_id использовать fd80mrhj8fl2oe87o4e1.
 * Создать в этой публичной подсети виртуалку с публичным IP, подключиться к ней и убедиться, что есть доступ к интернету.
3. Приватная подсеть.
 * Создать в VPC subnet с названием private, сетью 192.168.20.0/24.
 * Создать route table. Добавить статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс.
 * Создать в этой приватной подсети виртуалку с внутренним IP, подключиться к ней через виртуалку, созданную ранее, и убедиться, что есть доступ к интернету.

### Выполнение
***

1. В ходе выполнения создал 3 файла network.tf (содержит все настройки сети), sg.tf (настройки группы безопасности) и vm.tf (настройки виртуальных машин)
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture1/terraform$ cat network.tf
resource "yandex_vpc_network" "network" {
  name = "network"
}

resource "yandex_vpc_subnet" "public" {
  name           = "public"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}

resource "yandex_vpc_subnet" "private" {
  name           = "private"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network.id
  v4_cidr_blocks = ["192.168.20.0/24"]
  route_table_id = yandex_vpc_route_table.nat-instance-route.id
}

resource "yandex_vpc_route_table" "nat-instance-route" {
  name       = "nat-instance-route"
  network_id = yandex_vpc_network.network.id
  static_route {
    destination_prefix = "0.0.0.0/0"
    next_hop_address   = yandex_compute_instance.nat-instance.network_interface.0.ip_address
  }
}

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture1/terraform$ cat sg.tf
resource "yandex_vpc_security_group" "nat-instance-sg" {
  name       = "security"
  network_id = yandex_vpc_network.network.id

  egress {
    protocol       = "ANY"
    description    = "any"
    v4_cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    protocol       = "TCP"
    description    = "ssh"
    v4_cidr_blocks = ["0.0.0.0/0"]
    port           = 22
  }

  ingress {
    protocol       = "TCP"
    description    = "http"
    v4_cidr_blocks = ["0.0.0.0/0"]
    port           = 80
  }

  ingress {
    protocol       = "TCP"
    description    = "https"
    v4_cidr_blocks = ["0.0.0.0/0"]
    port           = 443
  }
}

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture1/terraform$ cat vm.tf 
# create vm in public web
resource "yandex_compute_instance" "public-vm" {
  name     = "public-vm"
  hostname = "public-vm"

  resources {
    core_fraction = 20
    cores         = 2
    memory        = 1
  }

  boot_disk {
    initialize_params {
      image_id = "fd8o6khjbdv3f1suqf69"
      size     = 5
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.public.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }
}

# create vm in private web without public IP
resource "yandex_compute_instance" "private-vm-1" {
  name     = "private-vm-1"
  hostname = "private-vm-1"

  resources {
    core_fraction = 20
    cores         = 2
    memory        = 1
  }

  boot_disk {
    initialize_params {
      image_id = "fd8o6khjbdv3f1suqf69"
      size     = 5
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.private.id
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }
}

# create vm with in private web with NAT
resource "yandex_compute_instance" "private-vm-2" {
  name     = "private-vm-2"
  hostname = "private-vm-2"

  resources {
    core_fraction = 20
    cores         = 2
    memory        = 1
  }

  boot_disk {
    initialize_params {
      image_id = "fd8o6khjbdv3f1suqf69"
      size     = 5
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.private.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }
}

# create nat instance
resource "yandex_compute_instance" "nat-instance" {
  name     = "nat-instance"
  hostname = "nat-instance"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd80mrhj8fl2oe87o4e1"
      size     = 5
    }
  }

  network_interface {
    subnet_id  = yandex_vpc_subnet.public.id
    nat        = true
    ip_address = "192.168.10.254"
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }
}
````

2. В результате было создано 4 виртуальных машины и проверены все настройки сети, которые были указаны в задании. Подготовку уже произвел, показываю результат.
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture1/terraform$ yc compute instance list
+----------------------+--------------+---------------+---------+-----------------+----------------+
|          ID          |     NAME     |    ZONE ID    | STATUS  |   EXTERNAL IP   |  INTERNAL IP   |
+----------------------+--------------+---------------+---------+-----------------+----------------+
| fhm2svrfrb1bm7fa27ta | public-vm    | ru-central1-a | RUNNING | 51.250.75.166   | 192.168.10.27  |
| fhmqm2um9nq4e16853ks | private-vm-1 | ru-central1-a | RUNNING |                 | 192.168.20.9   |
| fhms73c6vvafmgjh6iu9 | private-vm-2 | ru-central1-a | RUNNING | 158.160.106.140 | 192.168.20.19  |
| fhmtfua2p7g12r2k36cs | nat-instance | ru-central1-a | RUNNING | 158.160.123.208 | 192.168.10.254 |
+----------------------+--------------+---------------+---------+-----------------+----------------+

ubuntu@public-vm:~$ ping -c 5 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=58 time=19.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=58 time=19.3 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=58 time=19.4 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=58 time=19.4 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=58 time=19.4 ms

--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 19.239/19.352/19.424/0.066 ms

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture1/terraform$ ssh ubuntu@158.160.106.140
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-163-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
New release '22.04.3 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Wed Oct 25 11:40:45 2023 from 95.24.60.148
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@private-vm-2:~$ ssh ubuntu@192.168.20.9
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-163-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
New release '22.04.3 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Wed Oct 25 11:43:01 2023 from 192.168.20.19
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@private-vm-1:~$ ping -c 5 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=54 time=19.9 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=54 time=19.0 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=54 time=19.0 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=54 time=19.1 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=54 time=19.0 ms

--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 18.979/19.200/19.939/0.370 ms
````

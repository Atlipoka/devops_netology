1. Ранее мы уже использовали терраформ на лекциях по фиртуализации, поэтому процесс создания описывать не буду, начну с того, что касается Терраформа:
 * Сервисный аккуант уже был создан с ролью editor
    * ```
      root@vagrant:/home/vagrant/terraform# yc iam service-account list
      +----------------------+-------+
      |          ID          | NAME  |
      +----------------------+-------+
      | ajeqqv8273vi2m4uf1h5 | maxim |
      +----------------------+-------+
      ```
  * Создал iam ключ для сервисного аккаунта
      * ```
        yc iam key create \
        --service-account-id ajeqqv8273vi2m4uf1h5 \
        --folder-name b1gjjlp1h6jc8jeaclal \
        --output key.json
        ```
   * Создал CLI профиль
      * ```
        root@vagrant:/home/vagrant/terraform# yc config profile list
        default
        terraform ACTIVE
        ```
   * Присвоил значения для CLI профиля ``yc config set service-account-key key.json && yc config set cloud-id b1g9tskfs4f4itprsns9 && yc config set folder-id b1gjjlp1h6jc8jeaclal``
   * В рабочей папке создал файл main.tf и внес в него данные
      * ```
        terraform {
         required_providers {
            yandex = {
               source = "yandex-cloud/yandex"
            }
          }
         required_version = ">= 0.13"
        }
        ```
   * Cоздал файл с переменными var.tf и файл provisers.tf, где указал данные о провайдере yc
      * ```
        variable "yc_token" {
         default = ""
        }

        variable "yc_cloud_id" {
         default = ""
        }

        variable "yc_folder_id" {
         default = ""
        }

        variable "yc_region" {
         default = "ru-central1-a"
        }
        ```
       * ```
         provider "yandex" {
          token                    = var.yc_token
          cloud_id                 = var.yc_cloud_id
          folder_id                = var.yc_folder_id
          zone                     = var.yc_region
         }
         ```
2. Отвечу по порядку.
   * Если я правильно понял, то создать собственный образ можно при помощи утилиты Packer.
   * Заполнил план создания инфраструктуры, все исходники тут - [ЖМИ](https://github.com/Atlipoka/devops_netology/tree/main/Cloud_Terraform/lecture2/terraform)
   * Результат выполнения ``terraform plan``
```
root@vagrant:/home/vagrant/terraform# terraform plan
data.yandex_compute_image.ubuntu_image: Reading...
data.yandex_compute_image.ubuntu_image: Read complete after 0s [id=fd8kdq6d0p8sij7h5qe3]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_compute_instance.vm-test1 will be created
  + resource "yandex_compute_instance" "vm-test1" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + hostname                  = (known after apply)
      + id                        = (known after apply)
      + name                      = "test1"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd8kdq6d0p8sij7h5qe3"
              + name        = (known after apply)
              + size        = (known after apply)
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy {
          + host_affinity_rules = (known after apply)
          + placement_group_id  = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 2
        }

      + scheduling_policy {
          + preemptible = (known after apply)
        }
    }

  # yandex_vpc_network.network_terraform will be created
  + resource "yandex_vpc_network" "network_terraform" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "net_terraform"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.subnet_terraform will be created
  + resource "yandex_vpc_subnet" "subnet_terraform" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "sub_terraform"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.15.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

Plan: 3 to add, 0 to change, 0 to destroy.

```

1. Ранее мы уже инициализировали терраформ, поэтому процесс создания описывать не буду, начну с того, что касается Терраформа:
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
    * Присвоил значения для параметров ``yc config set service-account-key key.json && yc config set cloud-id b1g9tskfs4f4itprsns9 && yc config set folder-id b1gjjlp1h6jc8jeaclal``
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

         provider "yandex" {
            zone = "ru-central1-a"
        }
        ```
        

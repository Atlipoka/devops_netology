1. Распишу по порядку:
  * Создал корзину и назначил ей роль и сервисный аккаунт ``yc resource-manager folder add-access-binding netology --service-account-name netology --role admin``
  * Зарегестрировал корзину в терраформе:
 ```
 terraform {
 required_providers {
  yandex = {
    source = "yandex-cloud/yandex"
  }
}
required_version = ">= 0.13"

backend "s3" {
  endpoint   = "storage.yandexcloud.net"
  bucket     = "netology"
  region     = "ru-central1"
  key        = "terraform.tfstate"
  access_key = "YCA..."
  secret_key = "YCM..."

  skip_region_validation      = true
  skip_credentials_validation = true
}
}


provider "yandex" {
  token     = var.yc_token
  cloud_id  = var.yc_cloud_id
  folder_id = var.yc_folder_id
  zone      = var.yc_region
}
 ```

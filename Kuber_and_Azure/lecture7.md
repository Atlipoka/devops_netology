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

1. Распишу по порядку все операции с корзиной
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
 * вддпдапжв 

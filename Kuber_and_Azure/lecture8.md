# Домашнее задание к занятию «Безопасность в облачных провайдерах»

## Задание 1. Yandex Cloud
1. С помощью ключа в KMS необходимо зашифровать содержимое бакета:
 * создать ключ в KMS;
 * с помощью ключа зашифровать содержимое бакета, созданного ранее.
2. (Выполняется не в Terraform)* Создать статический сайт в Object Storage c собственным публичным адресом и сделать доступным по HTTPS:
 * создать сертификат;
 * создать статическую страницу в Object Storage и применить сертификат HTTPS;
 * в качестве результата предоставить скриншот на страницу с сертификатом в заголовке (замочек).
***
## Выполнение

1. Создаем корзину, ключ и загружаем объект без шифрования, затем добаваляем параметры шифрования в виде kms ключа
 * Создаем корзину, ключ ( но не применяем его к корзине) и загружаем объект в корзину.
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture3$ cat bucket_kms.tf
resource "yandex_iam_service_account_static_access_key" "sa-static-key" {
  service_account_id = "ajebtiua2igaepkna5i8"
  description        = "static access key for object storage"
}

resource "yandex_kms_symmetric_key" "kms" {
  name              = "kms"
  description       = "First kms key for bucket"
  default_algorithm = "AES_128"
  rotation_period   = "80h"
  }

resource "yandex_storage_bucket" "kabaev-bucket" {
  bucket     = "kabaev-bucket"
  access_key = yandex_iam_service_account_static_access_key.sa-static-key.access_key
  secret_key = yandex_iam_service_account_static_access_key.sa-static-key.secret_key
  acl        = "public-read-write"
  }

resource "yandex_storage_object" "devops" {
  bucket = "${yandex_storage_bucket.kabaev-bucket.bucket}"
  access_key = yandex_iam_service_account_static_access_key.sa-static-key.access_key
  secret_key = yandex_iam_service_account_static_access_key.sa-static-key.secret_key
  key    = "devops2.jpg"
  source = "./devops2.jpg"
  acl    = "public-read-write"
  }

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture3$ yc kms symmetric-key list
+----------------------+------+----------------------+-------------------+---------------------+--------+
|          ID          | NAME |  PRIMARY VERSION ID  | DEFAULT ALGORITHM |     CREATED AT      | STATUS |
+----------------------+------+----------------------+-------------------+---------------------+--------+
| abjhm71esbl8gqi3iuri | kms  | abj7v0crd089tat129qs | AES_128           | 2023-11-08 14:16:35 | ACTIVE |
+----------------------+------+----------------------+-------------------+---------------------+--------+

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture3$ yc storage bucket list
+---------------+----------------------+----------+-----------------------+---------------------+
|     NAME      |      FOLDER ID       | MAX SIZE | DEFAULT STORAGE CLASS |     CREATED AT      |
+---------------+----------------------+----------+-----------------------+---------------------+
| kabaev-bucket | b1gjjlp1h6jc8jeaclal |        0 | STANDARD              | 2023-11-08 14:16:35 |
+---------------+----------------------+----------+-----------------------+---------------------+


````
 * Теперь добавляем параметры шифрования к корзине
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture3$ cat bucket_kms.tf
...
resource "yandex_storage_bucket" "kabaev-bucket" {
  bucket     = "kabaev-bucket"
  access_key = yandex_iam_service_account_static_access_key.sa-static-key.access_key
  secret_key = yandex_iam_service_account_static_access_key.sa-static-key.secret_key
  acl        = "public-read-write"

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        kms_master_key_id = yandex_kms_symmetric_key.kms.id
        sse_algorithm     = "aws:kms"
      }
    }
  }
}
...
````
 * По поводу шифрования уже созданных файлов в бакете. Прочитав раздел о выборе способа шифрования, в документации [YC](https://cloud.yandex.ru/docs/kms/tutorials/encrypt/), я нашел способ ширования данных через YC CLI, но для файлов, размер которых меньше чем 32 КБ. Создал тестовый
файл и зашифровал его, затем, расшифровал повторно.
````
vagrant@vagrant:~/Netology_homeworks/Cloud/lecture3$ cat text.txt
Hello!
I want encrypt this file use kms key in YC.
hahahahhahaha!

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture3$ yc kms symmetric-crypto encrypt   --name kms   --plaintext-file ./text.txt   --ciphertext-file ./text.txt
key_id: abjhk6v75usgmf7klrff
version_id: abjmocoegi09hdpp7udu

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture3$ cat text.txt
abjmocoegi09hdpp7udu►▒◄1▒       ▒t▒►▒▒(8H7J▒▒,#G▒▒%]▒no-▒م▒zh▒▒▒▒n▒?2:k▒▒▒Y▒^▒↓X(▒Dj▒
T▒▒▒[▒(▒x.+Fb_▒ /"▒J▒0▒▒o▒▒'▒▒▒▒PS(9▒Zɣ▒

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture3$ yc kms symmetric-crypto decrypt   --name kms   --plaintext-file ./text.txt   --ciphertext-file ./text.txt
key_id: abjhk6v75usgmf7klrff
version_id: abjmocoegi09hdpp7udu

vagrant@vagrant:~/Netology_homeworks/Cloud/lecture3$ cat text.txt
Hello!
I want encrypt this file use kms key in YC.
hahahahhahaha!
````
2. Создаем статический сайт в YC на базе Object Starage, который будет доступен по HTTPS
 * Создаем сетрификат (я создал два для HTTP и DNS типом проверки)
![HTTP-SERT](https://github.com/Atlipoka/devops_netology/blob/main/Kuber_and_Azure/sert-http.png)
![DNS-SERT](https://github.com/Atlipoka/devops_netology/blob/main/Kuber_and_Azure/sert-dns.png)
 * Так-же, следуя инструкциям, я создал файлы для проверки в случае с http сертификатом и необходимые dns записи, в случае с dns сертификатом
![HTTP-VALID](https://github.com/Atlipoka/devops_netology/blob/main/Kuber_and_Azure/http-valid.png)
![DNS-VALID](https://github.com/Atlipoka/devops_netology/blob/main/Kuber_and_Azure/dns-valid.png)
* Пока сертификаты в статусе pending. Уже висят так в теечении 2х часов, подскажите, как долго они валидируются??

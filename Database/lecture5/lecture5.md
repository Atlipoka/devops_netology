1. Рапишу по шагам все этапы:
  * Текст Dockerfile:
  ```
   # Указываем базовый образ
   FROM elasticsearch:7.17.6
   # Копируем конфигурационный файл
   COPY elasticsearch.yml /usr/share/elasticsearch/config
   # Присваиваем права на директорию /var пользоваетелю elasticsearch
   RUN chown -R elasticsearch /var
   # создаем пользователя elasticsearch
   USER elasticsearch
   # открываем порты
   EXPOSE 9200 9300
  ```
  * Ссылка на созданный образ в DockerHub https://hub.docker.com/layers/maximkabaev/devops_netology/elasticsearch/images/sha256-f61d891c89427fe8e756dfec4ab714c107940cab8603e7bde40820bea9ad0e92?context=explore
  * Запрос и ответ от elasticsearch по пути ``/``:
    * ```
      root@vagrant:/home/vagrant/virt-homeworks/06-db-05-elasticsearch# curl localhost:9200
{
  "name" : "node1",
  "cluster_name" : "netology_test",
  "cluster_uuid" : "vDXlimNrRAqGEpn4sFy_Vg",
  "version" : {
    "number" : "7.17.6",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "f65e9d338dc1d07b642e14a27f338990148ee5b6",
    "build_date" : "2022-08-23T11:08:48.893373482Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
    ```

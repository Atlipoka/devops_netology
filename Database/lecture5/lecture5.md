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
    * ![task1](https://github.com/Atlipoka/devops_netology/blob/main/Database/lecture5/task1.png)
2. Распишу по шагам:
 * Создал индексы и получил результат используя Postman
   * ```
     green  open .geoip_databases N9sq5GuEQQ2UceE1Yn7ofQ 1 0 41 0 39.1mb 39.1mb
     yellow open ind-1            9BK0sEK_Ta2n0HWgArC_GQ 1 1  0 0   226b   226b
     yellow open ind-3            wJeqFPidQYyvo2PQaTWUHA 1 1  0 0   226b   226b
     yellow open ind-2            zFZkO4smQOmGfa_IYPsAlg 1 1  0 0   226b   226b
     ```
  * Состояние кластера получил по пути 
    * ```
      {
      "cluster_name": "netology_test",
      "status": "yellow",
      "timed_out": false,
      "number_of_nodes": 1,
      "number_of_data_nodes": 1,
      "active_primary_shards": 6,
      "active_shards": 6,
      "relocating_shards": 0,
      "initializing_shards": 0,
      "unassigned_shards": 3,
      "delayed_unassigned_shards": 0,
      "number_of_pending_tasks": 0,
      "number_of_in_flight_fetch": 0,
      "task_max_waiting_in_queue_millis": 0,
      "active_shards_percent_as_number": 66.66666666666666
       }```

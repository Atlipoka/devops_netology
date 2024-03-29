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
 * Создал индексы и получил результат по пути ``localhost:9200/_cat/indices``
   * ```
     green  open .geoip_databases N9sq5GuEQQ2UceE1Yn7ofQ 1 0 41 0 39.1mb 39.1mb
     yellow open ind-1            9BK0sEK_Ta2n0HWgArC_GQ 1 1  0 0   226b   226b
     yellow open ind-3            wJeqFPidQYyvo2PQaTWUHA 1 1  0 0   226b   226b
     yellow open ind-2            zFZkO4smQOmGfa_IYPsAlg 1 1  0 0   226b   226b
     ```
  * Состояние кластера получил по пути ``localhost:9200/_cluster/health``
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
       }
       ```
    * Кластер в статусе yellow потому, что 3 secondary шарда находяться в статусе unassigned.
3. Распишу по шагам:
 * Зарегестрировал репозиторий
   * ```
        root@vagrant:/usr/share/elasticsearch/snapshots# curl -X PUT "localhost:9200/_snapshot/netology_backup?verify=false&pretty" -H 'Content-Type: applica
        tion/json' -d'{"type": "fs","settings": {"location": "/usr/share/elasticsearch/snapshots"}}'
        {
        "acknowledged" : true
        }
     ``` 
 * Создал индекс test
   * ```
     yellow open test   9jdzujwbSJqNTledprrrdg 1 1 0 0 226b 226b
     ```
 * Создал снапшот, результат
   * ```
     elasticsearch@f60728af3f22:~/snapshots$ ll
     total 60
     drwxr-xr-x 1 elasticsearch root           4096 Nov  1 19:20 ./
     drwxrwxr-x 1 root          root           4096 Nov  1 19:19 ../
     -rw-rw-r-- 1 elasticsearch elasticsearch  1416 Nov  1 19:20 index-0
     -rw-rw-r-- 1 elasticsearch elasticsearch     8 Nov  1 19:20 index.latest
     drwxrwxr-x 6 elasticsearch elasticsearch  4096 Nov  1 19:20 indices/
     -rw-rw-r-- 1 elasticsearch elasticsearch 29340 Nov  1 19:20 meta-jMD_sR0IQcmnWvhi_dzKPg.dat
     -rw-rw-r-- 1 elasticsearch elasticsearch   703 Nov  1 19:20 snap-jMD_sR0IQcmnWvhi_dzKPg.dat
     ```
 * Удалил индекс test и создал индекс test-2
   * ```
     yellow open test-2 xkXsmszWSI65PF0QxGla2w 1 1 0 0 226b 226b
     ```
  * Восстановил индекс и получил итоговый список индексов
    * ```
      localhost:9200/_snapshot/netology_backup/snap/_restore
      {
      "indices": "test"
      }
      ```
     * ```
       yellow open test-2 xkXsmszWSI65PF0QxGla2w 1 1 0 0 226b 226b
       yellow open test   9jdzujwbSJqNTledprrrdg 1 1 0 0 226b 226b
       ```

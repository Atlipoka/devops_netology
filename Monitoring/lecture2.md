1. Скриншот поднятого контейнера с Grafana (использовал директорию help) с подключенным источником данных Prometeus
 * ![task1](https://github.com/Atlipoka/devops_netology/blob/main/Monitoring/lecture2-taks1.png)
2. Запросы и скриншоты получившихся панелей на дашборде Base Metrics
 * Free RAM - node_memory_MemFree_bytes
 * FileSystem size (/dev/sda1) - node_filesystem_size_bytes{device="/dev/sda1",fstype="vfat",mountpoint="/boot/efi"}
 * CPU LA (1,5,15) - node_load1,node_load5,node_load15
 * CPU usage in percent - 100 - (avg by (instance) (irate(node_cpu_seconds_total{job="nodeexporter",mode="idle"}[5m])) * 100)
 * ![task2](https://github.com/Atlipoka/devops_netology/blob/main/Monitoring/lecture2-taks2.png)
3. Скриншот полсе настройки алертов на панелях дашборда и настроенный канал нотификации
 * ![task3-1](https://github.com/Atlipoka/devops_netology/blob/main/Monitoring/lecture2-taks3-1.png)
 * ![task3-2](https://github.com/Atlipoka/devops_netology/blob/main/Monitoring/lecture2-taks3-2.png)
4. Json макет созданного дашборда
 * [JSON](https://github.com/Atlipoka/devops_netology/blob/main/Monitoring/Dashboard_JsonModel.json)

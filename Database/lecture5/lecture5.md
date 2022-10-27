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

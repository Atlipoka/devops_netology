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

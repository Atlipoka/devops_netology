Ответы на вопросы по порядку:
 1. Для мониторинга работы системы вижу следующие метрики:
  * Загрузка ЦПУ.
  * Загрузка и потребление RAM.
  * Кол-во занятого и свободного места на диске.
  * Кол-во информационных(100х), успешных(200х), перенаправления(300х) и неуспешных(400х и 500х ошибок) ответов от сервера.
  * И еще я бы подумал над кастомной метрикой, которая собирала бы инфрмацию по кол-ву успешных и неуспешных сформированных отчетов.
 2. Как я и писал выше, бизнес заказчику важно понимать базовые вещи и можно предложить настроить кол-во успешных или не успешных обращений к сервису. Так-же дополнить это и считать кол-во успешных или неуспешных сформированных отчетов. Еще можно добавить простой чекер, для  провеки работоспособности сервиса, например, если в течении 2-х минут от сервиса не поступает ответов, то это может означать что наблюдаютьс некие проблемы.
 3. В случае отсутсвия буджета на систему логирования, можно использовать беспланые решения или контейнерные приложения, например Zabbix.
 4. В данном случае не все коды учитываються в формуле summ_2xx_requests/summ_all_requests. Необходимо добавить в мониторинг дополнительные коды 100х и 300х, тогда картины станет более понятной и будет видно, реальный процент.
 5. О плюсах и минусах Pull и Push моделей
   * Плюсы Push Систем:
     * Cкорость передачи данных можно сократить используя протокол UDP.
     * Гибкая настройка отправки данных на мастер сервер.
   * Минусы Push Систем:
     * Если отваливаеться агент на сервере, то сбор логов прекращается и необходим досуп на сервер для отладки работы агента и выяснения причин, почему логи не отправляются.
     * При выборе UDP портокола отправки данных может происходить потеря данных, что никак и ничем не защищено.
     * Базово, в данной модели не предустмотрено безопасное взаимодействие агентов с сервером через единый Proxy.
   * Плюсы Pull Систем:
     * Легко и удобно контролировать все точки сбора логов.
     * Можно настроить единый proxy для всех агентов.
     * Можно запросить логи в любое удобно время, если это необходимо.
     * Узнать о проблеме с агентом можно прямо из интерфейса системы, что ускоряет процесс устранения неисправности.
   * Минусы Pull Систем:
     * Использует более медленный протоко отправки данных TCP
     * Более сложная система сбора данных с серверов.
6. Типы систем и их модели сбора данных
 * Prometheus - гибридная модель.
 * TICK - push модель.
 * Zabbix - гибридная модель.
 * VictoriaMetrics - гибридная модель.
 * Nagios - pull модель.
7. Прикладываю скриншот выполнения
 * ![Task-7](https://github.com/Atlipoka/devops_netology/blob/main/Monitoring/lecture1-task7.png)
8. Прикладываю скриншот выполнения
 * ![Task-8](https://github.com/Atlipoka/devops_netology/blob/main/Monitoring/lecture1-task8-1.png)
9. Прикладываю скриншот выполнения
 * ![Task-9](https://github.com/Atlipoka/devops_netology/blob/main/Monitoring/lecture1-task9-1.png)
10. Доп. задание:
 * Скрипт
```
# usr/bin/python3

## Import block
import datetime
import json
import re
import subprocess

## Variables block
log = datetime.datetime.now().strftime("%Y-%m-%d-awesome-monitoring.log")
dt = datetime.datetime.now()
dt = str(dt)

## Set in variables data in needed format
with open(log,'a') as date:
   date.write('\n'+dt+'\n')

## Get info about Memory and write it in log file
with open('/proc/meminfo','r',buffering=5) as f1, open(log,'a') as f2:
    f1.seek(0)
    f3=f1.read(55)
    f3=f3.replace('\n','","')
    f3=f3.replace(' ','')
    f3=f3.replace(':','": "')
    f2.write('{"Memory_Info": ["'+f3+'"]}'+'\n')

## Get info about CPU and write it in log file
with open('/proc/loadavg','r') as f4, open(log,'a') as f5:
    f6=f4.read(25)
    f6=f6.replace(' ','","')
    f5.write('{"CPU_Info": ["'+f6+'"]}'+'\n')

## Get info about Disk usage and write it in log file
with open('/proc/diskstats','r') as f7, open(log,'a') as f8:
    f7.seek(405)
    f9=f7.read(73)
    f9 =f9.replace(' ','","')
    f8.write('{"Disk_Info": ["'+f9+'"]}'+'\n')

## Get info about uptime and write it in log file
with open('/proc/uptime','r') as f10, open(log,'a') as f11:
    f12=f10.read(8)
    f11.write('{"UpTime": "'+f12+'"}')
```
 * Cron расписание - * * * * * python3 /home/vagrant/mem.py
 * Как импорт выглядит в файле - ![Task-10](https://github.com/Atlipoka/devops_netology/blob/main/Monitoring/lecture7-task10.png)


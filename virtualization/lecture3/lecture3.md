1. Рапишу по порядку как выполнял задание:
  * Зарегестрировал аккаунт на https://hub.docker.com и засетапил на виртуальную машину докер используя официальную инструкцию https://docs.docker.com/engine/install/ubuntu/. Проверил запустив первый контейнер - ``docker run hello-world``. Привязал акк на докерхабе к локальному репозиторию - ``docker login --username maximkabaev``. 
  * Скачал официальный образ nginx -``docker pull nginx``
  * Создал докерфайл с HTML кодом и сделал форк образа - ``docker build -f dockerfile.txt -t maximkabaev/devops_netology:v1.0 .``
  * Проброс портов ``docker run -d -p 8080:80 maximkabaev/devops_netology:v1.0``
  * Cсылка на репозиторий - https://hub.docker.com/repository/docker/maximkabaev/devops_netology.

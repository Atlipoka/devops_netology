1. Образ создан, скриншот
 * ![Task1](https://github.com/Atlipoka/devops_netology/blob/main/virtualization/lecture4/Lecture4-Task1.png)
2. Виртуальная машина создана, распишу по шагам:
 * Создал сервисный аккаунт с правами editor
  * ![Task2-1](https://github.com/Atlipoka/devops_netology/blob/main/virtualization/lecture4/Lecture4-task2-1.png)
 * Создал IAM склю для сервисного аккаунта и сложил его в key.json - ``yc iam key create --service-account-name maxim --output key.json``
 * Отредактировал все конфигурационные файлы, которые были указаны в лекции, проверил конфиг на виладность - ``terraform validate``, проверил план работ - ``terraform plan``, запустил создание ВМ - ``terraform apply -auto-approve``.
  * ![Task2-2](https://github.com/Atlipoka/devops_netology/blob/main/virtualization/lecture4/Lecture4-task2-2.png)

1. Отвечу по порядку:
 * Отличие режимов replicated и global в том, что в режиме global, доставка производиться на каждую ноду, а в режиме replicated означает, что n-ое кол-во контейнеров для данного сервиса будет запущено на всех доступных нодах.
 * Определение первой manager ноды производиться при вводе команды docker swarm init. После ввода команды нода, с которой был инициализирован запрос, автоматически становиться manager. Для дальнейшего управления ролями используются команды docker node promote 'dns name node' (для добавления роли manager) и docker nome demote 'dns name node' (для удаления роли manager).
 * Overlay сети позволяют демонам докера взаимодействовать друг с другом в режиме кластера, применяя дополнительную логическую надстройку над основной сетью.
2. Результат работ по развертыванию Docker Swarm кластера в Яндекс.Облаке
 * ![Task2](https://github.com/Atlipoka/devops_netology/blob/main/virtualization/lecture5/Lecture5-task2.png)
3. Ниже вывод стека сервисов в кластере Docker Swarm
 * ![Task3](https://github.com/Atlipoka/devops_netology/blob/main/virtualization/lecture5/Lecture5-task3.png)
4. После применения команды на лидере docker swarm update --autolock=true была активирована функция автоматического блокировки роя, для разблокировки необходимо вводить  TLS ключ, который выдается просле ввода команды. После перезапуска сервиса docker каждая управляющая нода блокируется и доступ управляющим нода можео получить введя TLS ключ. Для разблокировки необходимо использовать команду docker swarm unlock, после чего необходимо будет ввести ключ шифрования.
 * ![Task4](https://github.com/Atlipoka/devops_netology/blob/main/virtualization/lecture5/Lecture5-task4.png)

1. При выполнении команды скрипта из задания 1, мы получим ошибку, т.к. пытаемся применить операцию сложения между строкой и числом.
 * Для получения значения 12 необходимо назначить переменные a и b строкового типа  a = '1', b = '2', тогда при выполнении c = a + b, переменная с будет равна 12
 * Для получения значения 3 необходимо назначить переменные a и b числового типа  a = 1, b = 2, тогда при выполнении c = a + b, переменная с будет равна 3.
2. Ниже прикладываю свой вариант скрипта. Хотел бы уточнить все ли правильно выполнено, т.к. задание было относительно локального репозитория и я не стал добавлять неотселиваемые файлы. Если есть заменчания то прошу пометить как было бы лучше или в какую сторону посмотреть, может вообще применить другую логику для скрипта.
 *        #!/usr/bin/env python3
          import os
          bash_command = ["cd /home/vagrant/Python_practice", "git status"]
          result_os = os.popen(' && '.join(bash_command)).read()
          is_change = False
          for result in result_os.split('\n'):
          if result.find('new file') != -1:
                prepare_result = result.replace('\tnew file: ','Directory: '+ os.path.abspath('') + ' New file: ')
                print(prepare_result)
          if result.find('modified') != -1:
                prepare_result = result.replace('\tmodified: ','Directory: '+ os.path.abspath('') + ' Modified: ')
                print(prepare_result)
                break

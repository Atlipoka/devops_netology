1. При просмотре сисетмных вызово командой /bin/bash -c 'cd /tmp' вызов команды cd будет выглядеть как chdir ("/tmp").
2. Изсходя из анализа данных и чтения документации, БД команды file это magic.mgc, файл находится по пути - /usr/share/misc/magic.mgc
3. В лекции была информация о способе восстановления файла, мне кажется она подходит для данной ситуации. Если при приложение пишет лог в файл, то вероятно оно запущено и имеет pid процесса, необходимо данный pid найти (любой командой ps, pidoff, pstree), далее через lfos убратиться к процессу с указание имени файла и убедиться что файл дейстивтельно удален, пометка об этом будет, далее мы увидим инфомацию о файле и дескрипторе, котрый используется для направления потока в лог-файл и перенаправить данный поток в другой файл. Скриншоты - ![Screen task 3 exaple 1](https://github.com/Atlipoka/devops_netology/blob/main/OperationSystem/OS-lecture1-task3_example1.png), ![Screen task 3 exaple 2](https://github.com/Atlipoka/devops_netology/blob/main/OperationSystem/OS-lecture1-task3_example2.png)
4. Зомби процессы не занимают никаких ресурсов.
5. Первыми команндами open было открытие библиотек по пути /lib/x86-64-lixun-gnu/{libselinux.so.1,libc.so.6,libpcre2-8.so.0}
6. В документации к команде указано, что информация та-же дотупна по пути /proc/sys/kernel/{ostype, hostname, osrelease, version,domainname}. Полная цитата - Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version,domainname}.
7. Послдеовательность команд через ; и через && разная, при перечислении команд при момощи ; , команды буут выполняться последовательно не обращая внимания на результаты предыдущих команд,а вот если при перечислении команд использовать &&, то в данном случае команды не будут выполняться последователно, если одна из команд завершится с ошибкой. Если использовать команду set -e с &&, то ошибочные команды будут игнорироваться и продолжат выполняться последовательно.
8. Аргументы команды set -euxo pipefail:
    * -e завершение команды, если возвращается ненулевое значение.
    * -u интерпретирование неустановленных переменных как ошибочные.
    * -х отображение коанд с аргументами во время выполнения сценария.
    * -o включение всех режимов работы или активизация определенного.
    * -pipefal заменят код завершения конвеера последней неудачной операцией или возвращает нулевой код, если все коанды завершились успешно.
 * Таким образом, команду set можно использовать для проверки и отладки сценариев.
9. Используя команду ps -o stat выяснил свои статысу процессов S,S,S,R+. В оазделе /PROCESS STATE CODES указаны следующие доп. параметры:
   * < высокий приоритет
   * N низкий приоритет
   * L страница забликированная в памяти
   * s лидер сессии
   * l многопоточный
   * (+) подгруппа основоной группы
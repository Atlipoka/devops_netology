2. Моя версия переделанного скритпа, которая мне кажется подходящей
 *      while ((1==1))
        do
        a=$(curl -s -o /dev/null -w "%{http_code}" https://ya.ru)
        if (($a==0))
        then
         ( echo "http code $a" ; date ) > /home/vagrant/curl.log
        fi

        if (($a!=0))
         then exit
        fi
        done
3. Скрипт ниже
 *      #!/usr/bin/env bash
        addres=(192.168.0.1:80 173.194.222.113:80 87.250.250.242:80)
        for a in {1..5}
        do
                for i in ${addres[@]}
                do
                ( echo "proverka $i" ; curl --max-time 5 $i; ) >> /home/vagrant/script2.log
                done
        done

### Как сдавать задания

Вы уже изучили блок «Системы управления версиями», и начиная с этого занятия все ваши работы будут приниматься ссылками на .md-файлы, размещённые в вашем публичном репозитории.

Скопируйте в свой .md-файл содержимое этого файла; исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-01-bash/README.md). Заполните недостающие части документа решением задач (заменяйте `???`, ОСТАЛЬНОЕ В ШАБЛОНЕ НЕ ТРОГАЙТЕ чтобы не сломать форматирование текста, подсветку синтаксиса и прочее, иначе можно отправиться на доработку) и отправляйте на проверку. Вместо логов можно вставить скриншоты по желани.

---


# Домашнее задание к занятию "4.1. Командная оболочка Bash: Практические навыки"

## Обязательная задача 1

Есть скрипт:
```bash
a=1
b=2
c=a+b
d=$a+$b
e=$(($a+$b))
```

Какие значения переменным c,d,e будут присвоены? Почему?

| Переменная  | Значение | Обоснование |
| ------------- | ------------- | ------------- |
| `c`  | а+b  | в данном случае не было указан символ переменной $, по сути, мы просто записали в переменную a+b |
| `d`  | 1+2  | хоть числа и целые, но арфиметической операции сложения не произошло потому, что для сложения целых чисел используется комманда вида $q=$(($e+$t)) |
| `e`  | 3  | в данном случае была исользована арфиметическая команда для сложения целых чисел |


## Обязательная задача 2
На нашем локальном сервере упал сервис и мы написали скрипт, который постоянно проверяет его доступность, записывая дату проверок до тех пор, пока сервис не станет доступным (после чего скрипт должен завершиться). В скрипте допущена ошибка, из-за которой выполнение не может завершиться, при этом место на Жёстком Диске постоянно уменьшается. Что необходимо сделать, чтобы его исправить:
```bash
while ((1==1)
do
	curl https://localhost:4757
	if (($? != 0))
	then
		date >> curl.log
	fi
done
```

### Ваш скрипт:
```bash
while ((1==1))
do
        a=$(curl -s -o /dev/null -w "%{http_code}" https://ya.ru)
        if (($a==0))
        then
         ( echo "http code $a" ; date ) > /home/vagrant/curl.log
        fi

        if (($a!=0))
         then exit
        fi
done
```

## Обязательная задача 3
Необходимо написать скрипт, который проверяет доступность трёх IP: `192.168.0.1`, `173.194.222.113`, `87.250.250.242` по `80` порту и записывает результат в файл `log`. Проверять доступность необходимо пять раз для каждого узла.

### Ваш скрипт:
```bash
???
```

## Обязательная задача 4
Необходимо дописать скрипт из предыдущего задания так, чтобы он выполнялся до тех пор, пока один из узлов не окажется недоступным. Если любой из узлов недоступен - IP этого узла пишется в файл error, скрипт прерывается.

### Ваш скрипт:
```bash
???
```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Мы хотим, чтобы у нас были красивые сообщения для коммитов в репозиторий. Для этого нужно написать локальный хук для git, который будет проверять, что сообщение в коммите содержит код текущего задания в квадратных скобках и количество символов в сообщении не превышает 30. Пример сообщения: \[04-script-01-bash\] сломал хук.

### Ваш скрипт:
```bash
???
```
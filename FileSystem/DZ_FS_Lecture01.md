1. Насколько я понял, что иметь хардлинки с разными правама и создателями нельзя. Все объясняется тем, что любой хардлинк файла является дубликатом файла, на который он ссылается и воспринимается системой как единое целое.
4. Используя интерактивный режим команды fdisk создал два раздела по 1Gb, скриншот - 
* ![Task4](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task4.png)
5. С помощью команды sfdisk создал скопировал таблицу разделов с /dev/sdb  и перенесс ее на /dev/sdc, для чтения тадлицы использовал команду sfdisk -d /dev/sdb > /home/vagrant/patritions-sdb.txt,  для записи ее на диск команду - sfdisk /dev/sdc < /home/vagrant/patritions-sdb.txt, скриншот -
* ![Task5](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task5.png)
6. Создал RAID1 с помощью mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdb2 (ниже общий скрриншот для задания 6 и 7)
7. Создал RAID0 с помощью mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdc1 /dev/sdc2 (ниже общий скрриншот для задания 6 и 7)
* 

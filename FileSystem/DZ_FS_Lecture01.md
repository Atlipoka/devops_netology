1. Насколько я понял, что иметь хардлинки с разными правама и создателями нельзя. Все объясняется тем, что любой хардлинк файла является дубликатом файла, на который он ссылается и воспринимается системой как единое целое.
4. Используя интерактивный режим команды fdisk создал два раздела по 1Gb, скриншот - 
* ![Task4](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task4.png)
5. С помощью команды sfdisk создал скопировал таблицу разделов с /dev/sdb  и перенесс ее на /dev/sdc, для чтения тадлицы использовал команду sfdisk -d /dev/sdb > /home/vagrant/patritions-sdb.txt,  для записи ее на диск команду - sfdisk /dev/sdc < /home/vagrant/patritions-sdb.txt, скриншот -
* ![Task5](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task5.png)
6. Создал RAID1 с помощью mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdb2 (ниже общий скрриншот для задания 6 и 7)
7. Создал RAID0 с помощью mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdc1 /dev/sdc2 (ниже общий скрриншот для задания 6 и 7)
* ![Task6,7](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task6,7.png)
8. С помощью команды pvcreate создал два независимы тома, использовал команд pvcreate /dev/md0 и pvcreate /dev/md1
* ![Task8](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task8.png)
9. Создал общую группу томов с помощью команды vgcreate vol_grp1 /dev/md0 /dev/md1
* ![Task9](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task9.png)
10. Создал логический том на 100 МБ с расположением на RAID0 lvcreate -L 100M vol_rgp1 /dev/md0
* ![Task10](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task10.png)
11. Создал файловую систему ext4 с помощью команды mkfs.ext4 /dev/vol_grp1/lvol0
* ![Task11](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task11.png)
12. Примонтировал раздел используя команду mount /dev/vol_grp1/lvol0 /tmp/new
* ![Task12](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task12.png)
13. Файл скачан
* ![Task13](https://github.com/Atlipoka/devops_netology/blob/main/FileSystem/FS-task13.png) 

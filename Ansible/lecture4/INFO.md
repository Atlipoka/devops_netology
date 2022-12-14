Плейбук site.yml разделен на 2 логические части:
 * Первая часть скачивает .deb пакеты clickhouse и устанавливает их, после создает БД в clickhouse servev.
 * Вторая часть плейбука скачивает vector и устанавливает его.
 * Третья часть скачивает nginx и git
 * Четвернтая

Внесены изменения от 14.12.2022

Плейбук разделился на роли, ниже ссылки на роли и остновной плейбук

* [vector-role](https://github.com/Atlipoka/vector-role)
* [ligthouse-role]()
* [main playbook](https://github.com/Atlipoka/devops_netology/tree/main/Ansible/lecture4)

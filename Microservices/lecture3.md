1. Для настройки CI\CD процессов в облачных сервисах, можно использовать облачные платформы для развертывания инфраструктуры, такие как MS AZURE, AWS, Google Clouds и YC. Все перечисленные платформы, можно использовать как базу для разворачивания сервисов. В силу обстоятельств, возьму за основу облачную платформу YC и применимо к ней буду описывать ресурсы, которые можно использовать для обеспечения CI\CD процессов.

| Стэк или единичное ПО | Git | репозиторий на каждый сервис | запуск сборки по событию из системы контроля версий | запуск сборки по кнопке с указанием параметров | возможность привязать настройки к каждой сборке | возможность создания шаблонов для различных конфигураций сборок | возможность безопасного хранения секретных данных (пароли, ключи доступа) | несколько конфигураций для сборки из одного репозитория | кастомные шаги при сборке | собственные докер-образы для сборки проектов | возможность развернуть агентов сборки на собственных серверах | возможность параллельного запуска нескольких сборок | возможность параллельного запуска тестов |
|--------------------------------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|GitHub+Jenkins+YC(container registry)       | + | + | + | + | + | + | + | + |+|:---:|:---:|:---:|:---:|
|GitHub+TeamSity+YC(container registry)+Vault| + | + | + | + | + | + | + | + |+|:---:|:---:|:---:|:---:|
|GitLab+Vault                                | + | + | + | + | + | + | + | + |+|:---:|:---:|:---:|:---:|
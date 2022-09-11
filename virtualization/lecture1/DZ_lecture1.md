1. Мое понимание типов виртуализации:
 * Полная виртуализация - используется может использоваться без ОС, т.к. является ОС;
 * Паравиртуализация - необходима ОС, для работы гипервизова, причем, в большинстве случаев в роли основной ОС может быть использована любая ОС независимо от того, на базе какой ОС будет разворачиваться дополнительная ВМ.
 * Виртуализация на основе ОС - позволяет работать с ОС, но только с теми, которые содержат тоже ядро, что и основная система, на базе которо были развернуты дополнительные ОС.
2. Мое мнение по выбору типа соотношения физических серверов и приложений, ИС, в зависимости от условий использования.
 * Физицеские севера - Высоконагруженная база данных, чувствительная к отказу и Различные web-приложения. Здесь выбор в сторону физических серверов был сделан был потому, что обычно в случае с БД используют полную виртуализацию, создают кластер из нескольких сервером ВМ БД и в случае отказа какой либо ВМ, быстро васстанавливают ее. Такая же ситуация с приложенями, если делать целевое решение для бизнеса, то так же важна отказоустойчивость и важно иметь несколько взаимозаменяющих ВМ в случае отказа одной из них.
 * Паравиртуализация - Системы, выполняющие высокопроизводительные расчеты на GPU. Здесь выбор пал на паравиртуилизацию потому, что в случае паравиртуализации удобно работать с ядром и выделять ресурсы средсвом гипервизора и устанавливать лимиты использования. Не стал выбирать виртуализацию на уровне ОС, т.к. они работаю с ядром напрямую и мне не совсем понятно, как они могут нагружать основную ОС.
 * Виртуализация уровня ОС - Windows системы для использования бухгалтерским отделом. Удобно использовать одно семейсвто ОС и приложений, простота в управлении и использовании.
3. Распишу по пунктам каждый сценарий.
 *  Полная виртуализация на базе VMVare. Обычно используют несколько физических серверов с полной виртуализацией, т.к. 100 ВМ это крупный бизнес. Удобно управлять ВМ, вводить и выводить ВМ, настраивать репликацию, миграцию, управлять балансировкой. В общем, можно использовать vSphere, которые удобно управляют ВМ и дают большое кол-во возможностей для администрирования.
 *  Полная виртуализация на базе Xen. Исходя из полученной информации на лекции, данный тип гипервозора работает со всеми ОС, достаточно стабильна и в ходе работы мало ошибок. Легко управлять и настраивать ОС, низкий порог вхождения для тех, кто сталкивается впервые.
 *  Паравиртуализация на базе XenPV. Выбрал данный тип гипервизора т.к. он высокопроизодителен, более подходит под семейство Windows, чем KVM.
 *  Бесплатно - Полная виртуализация на базе KVM, платно - VMVare. Так как используется семейство Linux и необходима тестовая среда, для уменьшения затрат можно использовать KVM, максимальная эффектиность. Если ресурсы позволяют, можно использовать VMVare, удобно администрировать и более надежно.
4. В случае использования нескольких гипервизоров могут возникать проблемы в администрировании. Все сиситемы усправления разные, используют разные механизмы, имеют разные минусы и плюсы. Если есть возможность, то переводить полную виртуализацию используюя один вид гипервизора, для простоты администрировния и последующего использования с учетом дальнейшего развития. В роли паравиртуализации или виртуализации на базе ОС, можно использовать разные виды гипервизора для наибольшей еффективности, в зависимотси от задач.
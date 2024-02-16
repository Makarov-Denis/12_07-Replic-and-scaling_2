# Домашнее задание к занятию "`Репликация и масштабирование. Часть 2`" - `Макаров Денис`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1
Опишите основные преимущества использования масштабирования методами:

 - активный master-сервер и пассивный репликационный slave-сервер;
 - master-сервер и несколько slave-серверов;
 - активный сервер со специальным механизмом репликации — distributed replicated block device (DRBD);
 - SAN-кластер.

### Решение:

  ### *Активный master-сервер и пассивный репликационный slave-сервер:*

Я бы назвал это начальным уровнем репликации. Преимущество тут только в том, что можно снизить нагрузку чтения с Мастера, который одновременно трудится и на запись. Если же БД начнет увеличичиваться, то придется вертикально расширять каждый сервер, особенно Мастер. Также на скорость чтения будет влиять вид записи: синхронная и асинохронная. В случае синхронной записи есть гарантия, что данные не потеряются, но скорость ответа будет ниже.

  ### *Мaster-сервер и несколько slave-серверов:*

Та же схема, что и выше. Но тут приемущество, что Мастер еще больше освобождается от чтения. Также если БД стала достаточно велика, ее части можно реплицировать отдельно на каждый Слейв. Например, один Слейв реплицирует название товара и цену, второй - доступнок количество и т.д. А расширять вертикально можно только Мастер. И опять же очень важен вид записи.

 ### *Активный сервер со специальным механизмом репликации — distributed replicated block device (DRBD):*

Этот вариант является программным решением для хранения данных. DRBD включает в себя копию данных на двух устройствах хранения, так что в случае сбоя одного из них можно использовать данные на другом. Тут происходит реплицикация блочных устройств (жесткие диски, разделы, логические тома и т.д.) между хостами по сети и зеркально отображает их содержимое. По сути это вариант можно назвать даже RAID1 для БД. Данный вариант может применяться для ряда задач в кластерах высокой доступности (High Availability).

### *SAN-кластер:*
SAN - Storage Area Network - Сеть хранения данных. Название говорит само за себя. Это "выделенная сеть", отделенная от локальных (LAN) и глобальных (WAN) сетей. Преимущества - разделяемый доступ к дискам и ленточным устройствам, большая производительность репликации данных, лучшая масштабируемость дисковой системы и расширенная отказоустойчивость. Архитектура SAN так же обеспечивает возможность безболезненного роста. Техника управления емкостью хранилищ данных может быть использована что бы обеспечить постоянное ее увеличение по мере необходимости. Если необходимо увеличение вычислительной мощности, SAN позволяет добавлять новые серверы таким образом, что они будут иметь разделяемый доступ к данным. Для увеличения скорости доступа к данным, могут быть созданы несколько их копий, это позволяет преодолеть ограничение производительности одного диска.

---

### Задание 2

Разработайте план для выполнения горизонтального и вертикального шаринга базы данных. База данных состоит из трёх таблиц:

 - пользователи,
 - книги,
 - магазины (столбцы произвольно).

   Опишите принципы построения системы и их разграничение или разбивку между базами данных.

*Пришлите блоксхему, где и что будет располагаться. Опишите, в каких режимах будут работать сервера.*

### Решение:

В данной БД самый большой объем будет занимать информация о книгах (авторы, названия, жанры, годы издания). И этот объем настолько велик, что именно разделу книг я бы сделал горизонтальное масштабирование, например, по названиям: один сервер - от А до К, второй - от Л до Т, третий - от У до Я. И если сеть магазинов не очень большая, то данные по пользователям и магазинам вполне можно уместить и на одном сервере. И в приведенной схеме все сервера поставить в режим Мастер-Мастер.


![image](https://github.com/Makarov-Denis/12_07-Replic-and-scaling_2/assets/148921246/1d7fb019-81ef-4beb-a7d9-af747a46f9c9)


Если же сеть магазинов очень велика, то помимо горизонтального масштабирования можно применить и вертикальный способ. На отдельные сервера вынести, например, пользователей, т.к. в каждом районе города, да и в других городах, их может быть много. Тут можно главный сервер сделать Мастером, остальные сделать Слейв.

![image](https://github.com/Makarov-Denis/12_07-Replic-and-scaling_2/assets/148921246/0a944a31-3854-458c-a43f-7ee8b81a9ddc)


---



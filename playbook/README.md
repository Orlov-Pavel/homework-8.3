# Playbook для установки clickhouse и vector.
## Содержание:  
[Описание Playbook](#описание-playbook)  
[Описание inventory файла](#описание-inventory-файла)  
[Описание файлов переменных](#описание-файлов-переменных)
## Описание Playbook
Playbook содержит 4 плея:
1. [Install Clickhouse](#плей-install-clickhouse)
2. [Install Vector](#плей-install-vector)
3. [Install Nginx](#плей-install-nginx)
4. [Install LightHouse](#плей-install-lighthouse)
### Плей Install Clickhouse 
Плей ```Install Clickhouse``` устанавливает clickhouse на все машины в группе clickhouse указанные в inventory файле.  
Процесс выполнения плея:
1. Таск ```Get clickhouse distrib```. Скачивает дистрибутивы перечисленные в переменных. Сначала task ищет дистрибутивы без архитектуры, если не находит, то берет дистрибутив с архитектурой x86_64.
2. Таск ```Install clickhouse packages```. Устанавливает скачанные по списку дистрибутивы. Данный таск, в случае если были внесены изменения, вызывает handler ```Start clickhouse service```.
3. Таск ```Start clickhouse service```. Убеждается что service "clickhouse-server" запущен, либо запускает его.
4. Таск ```Create database```. Создаёт базу данных "logs".

Хендлеры в порядке их выполнения:
1. Хэндлер ```Start clickhouse service``` перезапускает сервис "clickhouse-server" в случае если был вызван в таске ```Install clickhouse packages```.

### Плей Install Vector
Плей ```Install Vector``` устанавливает vector на все машины в группе Vector указанные в inventory файле.  
Процесс выполнения плея:
1. Таск ```Get vector distrib```. Скачивает дистрибутив vector версии указанной в переменных.
2. Таск ```Install vector packages```. Устанавливает скачанный в первом таске пакет. Данный таск, в случае если были внесены изменения, вызывает handler ```Start vector service```.

Хендлеры в порядке их выполнения:
1. Хэндлер ```Start vector service``` перезапускает сервис "vector" в случае если был вызван в таске ```Install vector packages```.

### Плей Install Nginx
Плей ```Install Nginx``` устанавливает Nginx на все машины в группе LightHouse указанные в inventory файле для возможности в дальнейшем запустить на нём сам LightHouse.  
Процесс выполнения плея:
1. Таск ```Install nginx``` устанавливает Nginx, указанной в переменных версии, из репозитория Ubuntu. Данный таск, в случае если были внесены изменения, вызывает handler ```Restart nginx service```.

Хендлеры в порядке их выполнения:
1. Хэндлер ```Restart nginx service``` перезапускает сервис "nginx" в случае если был вызван в таске ```Install nginx```.

### Плей Install LightHouse
Плей ```Install LightHouse``` устанавливает LightHouse на все машины в группе LightHouse указанные в inventory файле.  
Процесс выполнения плея:

Претаски в порядке их выполнения:
1. Претаск ```install unzip``` устанавливает утилиты "unzip" из репозитория ubuntu. Утилита необходима для распаковки скачиваемого, во время выполнения плея, дистрибутива LightHouse.

Таски в порядке их выполнения:
1. Таск ```Get lighthouse distrib``` скачивает дистрибутив LightHouse из гита разработчиков.
2. Таск ```Unarchive lighthouse distrib into nginx``` распаковывает дистрибутив LightHouse в директорию "/var/www/html/". Данный таск, в случае если были внесены изменения, вызывает handler ```Restart nginx service```.
3. Таск ```Make nginx config``` подготавливает из шаблона конфигурационный файл для Nginx, подставлая в него порт, указанный в переменных. Данный таск, в случае если были внесены изменения, вызывает handler ```Restart nginx service```.
4. Таск ```Remove lighthouse distrib``` удаляет скачанный во 2-ом таске дистрибутив.

Хендлеры в порядке их выполнения:
1. Хэндлер ```Restart nginx service``` перезапускает сервис "nginx" в случае если был вызван в тасках ```Unarchive lighthouse distrib into nginx``` или ```make nginx config```.


## Описание inventory файла
Inventory файл prod.yml находится [здесь](./inventory/prod.yml).  
В inventory файле обязательно должны быть перечислены хосты для трех групп: clickhouse, vector и lighthouse. Плейбуком предполагается что хосты группы clickhouse будут на базе ОС CentOS, а хосты групп vector и lighthouse будут на базе ОС ubuntu.

## Описание файлов переменных
В данном проекте 3 файлах переменных для групп хостов [clickhouse](./group_vars/clickhouse/vars.yml), [vector](./group_vars/vector/vars.yml) и [lighthouse](./group_vars/lighthouse/vars.yml).

### Файл переменных clickhouse
Файл переменных clickhouse в обязательном порядке должен содержать следующие переменные:
Переменная | Описание
---------- | --------
clickhouse_version | Требуемая к установке версия дистрибутива
clickhouse_packages | Список требуемых к установке пакетов

### Файл переменных vector
Файл переменных vector в обязательном порядке должен содержать следующие переменные:
Переменная | Описание
---------- | --------
vector_version | Требуемая к установке версия дистрибутива

### Файл переменных lighthouse
Файл переменных lighthouse в обязательном порядке должен содержать следующие переменные:
Переменная | Описание
---------- | --------
nginx_version | Требуемая к установке версия nginx
lighthouse_port | Порт на котором будет доступен lighthouse
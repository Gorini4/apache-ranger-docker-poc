# Практика по Apache Ranger
Основано на материалах [Apache Ranger Docker POC With Hadoop(HDFS, Hive, Presto)](https://kadencho.medium.com/hands-on-apache-ranger-docker-poc-with-hadoop-hdfs-hive-presto-814344a03a17).

## Цель
В процессе выполнения вы 
1. развернете локальный набор сервисов, включающих Ranger и HDFS
1. настроите политики доступа к HDFS

## Инструкция

### Развертывание
1. Склонируйте проект командой
```bash
git clone https://github.com/Gorini4/apache-ranger-docker-poc
```
2. Перейдите в директорию проекта
```bash
cd apache-ranger-docker-poc
```
3. Запустите первый набор сервисов (Ranger)
```bash
cd docker-composes/ranger
docker-compose up -d --build
```
4. Запустите второй набор сервисов (Hadoop)
```bash
cd ../hadoop
docker-compose up -d --build
```
5. Проверьте, что доступны сервисы на следующих адресах:

http://localhost:6080 (Ranger Admin, login `admin`, password `rangeradmin1`)

http://localhost:9870 (HDFS NameNode)

6. Добавьте в Ranger сервис HDFS. Для этого перейдите по адресу http://localhost:6080 и нажмите "+" на плашке HDFS и заполните форму, как приведено на изображении ниже.
![](https://miro.medium.com/max/1400/1*MZIsgJzCAOot2xu3erP1lg.png)

### Создание файловой структуры на HDFS
1. Войдите в контейнер
```bash
docker exec -it datanode bash
```
2. Выполните команды для создания папок и установки прав
```bash
/opt/hadoop-3.2.1/bin/hdfs dfs -mkdir /tmp/allowed_dir /tmp/denied_dir
/opt/hadoop-3.2.1/bin/hdfs dfs -chmod 000 /tmp/allowed_dir /tmp/denied_dir
```
3. Убедитесь, что папки созданы корректно командой
```bash
/opt/hadoop-3.2.1/bin/hdfs dfs -ls /tmp
```
Вывод должен выглядеть так
```bash
Found 3 items
d---------   - root supergroup          0 2022-06-13 22:08 /tmp/allowed_dir
d---------   - root supergroup          0 2022-06-13 22:08 /tmp/denied_dir
drwx-wx-wx   - root supergroup          0 2022-06-07 21:53 /tmp/hive
```

### Создание тестового пользователя
1. Не выходя из контейнера, выполните команду
```bash
adduser --shell /bin/bash testuser
```
Установите произвольный пароль, на остальные вопросы просто нажмите Enter.

2. В веб-интерфейсе Ranger Admin перейдите в ‘Settings’ -> ‘Users/Groups/Roles’ -> ‘Add New User’ и заполните форму, указав User Name - testuser, Select Role - User. Остальные поля можно заполнить произвольно. Нажмите Save.

### Настройка политик
1. Перейдите в сервис hdfs на главной странице.
![](https://miro.medium.com/max/1400/1*nRQfUduWEXsQhZKVRkamwg.png)
2. Нажмите кнопку "Add New Policy" и заполните форму:
- Policy Name: произвольно
- Resource Path: /tmp/allowed_dir
- В строке Allow Conditions: Select User - testuser, Permissions - выберите все пункты (Read, Write, Execute).
3. Нажмите Save.

### Проверка политик
1. Войдите в контейнер
```bash
docker exec -it datanode bash
```
2. Переключитесь на тестового пользователя
```bash
su testuser
```
3. Выполните команды на создание файлов в тестовых директориях
```bash
/opt/hadoop-3.2.1/bin/hdfs dfs -touch /tmp/allowed_dir/test.txt
/opt/hadoop-3.2.1/bin/hdfs dfs -touch /tmp/denied_dir/test.txt
```
4. Первая команда должна выполниться успешно, а вторая вернуться с ошибкой
```
touch: Permission denied: user=testuser, access=EXECUTE, inode="/tmp/denied_dir"
```

### Просмотр логов аудита
1. Перейдите в ‘Audit’.
2. В списке операций найдите логи команд из предыдущего раздела

## Способ сдачи
Пришлите в чат с преподавателем скриншот логов аудита.

## Критерии сдачи
На скриншоте присутствуют требуемые операции.
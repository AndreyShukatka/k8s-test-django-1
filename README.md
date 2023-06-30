# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри контейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Как запустить dev-версию

Запустите базу данных и сайт:

```shell-session
$ docker-compose up
```

В новом терминале не выключая сайт запустите команды для настройки базы данных:

```shell-session
$ docker-compose run web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker-compose run web ./manage.py createsuperuser
```

Для тонкой настройки Docker Compose используйте переменные окружения. Их названия отличаются от тех, что задаёт docker-образа, сделано это, чтобы избежать конфликта имён. Внутри docker-compose.yaml настраиваются сразу несколько образов, у каждого свои переменные окружения, и поэтому их названия могут случайно пересечься. Чтобы не было конфликтов к названиям переменных окружения добавлены префиксы по названию сервиса. Список доступных переменных можно найти внутри файла [`docker-compose.yml`](./docker-compose.yml).

## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgresSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).

## Деплой проекта:
1. Установить:

[Kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/)
[minikube](https://minikube.sigs.k8s.io/docs/)
[VirtualBox](https://www.virtualbox.org) - для Linux\Windows\MacOS на процессорах Интел.
[DockerDesktop](https://www.docker.com) - MacOS для Mac на M1.

2. Запустите PostgresSQL
3. Создайте базу данных в поде Postgresql:
```
CREATE DATABASE {DB_NAME};
CREATE USER {DB_USER} WITH ENCRYPTED PASSWORD {PASSWORD};
GRANT ALL PRIVILEGES ON DATABASE {DB_NAME} TO {DB_USER};
```
4. Внести переменные окружения:
  - Создаем файл с названием `postgres-config.properties`
  - Записываем в него наши переменные окружения в формате:
```shell
SECRET_KEY=Ваш секретный ключ
DEBUG=False или True
DATABASE_URL=Ссылка на вашу базу данных PostgreSQL
ALLOWED_HOSTS=Список ваших хостов
```
  - Далее прописываем наш `ConfigMap` в системе с помощью команды:
```shell
kubectl create configmap postgres-config --from-env-file=postgres-config.properties
```
5. Запустить манифесты:
```shell
kubectl apply -f django-app.yaml
```
6. Проверить запущенные поды:
```shell
kubectl get pods
```
7. Зайти в shell 
```shell
kubectl exec --stdin --tty <НАЗВАНИЕ ПОДА>> -- sh
```
8. Сделать миграцию
```python
./manage.py migrate
```
9. Создать суперпользователя
```python
./manage.py createsuperuser
```
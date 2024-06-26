# Django Site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри контейнера Django приложение запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет
сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и
Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и
Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Как подготовить окружение к локальной разработке

Код в репозитории полностью докеризирован, поэтому для запуска приложения вам понадобится Docker. Инструкции по его
установке ищите на официальных сайтах:

- [Get Started with Docker](https://www.docker.com/get-started/)

Вместе со свежей версией Docker к вам на компьютер автоматически будет установлен Docker Compose. Дальнейшие инструкции
будут его активно использовать.

## Как запустить сайт для локальной разработки

Запустите базу данных и сайт:

```shell
$ docker compose up
```

В новом терминале, не выключая сайт, запустите несколько команд:

```shell
$ docker compose run --rm web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker compose run --rm web ./manage.py createsuperuser  # создаём в БД учётку суперпользователя
```

Готово. Сайт будет доступен по адресу [http://127.0.0.1:8080](http://127.0.0.1:8080). Вход в админку находится по
адресу [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/).

## Как вести разработку

Все файлы с кодом django смонтированы внутрь докер-контейнера, чтобы Nginx Unit сразу видел изменения в коде и не
требовал постоянно пересборки докер-образа -- достаточно перезапустить сервисы Docker Compose.

### Как обновить приложение из основного репозитория

Чтобы обновить приложение до последней версии подтяните код из центрального окружения и пересоберите докер-образы:

``` shell
$ git pull
$ docker compose build
```

После обновлении кода из репозитория стоит также обновить и схему БД. Вместе с коммитом могли прилететь новые миграции
схемы БД, и без них код не запустится.

Чтобы не гадать заведётся код или нет — запускайте при каждом обновлении команду `migrate`. Если найдутся свежие
миграции, то команда их применит:

```shell
$ docker compose run --rm web ./manage.py migrate
…
Running migrations:
  No migrations to apply.
```

### Как добавить библиотеку в зависимости

В качестве менеджера пакетов для образа с Django используется pip с файлом requirements.txt. Для установки новой
библиотеки достаточно прописать её в файл requirements.txt и запустить сборку докер-образа:

```sh
$ docker compose build web
```

Аналогичным образом можно удалять библиотеки из зависимостей.

## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно
лишь, чтобы оно никому не было
известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE`
или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт
ответит ошибкой 400. Можно перечислить несколько адресов через запятую,
например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не
поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).

# Деплой в Minikube (Windows, Hyper-V)

### 1. Установите

#### [kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/)

Проверить результат:

```sh
kubectl version --client
```

#### [minikube](https://minikube.sigs.k8s.io/docs/start/)

Проверить результат:

```sh
minikube version
```

#### [Docker](https://docs.docker.com/get-docker/)

"Поднимите" локальный K8s Cluster

```sh
minikube start --driver=hyperv
```

Проверить результат:

```sh
kubectl cluster-info
```

`-> Kubernetes control plane is running at https://172.20.55.46:8443
CoreDNS is running at https://172.20.55.46:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.`

### 2. Создайте образ приложения. Соберите образ с приложением `django`. Перейдите в папку проекта.

```sh
cd 'ваш путь до папки проекта'\k8s-test-django\
```

```sh
docker image build ./backend_main_django/ -t имя_репозитория/имя_образа
```

Проверить результат:

```sh
docker image ls
```

`-> ... olberd/django-hub latest c841ac0b86f8 4 weeks ago 963MB ...`

Войдите в Docker Hub:

```sh
docker login
```

Загрузите образ в [dockerhub](https://hub.docker.com/)

```sh
docker push имя_репозитория/имя_образа
``` 

Запишите ссылку на образ в файле django-app.yaml    
Например:  
`image: olberd/django-hub`

### 3. Активируйте `ingress`

```sh
minikube addons enable ingress
kubectl apply -f .\kubernetes\ingress.yaml
```

Проверить:

```sh
kubectl get ingress
```

### 4. Запустите проект джанго

~~~
kubectl apply -f  .\kubernetes\django-app.yaml
~~~

### 5. Активируйте автоматический `clearsessions`

```sh
kubectl apply -f .\kubernetes\clearsessions_cron_job.yaml
```

Проверить:
```kubectl get cronjob```

### 6. Создание и подключение к БД PostgreSQL с помощью Minikube и [Helm](https://helm.sh/)

- добавить хранилище helm `postgresql` в кластер Minikube:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

- установите БД postgresql в кластере Minikube с параметрами:

```bash
helm install dev-pg bitnami/postgresql \  
  --set global.postgresql.auth.postgresPassword=<YOUR-POSTGRES-PASSWORD> \
  --set global.postgresql.auth.username=test_k8s \
  --set global.postgresql.auth.password=OwOtBep9Frut \
  --set global.postgresql.auth.database=test_k8s \
  --set global.postgresql.service.ports.postgresql=5432
```

- для доступа к созданной БД:

```bash
kubectl run dev-pg-postgresql-client --rm --tty -i --restart=Never \
  --namespace default \
  --image docker.io/bitnami/postgresql:16.2.0-debian-12-r5  --env="PGPASSWORD=<YOUR-POSTGRES-PASSWORD>" \
  --command -- psql --host dev-pg-postgresql -U test_k8s -d test_k8s -p 5432
```

Чтобы иметь к данной БД доступ в проекте джанго, необходимо в файле Secret.yaml прописать:

```bash
stringData:
  database_url: "postgres://test_k8s:OwOtBep9Frut@dev-pg-postgresql:5432/test_k8s"

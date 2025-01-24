# Развертывание в кластере Yandex-облака

Убедитесь что вы получили необходимые доступы и можете зайти
в [веб интерфейс Яндекс-облака.](https://console.cloud.yandex.ru/)

Перед началом работы убедитесь что у вас установлены следующие инструменты:

- Инструмент для управления кластерами
  Kubernetes: [Kubectl](https://kubernetes.io/docs/tasks/tools/). [Иструкция по настройке](https://yadi.sk/i/QDZGXUP7aG3KOw)
  kubectl для работы в Яндекс-облаке.
- Графический инструмент для управления кластерами Kubernetes - [Lens Desktop](https://k8slens.dev/)

Проверьте корректность настройки следующей командой:

```sh
kubectl cluster-info
```

## Выделенные ресурсы

Для развертывания тестового сайта джанго в Яндекс-облаке Вам должны быть выделены:
(https://sirius-env-registry.website.yandexcloud.net/edu-focused-lamarr.html)

- домен вида `edu-focused-lamarr.sirius-k8s.dvmn.org`
- отдельный namespace вида `edu-focused-lamarr`
- база данных с именем вида `edu-focused-lamarr`
- в указанном namespace должен находиться секрет с именем `postgres`, в ключах которого доступы к
  выше базе данных (`dsn`)

## Настройка переменных окружения Django

Откройте `env-django.yaml` и укажите нужные значения для переменных окружения DEBUG, ALLOWED_HOST и SECRET_KEY.

Для доступа к приложению через выданное доменное имя, укажите в манифесте сервиса (`service-django.yaml`) порт на
который настроен роутинг ALB.

## Запуск приложения

Регистрируем переменные конфигурации и секреты:

```sh
kubectl apply -f env-django.yaml
```

Запускаем миграции командой:

```sh
kubectl apply -f migrate.yaml
```

Чтобы создать суперпользователя, внесите в `createsuperuser.yaml` необходимые Вам значения

```sh
DJANGO_SUPERUSER_EMAIL: '<e-mail>'
DJANGO_SUPERUSER_USERNAME: '<username>'
DJANGO_SUPERUSER_PASSWORD: '<password>'
```

и примените манифест командой:

```sh
kubectl apply -f createsuperuser.yaml
```

Запускаем Deployment django:

```sh
kubectl apply -f deployment-django.yaml
```

Запускаем Service django:

```sh
kubectl apply -f service-django.yaml
```

Приложение должно быть доступно по [выданному Вам адресу](https://edu-focused-lamarr.sirius-k8s.dvmn.org/)

## Обновление версии приложения

Для запуска другой версии приложения, укажите в манифесте `deployment-django.yaml` образ на dockerhub с
нужным Вам тэгом. Затем примените внесенные изменения командой:

```sh
kubectl apply -f deployment-django.yaml
```
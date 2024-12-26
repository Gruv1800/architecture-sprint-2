# pymongo-api

# Задание 1,5,6
Схемы - https://disk.yandex.ru/d/_DZJGGXn5sV6lQ

или - https://github.com/Gruv1800/architecture-sprint-2/tree/feature/sprint_2/schemes

**финальный вариант** - https://github.com/Gruv1800/architecture-sprint-2/blob/feature/sprint_2/schemes/sprint2_5.drawio

# Задание 2-4
**Процесс запуска финального варианта** - https://github.com/Gruv1800/architecture-sprint-2/blob/feature/sprint_2/sharding-repl-cache/README.md

## Как запустить

Запускаем mongodb и приложение

```shell
docker compose up -d
```

Заполняем mongodb данными

```shell
./scripts/mongo-init.sh
```

## Как проверить

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080

### Если вы запускаете проект на предоставленной виртуальной машине

Узнать белый ip виртуальной машины

```shell
curl --silent http://ifconfig.me
```

Откройте в браузере http://<ip виртуальной машины>:8080

## Доступные эндпоинты

Список доступных эндпоинтов, swagger http://<ip виртуальной машины>:8080/docs
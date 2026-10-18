# Docker Nginx Load Balancer

Учебный DevOps-проект: два Nginx backend-сервера за отдельным Nginx-балансировщиком. Входящие HTTP-запросы распределяются между backend-серверами алгоритмом Round Robin.

## Архитектура

```text
Browser / curl
      |
      v
localhost:8080
      |
      v
Nginx load balancer
      |
      +------> server-a:80
      |
      +------> server-b:80
```

Все три контейнера находятся в пользовательской Docker-сети `devops-net`. Балансировщик обращается к backend-серверам по именам сервисов Docker Compose.

## Требования

- Docker Desktop или Docker Engine с Docker Compose
- Свободные порты `8080`, `8081` и `8082`

## Запуск

Из корня проекта выполните:

```bash
docker compose up -d
```

Compose создаст сеть и запустит три контейнера. При первом запуске Docker может скачать образ Nginx.

## Проверка

Балансировщик:

```bash
curl http://localhost:8080
```

Повторите запрос несколько раз. Ответы должны приходить от `SERVER-A` и `SERVER-B`.

Можно обратиться к backend-серверам напрямую:

- http://localhost:8081 — SERVER-A
- http://localhost:8082 — SERVER-B
- http://localhost:8080 — балансировщик

Проверить состояние контейнеров:

```bash
docker compose ps
```

Посмотреть логи балансировщика:

```bash
docker compose logs --tail 20 load-balancer
```

## Проверка отказа backend

Остановите первый backend:

```bash
docker compose stop server-a
```

Отправьте несколько запросов через балансировщик:

```bash
curl -i http://localhost:8080
```

При недоступности `server-a` Nginx может зарегистрировать ошибку подключения, а запрос — успешно пройти через `server-b`. Поведение зависит от состояния backend и настроек Nginx; это не заменяет активные health checks.

Восстановите backend:

```bash
docker compose start server-a
```

## Остановка

Остановить и удалить контейнеры и созданную Compose сеть:

```bash
docker compose down
```

HTML-страницы и конфигурационные файлы останутся в проекте.

## Что демонстрирует проект

- Docker Compose для описания нескольких сервисов
- Пользовательскую Docker-сеть
- Nginx reverse proxy и upstream-группу
- Round Robin load balancing
- Базовый эксперимент с отказом backend
- Диагностику через логи контейнера

# Команды проекта: Docker Nginx Load Balancer

Этот файл содержит команды, использованные для сборки, запуска и проверки учебного проекта. Команды сгруппированы по этапам, чтобы их было удобно показывать на собеседовании.

> Важно: часть проекта изначально собиралась вручную через `docker run`, а затем была перенесена в Docker Compose. Это два способа запуска одной лабораторной конфигурации. Для актуального запуска используйте раздел **Docker Compose**.

## 1. Создание Docker-сети и backend-контейнеров (ручной запуск)

```bash
docker network create devops-net
```

Создаёт отдельную bridge-сеть для контейнеров.

```bash
docker run -d --name server-a --network devops-net -p 8081:80 nginx
docker run -d --name server-b --network devops-net -p 8082:80 nginx
```

Запускает два Nginx-контейнера и публикует их порты на Mac.

Подключение уже созданных контейнеров к сети, если они были созданы без `--network`:

```bash
docker network connect devops-net server-a
docker network connect devops-net server-b
```

## 2. Настройка страниц backend-серверов

В контейнерах меняли файл `/usr/share/nginx/html/index.html`, чтобы ответы различались: `Hello from SERVER-A` и `Hello from SERVER-B`.

Пример команд для повторения этой настройки:

```bash
docker exec server-a sh -c 'echo "Hello from SERVER-A" > /usr/share/nginx/html/index.html'
docker exec server-b sh -c 'echo "Hello from SERVER-B" > /usr/share/nginx/html/index.html'
```

В версии Docker Compose страницы уже лежат в проекте в `html/server-a/index.html` и `html/server-b/index.html`, поэтому редактировать файлы внутри контейнеров не требуется.

## 3. Запуск балансировщика вручную

```bash
docker run -d --name load-balancer --network devops-net -p 8080:80 -v "$HOME/devops-lab/nginx.conf:/etc/nginx/nginx.conf:ro" nginx
```

Монтирует конфигурацию Nginx с Mac внутрь контейнера только для чтения.

> Этот пример использует прежний путь `~/devops-lab/nginx.conf`. Если запускаете проект из новой папки, используйте Compose ниже либо замените путь на актуальный.

## 4. Проверка работы

```bash
docker ps
curl http://localhost:8080
```

`docker ps` показывает запущенные контейнеры. Повторные запросы к балансировщику должны показывать ответы от `SERVER-A` и `SERVER-B`.

```bash
docker exec load-balancer nginx -t
```

Проверяет синтаксис конфигурации Nginx.

```bash
docker exec load-balancer nginx -s reload
```

Перечитывает конфигурацию Nginx без пересоздания контейнера.

## 5. Проверка отказа backend-сервера

```bash
docker stop server-a
curl http://localhost:8080
docker logs --tail 20 load-balancer
```

Останавливает `server-a`, проверяет ответ через балансировщик и показывает последние строки его логов. В нашем тесте балансировщик продолжал получать успешные ответы от `server-b`; в логах также были ошибки подключения к остановленному backend.

Восстановление:

```bash
docker start server-a
curl http://localhost:8080
```

После запуска `server-a` снова можно проверять ответы обоих backend-серверов.

## 6. Актуальный запуск через Docker Compose

Перейдите в каталог проекта — тот, где лежит `docker-compose.yml`:

```bash
cd /путь/к/devops-load-balancer
```

Если старые контейнеры с такими же именами ещё существуют, удалите именно контейнеры этой лабораторной работы:

```bash
docker rm -f load-balancer server-a server-b
```

Эта команда удаляет указанные контейнеры, но не удаляет файлы проекта.

Запуск:

```bash
docker compose up -d
docker compose ps
curl http://localhost:8080
```

Остановка проекта:

```bash
docker compose down
```

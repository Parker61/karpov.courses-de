# Практические примеры ENTRYPOINT и CMD

## Пример 1: Веб-сервер (рекомендуемый подход)

```dockerfile
FROM nginx:alpine
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

**Использование:**
```bash
# Запуск с параметрами по умолчанию
docker run myapp

# Запуск с дополнительными параметрами
docker run myapp -t /tmp/nginx.conf -g "daemon off;"
```

## Пример 2: Python приложение

```dockerfile
FROM python:3.9-slim
COPY app.py /app/
WORKDIR /app
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]
```

**Использование:**
```bash
# Запуск на порту 8080 (по умолчанию)
docker run myapp

# Запуск на другом порту
docker run myapp --port 3000

# Запуск с дополнительными параметрами
docker run myapp --port 3000 --debug
```

## Пример 3: Утилита командной строки

```dockerfile
FROM alpine:latest
RUN apk add --no-cache curl
ENTRYPOINT ["curl"]
CMD ["--help"]
```

**Использование:**
```bash
# Показать справку
docker run mycurl

# Скачать файл
docker run mycurl -o /tmp/file.txt https://example.com/file.txt

# Проверить заголовки
docker run mycurl -I https://example.com
```

## Пример 4: База данных

```dockerfile
FROM postgres:13
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["postgres"]
```

**Использование:**
```bash
# Запуск PostgreSQL
docker run -e POSTGRES_PASSWORD=mypassword mypostgres

# Запуск с дополнительными параметрами
docker run -e POSTGRES_PASSWORD=mypassword mypostgres postgres -c log_statement=all
```

## Пример 5: Сравнение поведения

### Dockerfile A (только CMD):
```dockerfile
FROM ubuntu:20.04
CMD ["echo", "Hello from CMD"]
```

### Dockerfile B (только ENTRYPOINT):
```dockerfile
FROM ubuntu:20.04
ENTRYPOINT ["echo", "Hello from ENTRYPOINT"]
```

### Dockerfile C (ENTRYPOINT + CMD):
```dockerfile
FROM ubuntu:20.04
ENTRYPOINT ["echo", "Hello from"]
CMD ["ENTRYPOINT+CMD"]
```

**Тестирование:**
```bash
# Dockerfile A
docker run imageA                    # Hello from CMD
docker run imageA echo "Override"    # Override

# Dockerfile B
docker run imageB                    # Hello from ENTRYPOINT
docker run imageB World             # Hello from ENTRYPOINT World

# Dockerfile C
docker run imageC                    # Hello from ENTRYPOINT+CMD
docker run imageC Docker            # Hello from Docker
```

## Частые ошибки и как их избежать

### ❌ Неправильно:
```dockerfile
FROM ubuntu
ENTRYPOINT echo "Hello"
CMD echo "World"
```

### ✅ Правильно:
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "Hello"]
CMD ["World"]
```

### ❌ Неправильно (смешивание форм):
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "Hello"]
CMD "World"
```

### ✅ Правильно:
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "Hello"]
CMD ["World"]
```

## Резюме

- **ENTRYPOINT** - неизменяемая команда
- **CMD** - параметры по умолчанию, которые можно переопределить
- **Комбинация** - лучший способ создать гибкий контейнер
- **Exec форма** - предпочтительная для производственных контейнеров
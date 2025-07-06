# Разница между ENTRYPOINT и CMD в Docker

## Основные понятия

**ENTRYPOINT** - определяет команду, которая ВСЕГДА будет выполняться при запуске контейнера.
**CMD** - определяет команду по умолчанию, которая может быть переопределена при запуске контейнера.

## Формы записи

### Shell форма:
```dockerfile
ENTRYPOINT command param1 param2
CMD command param1 param2
```

### Exec форма (рекомендуется):
```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]
CMD ["executable", "param1", "param2"]
```

## Объяснение таблицы

| Случай | No ENTRYPOINT | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT ["exec_entry", "p1_entry"] |
|--------|---------------|--------------------------------|---------------------------------------|
| **No CMD** | ❌ error, not allowed | `/bin/sh -c exec_entry p1_entry` | `exec_entry p1_entry` |
| **CMD ["exec_cmd", "p1_cmd"]** | `exec_cmd p1_cmd` | `/bin/sh -c exec_entry p1_entry` | `exec_entry p1_entry exec_cmd p1_cmd` |
| **CMD ["p1_cmd", "p2_cmd"]** | `p1_cmd p2_cmd` | `/bin/sh -c exec_entry p1_entry` | `exec_entry p1_entry p1_cmd p2_cmd` |
| **CMD exec_cmd p1_cmd** | `/bin/sh -c exec_cmd p1_cmd` | `/bin/sh -c exec_entry p1_entry` | `exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd` |

## Пошаговое объяснение каждого случая

### 1. No CMD (нет инструкции CMD)

- **No ENTRYPOINT**: ❌ Ошибка - контейнер не может запуститься без команды
- **ENTRYPOINT shell форма**: Выполняется через `/bin/sh -c exec_entry p1_entry`
- **ENTRYPOINT exec форма**: Выполняется напрямую `exec_entry p1_entry`

### 2. CMD ["exec_cmd", "p1_cmd"] (CMD в exec форме)

- **No ENTRYPOINT**: Просто выполняется `exec_cmd p1_cmd`
- **ENTRYPOINT shell форма**: CMD игнорируется, выполняется только ENTRYPOINT
- **ENTRYPOINT exec форма**: CMD добавляется как параметры к ENTRYPOINT

### 3. CMD ["p1_cmd", "p2_cmd"] (CMD только с параметрами)

- **No ENTRYPOINT**: Параметры выполняются как команда
- **ENTRYPOINT shell форма**: CMD игнорируется
- **ENTRYPOINT exec форма**: CMD параметры добавляются к ENTRYPOINT

### 4. CMD exec_cmd p1_cmd (CMD в shell форме)

- **No ENTRYPOINT**: Выполняется через shell
- **ENTRYPOINT shell форма**: CMD игнорируется
- **ENTRYPOINT exec форма**: Вся CMD команда передается как один параметр

## Практические примеры

### Пример 1: Только CMD
```dockerfile
FROM ubuntu
CMD ["echo", "Hello World"]
```
```bash
docker run myimage                    # Выведет: Hello World
docker run myimage echo "Goodbye"     # Выведет: Goodbye (CMD переопределен)
```

### Пример 2: Только ENTRYPOINT
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "Hello"]
```
```bash
docker run myimage                    # Выведет: Hello
docker run myimage World             # Выведет: Hello World
```

### Пример 3: ENTRYPOINT + CMD
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "Hello"]
CMD ["World"]
```
```bash
docker run myimage                    # Выведет: Hello World
docker run myimage Docker            # Выведет: Hello Docker
```

## Ключевые различия

1. **ENTRYPOINT** нельзя переопределить при запуске контейнера
2. **CMD** легко переопределяется аргументами при `docker run`
3. **Комбинация**: ENTRYPOINT + CMD позволяет создать команду с параметрами по умолчанию
4. **Shell vs Exec форма**: Exec форма не запускает shell, что более эффективно и безопасно

## Рекомендации

- Используйте **ENTRYPOINT** для основной команды приложения
- Используйте **CMD** для параметров по умолчанию
- Предпочитайте **exec форму** для лучшей производительности
- Комбинируйте ENTRYPOINT + CMD для гибкости
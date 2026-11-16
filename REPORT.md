# Отчёт по ДЗ-3

## 4 балла: один контейнер

Сборка и запуск:
```bash
docker build -t counter-app .
docker run -d --name counter -p 8080:8000 -e GREETING="Привет, ВШЭ" counter-app
```

Проверка счётчика:
```bash
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS http://127.0.0.1:8080/
curl -fsS http://127.0.0.1:8080/readyz
```

Результат:
```
{"greeting": "Привет, ВШЭ", "count": 0, "hostname": "de27f9103727", "storage": "sqlite"}
{"greeting": "Привет, ВШЭ", "count": 1, "hostname": "de27f9103727", "storage": "sqlite"}
{"greeting": "Привет, ВШЭ", "count": 2, "hostname": "de27f9103727", "storage": "sqlite"}
```

Логи (`docker logs counter`):
```
2026-11-16 15:23:50,965 listening port=8000 storage=sqlite
2026-11-16 15:25:18,038 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-16 15:25:18,057 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-16 15:25:18,072 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-16 15:25:18,080 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-16 15:25:18,088 http 172.17.0.1 "GET /readyz HTTP/1.1" 200 -
```
## 6 баллов: Compose и данные

Запуск:
```bash
cp .env.example .env
docker compose up -d --build
```

Увеличиваем счётчик до удаления контейнера:
```bash
curl -fsS -X POST http://127.0.0.1:8080/increment; echo
curl -fsS -X POST http://127.0.0.1:8080/increment; echo
curl -fsS -X POST http://127.0.0.1:8080/increment; echo
curl -fsS http://127.0.0.1:8080/; echo
```
```
{"greeting": "Привет, ВШЭ", "count": 1, "hostname": "980b33040d4f", "storage": "sqlite"}
{"greeting": "Привет, ВШЭ", "count": 2, "hostname": "980b33040d4f", "storage": "sqlite"}
{"greeting": "Привет, ВШЭ", "count": 3, "hostname": "980b33040d4f", "storage": "sqlite"}
{"greeting": "Привет, ВШЭ", "count": 3, "hostname": "980b33040d4f", "storage": "sqlite"}
```

Удаляем контейнер и создаём заново:
```bash
docker compose down
docker compose up -d
curl -fsS http://127.0.0.1:8080/; echo
docker volume ls
docker compose ps
```
```
{"greeting": "Привет, ВШЭ", "count": 3, "hostname": "6dd4d124e8c4", "storage": "sqlite"}
DRIVER    VOLUME NAME
local     hw_3_template_app_data
```

## 8 баллов: отдельное хранилище (Redis)

Запуск:
```bash
docker compose up -d --build
docker compose ps
```

### Проверка 1: данные сохраняются после down/up
```bash
curl -fsS -X POST http://127.0.0.1:8080/increment; echo
curl -fsS -X POST http://127.0.0.1:8080/increment; echo
curl -fsS -X POST http://127.0.0.1:8080/increment; echo
curl -fsS http://127.0.0.1:8080/; echo
docker compose down
docker compose up -d
sleep 10
curl -fsS http://127.0.0.1:8080/; echo
```
```
{"greeting": "Привет, ВШЭ", "count": 1, "hostname": "9894b4d35083", "storage": "redis"}
{"greeting": "Привет, ВШЭ", "count": 2, "hostname": "9894b4d35083", "storage": "redis"}
{"greeting": "Привет, ВШЭ", "count": 3, "hostname": "9894b4d35083", "storage": "redis"}
{"greeting": "Привет, ВШЭ", "count": 3, "hostname": "9894b4d35083", "storage": "redis"}
[+] Running 3/3
 ✔ Container hw_3_template-app-1    Removed
 ✔ Container hw_3_template-redis-1  Removed
 ✔ Network hw_3_template_default    Removed
[+] Running 3/3
 ✔ Network hw_3_template_default    Created
 ✔ Container hw_3_template-redis-1  Healthy
 ✔ Container hw_3_template-app-1    Started
{"greeting": "Привет, ВШЭ", "count": 3, "hostname": "43a99841ed09", "storage": "redis"}
```


### Проверка 2: Redis остановлен
```bash
docker compose stop redis
curl -s -o /dev/null -w "healthz: %{http_code}\n" http://127.0.0.1:8080/healthz
curl -s -o /dev/null -w "readyz: %{http_code}\n" http://127.0.0.1:8080/readyz
curl -s -o /dev/null -w "POST increment: %{http_code}\n" -X POST http://127.0.0.1:8080/increment
```
```
healthz: 200
readyz: 503
POST increment: 503
```

### Проверка 3: Redis снова запущен
```bash
docker compose start redis
sleep 10
docker compose ps
curl -fsS http://127.0.0.1:8080/; echo
curl -fsS -X POST http://127.0.0.1:8080/increment; echo
```
```
NAME                    IMAGE                COMMAND                  SERVICE   CREATED          STATUS                    PORTS
hw_3_template-app-1     hw_3_template-app    "python app.py"          app       About a minute ago   Up About a minute (healthy)   0.0.0.0:8080->8000/tcp, [::]:8080->8000/tcp
hw_3_template-redis-1   redis:7.4-alpine     "docker-entrypoint.s…"   redis     About a minute ago   Up 10 seconds (healthy)       6379/tcp
{"greeting": "Привет, ВШЭ", "count": 3, "hostname": "43a99841ed09", "storage": "redis"}
{"greeting": "Привет, ВШЭ", "count": 4, "hostname": "43a99841ed09", "storage": "redis"}
```

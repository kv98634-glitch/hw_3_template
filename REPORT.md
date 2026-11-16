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
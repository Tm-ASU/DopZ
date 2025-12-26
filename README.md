отчет 
# Мониторинг Docker-контейнеров с помощью Grafana и Prometheus


## Этап 1: Подготовка файлов конфигурации
---
1. Создаем новую директорию для проекта

 mkdir -p DOP

  cd DOP
  
---

## Этап 2: Запуск стека мониторинга

nano docker-compose.yml


version: '3.8'

services:
  webapp:
    image: nginx:alpine
    container_name: webapp
    ports:
      - "80:80"
    networks:
      - monitor-net

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:v0.47.2
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    privileged: true
    devices:
      - /dev/kmsg
    networks:
      - monitor-net

  prometheus:
    image: prom/prometheus:v2.47.1
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      - monitor-net

  grafana:
    image: grafana/grafana:10.1.5
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    networks:
      - monitor-net

networks:
  monitor-net:
    driver: bridge


nano prometheus.yml


global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

docker-compose up -d

docker ps


Должны быть видны 4 контейнера: webapp, cadvisor, prometheus, grafana.


---

## Этап 3: Проверка cAdvisor

Открываем в браузере:
🔗 http://localhost:8080
Убеждаемся, что:
Видны все запущенные контейнеры (включая webapp, prometheus и т.д.)
Отображаются метрики CPU, памяти, диска

---


## Этап 4: Проверка Prometheus


Открываем:
🔗 http://localhost:9090
Перейди: Status → Targets
→ Убеждаемся, что cadvisor имеет статус UP


## Этап 5: Подключение Grafana к Prometheus


Открываем:
🔗 http://localhost:3000
Входим:
Логин: admin
Пароль: admin
При первом входе — меняем пароль (можно оставить admin для теста)
Добавляем источник данных:
Configuration → Data Sources → Add data source
Выбераем Prometheus
В поле URL введи:
1
http://prometheus:9090


rate(container_cpu_usage_seconds_total{name="webapp"}[5m])


В терминале выполняем:


for i in {1..1000}; do curl -s http://localhost > /dev/null; done



sudo apt update
sudo apt install -y docker.io

sudo systemctl start docker
sudo systemctl enable docker

sudo systemctl status docker

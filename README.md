# Master of Relics — Infrastructure

Репозиторий для развёртывания инфраструктуры многопользовательской пошаговой тактической дуэли **Master of Relics**. Содержит конфигурации Docker Compose, мониторинга (Prometheus + Grafana), логирования (Loki + Promtail), резервного копирования и reverse-proxy (Nginx Proxy Manager).

## Технологический стек

| Компонент | Технология |
|-----------|-------------|
| **Контейнеризация** | Docker + Docker Compose |
| **База данных** | PostgreSQL 16-alpine |
| **Кэш/игровые сессии** | Redis + RediJSON |
| **Reverse Proxy** | Nginx Proxy Manager |
| **Мониторинг** | Prometheus + Grafana |
| **Логирование** | Loki + Promtail |
| **Резервное копирование** | postgres-backup-s3 (Yandex Object Storage) |
| **Хостинг** | Yandex Cloud + reg.ru |

## Структура проекта

```
.
├── docker-compose.yml # Оркестрация всех сервисов
├── .env.backup # Конфигурация бэкапов (не коммитится)
├── .env.backup.example # Пример конфигурации бэкапов
├── prometheus/
│ └── prometheus.yml # Конфигурация Prometheus
├── grafana/
│ ├── provisioning/ # Автоматическая настройка datasources
│ └── dashboards/ # Готовые дашборды Grafana
├── loki-config.yaml # Конфигурация Loki
├── promtail-config.yaml # Конфигурация Promtail (сбор логов)
└── npm-data/ # Данные Nginx Proxy Manager (создаётся автоматически)
```

## Запуск инфраструктуры

## Установка и настройка на удаленном сервере

```bash
1. sudo apt update
2. sudo apt install git-all
3. git clone https://github.com/Astrof123/Master-of-Relics-Deploy.git .
4. git clone https://github.com/Astrof123/Master-of-Relics-Frontend.git
5. git clone https://github.com/Astrof123/Master-of-Relics-Backend.git

6. Установка докера:

# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

Затем это:
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

7. Затем создаём .env файлы где это нужно.
8. Настраиваем пользователя:

# Добавить текущего пользователя в группу docker
sudo usermod -aG docker $USER

# Применить изменения (нужно выйти и зайти заново или выполнить)
newgrp docker

# Проверить, что пользователь в группе docker
groups $USER

# Перезапустить Docker сервис
sudo systemctl restart docker

# Проверить, что ошибка исчезла
docker ps

9. Остановить Apache:

sudo systemctl stop apache2

# Отключить автозапуск Apache2
sudo systemctl disable apache2

10. docker compose up --build

11. sudo ln -s /home/user/relics_app /app
```

### Запуск всех сервисов

```bash
# Запуск всех сервисов в фоновом режиме
docker compose -p masterofrelics up -d

# Просмотр логов всех сервисов
docker compose logs -f

# Просмотр логов конкретного сервиса
docker compose logs -f backend

# Остановка всех сервисов
docker compose down

# Перезапуск с пересборкой образов
docker compose up -d --build
```

## Сервисы
|Сервис | Описание |	Порты |
|-------|----------|----------|
db|	PostgreSQL 16 |	5432 (внутренний)
redis|	Redis с модулем RediJSON|	6379 (внутренний)
backend|	NestJS сервер (из отдельного репозитория)	|3000 (внутренний)
frontend|	React клиент (из отдельного репозитория)	|80/443 (через NPM)
nginx-proxy-manager|	Reverse proxy + SSL (Let's Encrypt)|	80, 443, 81 (admin)
prometheus|	Сбор метрик	9090
grafana|	Визуализация метрик и логов	|3001
loki|	Агрегация логов	3100
promtail|	Сбор логов из Docker-контейнеров	|9080 (внутренний)
postgres-backup|	Автоматический бэкап БД в S3|	—


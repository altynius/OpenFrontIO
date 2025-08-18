# 🚀 Установка и настройка OpenFront.io на своём сервере

Эта документация описывает пошаговый процесс развёртывания игрового сервера OpenFront.io на своём сервере.

## 📋 Требования

### Системные требования

- **ОС**: Ubuntu 20.04+ или другой Linux дистрибутив
- **RAM**: Минимум 4GB, рекомендуется 8GB+
- **CPU**: 2+ cores
- **Диск**: 20GB+ свободного места
- **Сеть**: Статический IP-адрес

### Программное обеспечение

- **Docker** (будет установлен автоматически)
- **Node.js 24+** (для локальной разработки)
- **npm v10.9.2+** (для локальной разработки)
- **SSH-доступ** к серверу

### Внешние сервисы

- **Docker Hub** аккаунт для хранения образов
- **Cloudflare** аккаунт с доменом
- **OpenTelemetry** endpoint для мониторинга (опционально)

## 🗝️ Необходимые ключи и токены

Перед началом получите следующие данные:

1. **Docker Hub**:

   - Username
   - Access Token

2. **Cloudflare**:

   - Account ID
   - API Token
   - Домен под управлением Cloudflare

3. **OpenTelemetry** (опционально):
   - OTLP Endpoint
   - Authorization Header

## 🎯 Способы развёртывания

Есть два основных сценария развёртывания:

### 1. Полное развёртывание (Production)

Автоматическая сборка, загрузка в Docker Hub и развёртывание на удалённом сервере.

### 2. Локальная разработка

Запуск сервера локально для разработки и тестирования.

---

## 🏭 Полное развёртывание (Production)

### Шаг 1: Настройка конфигурации

1. **Склонируйте репозиторий** (на локальной машине):

   ```bash
   git clone https://github.com/openfrontio/OpenFrontIO.git
   cd OpenFrontIO
   ```

2. **Создайте файл `.env`** на основе `example.env`:

   ```bash
   cp example.env .env
   ```

3. **Отредактируйте `.env`** со своими данными:

   ```bash
   # SSH Configuration
   SSH_KEY=~/.ssh/your-private-key
   
   # Docker Configuration
   DOCKER_USERNAME=your-docker-username
   DOCKER_REPO=openfront-server
   DOCKER_TOKEN=your_docker_hub_token
   
   # Admin credentials
   ADMIN_TOKEN=secure_random_admin_token_here
   
   # Cloudflare Configuration
   CF_ACCOUNT_ID=your_cloudflare_account_id
   CF_API_TOKEN=your_cloudflare_api_token
   DOMAIN=yourdomain.com
   
   # R2 Configuration (для файлового хранилища)
   R2_ACCESS_KEY=your_r2_access_key
   R2_SECRET_KEY=your_r2_secret_key
   R2_BUCKET=your-bucket-name
   
   # Server Hosts
   SERVER_HOST_STAGING=your.staging.server.ip
   SERVER_HOST_EU=your.production.server.ip
   SERVER_HOST_NBG1=your.backup.server.ip
   SERVER_HOST_MASTERS=your.masters.server.ip
   ```

4. **Создайте файл `.env.prod`** для production настроек:
   ```bash
   # Production-specific environment variables
   GAME_ENV=prod
   OTEL_EXPORTER_OTLP_ENDPOINT=https://your-telemetry-endpoint.com/v1/metrics
   OTEL_AUTH_HEADER="Bearer your-otel-token"
   OTEL_USERNAME=your-otel-username
   OTEL_PASSWORD=your-otel-password
   ```

### Шаг 2: Настройка сервера

1. **Подключитесь к серверу** по SSH:

   ```bash
   ssh root@your.server.ip
   ```

2. **Установите необходимые переменные окружения**:

   ```bash
   export OTEL_EXPORTER_OTLP_ENDPOINT="https://your-telemetry-endpoint.com/v1/metrics"
   export OTEL_AUTH_HEADER="Bearer your-otel-token"
   ```

3. **Загрузите и выполните скрипт настройки**:

   ```bash
   # Скопируйте setup.sh с локальной машины или загрузите с репозитория
   wget https://raw.githubusercontent.com/openfrontio/OpenFrontIO/main/setup.sh
   chmod +x setup.sh
   ./setup.sh
   ```

   Этот скрипт:

   - Обновит систему
   - Установит Docker
   - Создаст пользователя `openfront`
   - Настроит SSH-ключи
   - Установит Node Exporter и OpenTelemetry Collector
   - Настроит UDP буферы для QUIC

### Шаг 3: Сборка и развёртывание

На **локальной машине** выполните одну из команд:

#### Вариант А: Полная автоматическая сборка и развёртывание

```bash
# Syntax: ./build-deploy.sh [prod|staging] [eu|nbg1|staging|masters] [subdomain] [--enable_basic_auth]
./build-deploy.sh prod eu mysubdomain
```

#### Вариант Б: Пошаговое развёртывание

1. **Сборка Docker образа**:

   ```bash
   # Syntax: ./build.sh [prod|staging] [version_tag]
   ./build.sh prod $(date +"%Y%m%d-%H%M%S")
   ```

2. **Развёртывание на сервер**:
   ```bash
   # Syntax: ./deploy.sh [prod|staging] [eu|nbg1|staging|masters] [version_tag] [subdomain] [--enable_basic_auth]
   ./deploy.sh prod eu 20231201-143000 mysubdomain
   ```

### Шаг 4: Настройка Cloudflare Tunnel

После первого запуска контейнера:

1. **Проверьте логи контейнера**:

   ```bash
   docker logs openfront-prod-mysubdomain
   ```

2. **Найдите ссылку для авторизации Cloudflare Tunnel** в логах и перейдите по ней в браузере

3. **Авторизуйте туннель** в вашем Cloudflare аккаунте

### Шаг 5: Проверка работы

1. **Проверьте статус контейнеров**:

   ```bash
   docker ps
   ```

2. **Проверьте логи**:

   ```bash
   docker logs openfront-prod-mysubdomain
   docker logs node-exporter
   docker logs otel-collector
   ```

3. **Откройте игру в браузере**:
   ```
   https://mysubdomain.yourdomain.com
   ```

---

## 🛠️ Локальная разработка

### Шаг 1: Установка зависимостей

1. **Склонируйте репозиторий**:

   ```bash
   git clone https://github.com/openfrontio/OpenFrontIO.git
   cd OpenFrontIO
   ```

2. **Установите зависимости**:
   ```bash
   npm install
   ```

### Шаг 2: Запуск в режиме разработки

#### Полный запуск (клиент + сервер)

```bash
npm run dev
```

Это запустит:

- Webpack dev server на порту 9000 (клиент)
- Game server на порту 3000 (сервер)
- Автоматически откроет браузер

#### Только сервер

```bash
npm run start:server-dev
```

#### Только клиент

```bash
npm run start:client
```

### Шаг 3: Доступ к игре

Откройте браузер и перейдите по адресу:

```
http://localhost:9000
```

---

## 🔧 Управление сервером

### Обновление сервера

Для обновления уже развёрнутого сервера:

```bash
# Пересобрать и развернуть новую версию
./build-deploy.sh prod eu mysubdomain

# Или только развернуть уже собранный образ
./deploy.sh prod eu latest mysubdomain
```

### Управление контейнерами

```bash
# Остановить контейнер
docker stop openfront-prod-mysubdomain

# Запустить контейнер
docker start openfront-prod-mysubdomain

# Перезапустить контейнер
docker restart openfront-prod-mysubdomain

# Удалить контейнер
docker rm openfront-prod-mysubdomain

# Просмотр логов
docker logs -f openfront-prod-mysubdomain
```

### Мониторинг

#### Системные метрики

- **Node Exporter**: http://your.server.ip:9100/metrics
- **OpenTelemetry Collector**: форвардит метрики в ваш OTLP endpoint

#### Логи приложения

```bash
# Логи основного контейнера
docker logs -f openfront-prod-mysubdomain

# Логи мониторинга
docker logs -f node-exporter
docker logs -f otel-collector
```

---

## 🎛️ Конфигурация

### Переменные окружения

Основные переменные окружения, которые можно настроить:

```bash
# Игровая среда
GAME_ENV=prod | staging | dev

# Домен и поддомен
DOMAIN=yourdomain.com
SUBDOMAIN=mysubdomain

# Cloudflare
CF_ACCOUNT_ID=your_account_id
CF_API_TOKEN=your_api_token

# Docker
DOCKER_USERNAME=username
DOCKER_REPO=repository
DOCKER_TOKEN=token

# Мониторинг
OTEL_EXPORTER_OTLP_ENDPOINT=https://your-endpoint
OTEL_AUTH_HEADER="Bearer token"

# Безопасность
ADMIN_TOKEN=secure_token
BASIC_AUTH_USER=admin
BASIC_AUTH_PASS=password
```

### Порты

По умолчанию используются следующие порты:

- **3000**: Основной сервер
- **3001-3041**: Worker серверы (до 40 воркеров)
- **9000**: Webpack dev server (только для разработки)
- **9100**: Node Exporter
- **80**: Nginx (внутри контейнера)

### Архитектура

```
[Cloudflare Tunnel] → [Nginx] → [Node.js Server:3000]
                                      ↓
                              [Worker 1:3001]
                              [Worker 2:3002]
                              [Worker N:30XX]
```

---

## 🐛 Устранение неполадок

### Проблемы с Docker

```bash
# Проверить запущенные контейнеры
docker ps

# Проверить все контейнеры
docker ps -a

# Очистить неиспользуемые образы
docker image prune -a

# Проверить логи
docker logs container_name
```

### Проблемы с Cloudflare Tunnel

1. **Проверьте токены**: Убедитесь, что CF_API_TOKEN корректен
2. **Проверьте домен**: Домен должен быть под управлением Cloudflare
3. **Проверьте авторизацию**: Перейдите по ссылке авторизации в логах

### Проблемы с мониторингом

```bash
# Проверить Node Exporter
curl http://localhost:9100/metrics

# Проверить конфигурацию OpenTelemetry
docker exec -it otel-collector cat /etc/otelcol-contrib/config.yaml

# Перезапустить сервисы мониторинга
docker restart node-exporter otel-collector
```

### Проблемы с памятью

Если сервер использует много памяти:

```bash
# Проверить использование ресурсов
docker stats

# Ограничить память для контейнера
docker run -m 2g ...
```

---

## 📚 Дополнительные ресурсы

- **Репозиторий**: https://github.com/openfrontio/OpenFrontIO
- **Основной сайт**: https://openfront.io/
- **Discord разработчиков**: https://discord.gg/K9zernJB5z
- **Документация Cloudflare Tunnel**: https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/

---

## ✅ Checklist развёртывания

- [ ] Сервер с Ubuntu 20.04+ готов
- [ ] SSH-ключи настроены
- [ ] Docker Hub аккаунт создан
- [ ] Cloudflare домен настроен
- [ ] `.env` файлы созданы и заполнены
- [ ] `setup.sh` выполнен на сервере
- [ ] Сборка и развёртывание выполнены
- [ ] Cloudflare tunnel авторизован
- [ ] Игра доступна через браузер
- [ ] Мониторинг работает
- [ ] Логи проверены

**Готово! Ваш сервер OpenFront.io запущен! 🎉**

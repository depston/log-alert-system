# log-alert-system
# 🤖 Система мониторинга логов Linux с Telegram-ботом

Telegram-бот для удаленного управления мониторингом критических событий в логах Linux-сервера.

---

## ⚡ Быстрый старт

### 1. Клонирование и запуск
```bash
git clone <repository>
cd log-alert-system
chmod +x start.sh
./start.sh
```

### 2. Ручная настройка (альтернатива)
```bash
git clone <repository>
cd log-alert-system
cp env.example .env
# Отредактируйте .env с вашими настройками (см. ниже)
./docker-run.sh start
```

### 3. Настройка переменных окружения
Создайте файл `.env` на основе `env.example`:

```
BOT_TOKEN=your_bot_token_here
CHAT_ID=your_chat_id_here
LOGGER_MODE=normal  # normal | verbose | silent
LOG_FILES=/var/log/auth.log /var/log/syslog /var/log/kern.log
CHECK_INTERVAL=30
TZ=Europe/Moscow
COMPOSE_PROJECT_NAME=log-alert-system
```

- `BOT_TOKEN` — токен Telegram-бота
- `CHAT_ID` — ваш chat_id для уведомлений
- `LOGGER_MODE` — режим логирования: normal (по умолчанию), verbose (все события), silent (без уведомлений)
- `LOG_FILES` — список лог-файлов для мониторинга
- `CHECK_INTERVAL` — интервал проверки логов (сек)

### 4. Настройка Telegram
1. Создайте бота через @BotFather
2. Получите Chat ID через @userinfobot
3. Впишите значения в `.env`

### 5. Запуск
```bash
# Docker (рекомендуется)
./docker-run.sh start

# Или systemd (опционально)
sudo chmod +x install.sh
sudo ./install.sh
sudo systemctl start log-bot.service log-watcher.service
```

---

## 📁 Структура проекта

- `start.sh` — быстрый запуск всей системы
- `docker-run.sh` — управление контейнерами (start, stop, status, logs, test и др.)
- `config/bot_control.sh` — Telegram-бот для управления логгером
- `config/log_watcher.sh` — мониторинг логов и отправка событий в Telegram
- `scripts/settings.conf` — основные настройки логгера
- `env.example` — пример переменных окружения
- `test/generate_test_logs.sh` — генерация тестовых логов
- `nginx.conf` — конфигурация веб-интерфейса (опционально)

---

## 📱 Команды Telegram-бота

| Команда         | Описание                  |
|----------------|---------------------------|
| `/start`       | Начать работу              |
| `/status`      | Статус системы             |
| `/mode_normal` | Обычный режим              |
| `/mode_verbose`| Подробный режим            |
| `/mode_silent` | Отключить уведомления      |

---

## 🔧 Режимы работы

- **Normal**: критические события и предупреждения
- **Verbose**: все события системы
- **Silent**: уведомления отключены

---

## 🐳 Docker-команды

```bash
./docker-run.sh start      # Запуск системы
./docker-run.sh stop       # Остановка
./docker-run.sh status     # Статус контейнеров
./docker-run.sh logs       # Просмотр логов
./docker-run.sh test       # Запуск тестов
./docker-run.sh dev --web  # Режим разработки с веб-интерфейсом
```

---

## 🧪 Тестирование

### Быстрый запуск тестов
```bash
./docker-run.sh test
```

### Ручное тестирование
```bash
./config/bot_control.sh test
./config/log_watcher.sh test
./test/generate_test_logs.sh create
```

- Скрипт `generate_test_logs.sh` создаёт тестовые логи в `/tmp/test_logs`.
- Для проверки работы системы используйте тестовые команды и просматривайте логи через Docker или напрямую.

---

## 🛠️ Устранение неполадок

- **Проблемы с кодировкой**: `dos2unix scripts/settings.conf`
- **Проверка конфигурации**: `./config/bot_control.sh test`
- **Логи Docker**: `docker logs log-alert-bot`
- **Права на скрипты**: `chmod +x *.sh config/*.sh scripts/*.sh`
- **Проверка зависимостей**: убедитесь, что установлены `curl`, `jq`, `docker`, `docker-compose`

---

## ℹ️ Примечания

- Для работы веб-интерфейса используется nginx (порт 8080 по умолчанию).
- Все настройки логирования и фильтрации событий задаются в `scripts/settings.conf` или через переменные окружения.
- Для расширенного мониторинга можно подключить Grafana и Redis (см. опции docker-run.sh).

---

## ❗ Возможные ошибки и их решения

| Ошибка/Симптом                | Причина/Описание                                      | Решение                                   |
|-------------------------------|------------------------------------------------------|--------------------------------------------|
| `❌ Docker не установлен`      | Docker не найден в системе                           | Установите Docker и Docker Compose         |
| `❌ Файл .env не найден!`      | Не создан файл с переменными окружения               | Скопируйте `env.example` в `.env` и заполните |
| `❌ BOT_TOKEN не задан`        | Не указан токен Telegram-бота                        | Укажите BOT_TOKEN в `.env`                 |
| `❌ CHAT_ID не задан`          | Не указан chat_id                                    | Укажите CHAT_ID в `.env`                   |
| `permission denied` при запуске| Нет прав на выполнение скриптов                      | Выполните: `chmod +x *.sh config/*.sh scripts/*.sh` |
| `curl: command not found`      | Не установлен curl                                   | Установите: `sudo apt-get install curl`    |
| `jq: command not found`        | Не установлен jq                                     | Установите: `sudo apt-get install jq`      |
| `docker: command not found`    | Не установлен Docker                                 | Установите Docker                         |
| `Cannot connect to the Docker daemon` | Docker не запущен или нет прав | Запустите Docker: `sudo systemctl start docker` или перезагрузите систему |
| `Error: EACCES: permission denied` | Нет прав на доступ к логам или директориям | Запустите с sudo или настройте права доступа |
| Бот не отвечает в Telegram     | Неверный токен или chat_id, либо нет доступа в интернет | Проверьте значения в `.env` и интернет-соединение |
| Нет уведомлений о событиях     | Неправильный LOGGER_MODE или фильтры событий         | Проверьте настройки в `.env` и `settings.conf` |
| Логи не обновляются            | Неправильные пути к логам или нет новых событий      | Проверьте переменную LOG_FILES и права доступа |
| Ошибка кодировки               | Windows-формат файлов                                | Выполните: `dos2unix scripts/settings.conf` |
| `Address already in use`       | Порт уже занят другим процессом                      | Освободите порт или измените настройки      |

---

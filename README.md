# 📚 Spring Cloud Gateway — README

## 📌 Обзор проекта

Этот проект реализует **API Gateway** на основе **Spring Cloud Gateway**, который служит единой точкой входа для микросервисов `user-service` и `notification-service`. Шлюз выполняет маршрутизацию запросов, обеспечивая централизованное управление трафиком.

## 🏗️ Архитектура

```
┌─────────────────────────────────────────────────────────────┐
│                    Клиентское приложение                     │
│                   (например, браузер, мобильное)             │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │ Запросы на порт 8079
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                  API GATEWAY (gateway-service)              │
│                      Порт: 8079                             │
├─────────────────┬───────────────────────────────────────────┤
│ /api/users/**   │ → user-service (порт 8080)                │
│ /api/notifications/** │ → notification-service (порт 8081)  │
└─────────────────┴───────────────────────────────────────────┘
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
┌─────────────────────┐ ┌─────────────────────────────┐
│   user-service      │ │   notification-service      │
│   Порт: 8080        │ │   Порт: 8081                │
│                     │ │                             │
│ - CRUD пользователей│ │ - Отправка уведомлений      │
│ - Работа с БД       │ │ - Интеграция с Kafka        │
│ - Kafka producer    │ │ - Kafka consumer            │
└─────────────────────┘ └─────────────────────────────┘
```

## 📋 Требования

- **Java** 17 или выше
- **Maven** 3.6+
- **Spring Boot** 3.2.x
- **Spring Cloud** 2023.0.x

## 🚀 Быстрый старт

### 1. Клонирование и сборка

```bash
# Клонировать репозиторий
git clone <ваш-репозиторий>
cd gateway-service

# Собрать проект
mvn clean package
```

### 2. Запуск сервисов

**Порядок запуска важен:**

1. **Сначала запустите целевые сервисы:**
```bash
# В отдельном терминале - user-service
cd user-service
mvn spring-boot:run
# Сервис будет доступен на http://localhost:8080

# В отдельном терминале - notification-service
cd notification-service
mvn spring-boot:run
# Сервис будет доступен на http://localhost:8081
```

2. **Затем запустите шлюз:**
```bash
# В отдельном терминале - gateway-service
cd gateway-service
mvn spring-boot:run
# Шлюз будет доступен на http://localhost:8079
```

### 3. Проверка работоспособности

```bash
# Проверка шлюза
curl http://localhost:8079/actuator/health

# Проверка маршрутизации к notification-service
curl http://localhost:8079/api/notifications/health
# Ожидаемый ответ: "Notification service is running"

# Проверка маршрутизации к user-service
curl http://localhost:8079/api/users
# Ожидаемый ответ: список пользователей (или пустой массив)
```

## ⚙️ Конфигурация шлюза

Конфигурация находится в `src/main/resources/application.properties`:

```properties
# Базовые настройки
server.port=8079
spring.application.name=gateway-service

# Логирование (отладка)
logging.level.org.springframework.cloud.gateway=DEBUG
logging.level.reactor.netty=DEBUG

# Маршрут к user-service
spring.cloud.gateway.server.webflux.routes[0].id=user-service-route
spring.cloud.gateway.server.webflux.routes[0].uri=http://localhost:8080
spring.cloud.gateway.server.webflux.routes[0].predicates[0]=Path=/api/users/**

# Маршрут к notification-service
spring.cloud.gateway.server.webflux.routes[1].id=notification-service-route
spring.cloud.gateway.server.webflux.routes[1].uri=http://localhost:8081
spring.cloud.gateway.server.webflux.routes[1].predicates[0]=Path=/api/notifications/**
```

## 🔌 Доступные эндпоинты

Через шлюз доступны следующие маршруты:

### 📱 User Service (`/api/users`)
| Метод | Путь | Описание |
|-------|------|----------|
| GET | `/api/users` | Получить всех пользователей |
| GET | `/api/users/{id}` | Получить пользователя по ID |
| POST | `/api/users` | Создать нового пользователя |
| PUT | `/api/users/{id}` | Обновить пользователя |
| DELETE | `/api/users/{id}` | Удалить пользователя |
| GET | `/api/users/email/{email}` | Найти пользователя по email |

### 🔔 Notification Service (`/api/notifications`)
| Метод | Путь | Описание |
|-------|------|----------|
| POST | `/api/notifications/send` | Отправить уведомление |
| GET | `/api/notifications/health` | Проверка здоровья сервиса |

# 1 Аутентификация и авторизация (AuthService)


### Эндпоинт: /auth/login

**Метод:** POST
**Описание:** Аутентификация пользователя.
**Тело запроса:**

```json
{
  "username": "string",
  "password": "string"
}
```

**Ответ:**

```json
{
  "accessToken": "string",
  "refreshToken": "string"
}
```

### Эндпоинт: /auth/validate

**Метод:** POST
**Описание:** Проверка действительности токена.
**Тело запроса:**

```json

{
  "accessToken": "string"
}


```

**Ответ:**

```json

{
  "isValid": true
}


```

## 1.2. Управление устройствами (DeviceService)

## Эндпоинт: /devices

**Метод:** GET
**Описание:** Получение списка устройств, связанных с пользователем.
**Ответ:**

```json
[
  {
    "id": "UUID",
    "name": "string",
    "status": "on/off",
    "type": "string",
    "houseId": "UUID"
  }
]

```

## Эндпоинт: /devices/{id}/toggle

**Метод:** POST
**Описание:** Переключение состояния устройства.
**Параметры пути:**
```id (UUID) — уникальный идентификатор устройства.``
**Ответ:**

```json
{
  "id": "UUID",
  "status": "on/off"
}

```

## 1.3. Мониторинг данных (MonitoringService)

## Эндпоинт: /telemetry

**Метод:** GET
**Описание:** Получение последних данных телеметрии для устройства.
**Параметры запроса:**
``` deviceId (UUID) — идентификатор устройства.```
**Ответ:**

```json
{
  "deviceId": "UUID",
  "temperature": 22.5,
  "timestamp": "2024-11-29T12:00:00Z"
}

```

# 2. AsyncAPI (Kafka)

## 2.1. Публикация события об изменении состояния устройства (DeviceService → Kafka)
**Тема:** device.state.changed
**Сообщение**

```json
{
  "deviceId": "UUID",
  "newState": "on/off",
  "timestamp": "2024-11-29T12:00:00Z"
}

```

## 2.2. Получение новых данных телеметрии (Sensors → MonitoringService через Kafka)
**Тема:** telemetry.new.data
**Сообщение:**

```json
{
  "deviceId": "UUID",
  "temperature": 23.4,
  "timestamp": "2024-11-29T12:10:00Z"
}

```

## Документацию по API можете посмотреть в файле swagger.yaml через <a href="https://editor.swagger.io/">Swagger Editor</a> 
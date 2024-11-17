# Smart Village Ecosystem API

API для управления умным домом: управление устройствами, мониторинг температуры, управление сценариями и уведомления. Это документация для интеграции с микросервисами экосистемы.

---

## Содержание
1. [Описание проекта](#описание-проекта)
2. [Структура API](#структура-api)
3. [Микросервис «Управление устройствами»](#микросервис-управление-устройствами)
    - [Получение информации об устройстве](#11-получение-информации-об-устройстве)
    - [Обновление состояния устройства](#12-обновление-состояния-устройства)
    - [Отправка команды устройству](#13-отправка-команды-устройству)
4. [Микросервис «Мониторинг температуры»](#микросервис-мониторинг-температуры)
    - [Получение текущей температуры](#21-получение-текущей-температуры)
    - [Регистрация нового датчика](#22-регистрация-нового-датчика)
5. [Микросервис «Управление сценариями»](#микросервис-управление-сценариями)
    - [Создание сценария](#31-создание-сценария)
    - [Запуск сценария](#32-запуск-сценария)
    - [Удаление сценария](#33-удаление-сценария)
6. [Микросервис «Уведомления и события»](#микросервис-уведомления-и-события)
    - [Подписка на события](#41-подписка-на-события)
    - [Получение уведомлений](#42-получение-уведомлений)

---

## Описание проекта

Smart Village Ecosystem API — это программный интерфейс для взаимодействия с экосистемой умного дома. Экосистема позволяет:
- Управлять умными устройствами (освещением, отоплением, воротами и т.д.).
- Настраивать автоматические сценарии работы устройств.
- Получать уведомления о событиях.
- Мониторить телеметрию (например, текущую температуру).

### Основные технологии
- **REST API** для синхронных запросов.
- **AsyncAPI** для асинхронных уведомлений через Kafka.
- JSON — формат передачи данных.

---
openapi: 3.0.0
info:
  title: Smart Village Ecosystem API
  description: API for managing smart devices, temperature monitoring, scenarios, and notifications in the smart village ecosystem.
  version: 1.0.0
servers:
  - url: https://api.smartvillage.com
    description: Production server

paths:
  /devices/{deviceId}:
    get:
      summary: Get device information
      parameters:
        - name: deviceId
          in: path
          required: true
          description: The ID of the device
          schema:
            type: string
      responses:
        '200':
          description: Device information retrieved successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  deviceId:
                    type: string
                  name:
                    type: string
                  status:
                    type: string
                  lastUpdated:
                    type: string
                    format: date-time
                  type:
                    type: string
        '404':
          description: Device not found

  /devices/{deviceId}/state:
    put:
      summary: Update device state
      parameters:
        - name: deviceId
          in: path
          required: true
          description: The ID of the device
          schema:
            type: string
      requestBody:
        description: State update payload
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                state:
                  type: string
                  enum: [on, off]
      responses:
        '200':
          description: Device state updated successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  deviceId:
                    type: string
                  status:
                    type: string
                  message:
                    type: string
        '400':
          description: Invalid request
        '404':
          description: Device not found

  /devices/{deviceId}/commands:
    post:
      summary: Send command to device
      parameters:
        - name: deviceId
          in: path
          required: true
          description: The ID of the device
          schema:
            type: string
      requestBody:
        description: Command payload
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                command:
                  type: string
                parameters:
                  type: object
                  additionalProperties:
                    type: string
      responses:
        '200':
          description: Command sent successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  deviceId:
                    type: string
                  status:
                    type: string
                  message:
                    type: string
        '400':
          description: Invalid request
        '404':
          description: Device not found
        '500':
          description: Server error

  /sensors/{sensorId}/temperature:
    get:
      summary: Get current temperature from sensor
      parameters:
        - name: sensorId
          in: path
          required: true
          description: The ID of the sensor
          schema:
            type: string
      responses:
        '200':
          description: Temperature data retrieved successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  sensorId:
                    type: string
                  temperature:
                    type: number
                  unit:
                    type: string
                    enum: [C, F]
                  timestamp:
                    type: string
                    format: date-time
        '404':
          description: Sensor not found

  /sensors:
    post:
      summary: Register a new sensor
      requestBody:
        description: Sensor registration payload
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                sensorId:
                  type: string
                type:
                  type: string
                  enum: [temperature, humidity]
                location:
                  type: string
      responses:
        '201':
          description: Sensor registered successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  sensorId:
                    type: string
                  status:
                    type: string
                  message:
                    type: string
        '400':
          description: Invalid request

  /scenarios:
    post:
      summary: Create a new scenario
      requestBody:
        description: Scenario creation payload
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                actions:
                  type: array
                  items:
                    type: object
                    properties:
                      deviceId:
                        type: string
                      command:
                        type: string
                      parameters:
                        type: object
                        additionalProperties:
                          type: string
      responses:
        '201':
          description: Scenario created successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  scenarioId:
                    type: string
                  status:
                    type: string
                  message:
                    type: string
        '400':
          description: Invalid request

  /scenarios/{scenarioId}/execute:
    post:
      summary: Execute a scenario
      parameters:
        - name: scenarioId
          in: path
          required: true
          description: The ID of the scenario
          schema:
            type: string
      responses:
        '200':
          description: Scenario executed successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  scenarioId:
                    type: string
                  status:
                    type: string
                  message:
                    type: string
        '404':
          description: Scenario not found

  /scenarios/{scenarioId}:
    delete:
      summary: Delete a scenario
      parameters:
        - name: scenarioId
          in: path
          required: true
          description: The ID of the scenario
          schema:
            type: string
      responses:
        '200':
          description: Scenario deleted successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  scenarioId:
                    type: string
                  status:
                    type: string
                  message:
                    type: string
        '404':
          description: Scenario not found

  /notifications/subscribe:
    post:
      summary: Subscribe to notifications
      requestBody:
        description: Subscription payload
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                userId:
                  type: string
                eventType:
                  type: string
      responses:
        '201':
          description: Subscription created successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  subscriptionId:
                    type: string
                  status:
                    type: string
                  message:
                    type: string
        '400':
          description: Invalid request

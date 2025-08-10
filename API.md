<h1  style="color: pink;">Основные объекты API клиентского приложения "Машина времени"</h1>

### Пояснение: В данной работе опущены авторизация для упрощения описания метода. В методе по бронированию опущена логика опаты. т.к. допускается, что это отдельный метод API.

**Основные объекты API:**

1. **Destinations**
   - Список доступных мест назначения
   - Поля: `destinetion_id`, `name`, `description`, `time_period`, `available_slots`, `price_per_slot`

2. **Bookings**
   - Информация о бронированиях клиента
   - Поля: `booking_id`, `user_id`, `destination_id`, `start_date`, `end_date`, `booking_date`, `status`, `total_cost`

3. **Payments**
   - Информация о платежах за бронирования
   - Поля: `payment_id`, `user_id`, `booking_id`, `amount`, `payment_date`, `payment_method`, `status`

4. **Travel_history**
   - История путешествий клиента
   - Поля: `travel_id`, `booking_id`, `user_id`, `destination_id`, `departure_time`, `arrival_time`, `feedback_score`, `feedback_text`

5. **Users**
   - Информация о пользователях (клиентах)
   - Поля: `user_id`, `manager_id`, `username`, `email`, `password_hash`, `created_at`, `last_login`, `phone_number`, `address`

**Метод: Создание бронирования**

**Логика обработки:**
1. Проверить, что указанное место назначения (`destination_id`) существует и имеет доступные слоты на выбранный период.

```
SELECT available_slots
FROM Destinations
WHERE destination_id = @destination_id;
```
2. Создать новые записи в таблицах:
```
BEGIN TRANSACTION;

-- Обновляем доступные слоты
UPDATE Destinations
SET available_slots = available_slots - @days_count
WHERE destination_id = @destination_id;

-- Вставляем новое бронирование
INSERT INTO Bookings (
    user_id, 
    destination_id,
    start_date,
    end_date,
    booking_date,
    status,
    total_cost
) VALUES (
    @user_id,
    @destination_id,
    @start_date,
    @end_date,
    GETDATE(),
    'confirmed',
    @total_cost
);

COMMIT TRANSACTION;
```

**Структура запроса:**
```json
{
  "destination_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "destination_name": "string",
  "time_period": "string",
  "start_date": "2025-08-10T11:06:20.513Z",
  "end_date": "2025-08-10T11:06:20.513Z",
  "total_cost": 0.00,
  "user_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "last_name": "string",
  "first_name": "string"
}
```

**Структура ответа:**
```json
{
  "booking_id": "123e4567-e89b-12d3-a456-426655440000",
  "destination_id": "789e4567-e89b-12d3-a456-426655440002",
  "start_date": "2025-08-22T16:00:00",
  "end_date": "2715-02-02T08:30:00",
  "booking_date": "2025-08-10T12:15:00",
  "status": "Confirmed",
  "total_cost": 0.00
}
```
```Swagger```
```
openapi: "3.0.3"
info:
  title: Создание нового бронирования путешествия во времени
  version: 1.0.0
servers:
  - url: https://host.ru/api/v1.0
tags:
  - name: Manager
    description: Операции доступные для менеджера
  - name: User
    description: Операции доступные для пользователя
paths:
  /booking:
    post:
      tags:
        - Manager
        - User
      summary: Создать новое бронирование
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                destination_id:
                  type: string
                  format: uuid
                  description: Уникальный идентификатор пункта назначения
                destination_name:
                  type: string
                  description: Название пункта назначения
                time_period:
                  type: string
                  description: Временной период (эпоха) пункта назначения
                start_date:
                  type: string
                  format: date-time
                  description: Дата и время начала бронирования
                end_date:
                  type: string
                  format: date-time
                  description: Дата и время окончания бронирования
                total_cost:
                  type: number
                  format: double
                  description: Общая стоимость бронирования
                user_id:
                  type: string
                  format: uuid
                  description: Уникальный идентификатор пользователя
                last_name:
                  type: string
                  description: Фамилия пользователя
                first_name:
                  type: string
                  description: Имя пользователя
      responses:
        '201':
          description: Путешествие успешно забронировано
          content:
            application/json:    
              schema:
                type: object
                properties:
                  booking_id:
                    type: string
                    format: uuid
                    description: Уникальный идентификатор бронирования
                  destination_id:
                    type: string
                    format: uuid
                    description: Уникальный идентификатор пункта назначения
                  start_date:
                    type: string
                    format: date-time
                    description: Дата и время начала бронирования
                  end_date:
                    type: string
                    format: date-time
                    description: Дата и время окончания бронирования
                  booking_date:
                    type: string
                    format: date-time
                    description: Дата и время создания бронирования
                  status:
                    type: string
                    description: Статус бронирования
                  total_cost:
                    type: number
                    format: double
                    description: Общая стоимость бронирования
        '500':
          description: Внутренняя ошибка сервера
          content:
            application/json:
              schema:
                type: object
                properties:
                  status:
                    type: string
                    example: "500"
                  message:
                    type: string
                    example: "Произошла внутренняя ошибка сервера"
```

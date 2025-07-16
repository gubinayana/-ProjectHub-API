# Методы для управления задачами
В этом разделе описаны методы API для управления задачами: создание, получение списка, получение данных конкретного проекта, частичное обновление проекта, удаление.

## Оглавление
- [Вернуться в README](./README.md)
- [POST /tasks — Создание задачи](#post-tasks)
- [GET /tasks — Получение списка задач](#get-tasks)
- [GET /tasks/{id} — Получение данных задачи](#get-tasksid)
- [PATCH /tasks/{id} - Частичное обновление задачи](#patch-tasksid)
- [DELETE /tasks/{id} — Удаление задачи](#delete-tasksid)

### POST /tasks
**Назначение:** Создание новой задачи, добавление ее в систему.

**Контакт:** support@example.com

**Версия API:** v1 

**Базовый адрес:** api.example.com

**Описание:** Создание новой задачи, с указанием названия, описания, статуса, приоритетности, даты дедлайна, ID проекта, ID исполнителя задачи.

**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer <token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Тело запроса
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|-------|
|title| string| да| minLength: 3, maxLength: 100| Название задачи|
|description|string|нет|maxLength: 500| Описание задачи|
|status| string| да|enum: ["todo", "in_progress", "done"]| Статус задачи ( по умолчанию: "todo") |
|priority| integer| нет |minimum: 1, maximum: 5| Приоритетность задачи (где 1 - высокий приоритет, 5 - низкий приоритет)|
|dueDate| string| нет |format: date| Срок выполнения задачи |
|projectId| integer| да |minimum: 1|ID проекта, к которому привязана задача|
|assigneeId| integer| нет |minimum: 1|ID исполнителя задачи|

#### Пример тела запроса
```http
POST /tasks HTTP/1.1
Host: api.example.com
Content-Type: application/json
```

```json
{
  "title": "Провести тестирование нового релиза",
  "description": "Проверка функциональности всех модулей перед деплоем.",
  "status": "todo",
  "priority": 3,
  "dueDate": "2025-07-15",
  "projectId": 101,
  "assigneeId": 45
}
```

#### Параметры ответа
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
|id| integer| да| ID задачи|
|title| string| да| Название задачи|
|description|string|нет|Описание задачи|
|status| string| да|Статус задачи ( по умолчанию: "todo") |
|priority| integer| нет |Приоритетность задачи (где 1 - высокий приоритет, 5 - низкий приоритет)|
|dueDate| string| нет |Срок выполнения задачи |
|projectId| integer| да |ID проекта, к которому привязана задача|
|assigneeId| integer| нет |ID исполнителя задачи|
|createdAt| string| да| Дата создания записи. Формат: YYYY-MM-DDTHH:MM:SSZ |

#### Пример ответа: 
```json
{
  "id": 173,
  "title": "Провести тестирование нового релиза",
  "description": "Проверка функциональности всех модулей перед деплоем.",
  "status": "todo",
  "priority": 3,
  "dueDate": "2025-07-15",
  "projectId": 101,
  "assigneeId": 45,
  "createdAt": "2025-06-10T12:00:00Z"
}
```
#### Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
| `400`      |`{"error": "Invalid data format"}`| Bad Request — неверные данные запроса   | Проверьте формат данных |
| `409`      |`{"error": "Task with this title exists"}`| 	Conflict — задача с таким названием уже существует | Используйте другое название|  
| `500`      |`{"error": "Unexpected server error"}`| Internal Server Error — сбой сервера  | Повторите запрос позже или обратитесь в поддержку| 

#### Пример ответа с ошибкой
```json
{
  "error": "Task with this title exists"
}
```

### GET /tasks
**Назначение:** Получение списка задач.

**Контакт:** support@example.com

**Версия API:** v1 

**Базовый адрес:** api.example.com

**Описание:** Получение списка задач по ID проекта, ID ответственного с фильтрацией по статусу.

**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer <token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Параметры запроса (Query-параметры)
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|------|
|projectId| integer| да |minimum: 1|ID проекта, к которому привязана задача|
|assigneeId| integer| нет |minimum: 1|ID исполнителя задачи|
|status| string| нет |enum: ["todo", "in_progress", "done"]|Статус задачи|

#### Пример запроса
```http
GET /tasks?projectId=134&assigneeId=56&status=done HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Параметры ответа
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
| total | integer | да | Общее количество задач|
| page | integer | да | Номер текущей страницы |
| limit | integer | да | Количество элементов на странице |
| data | array | да | Список задач|

#### Параметры внутри массива `data`
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
|id| integer| да| ID задачи|
|title| string| да| Название задачи|
|status|string|нет|Статус задачи (Варианты: todo, in_progress, done) |
|priority| integer| да|Приоритетность задачи (где 1 - высокий приоритет, 5 - низкий приоритет)|
|dueDate| string| нет |Срок выполнения задачи |
|projectId| integer| да |ID проекта, к которому привязана задача|
|assigneeId| integer| нет |ID исполнителя задачи|
|createdAt| string| да| Дата создания записи. Формат: YYYY-MM-DDTHH:MM:SSZ |

#### Пример ответа: 
```json
{
  "total": 2,
  "page": 1,
  "limit": 10,
  "data": [
    {
      "id": 101,
      "title": "Составить отчёт по проекту",
      "status": "done",
      "priority": 1,
      "dueDate": "2025-06-01T18:00:00Z",
      "projectId": 134,
      "assigneeId": 56,
      "createdAt": "2025-05-10T09:30:00Z"
    },
    {
      "id": 102,
      "title": "Проверка финальной сборки",
      "status": "done",
      "priority": 3,
      "dueDate": "2025-06-03T14:00:00Z",
      "projectId": 134,
      "assigneeId": 56,
      "createdAt": "2025-05-12T13:15:00Z"
    }
  ]
}
```

#### Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
|`400`|`{"error": "Invalid query parameter 'status'"}`| Bad Request — неверный параметр запроса| Проверьте значения projectId, assigneeId, status|
|`500`|`{"error": "Unexpected server error"}`| Internal Server Error — сбой сервера| Повторите запрос позже или обратитесь в поддержку| 

#### Пример ответа с ошибкой
```json
{
  "error": "Invalid query parameter 'status'"
}
```

### GET /tasks/{id} 
**Назначение:** Получение данных о конкретной задаче.

**Контакт:** support@example.com

**Версия API:** v1 

**Базовый адрес:** api.example.com

**Описание:** Получение данных задачи по ID.

**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer <token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Параметры запроса (Path-параметры)
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|--------|
|id| integer| да| minimum: 1| Идентификатор задачи|

#### Пример запроса
```http
GET /tasks/401
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Параметры ответа
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
|id| integer| да| ID задачи|
|title| string| да| Название задачи|
|status|string|нет|Статус задачи (Варианты: todo, in_progress, done) |
|priority| integer| да|Приоритетность задачи (где 1 - высокий приоритет, 5 - низкий приоритет)|
|dueDate| string| нет |Срок выполнения задачи |
|projectId| integer| да |ID проекта, к которому привязана задача|
|assigneeId| integer| нет |ID исполнителя задачи|
|createdAt| string| да| Дата создания записи. Формат: YYYY-MM-DDTHH:MM:SSZ |

#### Пример ответа: 
```json
{
  "id": 401,
  "title": "Подготовить презентацию для демо",
  "status": "in_progress",
  "priority": 1,
  "dueDate": "2025-06-10T16:00:00Z",
  "projectId": 134,
  "assigneeId": 56,
  "createdAt": "2025-05-28T11:45:00Z"
}
```

#### Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
|`400`|`{"error": "Invalid ID format"}`| Bad Request — неверный формат ID| Проверьте корректность ID|
|`404`|`{"error": "Task not found"}`| Not Found — задача не найдена| Проверьте, существует ли задача с таким ID|
|`500`|`{"error": "Unexpected server error"}`| Internal Server Error — сбой сервера| Повторите запрос позже или обратитесь в поддержку|

#### Пример ответа с ошибкой
```json
{
  "error": "Task not found"
}
```
### PATCH /tasks/{id} 
**Назначение:** Частичное обновление задачи.

**Контакт:** support@example.com

**Версия API:** v1 

**Базовый адрес:** api.example.com

**Описание:** Частичное обновление данных задачи по ID: обновляет название, описание, статус, приоритет, дату окончания или ID ответственного. 

**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer <token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

#### Параметры запроса (Path-параметры)
|Поле|Тип|Обязательность|Описание|
|---|---|---------------|--------|
|id| integer| да | Идентификатор задачи|

#### Параметры тела запроса
|Поле|Тип|Обязательность| Ограничения| Описание|
|---|---|---------------|------|------|
|title| string| нет| minLength: 3, maxLength: 100| Название задачи|
|description|string|нет|maxLength: 500| Описание задачи|
|status| string| нет|enum: ["todo", "in_progress", "done"]| Статус задачи|
|priority| integer| нет |minimum: 1, maximum: 5| Приоритетность задачи (где 1 - высокий приоритет, 5 - низкий приоритет)|
|dueDate| string| нет |format: date| Срок выполнения задачи |
|assigneeId| integer| нет |minimum: 1|ID исполнителя задачи|
**Важно:** хотя поля необязательны, хотя бы одно из них должно быть передано.

#### Пример запроса
```http
PATCH /tasks/401 HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```
```json
{
  "title": "Обновлённая задача: презентация",
  "description": "Подготовить новую версию презентации с учётом обратной связи",
  "status": "in_progress",
  "priority": 2,
  "dueDate": "2025-06-12",
  "assigneeId": 56
}
```

#### Параметры ответа
|Поле|Тип|Обязательность|Описание|
|---|---|---------------|--------|
|id| integer | да| ID задачи|
|title| string| да|  Название задачи|
|description|string|нет| Описание задачи|
|status| string| да|Статус задачи|
|priority| integer| да |Приоритетность задачи (где 1 - высокий приоритет, 5 - низкий приоритет)|
|dueDate| string| нет |Срок выполнения задачи |
|assigneeId| integer| нет |ID исполнителя задачи|
|updatedAt| string | да | Дата обновления задачи. Формат: YYYY-MM-DDTHH:MM:SSZ|

#### Пример успешного ответа: 
```json
{
  "id": 401,
  "title": "Обновлённая задача: презентация",
  "description": "Подготовить новую версию презентации с учётом обратной связи",
  "status": "in_progress",
  "priority": 2,
  "dueDate": "2025-06-12",
  "assigneeId": 56,
  "updatedAt": "2025-05-26T12:00:00Z"
}
```

#### Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
|`400`| `{ "error": "No fields provided for update" }` | Bad Request — не передано ни одного поля| Укажите хотя бы одно поле для обновления|
|`404`| `{"error": "Task not found"}`| Not Found — задача с таким ID не найдена| Проверьте корректность ID|
| `409` |`{"error": "Task with this title exists"}`| Conflict — задача с таким названием уже существует |Используйте другое название|  
|`500`| `{ "error": "Unexpected server error" }`| Internal Server Error — сбой сервера| Повторите запрос позже или обратитесь в поддержку| 

#### Пример ответа с ошибкой
```json
{
 "error": "Task with this title exists" 
}
```

### DELETE /tasks/{id}
**Назначение:** Удаление задачи.
**Контакт:** support@example.com
**Версия API:** v1 
**Базовый адрес:** api.example.com
**Описание:** Удаление данных задачи по ID. 
**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer <token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Параметры запроса (Path-параметры)
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|--------|
|id| integer| да| minimum: 1| Идентификатор задачи| 

#### Пример запроса
```http
DELETE /tasks/201
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Пример успешного ответа
```http
HTTP/1.1 204 No Content
```

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
|`404`| `{ "error": "Task not found" }`| Not Found — задача с таким ID не найдена| Проверьте корректность ID|
|`500`| `{ "error": "Unexpected server error" }`| Internal Server Error — сбой сервера| Повторите запрос позже или обратитесь в поддержку| 

#### Пример ответа с ошибкой
```json
{
 "error": "Task not found"
}
```

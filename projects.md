# Методы для управления проектами
В этом разделе описаны методы API для управления проектами: создание, получение списка, получение данных конкретного проекта, частичное обновление проекта, удаление.

## Оглавление
- [Вернуться в README](./README.md)
- [POST /projects — Создание проекта](#post-projects)
- [GET /projects — Получение списка проектов](#get-projects)
- [GET /projects/{id} — Получение данных проекта](#get-projectsid)
- [PATCH /projects/{id} - Частичное обновление проекта](#patch-projectsid)
- [DELETE /projects/{id} — Удаление проекта](#delete-projectsid)

### POST /projects
**Назначение:** Создание нового проекта, добавление его в систему.
**Контакт:** support@example.com
**Версия API:** v1 
**Базовый адрес:** api.example.com
**Описание:** Создание нового проекта, с указанием названия, описания, дат создания и завершения, ID владельца проекта.
**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer<token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Тело запроса
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|-------|
|title| string| да| minLength: 3, maxLength: 100| Название проекта|
|description|string|нет|maxLength: 500| Описание проекта|
|startDate| string| да|format: date| Дата начала проекта|
|endDate| string| да |format: date| Дата окончания проекта (должна быть позже startDate)|
|ownerId| integer| да |minimum: 1| ID владельца проекта|

#### Пример тела запроса
```http
POST /projects HTTP/1.1
Host: api.example.com
Content-Type: application/json
```

```json
{
  "title": "Система мониторинга задач",
  "description": "Веб-приложение для внутреннего использования, позволяющее команде отслеживать задачи по проектам, видеть статус, сроки и исполнителей.",
  "startDate": "2025-06-01",
  "endDate":"2025-09-30",
  "ownerId": 42
}
```

#### Параметры ответа
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
|id| integer| да| ID проекта|
|title| string| да| Название проекта|
|description|string|нет|Описание проекта|
|startDate| string| да|Дата начала проекта|
|endDate| string| да |Дата окончания проекта (должна быть позже startDate)|
|ownerId| integer| да |ID владельца проекта|
|createdAt| string| да| Дата создания записи. Формат: YYYY-MM-DDTHH:MM:SSZ |

#### Пример ответа: 
```json
{
  "id": 101,
  "title": "Система мониторинга задач",
  "description": "Веб-приложение для внутреннего использования, позволяющее команде отслеживать задачи по проектам, видеть статус, сроки и исполнителей.",
  "startDate": "2025-06-01",
  "endDate": "2025-09-30",
  "ownerId": 42,
  "createdAt": "2025-06-10T12:00:00Z"
}
```
## Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
| `400`      |`{"error": "Invalid data format"}`| Bad Request — неверные данные запроса   | Проверьте формат данных |
| `409`      |`{"error": "Project with this title exists"}`| 	Conflict — проект с таким названием уже существует |Используйте другое название|  
| `500`      |`{"error": "Unexpected server error"}`| Internal Server Error — сбой сервера  | Повторите запрос позже или обратитесь в поддержку| 

## Пример ответа с ошибкой
```json
{
  "error": "Project with this title exists"
}
```

### GET /projects
**Назначение:** Получение списка проектов.
**Контакт:** support@example.com
**Версия API:** v1 
**Базовый адрес:** api.example.com
**Описание:** Получение списка проектов по ID владельца и фильтрации по статусу.

**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer<token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Параметры запроса (Query-параметры)
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|------|
|status| string| нет |enum: [ "active", "completed", "archived" ]|Статус проекта (активный, завершенный, в архиве)|
|ownerId |integer| нет | minimum: 1| ID владельца проекта|

#### Пример запроса
```http
GET /projects?status=active&ownerID=123 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Параметры ответа
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
| total | integer | да | Общее количество проектов|
| page | integer | да | Номер текущей страницы |
| limit | integer | да | Количество элементов на странице |
| data | array | да | Список проектов |

#### Параметры внутри массива `data`
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
|id| integer| да| ID проекта|
|title| string| да| Название проекта|
|description|string|нет|Описание проекта|
|startDate| string| да|Дата начала проекта|
|endDate| string| да |Дата окончания проекта (должна быть позже startDate)|
|ownerId| integer| да |ID владельца проекта|
|status| string| нет |Статус проекта (активный, завершенный, в архиве)|
|createdAt| string| да| Дата создания записи. Формат: YYYY-MM-DDTHH:MM:SSZ |

#### Пример ответа: 
```json
{
  "total": 2,
  "page": 1,
  "limit": 20,
  "data": [
    {
      "id": 201,
      "title": "Модуль интеграции с CRM",
      "description": "Разработка интеграции ProjectHub с корпоративной CRM-системой.",
      "startDate": "2025-04-01",
      "endDate": "2025-07-15",
      "ownerId": 123,
      "status": "active",
      "createdAt": "2025-04-01T09:30:00Z"
    },
    {
      "id": 208,
      "title": "Редизайн клиентского портала",
      "description": "Обновление интерфейса веб-портала для внешних клиентов.",
      "startDate": "2025-05-10",
      "endDate": "2025-09-01",
      "ownerId": 123,
      "status": "active",
      "createdAt": "2025-05-10T14:15:00Z"
    }
  ]
}

```

#### Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
|`400`|`{"error": "Invalid query parameter 'status'"}`| Bad Request — неверный параметр запроса| Проверьте значения status, ownerID|
|`500`|`{"error": "Unexpected server error"}`| Internal Server Error — сбой сервера| Повторите запрос позже или обратитесь в поддержку| 

#### Пример ответа с ошибкой
```json
{
  "error": "Invalid query parameter 'status'"
}
```

### GET /projects/{id} 
**Назначение:** Получение данных проекта.
**Контакт:** support@example.com
**Версия API:** v1 
**Базовый адрес:** api.example.com
**Описание:** Получение данных проекта по ID.
**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer<token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Параметры запроса (Path-параметры)
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|--------|
|id| integer| да| minimum: 1| Идентификатор проекта|

#### Пример запроса
```http
GET /projects/201
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Параметры ответа
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
|id| integer| да| ID проекта|
|title| string| да| Название проекта|
|description|string|нет|Описание проекта|
|startDate| string| да|Дата начала проекта|
|endDate| string| да |Дата окончания проекта (должна быть позже startDate)|
|ownerId| integer| да |ID владельца проекта|
|status| string| нет |Статус проекта (активный, завершенный, в архиве)|
|createdAt| string| да| Дата создания записи. Формат: YYYY-MM-DDTHH:MM:SSZ |

#### Пример ответа: 
```json
{
     "id": 208,
      "title": "Редизайн клиентского портала",
      "description": "Обновление интерфейса веб-портала для внешних клиентов.",
      "startDate": "2025-05-10",
      "endDate": "2025-09-01",
      "ownerId": 123,
      "status": "active",
      "createdAt": "2025-05-10T14:15:00Z"
}
```

#### Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
|`400`|`{"error": "Invalid ID format"}`| Bad Request — неверный формат ID| Проверьте корректность ID|
|`404`|`{"error": "Project not found"}`| Not Found — проект не найден| Проверьте, существует ли проект с таким ID|
|`500`|`{"error": "Unexpected server error"}`| Internal Server Error — сбой сервера| Повторите запрос позже или обратитесь в поддержку|

#### Пример ответа с ошибкой
```json
{
  "error": "Project not found"
}
```
### PATCH /projects/{id} 
**Назначение:** Частичное обновление проекта.
**Контакт:** support@example.com
**Версия API:** v1 
**Базовый адрес:** api.example.com
**Описание:** Частичное обновление данных проекта по ID: обновляет название, описание или дату окончания. 
**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer<token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

#### Параметры запроса (Path-параметры)
|Поле|Тип|Обязательность|Описание|
|---|---|---------------|--------|
|id| integer| да | Идентификатор проекта|

#### Параметры тела запроса
|Поле|Тип|Обязательность| Ограничения| Описание|
|---|---|---------------|------|------|
| title| 	string | нет |minLength: 3, maxLength: 100| Название проекта|
|description| string | нет |maxLength: 500|  Описание проекта|
|endDate| string | нет |format: date| Дата окончания проекта(должна быть позже startDate)|
**Важно:** хотя поля необязательны, хотя бы одно из них должно быть передано.

#### Пример запроса
```http
PATCH /projects/101 HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```
```json
{
  "title": "Редизайн клиентского портала",
  "description": "Обновлённый UI и UX для веб-интерфейса клиентов.",
  "endDate": "2025-10-15"
}
```

#### Параметры ответа
|Поле|Тип|Обязательность|Описание|
|---|---|---------------|--------|
|id| integer | да| ID проекта|
|title| 	string | да | Название проекта|
|description| string | нет| Описание проекта|
|endDate| string | нет|  Дата окончания проекта(должна быть позже startDate)|
|updatedAt| string | да | Дата последнего обновления информации. Формат: YYYY-MM-DDTHH:MM:SSZ |

#### Пример успешного ответа: 
```json
{
  "id": 101,
  "title": "Редизайн клиентского портала",
  "description": "Обновлённый UI и UX для веб-интерфейса клиентов.",
  "endDate": "2025-10-15",
  "updatedAt": "2025-05-26T12:00:00Z"
}
```

#### Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
|`400`| `{ "error": "No fields provided for update" }` | Bad Request — не передано ни одного поля| Укажите хотя бы одно поле для обновления|
|`404`| `{"error": "Project not found"}`| Not Found — проект с таким ID не найден| Проверьте корректность ID|
| `409` |`{"error": "Project with this title exists"}`| Conflict — проект с таким названием уже существует |Используйте другое название|  
|`500`| `{ "error": "Unexpected server error" }`| Internal Server Error — сбой сервера| Повторите запрос позже или обратитесь в поддержку| 

#### Пример ответа с ошибкой
```json
{
 "error": "Project with this title exists" 
}
```

### DELETE /projects/{id}
**Назначение:** Удаление проекта.
**Контакт:** support@example.com
**Версия API:** v1 
**Базовый адрес:** api.example.com
**Описание:** Удаление данных проекта по ID. 
**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer<token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Параметры запроса (Path-параметры)
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|--------|
|id| integer| да| minimum: 1| Идентификатор проекта|

#### Пример запроса
```http
DELETE /projects/201
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Пример успешного ответа
```http
HTTP/1.1 204 No Content
```
| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
|`404`| `{ "error": "Project not found" }`| Not Found — проект с таким ID не найден| Проверьте корректность ID|
| `409` |`{"error": "Cannot delete project with active tasks"}`| Conflict — проект нельзя удалить, если к нему привязаны активные задачи |Завершите или перенесите задачи, прежде чем удалять проект|  
|`500`| `{ "error": "Unexpected server error" }`| Internal Server Error — сбой сервера| Повторите запрос позже или обратитесь в поддержку| 

#### Пример ответа с ошибкой
```json
{
 "error": "Cannot delete project with active tasks"
}
```

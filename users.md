# Методы для работы с пользователями
В этом разделе описаны методы API для управления пользователями: создание, получение списка, просмотр конкретного пользователя.

## Оглавление
- [Вернуться в README](./README.md)
- [POST /users — Создание пользователя](#post-users)
- [GET /users — Получение списка пользователей](#get-users)
- [GET /users/{id} — Получение данных пользователя](#get-usersid)

### POST /users
**Назначение:** Создание нового пользователя, добавление его в систему.
**Контакт:** support@example.com
**Версия API:** v1 (если известно)
**Базовый адрес:** api.example.com
**Описание:** Создание нового пользователя, с указанием имени, email, и пароля.
**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer<token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Тело запроса
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|-------|
|name| string| да| minLength: 2, maxLength: 50| Имя пользователя|
|email|string|да|format: email|  Электронная почта |
|password| string| да|minLength: 8, pattern: ^(?=.*[A-Z])(?=.*\d).+$ — хотя бы одна заглавная буква и цифра| Пароль пользователя|
|role| string| да |enum: [«admin», «manager», «developer»]| Статус пользователя|

#### Пример тела запроса
```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
```

```json
{
  "name": "Anna Ivanova",
  "email": "anna@example.com",
  "password": "SecurePass123",
  "role":"admin"
}
```

#### Параметры ответа
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
|id| integer| да| ID пользователя|
|name| string | да| Имя пользователя |
|email| string| да| Email пользователя|
|role| string| да |Статус пользователя (Варианты: «admin», «manager», «developer»)|
|createdAt| string| да| Дата создания записи. Формат: YYYY-MM-DDTHH:MM:SSZ |

#### Пример ответа: 
```json
{
  "id": 123,
  "name": "Anna Ivanova",
  "email": "anna@example.com",
  "role":"admin",
  "createdAt": "2025-05-26T10:00:00Z"
}
```
## Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
| `400`      |`{"error": "Invalid email format"}`| Bad Request — неверный формат email   | Проверьте формат данных (email, пароль)|
| `409`      |`{"error": "User with this email exists"}`| 	Conflict — пользователь уже существует |Используйте другой email| 
| `500`      |`{"error": "Unexpected server error"}`| Internal Server Error — сбой сервера  | Повторите запрос позже или обратитесь в поддержку| 

## Пример ответа с ошибкой
```json
{
  "error": "Unexpected server error"
}
```

### GET /users
**Назначение:** Получение списка пользователей.
**Контакт:** support@example.com
**Версия API:** v1 (если известно)
**Базовый адрес:** api.example.com
**Описание:** Получение списка пользователей с поддержкой пагинации.
**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer<token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Параметры запроса (Query-параметры)
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|------|
|page| integer| нет |minimum: 1|  Номер страницы (по умолчанию 1)|
|limit|integer| нет | minimum: 1, maximum: 100| 	Количество элементов на странице (по умолчанию 20) |

#### Пример запроса
```http
GET /users?page=7&limit=60 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Параметры ответа
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
| total | integer | да | Общее количество пользователей |
| page | integer | да | Номер текущей страницы |
| limit | integer | да | Количество элементов на странице |
| data | array | да | Список пользователей |

#### Параметры внутри массива `data`
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
|id| integer| да| ID пользователя|
|name| string | да| Имя пользователя |
|email| string| да| Email пользователя|
|createdAt| string| да| Дата создания записи. Формат: YYYY-MM-DDTHH:MM:SSZ |

#### Пример ответа: 
```json
{
  "total": 100,
  "page": 7,
  "limit": 60,
  "data": [
    {
      "id": 1,
      "name": "Anna Ivanova",
      "email": "anna@example.com",
      "createdAt": "2025-05-26T10:00:00Z"
    },
    {
      "id": 2,
      "name": "Ivan Petrov",
      "email": "ivan@example.com",
      "createdAt": "2025-05-20T15:30:00Z"
    }
  ]
}
```

#### Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
|`400`|`{"error": "Invalid query parameter 'page'"}`| Bad Request — неверный параметр запроса| Проверьте значения page, limit|
|`500`|`{"error": "Unexpected server error"}`| Internal Server Error — сбой сервера| Повторите запрос позже или обратитесь в поддержку| 

#### Пример ответа с ошибкой
```json
{
  "error": "Unexpected server error"
}
```

### GET /users/{id}
**Назначение:** Получение информации о пользователе.
**Контакт:** support@example.com
**Версия API:** v1 (если известно)
**Базовый адрес:** api.example.com
**Описание:** Получение информации о пользователе по его ID.
**Авторизация:** Требуется токен доступа. Токен передаётся в заголовке `Authorization` в следующем формате: Authorization: Bearer<token>. 
`Пример: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

####  Параметры запроса (Path-параметры)
|Поле|Тип|Обязательность|Ограничения| Описание|
|---|---|---------------|--------|--------|
|id| integer| да| minimum: 1| Идентификатор пользователя|

#### Пример запроса
```http
GET /users/1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Параметры ответа
|Параметр| Тип | Обязателен | Описание|
|------|----------|----------|----|
|id| integer| да| ID пользователя|
|name| string | да| Имя пользователя |
|email| string| да| Email пользователя|
|createdAt| string| да| Дата создания записи. Формат: YYYY-MM-DDTHH:MM:SSZ |

#### Пример ответа: 
```json
{
      "id": 1,
      "name": "Anna Ivanova",
      "email": "anna@example.com",
      "createdAt": "2025-05-26T10:00:00Z"
}
```

#### Возможные ошибки

| Код ответа | Сообщение                   | Описание   | Рекомендации|
|------------|-----------------------------|----------------------------------------------------|-------|
|`400`|`{"error": "Invalid ID format"}`| Bad Request — неверный формат ID| Проверьте корректность ID|
|`404`|`{"error": "User not found"}`| Not Found — пользователь не найден| Проверьте, существует ли пользователь с таким ID|
|`500`|`{"error": "Unexpected server error"}`| Internal Server Error — сбой сервера| Повторите запрос позже или обратитесь в поддержку|

#### Пример ответа с ошибкой
```json
{
  "error": "Unexpected server error"
}
```
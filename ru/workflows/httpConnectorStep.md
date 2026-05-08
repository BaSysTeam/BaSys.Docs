# HTTP Connector

Шаг **HTTP Connector** (`http_connector`) выполняет HTTP-запрос к указанному URL и сохраняет результат для последующих шагов процесса. Поддерживаются методы GET, POST, PUT, PATCH и DELETE. Ответ может быть автоматически разобран из JSON или XML в объект, пригодный для дальнейшей обработки в скриптах и загрузчиках данных.

## Настройки

### Url

Адрес, на который отправляется запрос. Указывается полный URL, включая протокол.

### Method

HTTP-метод запроса. Допустимые значения:

| Значение | Метод  |
|----------|--------|
| 0        | GET    |
| 1        | POST   |
| 2        | PUT    |
| 3        | PATCH  |
| 4        | DELETE |

### Body

Тело запроса. Используется для передачи данных на сервер. Для метода GET тело отправляется, только если значение задано (нестандартное поведение, применяется для API, которые принимают тело в GET-запросах). Для POST, PUT и PATCH тело обязательно при типе `BodyKind` равном JSON или XML.

### BodyKind

Формат тела запроса. Определяет, каким образом данные будут сериализованы и отправлены:

| Значение | Формат                           | Content-Type по умолчанию                |
|----------|----------------------------------|------------------------------------------|
| 0        | Undefined                        | —                                        |
| 1        | JSON                             | `application/json`                       |
| 2        | FormData (multipart)             | `multipart/form-data`                    |
| 3        | x-www-form-urlencoded            | `application/x-www-form-urlencoded`      |
| 4        | XML                              | `application/xml`                        |

При `BodyKind = JSON` (по умолчанию) содержимое параметра `Body` отправляется как JSON-строка. При `FormData` и `x-www-form-urlencoded` тело формируется из коллекции `FormData`, а параметр `Body` не используется. При `XML` содержимое `Body` отправляется как XML; Content-Type можно переопределить через заголовок `Content-Type` в коллекции `Headers`.

### BodyEncoding

Управляет кодировкой тела запроса:

| Значение | Режим |
|----------|-------|
| -1       | None  |
| 0        | UTF-8 |

При значении `UTF-8` кодировка указывается в заголовке `Content-Type` через параметр `charset` (например, `application/json; charset=utf-8`). При значении `None` заголовок `Content-Type` устанавливается без `charset`.

### Timeout

Тайм-аут выполнения запроса в секундах. Значение по умолчанию — `100`. Если сервер не отвечает в течение указанного времени, запрос завершается с ошибкой. При значении `0` или отрицательном тайм-аут не устанавливается (используется системное значение по умолчанию).

### Headers

Коллекция HTTP-заголовков запроса. Каждый элемент содержит поля `Name`, `Value` и `Description`.

Заголовок `Content-Type` обрабатывается особым образом: его значение устанавливается как `Accept`-заголовок клиента. Все остальные заголовки добавляются напрямую.

Пример в JSON-описании процесса:

```json
"Headers": [
  {
    "Name": "Authorization",
    "Value": "Bearer eyJhbGci...",
    "Description": "Токен авторизации"
  },
  {
    "Name": "Content-Type",
    "Value": "application/json",
    "Description": ""
  }
]
```

### Parameters

Коллекция именованных параметров шага. Каждый элемент содержит поля `Name`, `Value` и `Description`. Параметры используются для хранения пользовательских пар «ключ — значение», которые могут обновляться программно через метод `UpdateParameters`.

### FormData

Коллекция полей формы. Используется при `BodyKind = FormData` или `BodyKind = x-www-form-urlencoded`. Каждый элемент содержит поля `Name`, `Value` и `Description`.

При `FormData` каждое поле отправляется как часть `multipart/form-data`. При `x-www-form-urlencoded` поля кодируются в стандартный формат URL-параметров.

### LogResponse

Если `true`, тело ответа записывается в лог процесса. По умолчанию — `false`. Включайте для отладки; в рабочем режиме рекомендуется отключать, чтобы не засорять лог большими ответами.

### AutoParse

Если `true` (по умолчанию), шаг автоматически определяет формат ответа и преобразует его в объект:

- Если ответ содержит XML (определяется по наличию `<soap` или `xml` в тексте), он преобразуется из XML в JSON, а затем в динамический объект.
- Если ответ начинается с `[`, он разбирается как JSON-массив.
- В остальных случаях ответ разбирается как JSON-объект.

Если `false`, результатом шага будет необработанная строка ответа.

При ошибке парсинга в лог записывается сообщение об ошибке, а результат устанавливается в `null`.

### BypassSslCertificate

Если `true`, проверка SSL-сертификата сервера отключается. По умолчанию — `false`. Используется для обращения к серверам с самоподписанными или недоверенными сертификатами (например, в тестовых средах).

### ReturnResponseInfo

Управляет структурой результата шага.

Если `false` (по умолчанию), результатом является только разобранное тело ответа (или строка при `AutoParse = false`). В случае ошибки HTTP (код ответа 4xx / 5xx) шаг завершается с ошибкой и процесс останавливается.

Если `true`, результатом является объект с метаинформацией об ответе:

```json
{
  "isSuccess": true,
  "statusCode": 200,
  "message": "OK",
  "data": { }
}
```

| Поле         | Тип     | Описание                                            |
|--------------|---------|-----------------------------------------------------|
| `isSuccess`  | boolean | Признак успешного ответа (код 2xx)                  |
| `statusCode` | number  | HTTP-код ответа                                     |
| `message`    | string  | Текстовое описание статуса (`ReasonPhrase`)         |
| `data`       | any     | Разобранное тело ответа                             |

При включённом `ReturnResponseInfo` шаг **не завершается с ошибкой** при неуспешном HTTP-коде. Это позволяет обработать ошибку в последующих шагах (например, в скрипте проверить `statusCode` и принять решение о дальнейших действиях).

### AllowSetCookieFromHeaders

Если `true`, отключает автоматическое управление cookies на уровне `HttpClientHandler`. По умолчанию — `false`. Используется для сценариев, в которых cookies должны устанавливаться вручную через заголовок `Set-Cookie` из ответа, а не обрабатываться встроенным cookie-контейнером.

## Результат шага

После выполнения результат сохраняется и становится доступен последующим шагам через `_data.<имя_шага>`.

Если ответ успешный и `ReturnResponseInfo = false`, результат — разобранные данные ответа.
Если `ReturnResponseInfo = true`, результат — объект с полями `isSuccess`, `statusCode`, `message` и `data`.

При ошибке HTTP и `ReturnResponseInfo = false` процесс останавливается (статус `Terminated`). При `ReturnResponseInfo = true` процесс продолжается, а информация об ошибке доступна в результате шага.

## Обработка ошибок

Шаг считается ошибочным в следующих случаях:

- HTTP-ответ с кодом ошибки (4xx / 5xx) при `ReturnResponseInfo = false`.
- Пустой ответ (response равен `null`) при `ReturnResponseInfo = false`.
- Указан неподдерживаемый HTTP-метод.

При возникновении ошибки процесс переходит в статус `Terminated` и дальнейшие шаги не выполняются. Чтобы обработать HTTP-ошибки в прикладной логике, включите `ReturnResponseInfo` и проверяйте поле `isSuccess` в следующем шаге.

## Пример: POST-запрос с JSON-телом

```json
{
  "Url": "https://api.example.com/v1/trigger/start",
  "BodyKind": 1,
  "BodyEncoding": 1,
  "Body": "{ \"token\": \"abc123\" }",
  "Method": 1,
  "Timeout": 100,
  "LogResponse": false,
  "AutoParse": true,
  "BypassSslCertificate": false,
  "ReturnResponseInfo": false,
  "AllowSetCookieFromHeaders": false,
  "Headers": [],
  "Parameters": [],
  "FormData": [],
  "KindName": "http_connector",
  "Title": "Request",
  "Name": "request",
  "IsActive": true
}
```

Шаг отправляет POST-запрос на указанный URL с JSON-телом. Ответ автоматически разбирается из JSON. Результат доступен в следующих шагах через `_data.request`.

## Пример: GET-запрос с заголовком авторизации

```json
{
  "Url": "https://api.example.com/v1/data",
  "BodyKind": 1,
  "BodyEncoding": 0,
  "Body": "",
  "Method": 0,
  "Timeout": 30,
  "LogResponse": true,
  "AutoParse": true,
  "BypassSslCertificate": false,
  "ReturnResponseInfo": true,
  "AllowSetCookieFromHeaders": false,
  "Headers": [
    {
      "Name": "Authorization",
      "Value": "Bearer eyJhbGci...",
      "Description": "API-токен"
    }
  ],
  "Parameters": [],
  "FormData": [],
  "KindName": "http_connector",
  "Title": "Fetch data",
  "Name": "fetch_data",
  "IsActive": true
}
```

Шаг выполняет GET-запрос с Bearer-токеном. Тело ответа записывается в лог (`LogResponse = true`). Благодаря `ReturnResponseInfo = true` результат содержит код статуса и признак успеха, что позволяет обработать ошибку в последующем скрипте:

```javascript
var response = _data.fetch_data;
if (!response.isSuccess) {
  return { error: true, code: response.statusCode };
}
return response.data;
```

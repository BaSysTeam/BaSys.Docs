# Подключение MCP-сервера BaSYS к Cursor

Эта статья показывает, как подключить встроенный [MCP-сервер BaSYS](mcp.md) к редактору [Cursor](https://cursor.com), чтобы ИИ-агент мог читать метаданные базы и импортировать их обратно прямо из проекта.

Cursor обнаруживает MCP-серверы по файлу `.cursor/mcp.json` в корне проекта. Однако MCP-сервер BaSYS защищён JWT-аутентификацией (см. [Авторизация](auth.md)): каждый запрос требует заголовка `Authorization: Bearer <access-токен>`, а сам токен короткоживущий (по умолчанию 1 час) и периодически обновляется. Статичный заголовок в `mcp.json` для этого не подходит — он «протухнет» вместе с токеном.

Чтобы решить эту задачу, между Cursor и BaSYS ставится небольшой локальный **прокси** (Node.js-скрипт). Он запускается Cursor как stdio-сервер, при старте получает access-токен, обновляет его перед истечением и при ответе `401`, а сами MCP-вызовы транслирует на HTTP-эндпоинт `/mcp` BaSYS.

```
Cursor  ──stdio (JSON-RPC)──►  mcp-proxy.js  ──HTTPS + Bearer──►  BaSYS /mcp
```

## Структура файлов

Все файлы размещаются в каталоге `.cursor` в корне проекта:

| Файл                              | Назначение                                                                 |
|-----------------------------------|----------------------------------------------------------------------------|
| `.cursor/mcp.json`                | Регистрация MCP-сервера в Cursor (запуск прокси).                          |
| `.cursor/mcp-proxy.js`            | Прокси stdio→HTTPS: авторизация, обновление токена и проброс запросов.     |
| `.cursor/basys-credentials.json`  | Учётные данные подключения к BaSYS. **Не коммитится** в репозиторий.       |
| `.cursor/basys-credentials.example.json` | Образец файла учётных данных (без реальных секретов).               |

> Для работы прокси требуется установленный [Node.js](https://nodejs.org) (LTS-версия). Проверить наличие можно командой `node --version`.

## Шаг 1. Учётные данные

Создайте файл `.cursor/basys-credentials.json`. Все значения здесь — плейсхолдеры, которые нужно подставить под своё окружение:

```json
{
  "url"     : "https://<host>",
  "dbName"  : "<dbName>",
  "login"   : "<login>",
  "password": "<password>"
}
```

| Поле       | Описание                                                                 |
|------------|--------------------------------------------------------------------------|
| `url`      | Базовый адрес хоста BaSYS (схема, хост и порт), например `https://localhost:44365`. Без завершающего слеша и без пути. |
| `dbName`   | Имя базы данных (тенанта), к которой подключается агент.                  |
| `login`    | Логин пользователя (email).                                              |
| `password` | Пароль пользователя.                                                     |

Чтобы случайно не закоммитить секреты, рядом удобно держать образец `.cursor/basys-credentials.example.json` с теми же ключами и плейсхолдерами вместо реальных значений, а сам `basys-credentials.json` добавить в `.gitignore`:

```gitignore
.cursor/basys-credentials.json
```

> Прокси отправляет логин и пароль на эндпоинт авторизации только при получении нового токена и далее работает по короткоживущему access-токену. Подробнее о схеме аутентификации см. [Авторизация](auth.md).

## Шаг 2. Прокси

Создайте файл `.cursor/mcp-proxy.js` со следующим содержимым. Скрипт не содержит секретов — все учётные данные он читает из `basys-credentials.json`.

```js
#!/usr/bin/env node
/**
 * BaSYS MCP stdio→HTTPS прокси.
 *
 * Читает JSON-RPC сообщения из stdin, пробрасывает их на HTTP MCP-эндпоинт
 * BaSYS с актуальным JWT-токеном и возвращает ответы в stdout.
 * Токен запрашивается автоматически при старте и обновляется за 1 минуту
 * до истечения или при получении 401.
 *
 * Конфигурация: .cursor/basys-credentials.json
 */
'use strict';

const https = require('https');
const http  = require('http');
const fs    = require('fs');
const path  = require('path');
const rl    = require('readline');

// ─── конфигурация ──────────────────────────────────────────────────

const CREDS_FILE        = path.join(__dirname, 'basys-credentials.json');
const REFRESH_BEFORE_MS = 60_000; // обновить токен за 60 с до истечения

// ─── состояние токена ──────────────────────────────────────────────

let currentToken   = null;
let tokenExpiresAt = 0;

// ─── логирование ───────────────────────────────────────────────────

function log(msg) {
  process.stderr.write(`[mcp-proxy] ${msg}\n`);
}

// ─── загрузка учётных данных ───────────────────────────────────────

function loadCredentials() {
  if (!fs.existsSync(CREDS_FILE)) {
    log(`Файл учётных данных не найден: ${CREDS_FILE}`);
    log('Создайте .cursor/basys-credentials.json по образцу .cursor/basys-credentials.example.json');
    process.exit(1);
  }
  try {
    return JSON.parse(fs.readFileSync(CREDS_FILE, 'utf8'));
  } catch (e) {
    log(`Не удалось прочитать файл учётных данных: ${e.message}`);
    process.exit(1);
  }
}

// ─── HTTP-утилита ──────────────────────────────────────────────────

function request(urlStr, options, body) {
  return new Promise((resolve, reject) => {
    const u    = new URL(urlStr);
    const lib  = u.protocol === 'https:' ? https : http;
    const port = u.port || (u.protocol === 'https:' ? 443 : 80);

    const reqOptions = {
      hostname          : u.hostname,
      port,
      path              : u.pathname + (u.search || ''),
      method            : options.method || 'POST',
      headers           : { ...(options.headers || {}) },
      rejectUnauthorized: false, // разрешить самоподписанный сертификат в dev
    };

    if (body) {
      reqOptions.headers['Content-Length'] = Buffer.byteLength(body);
    }

    const req = lib.request(reqOptions, (res) => resolve(res));
    req.on('error', reject);
    if (body) req.write(body);
    req.end();
  });
}

async function readBody(res) {
  return new Promise((resolve, reject) => {
    let data = '';
    res.on('data', chunk => (data += chunk));
    res.on('end',  () => resolve(data));
    res.on('error', reject);
  });
}

// ─── аутентификация ────────────────────────────────────────────────

async function fetchToken(creds) {
  log('Запрашиваем новый JWT-токен...');

  const body = JSON.stringify({ Login: creds.login, Password: creds.password });
  const res  = await request(
    `${creds.url}/api/public/v1/Auth/${creds.dbName}`,
    {
      method : 'POST',
      headers: { 'Content-Type': 'application/json' },
    },
    body,
  );

  const data = await readBody(res);

  if (res.statusCode !== 200) {
    throw new Error(`Ошибка авторизации (${res.statusCode}): ${data}`);
  }

  const parsed    = JSON.parse(data);
  // ASP.NET Core сериализует ответ в camelCase: token / expiresAtUtc
  currentToken    = parsed.token ?? parsed.Token;
  const expiresAt = parsed.expiresAtUtc ?? parsed.ExpiresAtUtc;
  tokenExpiresAt  = new Date(expiresAt).getTime();
  log(`Токен получен, истекает: ${expiresAt}`);
  return currentToken;
}

async function ensureToken(creds) {
  if (currentToken && Date.now() < tokenExpiresAt - REFRESH_BEFORE_MS) {
    return currentToken;
  }
  return fetchToken(creds);
}

// ─── разбор SSE ────────────────────────────────────────────────────

function parseSseMessages(res) {
  return new Promise((resolve, reject) => {
    const messages = [];
    let   buffer   = '';

    res.on('data', chunk => {
      buffer += chunk.toString();
      const lines = buffer.split('\n');
      buffer = lines.pop(); // оставляем неполную строку
      for (const line of lines) {
        if (line.startsWith('data: ')) {
          const payload = line.slice(6).trim();
          if (payload) messages.push(payload);
        }
      }
    });

    res.on('end', () => {
      if (buffer.startsWith('data: ')) {
        const payload = buffer.slice(6).trim();
        if (payload) messages.push(payload);
      }
      resolve(messages);
    });

    res.on('error', reject);
  });
}

// ─── проброс MCP-запроса ───────────────────────────────────────────

async function forwardToMcp(messageBody, creds) {
  const jwt = await ensureToken(creds);

  const res = await request(
    `${creds.url}/mcp`,
    {
      method : 'POST',
      headers: {
        'Content-Type' : 'application/json',
        'Accept'       : 'application/json, text/event-stream',
        'Authorization': `Bearer ${jwt}`,
      },
    },
    messageBody,
  );

  if (res.statusCode === 401) {
    currentToken = null; // принудительное обновление при следующем вызове
    throw new Error('401 Unauthorized');
  }

  const contentType = res.headers['content-type'] || '';

  if (contentType.includes('text/event-stream')) {
    return parseSseMessages(res);
  }

  const data = await readBody(res);
  return data.trim() ? [data.trim()] : [];
}

// ─── отправка JSON-RPC ошибки в stdout ─────────────────────────────

function writeError(rawMessage, errorMessage) {
  try {
    const req = JSON.parse(rawMessage);
    const out = JSON.stringify({
      jsonrpc: '2.0',
      id     : req.id ?? null,
      error  : { code: -32603, message: errorMessage },
    });
    process.stdout.write(out + '\n');
  } catch {
    // не удалось распарсить входящее сообщение — ничего не пишем
  }
}

// ─── main ──────────────────────────────────────────────────────────

async function main() {
  const creds = loadCredentials();

  // получаем токен заранее, чтобы не задерживать первый запрос
  await fetchToken(creds);

  const reader = rl.createInterface({ input: process.stdin, terminal: false });

  reader.on('line', async (line) => {
    const trimmed = line.trim();
    if (!trimmed) return;

    try {
      const messages = await forwardToMcp(trimmed, creds);
      for (const msg of messages) {
        process.stdout.write(msg + '\n');
      }
    } catch (err) {
      // при 401 — одна попытка с обновлённым токеном
      if (err.message.includes('401')) {
        try {
          await fetchToken(creds);
          const messages = await forwardToMcp(trimmed, creds);
          for (const msg of messages) {
            process.stdout.write(msg + '\n');
          }
        } catch (retryErr) {
          log(`Повторная попытка после 401 не удалась: ${retryErr.message}`);
          writeError(trimmed, retryErr.message);
        }
        return;
      }
      log(`Ошибка: ${err.message}`);
      writeError(trimmed, err.message);
    }
  });

  reader.on('close', () => process.exit(0));
}

main().catch(err => {
  log(`Fatal: ${err.message}`);
  process.exit(1);
});
```

> Параметр `rejectUnauthorized: false` отключает проверку TLS-сертификата и нужен только для работы с локальным dev-хостом, использующим самоподписанный сертификат. На стенде с доверенным сертификатом этот параметр следует убрать.

## Шаг 3. Регистрация сервера в Cursor

Создайте файл `.cursor/mcp.json`, который указывает Cursor запускать прокси как stdio-сервер:

```json
{
  "mcpServers": {
    "basys-mcp": {
      "command": "node",
      "args": [".cursor/mcp-proxy.js"]
    }
  }
}
```

| Поле      | Описание                                                                 |
|-----------|--------------------------------------------------------------------------|
| `basys-mcp` | Имя сервера, под которым он отображается в настройках Cursor (можно изменить). |
| `command` | Исполняемый файл для запуска сервера — `node`.                            |
| `args`    | Аргументы запуска: путь к скрипту прокси относительно корня проекта.      |

## Шаг 4. Проверка подключения

1. Перезапустите Cursor (или откройте проект заново), чтобы он перечитал `.cursor/mcp.json`.
2. Откройте **Settings → MCP** (или **Cursor Settings → Tools & MCP**). Сервер `basys-mcp` должен отобразиться со статусом, что он запущен, и списком доступных инструментов.
3. Среди инструментов должны появиться `validate_metadata` и `import_metadata`, а среди ресурсов — `data_types`, `meta_object_kinds` и `meta_object_kind_by_uid` (см. описание в статье [MCP-сервер](mcp.md)).
4. Попросите агента, например, «выведи список видов метаобъектов» — он обратится к ресурсу MCP и вернёт данные из вашей базы.

Если сервер не запустился, диагностические сообщения прокси (с префиксом `[mcp-proxy]`) выводятся в `stderr` и видны в журнале MCP-сервера в Cursor.

## Как работает прокси

- **Запуск.** При старте прокси читает `basys-credentials.json` и сразу запрашивает access-токен, чтобы не задерживать первый запрос агента.
- **Проброс вызовов.** Каждое JSON-RPC сообщение от Cursor (одна строка из stdin) транслируется `POST`-запросом на `/<host>/mcp` с заголовком `Authorization: Bearer <access-токен>` и `Accept: application/json, text/event-stream`. Ответ (обычный JSON или поток SSE) разбирается и возвращается в stdout.
- **Обновление токена.** Перед каждым вызовом прокси проверяет срок жизни токена и обновляет его за 60 секунд до истечения. Если сервер всё же вернул `401`, прокси сбрасывает токен, получает новый и **один раз** повторяет запрос.

Прокси использует упрощённую модель: он получает новый токен по логину и паролю и не задействует refresh-токен. Это допустимо для локального инструмента разработчика; в более строгих сценариях логику можно расширить обменом refresh-токена (см. [Авторизация](auth.md)).

## Безопасность

- **Не коммитьте `basys-credentials.json`.** Файл содержит логин и пароль — держите его вне системы контроля версий (через `.gitignore`), а в репозитории оставляйте только образец `basys-credentials.example.json` с плейсхолдерами.
- **Используйте HTTPS.** На рабочих стендах подключайтесь по HTTPS к доверенному сертификату и убирайте из прокси `rejectUnauthorized: false`.
- **Ограничивайте права пользователя.** Для подключения используйте учётную запись с минимально необходимыми правами на работу с метаданными базы.

## Возможные проблемы

| Симптом                                              | Причина и решение                                                                 |
|------------------------------------------------------|-----------------------------------------------------------------------------------|
| Сервер не появился в списке MCP в Cursor             | Проверьте, что установлен Node.js (`node --version`) и путь в `args` указан верно. |
| `Файл учётных данных не найден`                      | Не создан `.cursor/basys-credentials.json`. Скопируйте образец и заполните значения. |
| `Ошибка авторизации (401)`                           | Неверные `login`/`password` или `dbName`. Проверьте учётные данные (см. [Авторизация](auth.md)). |
| Ошибки TLS / сертификата                             | Самоподписанный сертификат dev-хоста. Для локальной разработки в прокси оставлен `rejectUnauthorized: false`. |
| Агент не видит инструменты или ресурсы               | Перезапустите Cursor, чтобы он перечитал `.cursor/mcp.json`, и проверьте журнал MCP. |

## Связанные разделы

- [MCP-сервер](mcp.md) — описание ресурсов и инструментов MCP-сервера BaSYS.
- [Авторизация](auth.md) — получение и обновление JWT-токенов для доступа к MCP и публичному API.

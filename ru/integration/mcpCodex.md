# Подключение MCP-сервера BaSYS к Codex

Эта статья демонстрирует, как подключить встроенный [MCP-сервер BaSYS](mcp.md) к [OpenAI Codex](https://developers.openai.com/codex), чтобы агент мог читать метаданные базы и импортировать их обратно прямо из проекта.

Codex поддерживает MCP-серверы в CLI и IDE-расширении. Настройки MCP хранятся в `config.toml`: глобально в `~/.codex/config.toml` или локально в проекте в `.codex/config.toml` для доверенных репозиториев. Для BaSYS удобнее использовать проектную конфигурацию, потому что MCP-сервер, правила, навыки и подсказки агента относятся к конкретному репозиторию с метаданными.

MCP-сервер BaSYS защищён JWT-аутентификацией (см. [Авторизация](auth.md)): каждый запрос требует заголовка `Authorization: Bearer <access-токен>`, а сам токен короткоживущий (по умолчанию 1 час) и периодически обновляется. Статичный заголовок в `config.toml` для этого не подходит — он «протухнет» вместе с токеном.

Чтобы решить эту задачу, между Codex и BaSYS ставится небольшой локальный **прокси** (Node.js-скрипт). Codex запускает его как stdio MCP-сервер; при старте прокси получает access-токен, обновляет его перед истечением и при ответе `401`, а сами MCP-вызовы транслирует на HTTP-эндпоинт `/mcp` BaSYS.

```
Codex  ──stdio (JSON-RPC)──►  basys-mcp-proxy.js  ──HTTPS + Bearer──►  BaSYS /mcp
```

Готовый прокси и шаблоны настроек уже опубликованы в репозитории [BaSYS.AI.KIT](https://github.com/BaSysTeam/BaSYS.AI.KIT) — писать скрипт вручную не нужно. Достаточно скопировать `dist/codex-template` в свой проект, настроить доступ к BaSYS и запустить Codex из корня репозитория.

## Готовый набор файлов

Все нужные файлы лежат в каталоге [`dist/codex-template`](https://github.com/BaSysTeam/BaSYS.AI.KIT/tree/main/dist/codex-template) репозитория BaSYS.AI.KIT и размещаются в корне вашего проекта:

| Файл | Назначение |
|------|------------|
| [`.codex/config.toml`](https://github.com/BaSysTeam/BaSYS.AI.KIT/blob/main/dist/codex-template/.codex/config.toml) | Проектная конфигурация Codex: регистрация MCP-сервера `basys-mcp` и параметры запуска прокси. |
| [`.codex/mcp/basys-mcp-proxy.js`](https://github.com/BaSysTeam/BaSYS.AI.KIT/blob/main/dist/codex-template/.codex/mcp/basys-mcp-proxy.js) | Прокси stdio→HTTPS: авторизация, обновление токена и проброс MCP-запросов. |
| [`.codex/mcp/basys-credentials.example.json`](https://github.com/BaSysTeam/BaSYS.AI.KIT/blob/main/dist/codex-template/.codex/mcp/basys-credentials.example.json) | Образец локального файла учётных данных (без реальных секретов). |
| `.codex/mcp/basys-credentials.json` | Ваши учётные данные подключения к BaSYS, если не используются переменные окружения. **Не коммитится** в репозиторий. |
| [`gitignore.codex-snippet`](https://github.com/BaSysTeam/BaSYS.AI.KIT/blob/main/dist/codex-template/gitignore.codex-snippet) | Фрагмент для `.gitignore`, исключающий локальный файл с секретами. |
| [`AGENTS.md`](https://github.com/BaSysTeam/BaSYS.AI.KIT/blob/main/dist/codex-template/AGENTS.md) | Постоянные инструкции Codex для репозитория метаданных BaSYS. |
| `.agents/skills` | Навыки Codex уровня репозитория для типовых сценариев создания метаданных. |
| `.agents/references/basys` | Справочные правила BaSYS, которые навыки и агент читают перед изменениями. |
| `.codex/rules/default.rules` | Шаблон правил разрешений для команд Codex. Не используется для доменных правил BaSYS. |

Минимально необходимый набор для подключения MCP: `.codex/config.toml`, `.codex/mcp/basys-mcp-proxy.js`, `.codex/mcp/basys-credentials.example.json` (если не используете только переменные окружения) и запись из `gitignore.codex-snippet` в `.gitignore`. Остальные файлы не обязательны для самого MCP, но рекомендуются для полноценной работы Codex с метаданными BaSYS.

> Для работы прокси требуется установленный [Node.js](https://nodejs.org) (LTS-версия). Проверить наличие можно командой `node --version`.

## Шаг 1. Скопируйте шаблон в проект

Получите содержимое каталога `dist/codex-template` из репозитория любым удобным способом и положите его в корень проекта с метаданными. Например, склонируйте репозиторий и скопируйте файлы:

```bash
git clone https://github.com/BaSysTeam/BaSYS.AI.KIT.git
cp -r BaSYS.AI.KIT/dist/codex-template/AGENTS.md ./AGENTS.md
cp -r BaSYS.AI.KIT/dist/codex-template/.agents ./.agents
cp -r BaSYS.AI.KIT/dist/codex-template/.codex ./.codex
cat BaSYS.AI.KIT/dist/codex-template/gitignore.codex-snippet >> .gitignore
```

В PowerShell то же самое можно выполнить так:

```powershell
Copy-Item -Recurse -Force .\BaSYS.AI.KIT\dist\codex-template\AGENTS.md .\
Copy-Item -Recurse -Force .\BaSYS.AI.KIT\dist\codex-template\.agents .\
Copy-Item -Recurse -Force .\BaSYS.AI.KIT\dist\codex-template\.codex .\
Get-Content .\BaSYS.AI.KIT\dist\codex-template\gitignore.codex-snippet | Add-Content .\.gitignore
```

Если в проекте уже есть `AGENTS.md`, `.agents` или `.codex`, не перезаписывайте их вслепую: перенесите только недостающие части и сохраните существующие инструкции проекта.

Фрагмент `gitignore.codex-snippet` добавляет в `.gitignore` строку:

```gitignore
.codex/mcp/basys-credentials.json
```

## Шаг 2. Заполните учётные данные

Прокси умеет получать учётные данные двумя способами — из переменных окружения или из локального файла. Для Codex-шаблона предпочтительны переменные окружения: в `.codex/config.toml` они перечислены в `env_vars`, поэтому Codex передаёт их процессу MCP-сервера при запуске.

### Вариант A. Переменные окружения

Задайте переменные перед запуском Codex:

| Переменная | Соответствует полю |
|------------|--------------------|
| `BASYS_URL` | `url` |
| `BASYS_DB_NAME` | `dbName` |
| `BASYS_LOGIN` | `login` |
| `BASYS_PASSWORD` | `password` |

Пример для bash:

```bash
export BASYS_URL="https://<host>:<port>"
export BASYS_DB_NAME="<dbName>"
export BASYS_LOGIN="<login>"
export BASYS_PASSWORD="<password>"
codex
```

Пример для PowerShell:

```powershell
$env:BASYS_URL = "https://<host>:<port>"
$env:BASYS_DB_NAME = "<dbName>"
$env:BASYS_LOGIN = "<login>"
$env:BASYS_PASSWORD = "<password>"
codex
```

### Вариант Б. Локальный файл учётных данных

Если полный набор переменных окружения не задан, прокси читает файл `.codex/mcp/basys-credentials.json`. Скопируйте образец `.codex/mcp/basys-credentials.example.json` и подставьте значения своего окружения вместо плейсхолдеров:

```json
{
  "url": "https://<host>:<port>",
  "dbName": "<dbName>",
  "login": "<login>",
  "password": "<password>"
}
```

| Поле | Описание |
|------|----------|
| `url` | Базовый адрес хоста BaSYS (схема и хост, при необходимости порт). Порт указывается, только если хост слушает нестандартный порт, например локальный dev-хост `https://localhost:44365`; для публичных адресов он обычно не нужен — `https://test.thebasys.com`. Без завершающего слеша и без пути. |
| `dbName` | Имя базы данных (тенанта), к которой подключается агент. |
| `login` | Логин пользователя (email). |
| `password` | Пароль пользователя. |

Файл `.codex/mcp/basys-credentials.json` содержит секреты и **не должен попадать** в систему контроля версий. В репозиторий добавляйте только образец `basys-credentials.example.json` с плейсхолдерами.

> Прокси отправляет логин и пароль на эндпоинт авторизации только при получении нового токена и далее работает по короткоживущему access-токену. Подробнее о схеме аутентификации см. [Авторизация](auth.md).

## Шаг 3. Как зарегистрирован сервер в Codex

Регистрацию выполнять вручную не нужно — файл `.codex/config.toml` из шаблона уже указывает Codex запускать прокси как stdio MCP-сервер:

```toml
[mcp_servers.basys-mcp]
command = "node"
args = [".codex/mcp/basys-mcp-proxy.js"]
env_vars = [
  "BASYS_URL",
  "BASYS_DB_NAME",
  "BASYS_LOGIN",
  "BASYS_PASSWORD",
]
startup_timeout_sec = 20
tool_timeout_sec = 60
enabled = true
```

| Поле | Описание |
|------|----------|
| `basys-mcp` | Имя сервера, под которым он отображается в Codex (можно изменить). |
| `command` | Исполняемый файл для запуска сервера — `node`. |
| `args` | Аргументы запуска: путь к скрипту прокси относительно корня проекта. |
| `env_vars` | Список переменных окружения, которые Codex передаёт MCP-процессу. |
| `startup_timeout_sec` | Таймаут запуска MCP-сервера в секундах. |
| `tool_timeout_sec` | Таймаут выполнения инструментов MCP в секундах. |
| `enabled` | Признак активности сервера. Чтобы временно отключить его, задайте `false`. |

Проектный `.codex/config.toml` применяется, когда Codex запущен из корня доверенного проекта. CLI и IDE-расширение Codex используют одну и ту же конфигурацию, поэтому после настройки сервер доступен в обоих клиентах.

## Шаг 4. Проверка подключения

1. Запустите Codex из корня проекта или откройте этот проект в IDE-расширении Codex.
2. Откройте `/mcp` в интерфейсе Codex и убедитесь, что сервер `basys-mcp` включён.
3. Среди инструментов должны появиться `validate_metadata` и `import_metadata`, а среди ресурсов — `data_types`, `meta_object_kinds` и `meta_object_kind_by_uid` (см. описание в статье [MCP-сервер](mcp.md)).
4. Попросите агента, например, «выведи список видов метаобъектов» — он обратится к ресурсу MCP и вернёт данные из вашей базы.

Если сервер не запустился, диагностические сообщения прокси (с префиксом `[basys-mcp-proxy]`) выводятся в `stderr` и видны в журнале MCP-сервера Codex.

## Дополнительные файлы шаблона

Помимо подключения MCP, `codex-template` содержит файлы, которые помогают Codex работать с репозиторием метаданных как с metadata-as-code проектом:

- **`AGENTS.md`** — постоянные инструкции: что нельзя менять, где искать схемы и UIDs, какие соглашения BaSYS соблюдать.
- **`.agents/skills`** — навыки для повторяемых сценариев, например создания справочника, регистра, операции, форм и записей.
- **`.agents/references/basys`** — справочные документы по областям BaSYS: формы, команды, workflow, меню, отчёты и общие соглашения.
- **`.codex/rules`** — правила разрешений для выполнения команд Codex. Доменные инструкции BaSYS туда не помещаются.

Эти файлы можно скопировать вместе с MCP-настройкой. Они не заменяют MCP-сервер: MCP даёт агенту доступ к живым данным и инструментам BaSYS, а `AGENTS.md`, skills и references задают локальные правила работы с проектом.

## Как работает прокси

Исходный код прокси находится в файле [`basys-mcp-proxy.js`](https://github.com/BaSysTeam/BaSYS.AI.KIT/blob/main/dist/codex-template/.codex/mcp/basys-mcp-proxy.js). Скрипт не содержит секретов — все учётные данные он читает из переменных окружения либо из `.codex/mcp/basys-credentials.json`. Логика работы:

- **Запуск.** При старте прокси читает учётные данные и сразу запрашивает access-токен, чтобы не задерживать первый запрос агента.
- **Проброс вызовов.** Каждое JSON-RPC сообщение от Codex (одна строка из stdin) транслируется `POST`-запросом на `<url>/mcp` с заголовком `Authorization: Bearer <access-токен>` и `Accept: application/json, text/event-stream`. Ответ (обычный JSON или поток SSE) разбирается и возвращается в stdout.
- **Обновление токена.** Перед каждым вызовом прокси проверяет срок жизни токена и получает новый за 60 секунд до истечения. Если сервер всё же вернул `401`, прокси сбрасывает токен, получает новый и **один раз** повторяет запрос.

Прокси использует упрощённую модель: он получает новый токен по логину и паролю и не задействует refresh-токен. Это допустимо для локального инструмента разработчика; в более строгих сценариях логику можно расширить обменом refresh-токена (см. [Авторизация](auth.md)).

> В прокси параметр `rejectUnauthorized: false` отключает проверку TLS-сертификата и нужен только для работы с локальным dev-хостом, использующим самоподписанный сертификат. На стенде с доверенным сертификатом этот параметр следует убрать.

## Безопасность

- **Не коммитьте `.codex/mcp/basys-credentials.json`.** Файл содержит логин и пароль — держите его вне системы контроля версий, а в репозитории оставляйте только образец `basys-credentials.example.json` с плейсхолдерами.
- **Не храните секреты в `.codex/config.toml`.** Этот файл рассчитан на общий проектный шаблон; используйте `env_vars` или локальный файл, исключённый из Git.
- **Используйте HTTPS.** На рабочих стендах подключайтесь по HTTPS к доверенному сертификату и убирайте из прокси `rejectUnauthorized: false`.
- **Ограничивайте права пользователя.** Для подключения используйте учётную запись с минимально необходимыми правами на работу с метаданными базы.

## Возможные проблемы

| Симптом | Причина и решение |
|---------|-------------------|
| Сервер `basys-mcp` не появился в `/mcp` | Проверьте, что Codex запущен из корня проекта, проект доверенный, установлен Node.js (`node --version`), а путь в `args` указан верно. |
| `Credentials file not found` | Не заданы все переменные `BASYS_*` и не создан `.codex/mcp/basys-credentials.json`. Задайте переменные окружения либо скопируйте образец и заполните значения. |
| `Authentication failed (401)` | Неверные `login`/`password` или `dbName`. Проверьте учётные данные (см. [Авторизация](auth.md)). |
| Ошибки TLS / сертификата | Самоподписанный сертификат dev-хоста. Для локальной разработки в прокси оставлен `rejectUnauthorized: false`; для рабочих стендов используйте доверенный сертификат. |
| Агент не видит инструменты или ресурсы | Перезапустите Codex, чтобы он заново поднял MCP-процесс, и проверьте журнал MCP. |
| Изменили переменные окружения, но сервер использует старые значения | Завершите текущую сессию Codex и запустите её заново из окружения с новыми `BASYS_*`. |
| Инструмент завершается по таймауту | Увеличьте `tool_timeout_sec` в `.codex/config.toml`, если операция импорта или чтения метаданных в вашем окружении занимает больше 60 секунд. |

## Связанные разделы

- [BaSYS.AI.KIT](https://github.com/BaSysTeam/BaSYS.AI.KIT) — репозиторий с готовыми файлами прокси, настроек, инструкций и навыков для ИИ-агента BaSYS.
- [Подключение MCP-сервера к Cursor](mcpCursor.md) — аналогичная настройка прокси для редактора Cursor.
- [Подключение MCP-сервера к OpenCode](mcpOpenCode.md) — аналогичная настройка прокси для терминального агента OpenCode.
- [MCP-сервер](mcp.md) — описание ресурсов и инструментов MCP-сервера BaSYS.
- [Авторизация](auth.md) — получение и обновление JWT-токенов для доступа к MCP и публичному API.

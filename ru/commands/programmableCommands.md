# Программируемые команды

Программируемая команда — это блок кода на JavaScript, который выполняется по нажатию пользователем кнопки или пункта меню. Команды используются в автоматических формах и в формах, созданных конструктором.

В автоматических формах команды размещаются в подменю «Действия» шапки или соответствующей табличной части (в зависимости от того, к какому объекту команду отнесли при создании). В формах конструктора команда привязывается к кнопке или пункту меню по имени.

Особыми видами программируемых команд являются команды **подбора** и **заполнения** табличной части. Они упрощают программирование типовых действий: разработчику достаточно задать выражение — источник данных, а остальную логику система формирует автоматически. При необходимости подбор и заполнение могут быть запрограммированы полностью — как обычные команды.

## Контекст выполнения команды

Команда выполняется на клиенте (в браузере). В контексте выполнения доступен текущий объект данных (DataObject) через специальные обозначения:

| Обозначение | Описание                                                                  |
| :---------- | :------------------------------------------------------------------------ |
| `$h`        | Шапка объекта. К полям шапки можно обращаться через `$h.<имя_колонки>`.   |
| `$t`        | Табличные части объекта. Доступ — `$t.<имя_таблицы>`.                     |
| `$r`        | Текущая строка. Используется в командах, привязанных к табличной части.   |

В команде доступны все функции [библиотеки BaSYS.Fx](../calculations/index.md), включая [построитель запросов](../calculations/queryBuilder.md) `from(...)` для обращения к базе данных, функции для работы с [таблицами](../calculations/dataTable.md), [условные функции](../calculations/index.md), а также набор специальных функций, описанных ниже.

## Функции, доступные в командах

| Функция                           | Возвращает | Описание                                                       |
| :-------------------------------- | :--------- | :------------------------------------------------------------- |
| [close](#close)                   | void       | Закрытие формы и возврат к списку.                             |
| [getIsModified](#getismodified)   | boolean    | Получение значения флага `isModified`.                         |
| [getIsWaiting](#getiswaiting)     | boolean    | Получение значения флага `isWaiting`.                          |
| [openDialog](#opendialog)         | void       | Открытие модальной формы.                                      |
| [openPickUp](#openpickup)         | void       | Открытие диалога подбора по источнику данных.                  |
| [recalculate](#recalculate)       | Promise    | Полный пересчёт формул объекта.                                |
| [refresh](#refresh)               | void       | Обновление табличной части (TableView) по имени.               |
| [runWorkflow](#runworkflow)       | Promise    | Запуск процесса (workflow) на сервере.                         |
| [save](#save)                     | Promise    | Сохранение текущего объекта данных.                            |
| [setIsModified](#setismodified)   | void       | Установка флага `isModified`.                                  |
| [setIsWaiting](#setiswaiting)     | void       | Установка флага `isWaiting` (отображение индикатора ожидания). |

> **Примечание.** Функции `openPickUp` и `openDialog` доступны не во всех контекстах выполнения. `openPickUp` доступна для команд, привязанных к табличной части. `openDialog` доступна в шапке автоматической формы и в формах конструктора.

## close

Закрывает форму редактирования объекта и возвращает пользователя к списку.

### Синтаксис
```javascript
close()
```

### Параметры
нет

### Возвращаемое значение
void

### Пример
```javascript
save();
close();
```

## getIsModified

Возвращает текущее значение флага `isModified` объекта данных. Флаг устанавливается в `true`, когда пользователь внёс изменения, ещё не сохранённые в базе данных.

### Синтаксис
```javascript
getIsModified()
```

### Параметры
нет

### Возвращаемое значение
boolean

### Пример
```javascript
if (getIsModified()) {
  await save();
}
```

## getIsWaiting

Возвращает текущее значение флага `isWaiting`. Флаг указывает, выполняется ли в текущий момент длительная операция и отображается ли пользователю индикатор ожидания.

### Синтаксис
```javascript
getIsWaiting()
```

### Параметры
нет

### Возвращаемое значение
boolean

## openDialog

Открывает модальную форму, описанную в метаданных. Параметры передаются в форму через свойство `parameters`. Поддерживаются обратные вызовы `onClose` и `onError`, выполняющиеся при закрытии диалога или возникновении ошибки.

### Синтаксис
```javascript
openDialog(config)
```

### Параметры
- *config*: object — конфигурация диалога. Поддерживаемые поля:
  - `kind`: string — имя вида метаданных (например, `'operation'`, `'catalog'`);
  - `name`: string — имя метаобъекта;
  - `formName`: string — имя формы метаобъекта, которая будет открыта в диалоге;
  - `title`: string — заголовок диалога;
  - `width`: string — ширина диалога (CSS-значение, например `'40rem'`);
  - `parameters`: object — произвольный объект параметров, передаваемых в открываемую форму;
  - *onClose (необязательный)*: `(result?: any) => void` — функция, вызываемая при закрытии диалога. В аргумент передаётся результат, переданный закрывшейся формой;
  - *onError (необязательный)*: `(error?: any) => void` — функция, вызываемая при возникновении ошибки в диалоге.

### Возвращаемое значение
void

### Пример
Открытие формы создания новой задачи и обновление списка задач после её закрытия.

```javascript
openDialog({
  kind: 'operation',
  name: 'task',
  formName: 'create_task',
  title: 'Новая задача',
  width: '40rem',
  parameters: {
    author: $h.текущее_физлицо,
    contract_draft: $h.number,
    regime: 'new'
  },
  onClose: () => {
    refresh('list_tasks');
  }
});
```

## openPickUp

Открывает диалог подбора. В диалоге отображается переданный источник данных, и пользователь может выбрать одну или несколько строк, которые будут добавлены в указанную табличную часть текущего объекта.

### Синтаксис
```javascript
openPickUp(source, tableName, settings)
```

### Параметры
- *source*: [DataTable](../calculations/dataTable.md) — таблица данных, из которой пользователь выполняет подбор.
- *tableName*: string — имя табличной части текущего объекта (получателя данных).
- *settings (необязательный)*: object — настройки диалога подбора (заголовок, размеры, список колонок и т. п.).

### Возвращаемое значение
void

### Пример
```javascript
var source = await from('catalog.сотрудники')
  .select(['id as сотрудники', 'title as сотрудники_display'])
  .orderBy('сотрудники_display')
  .query();

openPickUp(source, 'table_2');
```

## recalculate

Запускает полный пересчёт формул текущего объекта данных. Применяется после программного изменения значений полей, чтобы пересчитать зависимые поля.

### Синтаксис
```javascript
await recalculate()
```

### Параметры
нет

### Возвращаемое значение
Promise

### Пример
```javascript
$h.статус = 'у_директора';
await recalculate();
await save();
```

## refresh

Обновляет данные элемента TableView по имени. Применяется, например, после серверной операции, изменившей записи, отображаемые в таблице.

### Синтаксис
```javascript
refresh(tableViewName)
```

### Параметры
- *tableViewName*: string — имя элемента TableView, размещённого на форме.

### Возвращаемое значение
void

### Пример
```javascript
await runWorkflow('contract_visa_manual', 'create_approval_records', [
  { name: 'number', dataType: 'integer', value: $h.number }
]);
refresh('list');
```

## runWorkflow

Запускает процесс (workflow) на сервере и возвращает результат указанного шага. Используется для выполнения серверной логики, обращений к внешним системам, импорта данных, формирования табличных результатов и т. п.

### Синтаксис
```javascript
await runWorkflow(name, resultStepName, parameters, timeout)
```

### Параметры
- *name*: string — имя процесса.
- *resultStepName*: string — имя шага процесса, результат которого должен быть возвращён.
- *parameters (необязательный)*: object[] — массив параметров процесса. Каждый параметр — объект с полями:
  - `name`: string — имя параметра;
  - `dataType`: string — тип данных параметра (`'integer'`, `'string'`, `'boolean'`, `'date'` и т. д.);
  - `value`: any — значение параметра.
- *timeout (необязательный)*: number — таймаут ожидания результата в секундах. Значение по умолчанию: 15.

### Возвращаемое значение
Promise — при успешном завершении возвращает данные шага. Если шаг возвращает табличный результат, функция возвращает объект [DataTable](../calculations/dataTable.md), пригодный для последующей обработки или загрузки в табличную часть.

### Пример
Запуск процесса формирования записей согласования с тремя параметрами и обновление списка после выполнения.

```javascript
setIsWaiting(true);

await runWorkflow('contract_visa_manual', 'create_approval_records', [
  { name: 'number', dataType: 'integer', value: $h.number },
  { name: 'person', dataType: 'integer', value: $h.visa_manual },
  { name: 'order',  dataType: 'integer', value: $h.visa_manual_order }
]);

refresh('list');
setIsWaiting(false);
```

## save

Сохраняет текущий объект данных в базе. После успешного сохранения флаг `isModified` сбрасывается.

### Синтаксис
```javascript
await save()
```

### Параметры
нет

### Возвращаемое значение
Promise

### Пример
```javascript
$h.статус = 'согласован';
await recalculate();
await save();
```

## setIsModified

Устанавливает флаг `isModified` объекта данных. Используется для принудительной отметки объекта как изменённого (например, после программного заполнения табличной части), либо для сброса флага после сохранения.

### Синтаксис
```javascript
setIsModified(value)
```

### Параметры
- *value*: boolean — новое значение флага.

### Возвращаемое значение
void

### Пример
```javascript
$t.table_1.clear();
$t.table_1.load(source);
setIsModified(true);
```

## setIsWaiting

Устанавливает флаг `isWaiting`. При значении `true` отображается индикатор ожидания и блокируется ввод. Применяется на время выполнения длительных операций — обращений к серверу, запуска процессов, тяжёлых расчётов.

### Синтаксис
```javascript
setIsWaiting(value)
```

### Параметры
- *value*: boolean — новое значение флага.

### Возвращаемое значение
void

### Пример
```javascript
setIsWaiting(true);
var data = await from('catalog.номенклатура').select(['id', 'title']).query();
setIsWaiting(false);
```

## Примеры

### Команда подбора. Упрощённый вариант

Команда подбора в упрощённом варианте создаётся как команда вида **«Подбор»** (`Kind = 2`). Разработчику достаточно задать выражение — источник данных в параметре `data_source`. Система автоматически вызовет диалог подбора и при выборе пользователем строк добавит их в указанную табличную часть.

В метаданных это выглядит так:

```json
{
  "Kind": 2,
  "Title": "Список сотрудников ( по проекту )",
  "Name": "pick_up_project",
  "Parameters": [
    {
      "Name": "data_source",
      "Value": "(await from('operation.штатное_расписание')\n  .select(['сотрудники as id', 'сотрудники as сотрудник', 'полное_имя', 'должность', 'подразделение', 'ставка_час as ставка'])\n  .getDisplays()\n  .orderBy('полное_имя')\n  .where('проект_заказ = @project AND уволен = @flagFalse')\n  .parameter('project', $h.проект_заказ, 11)\n  .parameter('flagFalse', false, 3)\n  .query())\n  .deleteColumn('id_display')\n  .addColumn({name: 'кту', dataType: 'number'})\n  .process(row => row.кту = 1)"
    }
  ]
}
```

В читаемом виде содержимое параметра `data_source`:

```javascript
(await from('operation.штатное_расписание')
  .select([
    'сотрудники as id',
    'сотрудники as сотрудник',
    'полное_имя',
    'должность',
    'подразделение',
    'ставка_час as ставка'
  ])
  .getDisplays()
  .orderBy('полное_имя')
  .where('проект_заказ = @project AND уволен = @flagFalse')
  .parameter('project', $h.проект_заказ, 11)
  .parameter('flagFalse', false, 3)
  .query())
  .deleteColumn('id_display')
  .addColumn({ name: 'кту', dataType: 'number' })
  .process(row => row.кту = 1)
```

### Команда подбора. Полный вариант

В полном варианте команда создаётся как обычная программируемая команда (`Kind = 0`). Источник данных формируется кодом команды, после чего вручную вызывается функция `openPickUp`. Такой вариант применяется, если перед открытием диалога подбора требуется сложная подготовка данных: соединение нескольких таблиц, фильтрация, обогащение колонок и т. п.

```javascript
var tableSaldo = await from('records.монтаж')
  .select([
    'марки',
    'Sum(колво) as колво_осталось_шт',
    'Sum(колво_тн) as колво_осталось_тн'
  ])
  .groupBy(['марки'])
  .where('шифр_км = @km')
  .parameter('km', $h.шифр_км, 11)
  .query();

var markList = tableSaldo.toArray('марки');

var operationDate = $h.date;

var result = await from('catalog.марки')
  .select([
    'id as марки',
    'title as марки_display',
    'наименование_марки',
    'шифр_км',
    'проект_заказ',
    'вес_всех_кг',
    'колво as колво_чертежи_шт',
    'вес_одной_кг'
  ])
  .where('cat_марки.проект_заказ = @project AND cat_марки.шифр_км = @km AND cat_марки.id in @markList')
  .parameter('project', $h.проект_заказ, 11)
  .parameter('km', $h.шифр_км, 11)
  .parameter('markList', markList)
  .getDisplays()
  .query();

result = result
  .addColumn('колво_чертежи_тн', 'number')
  .process(r => { r.колво_чертежи_тн = r.вес_всех_кг / 1000; })
  .addColumn('дата', 'datetime')
  .process(r => { r.дата = operationDate; })
  .leftJoin(tableSaldo, (pr, jn) => pr.марки === jn.марки);

openPickUp(result, 'table_1');
```

### Команда заполнения. Упрощённый вариант

Команда заполнения в упрощённом варианте создаётся как команда вида **«Заполнение»** (`Kind = 1`). Разработчику достаточно задать выражение — источник данных в параметре `data_source` и флаг `clear`, указывающий, очищать ли табличную часть перед загрузкой. Система автоматически выполнит запрос, очистит при необходимости таблицу и загрузит результат.

```json
{
  "Kind": 1,
  "Title": "Сотрудники по группе",
  "Name": "сотрудники",
  "Parameters": [
    {
      "Name": "data_source",
      "Value": "await from('catalog.сотрудники')\n  .where('группа = 1')\n  .select(['id as сотрудники', 'title as сотрудники_display'])\n  .orderBy('title')\n  .query()"
    },
    { "Name": "clear", "Value": "true" }
  ]
}
```

Под выполнение системой генерируется примерно следующий код:

```javascript
var source = await from('catalog.сотрудники')
  .where('группа = 1')
  .select(['id as сотрудники', 'title as сотрудники_display'])
  .orderBy('title')
  .query();

$t.table_2.clear().load(source);
```

### Команда заполнения. Полный вариант

В полном варианте табличная часть заполняется обычной программируемой командой (`Kind = 0`). Подходит, когда требуется управлять состоянием формы (`setIsWaiting`, `setIsModified`), вызывать процессы, выполнять предварительные расчёты и т. п.

```javascript
setIsWaiting(true);

var source = await runWorkflow('в_заявку_мтс_exls', 'result', [
  { name: 'number',  dataType: 'integer', value: $h.number },
  { name: 'project', dataType: 'integer', value: $h.площадка },
  { name: 'code_rd', dataType: 'integer', value: $h.шифр_рд }
], 300);

$t.table_1.clear();
$t.table_1.load(source);

setIsModified(true);
setIsWaiting(false);
```

### Вызов процесса

В данном примере по нажатию кнопки запускается процесс согласования договора. Процесс принимает четыре параметра, после его завершения список согласований обновляется, а результат анализируется в коде команды для определения итогового статуса. При полном согласовании статус документа изменяется, объект сохраняется и запускается процесс отправки уведомления.

```javascript
setIsWaiting(true);

await runWorkflow('contract_set_approve', 'set_approve', [
  { name: 'number',   dataType: 'integer', value: $h.number },
  { name: 'person',   dataType: 'integer', value: $h.текущее_физлицо },
  { name: 'decision', dataType: 'integer', value: 1 },
  { name: 'comment',  dataType: 'string',  value: $h.approve_comment }
]);

refresh('list');

var records = await from('register.согласование_договоров')
  .where('проект_договора = @number')
  .parameter('number', $h.number)
  .query();

var countApproved = records.clone().filter(row => row.решение == 1).count();
var approved = countApproved == records.rows.length;

if (approved) {
  $h.статус = 'у_директора';
  await recalculate();
  await save();

  await runWorkflow('contract_lawyer_message', 'messages', [
    { name: 'number', dataType: 'integer', value: $h.number }
  ]);
}

$h.approve_comment = '';
setIsModified(false);
setIsWaiting(false);
```

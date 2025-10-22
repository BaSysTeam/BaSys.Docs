# Построение и выполнение запросов

Для выполнения запросов к базе данных в системе **BaSYS** применяется следующий подход. Нами был разработан построитель запросов (QueryBuilder), с помощью которого формируется запрос к базе данных. Построитель создаётся с помощью функции `from()`, в которую передаётся имя таблицы, к которой будет выполнен запрос. Далее, с использованием методов построителя, производится поэтапная сборка запроса. Построитель поддерживает использование цепочки вызовов, что позволяет удобно формировать запрос. Методы построителя, в отличие от SQL, могут вызываться в произвольном порядке — например, **orderBy** можно вызвать до **select**, и это не повлияет на результат. Запрос отправляется на выполнение методом **query**.

Важно понимать, что выполнение запросов к базе данных является асинхронной операцией. Поэтому метод **query** построителя запроса возвращает объект **Promise**. Для получения результата запроса обязательно следует использовать ключевое слово **await**. 

```javascript
// Выполняется запрос к регистру currency_rates.
// Полученный результат запроса присваивается в переменную tableResult.
var tableResult = await from('register.currency_rates').query();
```

Если результатом запроса является таблица, он возвращается в виде объекта **DataTable**. После получения результата можно продолжить обработку данных с помощью методов этого объекта.

```javascript
var table_payment = 
// Выполняем запрос к таблице операции payment.
(await from("operation.payment").query())
// После получения данных продолжаем обработку методами объекта DataTable.
  .groupBy(['partner'], [{name: 'amount', alias: 'amount_p'}]);
```

## Создание построителя запроса

### **from**
Создает построитель запроса.

#### Синтаксис
```javascript
from('kindName.objectName')
```
#### Параметры
objectName - string, описание объекта к которому выполняется запрос. Используется следующий шаблон `<kindName>.<objectName>`, где kindName - имя вида объекта метаданных, objectName - имя объекта метаданных. Например: catalog.currency. 

#### Возвращаемое значение
QueryBuilder.

## Методы построителя запроса

| Метод                   | Возвращает   | Описание                                   |
| :---------------------- | :----------- | :----------------------------------------- |
| [select](#select)       | QueryBuilder | Задаёт перечень извлекаемых полей          |
| [orderBy](#orderby)     | QueryBuilder | Задаёт выражение сортировки                |
| [parameter](#parameter) | QueryBuilder | Устанавливает параметр                     |
| [top](#top)             | QueryBuilder | Задаёт количество выбираемых первых строк  |
| [where](#where)         | QueryBuilder | Задаёт условие фильтрации                  |
| [query](#query)         | Promise      | Отправляет запрос на выполнение            |

### **select**
Задает перечень извлекаемых полей, подерживается синтаксис выражений на SQL диалекте текущей 
базы данных.

### Синтаксис
```javascript
select(selectExpressions)
```
### Параметры
selectExpressions - string[], массив имен извлекаемых полей или SQL выражений на диалекте текущей базы данных.

### Возвращаемое значение
QueryBuilder.

#### Пример
```javascript
from("operation.supply")
// Задаем алиас period для поля date, как принято в SQL с использованием as.
  .select(["number", "date as period", "amount"])
  .query()
```

### **orderBy** 
Задаёт выражение сортировки в построителе запроса.

#### Синтаксис
```javascript
orderBy(orderExpression)
```
#### Параметры
orderExpression - string, выражение сортировки на диалекте SQL текущей базы данных.

#### Возвращаемое значение
QueryBuilder.

#### Пример
```javascript
from("operation.supply")
  .select(["number", "date as period", "amount"])
  .orderBy('amount desc, date desc')
  .query()
```

### **top** 
Ограничивает выборку указанным числом первых элементов.

#### Синтаксис
```javascript
top(numberOfItems)
```
#### Параметры
numberOfItems - number, число первых элементов, включаемых в выборку.

#### Возвращаемое значение
QueryBuilder

### **where**
Задаёт условие фильтрации для построителя запроса. Поддерживается параметризация: параметры указываются через символ `@` вне зависимости от используемой базы данных. Значения параметров необходимо установить с помощью метода **parameter** объекта QueryBuilder.

#### Синтаксис
```javascript
where(whereExpression)
```
#### Параметры
whereExpression - string, выражение фильтрации на диалекте SQL используемой базы данных.

#### Возвращаемое значение
QueryBuilder

#### Пример
```javascript
from("register.currency_rates")
  .where("currency = @currency")
  .parameter("currency", 'usd')
  .query()
```

### **parameter**
Устанавливает значение параметра запроса.

#### Синтаксис
```javascript
parameter(parameterName, parameterValue, dbType)
```
#### Параметры
parameterName - (string, обязательный). Имя параметра запроса. Задается **без** символа @.
parameterValue - (any, обязательный). Значение параметра запроса.
dbType - (number, опциональный). Числовые значения перечисления [System.DbType](https://learn.microsoft.com/ru-ru/dotnet/api/system.data.dbtype?view=net-8.0). Указывается в тех случаях, когда система не может определить тип параметра автоматически.

#### Возвращаемое значение
QueryBuilder

#### Пример
```javascript
from("register.currency_rates")
  .where("currency = @currency")
  .parameter("currency", 'usd')
  .query()
```

### **query** 
Отправляет собранный запрос на выполнение.

#### Синтаксис
```javascript
query()
```
#### Параметры
нет
#### Возвращаемое значение
Promise

## Примеры

### Пример 1

Получение курса валюты из журнала currency_rates.

```javascript
(await from("register.currency_rates")
  .where("currency = @currency")
  .parameter("currency", $h.currency)
  .orderBy("period desc")
  .query()
).rows[0].rate
```

### Пример 2

Получение валюты кассы из справочника cash_box

```javascript
(await from('catalog.cash_box')
 .where('id = @id')
 .parameter('id', $h.cash_box)
 .query()
)
.rows[0].currency;
```

### Пример 3. Информация о прикрепленных файлах.

Если для объекта метаданных настроено использование прикреплённых файлов, информацию о них можно получить запросом к «виртуальной» таблице по следующему шаблону:  
`<kindName>.<objectName>.attached_files`.

В примере ниже при установке параметра **id** явно задано значение [`System.Data.DbType`](https://learn.microsoft.com/en-us/dotnet/api/system.data.dbtype?view=net-9.0) — `11` (это `DbType.Int32`), соответствующее целочисленному типу `int`.

```javascript
var filesData = await from("operation.тарифы_дорожные.attached_files")
  .select(['uid', 'filename'])
  .where("objectuid = @id")
  // 11 - System.DbType.Int32
  .parameter("id", $h.number, 11)
  .query();

return filesData;
```

### Пример 4. Запрос к табличной части.

Если `DataObject` содержит табличные части (*DetailsTables*), к ним можно обратиться, выполнив запрос к таблице вида:  
`<kindName>.<objectName>.<detailsTableName>`.

В примере ниже извлекаются все строки табличной части **время** у объекта вида **operation** с именем **ведомость**. Возвращаются только строки того объекта, у которого идентификатор равен `3`. При установке параметра **id** явно задано значение [`System.Data.DbType`](https://learn.microsoft.com/en-us/dotnet/api/system.data.dbtype?view=net-9.0) — `11` (это `DbType.Int32`), соответствующее целочисленному типу `int`.

```javascript
var tableTime = await from("operation.ведомость.время")
  .where("object_uid = @id")
  .parameter("id", 3, 11)
  .query();

return tableTime;
```
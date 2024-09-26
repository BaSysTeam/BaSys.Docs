# Работа с табличными данными.

Для обработки табличных данных предназначен класс DataTable. Объект DataTable содержит коллекцию columns,
хранящую информацию о колонках таблицы. Данные хранятся в коллекции строк rows. Обрабатывать данные и изменять структуру
таблицы рекомендуется при помощи методов объекта DataTable. Многие методы объекта DataTable поддерживают конвейерный
способ работы, то есть можно вызывать цепочки методов для последовательного выполнения действий.  
Не рекомендуется непосредственно изменять колонки объекта DataTable через коллекцию columns, так как это
может привести к рассогласованию данных.

## Создание объекта DataTable

Объект DataTable может быть создан при помощи функции createTable или путем инициализации объекта с помощью ключевого
слова new.
Также объект DataTable возвращается в качестве результата запросов к БД.

```javascript
createTable([
    {name: 'product', dataType: 'string'},
    {name: 'quantity', dataType: 'number'},
    {name: 'amount', dataType: 'number'}
])
```

## Методы

| Метод                                                      | Возвращает      | Описание                            |
| :--------------------------------------------------------- | :-------------- | :---------------------------------- |
| [addColumn](#addcolumn)                                    | DataTable       | Добавление колонки в таблицу        |
| [addRow](#addrow)                                          | DataTable       | Добавление строки в таблицу         |
| [clear](#clear)                                            | DataTable       | Очистка строк таблицы               |
| [clone](#clone)                                            | DataTable       | Создание копии таблицы              |
| [deleteColumn](#deletecolumn)                              | DataTable       | Удаление колонки таблицы            |
| [distributeFifo](dataTableDistribution.md#distribute-fifo) | DataTable       | Распределение FIFO                  |
| [distributeLifo](dataTableDistribution.md#distribute-lifo) | DataTable       | Распределение LIFO                  |
| [innerJoin](dataTableJoins.md#inner-join)                  | DataTable       | Внутреннее соединение таблиц        |
| [filter](#filter)                                          | DataTable       | Фильтрация строк таблицы            |
| [fullJoin](dataTableJoins.md#full-join)                    | DataTable       | Полное внешнее соединение таблиц    |
| [getColumn](#getcolumn)                                    | DataTableColumn | Поиск колонки таблицы по имени      |
| [leftJoin](dataTableJoins.md#left-join)                    | DataTable       | Левое внешнее соединение таблиц     |
| [load](dataTable.md#load)                                  | DataTable       | Добавление данных в таблицу         |
| [newRow](#newrow)                                          | Object          | Создание новой строки таблицы       |
| [rightJoin](dataTableJoins.md#right-join)                  | DataTable       | Правое внешнее соединение таблиц    |
| [orderBy](#orderby)                                        | DataTable       | Сортировка строк таблицы            |
| union                                                      | DataTable       | Объединение уникальных строк таблиц |
| unionAll                                                   | DataTable       | Объединение всех строк таблиц       |


## addColumn

Добавляет колонку в таблицу.

### Синтаксис
```javascript
dataTable.addColumn(column)
```
### Параметры
- column: DataTableColumn - описание колонки таблицы.

### Возвращаемое значение
DataTable

### Пример
```javascript
createTable().addColumn({ name: 'period', dataType: 'date'})
```

## addRow
Добавляет строку в таблицу. Данные строки могут быть переданы в виде массива значений. 
В этом случае важно соблюдать позицию значения в массиве. Кроме того данные могут быть переданы в виде объекта.
Пропущенные значения будут инициализированы значениями по-умолчанию.

### Синтаксис
```javascript
dataTable.addRow(data)
```

### Параметры
- data: object[] - массив, содержащий данные добавляемой строки.
  
или

- data: object - объект, содержащий данные добавляемой строки.

### Возвращаемое значение
DataTable

### Пример
```javascript
// Создание таблицы
var tableRates = createTable([{ name: 'period', dataType: 'date'}, { name: 'person'}, { name: 'rate', dataType: 'number'}, { name: 'isWorking', dataType: 'boolean'}])
  // Добавление строки таблицы. Данные передаются в виде массива значений. Важна соблюдать позицию значения в массиве.
  .addRow(['2024-09-01', 'Person 1', 1000, true])
  // Добавление строки таблицы. Данные передаются в виде объекта.
  .addRow({person: 'Person 2', period: '2024-08-01', isWorking: false, rate: 2000});
return tableRates;
```

## clear
Очищает строки таблицы. Колонки остаются неизменными.

### Синтаксис
```javascript
dataTable.clear()
```
### Параметры
нет

### Возвращаемое значение
DataTable

## clone

Создает копию таблицы. 
Копируется как структура таблицы (колонки) так и данные.

### Синтаксис
```javascript
dataTable.clone()
```

### Параметры
нет

### Возвращаемое значение
DataTable

### Пример
```javascript
// Создание таблицы товаров и добавление строки в таблицу.
var productsTable = createTable([{ name: 'product'}, { name: 'quantity', dataType: 'number'}]).addRow(['product 1', 100]);
// Создание копии таблицы и добавление новой колонки. Таблица товаров productsTable остается неизменной.
var copyTable = productsTable.clone().addColumn({ name: 'amount', dataType: 'number'});
return copyTable;
```

## deleteColumn

Удаляет колонку таблицы.

### Синтаксис
```javascript
dataTable.deleteColumn(columnName)
```

### Параметры
- columnName: string - имя колонки таблицы.

### Возвращаемое значение
DataTable

### Пример
```javascript
createTable([{ name: 'product'}, { name: 'quantity', dataType: 'number'}, { name: 'price', dataType: 'number'}])
  .addRow(['product 1', 100, 10]).deleteColumn('quantity')
```

## filter
Фильтрует строки таблицы на основе заданного условия. Условие передается в виде стрелочной функции (предиката), 
которая принимает строку таблицы в качестве аргумента и возвращает логическое значение (true или false). 
Если предикат возвращает true, строка включается в результат фильтрации, в противном случае строка исключается.

### Синтаксис
```javascript
dataTable.filter(predicate)
```
### Параметры
- predicate: (row) => boolean — стрелочная функция, принимающая в качестве аргумента объект строки таблицы 
  и возвращающая true, если строка должна быть включена в результат фильтрации, и false в противном случае.

### Возвращаемое значение
DataTable

### Пример
```javascript
// Создание таблицы.
var tableRates = createTable([
  { name: 'period', dataType: 'date'}, 
  { name: 'person'}, 
  { name: 'rate', dataType: 'number'}, 
  { name: 'isWorking', dataType: 'boolean'}])
  // Добавление данных в таблицу.
  .load([
    ['2024-09-01', 'Person 1', 1000, true],
    ['2024-09-01', 'Person 2', 2000, true],
    ['2024-09-01', 'Person 3', 3000, true],
    ['2024-09-01', 'Person 4', 4000, true],
  ])
  // Выполнение фильтрации.
  .filter(row=> row.rate > 2000)
return tableRates;
```

## getColumn

Выполняет поиск колонки по имени.

### Синтаксис
```javascript
dataTable.getColumn(columnName)
```

### Параметры
- columnName: string - имя колонки.

### Возвращаемое значение
DataTableColumn

### Пример
```javascript
// Создание таблицы товаров и добавление строки в таблицу.
var productsTable = createTable([{ name: 'product'}, { name: 'quantity', dataType: 'number'}]).addRow(['product 1', 100]);
// Поиск колонки по имени.
var column = productsTable.getColumn('quantity');
return column;
```
## load
Добавляет данные в таблицу. Данные могут быть переданы в виде массива, 
содержащего массивы значений полей строк или массива объектов. 
Допускается смешение массивов значений и объектов в рамках одного набора данных.

### Синтаксис
```javascript
dataTable.load(data)
```
### Параметры
- data: any[] - массив объектов или массив, содержащий массивы значений полей строк.

### Возвращаемое значение
DataTable

### Пример
```javascript
// Создание таблицы.
var tableRates = createTable([
  { name: 'period', dataType: 'date'}, 
  { name: 'person'}, 
  { name: 'rate', dataType: 'number'}, 
  { name: 'isWorking', dataType: 'boolean'}])
  // Добавление данных в таблицу. Первая строка передается как массив значений полей строки. 
  // В этом случае важно соблюдать порядок значений.
  // Вторая строка добавляется в виде объекта.
  .load([
    ['2024-09-01', 'Person 1', 1000, true],
    { person: 'Person 2', period: '2024-08-01', isWorking: false, rate: 2000}
  ])
return tableRates;
```

## newRow
Создает новую строку таблицы и заполняет значениями по умолчанию.

### Синтаксис
```javascript
dataTable.newRow(pushNewRow)
```
### Параметры
-*pushNewRow (необязательный)*: boolean - указывает на необходимость поместить новую строку 
в коллекцию строк таблицы. Значение по умолчанию: true.

### Возвращаемое значение
Object

### Пример
Создает новую строку в таблице, заполняет значениями по умолчанию 
и помещает новую строку в коллекцию колонок таблицы.

```javascript
createTable([{ name: 'product'}, { name: 'quantity', dataType: 'number'}]).newRow();
```

## orderBy
Сортирует строки таблицы на основе заданного условия. Условие сортировки передается 
в виде стрелочной функции (компаратора), которая сравнивает две строки таблицы и возвращает 
число для определения порядка их следования.

### Синтаксис
```javascript
dataTable.orderBy(comparator)
```
### Параметры
- comparator: (a, b) => number — стрелочная функция, задающая условие сортировки. Принимает два объекта строки a и b, и возвращает:
   - отрицательное число, если строка a должна предшествовать строке b (сортировка по возрастанию);
   - положительное число, если строка b должна предшествовать строке a (сортировка по убыванию);
   - 0, если строки равны и их порядок не изменяется.

### Возвращаемое значение
DataTable — возвращает текущий объект DataTable с отсортированными строками.

### Примеры

Выполнение сортировки по числовой колонке по возрастанию.
```javascript
// Создание таблицы.
var tableRates = createTable([
  { name: 'period', dataType: 'date'}, 
  { name: 'person'}, 
  { name: 'rate', dataType: 'number'}, 
  { name: 'isWorking', dataType: 'boolean'}])
  // Добавление данных в таблицу.
  .load([
    ['2024-09-01', 'Person 2', 2000, true],
    ['2024-05-01', 'Person 3', 3000, true],
    ['2024-11-01', 'Person 1', 1000, false],
    ['2024-04-01', 'Person 4', 4000, true],
  ])
  // Выполнение сортировки по колонке rate по возрастанию.
  .orderBy( (a, b)=> a.rate - b.rate)
return tableRates;
```

Выполнение сортировки по колонке дат по убыванию.
```javascript
// Создание таблицы.
var tableRates = createTable([
  { name: 'period', dataType: 'date'}, 
  { name: 'person'}, 
  { name: 'rate', dataType: 'number'}, 
  { name: 'isWorking', dataType: 'boolean'}])
  // Добавление данных в таблицу.
  .load([
    [new Date('2024-09-01'), 'Person 2', 2000, true],
    [new Date('2024-05-01'), 'Person 3', 3000, true],
    [new Date('2024-11-01'), 'Person 1', 1000, false],
    [new Date('2024-04-01'), 'Person 4', 4000, true],
  ])
  // Выполнение сортировки по колонке period по убыванию.
  .orderBy( (a, b)=> b.period.getTime() - a.period.getTime())
return tableRates;
```

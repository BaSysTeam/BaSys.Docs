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

| Метод                                                      | Возвращает      | Описание                                |
| :--------------------------------------------------------- | :-------------- | :-------------------------------------- |
| [addColumn](#addcolumn)                                    | DataTable       | Добавление колонки в таблицу            |
| [addRow](#addrow)                                          | DataTable       | Добавление строки в таблицу             |
| [avg](#avg)                                                | DataTable       | Вычисление среднего по колонке таблицы  |
| [clear](#clear)                                            | DataTable       | Очистка строк таблицы                   |
| [clone](#clone)                                            | DataTable       | Создание копии таблицы                  |
| [count](#count)                                            | DataTable       | Подсчет количества строк в таблице      |
| [deleteColumn](#deletecolumn)                              | DataTable       | Удаление колонки таблицы                |
| [distributeFifo](dataTableDistribution.md#distribute-fifo) | DataTable       | Распределение FIFO                      |
| [distributeLifo](dataTableDistribution.md#distribute-lifo) | DataTable       | Распределение LIFO                      |
| [innerJoin](dataTableJoins.md#inner-join)                  | DataTable       | Внутреннее соединение таблиц            |
| [filter](#filter)                                          | DataTable       | Фильтрация строк таблицы                |
| [fullJoin](dataTableJoins.md#full-join)                    | DataTable       | Полное внешнее соединение таблиц        |
| [getColumn](#getcolumn)                                    | DataTableColumn | Поиск колонки таблицы по имени          |
| [groupBy](#groupby)                                        | DataTable       | Группировка таблицы                     |
| [leftJoin](dataTableJoins.md#left-join)                    | DataTable       | Левое внешнее соединение таблиц         |
| [load](dataTable.md#load)                                  | DataTable       | Добавление данных в таблицу             |
| [max](#max)                                                | DataTable       | Вычисление максимума по колонке таблицы |
| [min](#min)                                                | DataTable       | Вычисление минимума по колонке таблицы  |
| [newRow](#newrow)                                          | Object          | Создание новой строки таблицы           |
| [rightJoin](dataTableJoins.md#right-join)                  | DataTable       | Правое внешнее соединение таблиц        |
| [process](#process)                                        | DataTable       | Обработка строк таблицы                 |
| [orderBy](#orderby)                                        | DataTable       | Сортировка строк таблицы                |
| [sum](#sum)                                                | DataTable       | Вычисление суммы по колонке таблицы     |
| [union](#union)                                            | DataTable       | Объединение уникальных строк таблиц     |
| [unionAll](#unionall)                                      | DataTable       | Объединение всех строк таблиц           |


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

## avg
Вычисляет среднее значений по указанной колонке таблицы. 

### Синтаксис
```javascript
dataTable.min(columnName)
```
### Параметры
- columnName: string - имя колонки таблицы, по которой вычисляется среднее значение.
 
### Возвращаемое значение
number

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
    [new Date('2024-09-01'), 'Person 2', 2000, true],
    [new Date('2024-05-01'), 'Person 3', 3000, true],
    [new Date('2024-11-01'), 'Person 1', 1000, false],
    [new Date('2024-04-01'), 'Person 4', 4000, true],
  ]);
// Вычисление среднего по колонке 'rate'.
return tableRates.avg('rate');
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

## count
Подсчитывает количество строк в таблице. Если указано имя колонки, подсчитываются только строки, 
в которых значения в указанной колонке не пустые. Проверка на заполненность значений выполняется 
в соответствии с правилами JavaScript, где значения null, undefined, пустая строка (''), 0, false считаются пустыми.

### Синтаксис
```javascript
dataTable.count(columnName)
```
### Параметры
- *columnName (необязательный)*: string - имя колонки, по которой выполняется проверка на заполненность значений. 
  Если параметр не указан, функция вернёт общее количество строк в таблице. Если параметр указан, 
  возвращается количество строк, где значения в данной колонке не пустые. 
 
### Возвращаемое значение
- number - Количество строк в таблице или количество строк с заполненными значениями в указанной колонке.

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
    [new Date('2024-09-01'), 'Person 2', 2000, true],
    [new Date('2024-05-01'), 'Person 3', 3000, true],
    [new Date('2024-11-01'), 'Person 1', 1000, false],
    [new Date('2024-04-01'), 'Person 4', 4000, true],
  ]);
// Вычисление количества строк с заполненным полем isWorking.
return tableRates.count('isWorking');
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

## groupBy
Выполняет группировку строк таблицы по указанным ключевым колонкам и применяет агрегатные функции к выбранным колонкам.

### Синтаксис
```javascript
dataTable.groupBy(keyColumns, groupingColumns)
```
### Параметры
- keyColumns: string[] - Массив строк с именами колонок, по которым будут сгруппированы строки. 
  Эти колонки будут использоваться как уникальные идентификаторы групп.
- groupingColumns: [GroupingColumn](groupingColumn.md)[] - Массив объектов, описывающих колонки для агрегации и применяемые к ним агрегатные функции. Каждый объект может включать имя колонки, псевдоним и агрегатную функцию.
-  
Допустимые значения агрегатных функций:
  - 'avg' - среднее значение;
  - 'count' - количество элементов;
  - 'max' - максимальное значение;
  - 'min' - минимальное значение;
  - 'sum' - сумма значений.

### Возвращаемое значение
DataTable - сгруппированная таблица данных.

### Пример
```javascript
// Создание таблицы.
var tableProducts = createTable(
  [{name: 'product'}, { name: 'product_group'}, { name: 'store'}, { name: 'quantity', dataType: 'number'}])
  // Заполнение таблицы данными.
  .load([
    ['Product 1', 'Group 1', 'Store 1', 86],
    ['Product 19', 'Group 3', 'Store 3', 89],
    ['Product 16', 'Group 4', 'Store 1', 75],
    ['Product 14', 'Group 2', 'Store 1', 18],
    ['Product 12', 'Group 2', 'Store 3', 77],
    ['Product 25', 'Group 3', 'Store 2', 32]
  ])
  // Выполнение группировки по полю product_group c вычисленим различных агрегатных функций.
  .groupBy(['product_group'], [
    'quantity', 
    {name: 'quantity', alias:'q_max', aggregate:'max'},
    {name: 'quantity', alias:'q_min', aggregate:'min'},
    {name: 'quantity', alias:'q_avg', aggregate:'avg'},
    {name: 'quantity', alias:'q_count', aggregate:'count'}
  ]);

return tableProducts;
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

## max
Вычисляет максимальное значение по указанной колонке таблицы. 

### Синтаксис
```javascript
dataTable.max(columnName)
```
### Параметры
- columnName: string - имя колонки таблицы, по которой вычисляется максимальное значение.
 
### Возвращаемое значение
number

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
    [new Date('2024-09-01'), 'Person 2', 2000, true],
    [new Date('2024-05-01'), 'Person 3', 3000, true],
    [new Date('2024-11-01'), 'Person 1', 1000, false],
    [new Date('2024-04-01'), 'Person 4', 4000, true],
  ]);
// Вычисление максимума по колонке 'rate'.
return tableRates.max('rate');
```

## min
Вычисляет минимальное значение по указанной колонке таблицы. 

### Синтаксис
```javascript
dataTable.min(columnName)
```
### Параметры
- columnName: string - имя колонки таблицы, по которой вычисляется минимальное значение.
 
### Возвращаемое значение
number

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
    [new Date('2024-09-01'), 'Person 2', 2000, true],
    [new Date('2024-05-01'), 'Person 3', 3000, true],
    [new Date('2024-11-01'), 'Person 1', 1000, false],
    [new Date('2024-04-01'), 'Person 4', 4000, true],
  ]);
// Вычисление минимума по колонке 'rate'.
return tableRates.min('rate');
```

## process
Обрабатывает строки таблицы, выполняя переданную стрелочную функцию для каждой строки.
Функция может изменять содержимое строки или выполнять другие операции.

### Синтаксис
```javascript
dataTable.process(action)
```
### Параметры
- action: (row) => void - стрелочная функция, исполняемая для каждой строки таблицы.
  Параметры функции:
  - row: объект, представляющий строку таблицы, который может быть изменён в процессе обработки.

### Возвращаемое значение
DataTable - возвращает текущий объект DataTable, что позволяет использовать метод в цепочке вызовов.

### Пример
```javascript
// Создание таблицы.
var tableProducts = createTable([ 
  { name: 'product'}, 
  { name: 'price', dataType: 'number'}, 
  { name: 'quantity', dataType: 'number'}, 
  { name: 'amount', dataType: 'number'}])
  // Заполнение данных таблицы.
  .load([['Product 1', 100, 5], ['Product 2', 200, 10]])
  // Вычисление значения в колонке amount.
  .process(row=> row.amount = row.quantity * row. price);
return tableProducts;
```

## sum
Вычисляет сумму значений по указанной колонке таблицы. 

### Синтаксис
```javascript
dataTable.sum(columnName)
```
### Параметры
- columnName: string - имя колонки таблицы, по которой вычисляется сумма.
 
### Возвращаемое значение
number

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
    [new Date('2024-09-01'), 'Person 2', 2000, true],
    [new Date('2024-05-01'), 'Person 3', 3000, true],
    [new Date('2024-11-01'), 'Person 1', 1000, false],
    [new Date('2024-04-01'), 'Person 4', 4000, true],
  ]);
// Вычисление суммы по колонке 'rate'.
return tableRates.sum('rate');
```

## union
Осуществляет объединение текущей таблицы с другой таблицей, добавляя только уникальные строки. 
В отличие от unionAll, выполняется проверка на уникальность строк, что может замедлять выполнение, 
но гарантирует отсутствие дубликатов.

### Синтаксис
```javascript
dataTable.union(tableToUnion)
```
### Параметры
- tableToUnion: DataTable - таблица, которую нужно объединить с текущей таблицей. 
Строки из этой таблицы будут добавлены только в случае, если они отсутствуют в основной таблице.

### Возвращаемое значение
DataTable - Объединённая таблица, содержащая уникальные строки из обеих таблиц.

### Пример

Данный пример демонстрирует объединение двух таблиц. 
Строка ['product_1', 10, 100] в обеих таблицах одинаковая, поэтому она не будет добавлена второй раз. 
Однако строка ['product_3', 20, 4000] будет добавлена, так как она уникальна.

```javascript
// Определение колонок таблиц.
var columns = [{name: 'product'}, {name: 'quantity', dataType: 'number'}, {name: 'amount', dataType: 'number'}];
// Создание основной таблицы.
var mainTable = createTable(columns)
  .addRow(['product_1', 10, 100])
  .addRow(['product_2', 5, 2000]);
// Создание таблица для объединения.
var tableToUnion = createTable(columns)
  .addRow(['product_1', 10, 100])
  .addRow( ['product_3', 20, 4000]);
// Выполнение объединения таблиц.
return mainTable.union(tableToUnion);
```

## unionAll
Осуществляет объединение текущей таблицы с другой таблицей, добавляя все строки. 
В отличие от union, проверка на уникальность строк не выполняется.

### Синтаксис
```javascript
dataTable.union(tableToUnion)
```
### Параметры
- tableToUnion: DataTable - таблица, которую нужно объединить с текущей таблицей. 

### Возвращаемое значение
DataTable - Объединённая таблица, содержащая все строки из обеих таблиц.

### Пример

Данный пример демонстрирует объединение двух таблиц. 
Строка ['product_1', 10, 100] в обеих таблицах одинаковая, поэтому в результирующей таблице она будет задублирована.

```javascript
// Определение колонок таблиц.
var columns = [{name: 'product'}, {name: 'quantity', dataType: 'number'}, {name: 'amount', dataType: 'number'}];
// Создание основной таблицы.
var mainTable = createTable(columns)
  .addRow(['product_1', 10, 100])
  .addRow(['product_2', 5, 2000]);
// Создание таблица для объединения.
var tableToUnion = createTable(columns)
  .addRow(['product_1', 10, 100])
  .addRow( ['product_3', 20, 4000]);
// Выполнение объединения таблиц.
return mainTable.unionAll(tableToUnion);
```
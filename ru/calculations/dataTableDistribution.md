# Распределение FIFO / LIFO
Классическим применением данных функций является задача распределения товаров по партиям.
Например, на склад компании поступал товар различными партиями по различной стоимости.
При продаже товара необходимо списать себестоимость товаров по партиям. Возможны два варианта списания:
FIFO (первый пришел - первый ушел), LIFO (последний пришел - первый ушел).

В операции распределения принимают участие две таблицы. Основная таблица, 
которая содержит данные к распределению (primary). В примере с распределением товаров - это таблица товаров к распределению.
И таблица по которой выполняется распределение (secondary). В примере с распределением товаров - это таблица партий товаров.

При настройке колонок результирующей таблицы (если таковое требуется) следует использовать данные имена таблиц: 'primary', 'secondary'.

## Distribute FIFO
Выполняет распределение по FIFO.

### Синтаксис
```javascript
primaryTable.distributeFifo(secondaryTable, predicate, sortColumn, distributeColumn, columnSettings)
```

### Параметры
- secondaryTable: DataTable - таблица по которой выполняется распределение.
- predicate: Function - стрелочная функция, задающая условия распределения.
- sortColumn: string - имя колонки, по которой будет выполнена сортировка перед распределением.
- distributeColumn: string - имя колонки, по которой выполняется распределение. 
Должно совпадать в основной таблице (primary) и таблице распределения (secondary).
- *columnSettings (необязательный)*: columnDescription[] - массив настроек
  колонок результирующей таблицы.

### Возвращаемое значение
DataTable

## Distribute LIFO
Выполняет распределение по LIFO.

### Синтаксис
```javascript
primaryTable.distributeLifo(secondaryTable, predicate, sortColumn, distributeColumn, columnSettings)
```

### Параметры
- secondaryTable: DataTable - таблица по которой выполняется распределение.
- predicate: Function - стрелочная функция, задающая условия распределения.
- sortColumn: string - имя колонки, по которой будет выполнена сортировка перед распределением.
- distributeColumn: string - имя колонки, по которой выполняется распределение.
  Должно совпадать в основной таблице (primary) и таблице распределения (secondary).
- *columnSettings (необязательный)*: columnDescription[] - массив настроек
  колонок результирующей таблицы.

### Возвращаемое значение
DataTable

## Примеры

Выполнение распределения товаров по партиям по FIFO. 
Колонки результирующей таблицы создаются автоматически.

```javascript
// создание таблицы партий.
var batchTable = createTable(
        [ {name: 'period', dataType: 'date'},
          {name: 'product'},
          {name: 'batch'},
          {name: 'quantity', dataType: 'number'},
          {name: 'cost', dataType: 'number'},
          {name: 'unit_cost', dataType: 'number'} ])
        // Заполнение таблицы партий.
        .load([
          ['2024-09-01', 'product 1', 'batch 1', 10, 1000, 0],
          ['2024-09-01', 'product 2', 'batch 1', 50, 500, 0],
          ['2024-09-10', 'product 1', 'batch 2', 10, 2000, 0],
          ['2024-09-10', 'product 2', 'batch 2', 100, 1200, 0]])
        // Вычисление себестоимости единицы товара.  
        .process(row=> row.unit_cost = row.cost / row.quantity);
// Создание таблицы товаров
var productsTable = createTable([{name: 'product'},
  {name: 'price', dataType: 'number'},
  {name: 'quantity', dataType: 'number'},
  {name: 'amount', dataType: 'number'}])
        // Заполнение таблицы товаров данными.  
        .addRow(['product 1', 500, 15, 0]).addRow(['product 2', 30, 400, 0])
        // Выполнение распределения по FIFO.  
        .distributeFifo(batchTable, (primaryRow, secondaryRow)=> primaryRow.product == secondaryRow.product, 'period', 'quantity')
        // Вычисление стоимости и себестоимости после распределения.  
        .process(row => row.amount = row.price * row.quantity).process(row=> row.cost = row.unit_cost*row.quantity);
return productsTable;
```

Выполнение распределения товаров по партиям по FIFO.
Колонки результирующей таблицы настраиваются явно.

```javascript
// создание таблицы партий.
var batchTable = createTable(
        [ {name: 'period', dataType: 'date'},
          {name: 'product'},
          {name: 'batch'},
          {name: 'quantity', dataType: 'number'},
          {name: 'cost', dataType: 'number'},
          {name: 'unit_cost', dataType: 'number'} ])
        // Заполнение таблицы партий.
        .load([
          ['2024-09-01', 'product 1', 'batch 1', 10, 1000, 0],
          ['2024-09-01', 'product 2', 'batch 1', 50, 500, 0],
          ['2024-09-10', 'product 1', 'batch 2', 10, 2000, 0],
          ['2024-09-10', 'product 2', 'batch 2', 100, 1200, 0]])
        // Вычисление себестоимости единицы товара.  
        .process(row=> row.unit_cost = row.cost / row.quantity);
// Создание таблицы товаров
var productsTable = createTable([{name: 'product'},
  {name: 'price', dataType: 'number'},
  {name: 'quantity', dataType: 'number'},
  {name: 'amount', dataType: 'number'}])
        // Заполнение таблицы товаров данными.  
        .addRow(['product 1', 500, 15, 0]).addRow(['product 2', 30, 400, 0])
        // Выполнение распределения по FIFO.  
        .distributeFifo(batchTable, (primaryRow, secondaryRow)=> primaryRow.product == secondaryRow.product, 'period', 'quantity',
// Настройка колонок результирующей таблицы.                  
                [{table: 'primary', name: 'product', alias: 'item'},
                  {table: 'primary', name: 'price'},
                  {table: 'primary', name: 'quantity'},
                  {table: 'secondary', name: 'period'},
                  {table: 'secondary', name: 'batch'},
                  {table: 'secondary', name: 'unit_cost'}])
        // Вычисление стоимости и себестоимости после распределения.  
        .addColumn({name: 'amount', dataType: 'number'}).addColumn({name: 'cost', dataType: 'number'})
        .process(row => row.amount = row.price * row.quantity).process(row=> row.cost = row.unit_cost*row.quantity);
return productsTable;
```

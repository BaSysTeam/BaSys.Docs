# Соединения таблиц

## Inner join
Выполняет внутреннее соединение таблиц.

### Синтаксис
```javascript
primaryTable.innerJoin(secondaryTable, predicate, columnSettings)
```
### Параметры
- secondaryTable: DataTable - присоединяемая таблица данных.
- predicate: Function - стрелочная функция, задающая условие соединения.
- *columnSettings (необязательный)*: columnDescription[] - массив настроек 
колонок результирующей таблицы.  

### Возвращаемое значение
DataTable

## Left join
Выполняет левое внешнее соединение таблиц.

### Синтаксис
```javascript
primaryTable.leftJoin(secondaryTable, predicate, columnSettings)
```
### Параметры
- secondaryTable: DataTable - присоединяемая таблица данных.
- predicate: Function - стрелочная функция, задающая условие соединения.
- *columnSettings (необязательный)*: columnDescription[] - массив настроек
  колонок результирующей таблицы.

### Возвращаемое значение
DataTable

## Right join
Выполняет правое внешнее соединение таблиц.

### Синтаксис
```javascript
primaryTable.rightJoin(secondaryTable, predicate, columnSettings)
```
### Параметры
- secondaryTable: DataTable - присоединяемая таблица данных.
- predicate: Function - стрелочная функция, задающая условие соединения.
- *columnSettings (необязательный)*: columnDescription[] - массив настроек
  колонок результирующей таблицы.

### Возвращаемое значение
DataTable

## Full join
Выполняет полное внешнее соединение таблиц.

### Синтаксис
```javascript
primaryTable.fullJoin(secondaryTable, predicate, columnSettings)
```
### Параметры
- secondaryTable: DataTable - присоединяемая таблица данных.
- predicate: Function - стрелочная функция, задающая условие соединения.
- *columnSettings (необязательный)*: columnDescription[] - массив настроек
  колонок результирующей таблицы.

### Возвращаемое значение
DataTable

## Примеры

Выполнение соединения без настройки колонок результирующей таблицы. 
Колонки таблицы-результата настраиваются автоматически. 

```javascript
var tableRates = createTable([{name:'person'}, {name: 'position'}, {name: 'rate', dataType: 'number'}])
    .addRow(['person_1', 'developer', 100]).addRow(['person_2', 'administrator', 200]).addRow(['person_3', 'developer', 300]);
createTable([{name:'person'}, {name: 'department'}, {name:'hours', dateType:'number'}])
    .addRow(['person_1', 'department 1', 40])
    .addRow(['person_2', 'department 1', 30])
    .addRow(['person_4', 'department 2', 20])
    .addRow(['person_2', 'department 2', 25])
    .fullJoin(tableRates, (primaryRow, joinedRow)=> primaryRow.person == joinedRow.person)
```

Выполнение соединения с явной настройкой колонок результирующей таблицы.

```javascript
var tableRates = createTable([{name:'person'}, {name: 'position'}, {name: 'rate', dataType: 'number'}])
  .addRow(['person_1', 'developer', 100]).addRow(['person_2', 'administrator', 200]).addRow(['person_3', 'developer', 300]);
createTable([{name:'person'}, {name: 'department'}, {name:'hours', dateType:'number'}])
  .addRow(['person_1', 'department 1', 40])
  .addRow(['person_2', 'department 1', 30])
  .addRow(['person_4', 'department 2', 20])
  .addRow(['person_2', 'department 2', 25])
  .fullJoin(tableRates, (primaryRow, joinedRow)=> primaryRow.person == joinedRow.person, 
            [{table: 'primary', name:'person'}, {table: 'primary', name: 'hours'}, {table: 'secondary', name:'rate'}])
```

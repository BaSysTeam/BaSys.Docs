# WorkflowParameter
Описывает параметр процесса (workflow).

## Свойства

- **name**: string - имя параметра. Должно быть уникально в пределах процесса.
- **dataType**: string - тип данных процесса. Допустимые значения:
   - 'string'
   - 'number'
   - 'date'
   - 'boolean'
   - 'integer'
 - **value**: any - значение параметра.


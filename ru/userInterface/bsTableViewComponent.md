# Компонент `BsTableViewComponent`

Компонент отображает табличное представление (форму списка) с фильтрами, сортировкой и постраничной загрузкой данных. Базируется на [PrimeVue 3 DataTable](https://v3.primevue.org/datatable/), работает в режиме `lazy` и подгружает данные через `TableViewProvider`. В конструкторе форм компонент доступен под именем `bs-table-view`, а внутри программируемых компонентов — как `BsTableViewComponent`.

Типовые сценарии использования:

- форма списка для объекта метаданных (стандартный вариант, собираемый `ListFormBuilder`);
- встроенный список внутри программируемого компонента с предопределёнными фильтрами и сортировкой (например, «Задачи мне» / «Задачи от меня»);
- список с произвольным запросом, заданным выражением `from('…').select(…).query()` и контекстом данных формы;
- табличный вывод данных с раскраской строк через теги (`format: 'tag'`), бейджами (`format: 'badge.info'`) и иконками булевых значений.

## Источник данных

Свойство `dataSource` принимает два вида значений:

- **Имя объекта метаданных** (например, `"operation.task"`) — компонент обращается к серверу через `TableViewProvider.query` и поддерживает постраничную загрузку, серверную сортировку и серверные фильтры (включая фиксированные фильтры и сортировку).
- **Выражение запроса**, содержащее вызов `from(`, — компонент вычисляет выражение через `ExpressionEvaluator` в контексте `data` (см. свойство `data` ниже). В этом режиме данные загружаются разом, серверная пагинация не применяется.

## Props

- **dataSource** (`string`, обязательный) — источник данных: имя объекта метаданных или выражение запроса с `from(...)`.
- **columns** (`TableViewColumnViewModel[]`, обязательный) — описание колонок (см. раздел «Колонки»). В JSON-форме конструктора колонки задаются как дочерние элементы `bs-table-view-column`.
- **paginator** (`boolean`, необязательный, по умолчанию `true`) — отображать постраничную навигацию под таблицей.
- **height** (`string`, необязательный) — высота таблицы. Поддерживается специальное значение `stretch` (таблица растягивается на доступное пространство страницы) либо произвольное CSS-значение (`"400px"`, `"50vh"` и т. п.). Если не задано, используется внутреннее значение по умолчанию.
- **data** (`any`, необязательный) — контекст данных, в котором вычисляется выражение `dataSource` (используется только в режиме запроса с `from(...)`).
- **fixedFilters** (`Record<string, any | FixedFilterValue>`, необязательный) — фильтры, которые применяются всегда и переопределяют пользовательские для тех же полей. Конвертируются в формат PrimeVue через `FixedFiltersHelper.convertToPrimeVueFormat`. Допустимы как «значения», так и объекты вида `{ value, matchMode, columnDataType }`.
- **fixedSort** (`FixedSort`, необязательный) — фиксированная сортировка вида `{ field: string, order: 1 | -1 }`. Применяется при каждом обновлении.
- **rowTransformer** (`(rows: any[]) => any[] | Promise<any[]>`, необязательный) — функция постобработки строк после загрузки. Может быть асинхронной. Используется, например, для подсчёта производных полей (счётчик сообщений, преобразование «пустых» дат и т. п.).
- **size** (`'normal' | 'compact' | 'small'`, необязательный, по умолчанию `'normal'`) — масштаб таблицы. Влияет на размер шрифта и иконок:

| Значение `size` | Размер шрифта таблицы |
| --------------- | --------------------- |
| `normal`        | `1rem`                |
| `compact`       | `0.85rem`             |
| `small`         | `0.75rem`             |

## События

- **rowSelect** (`row: any`) — выделена строка таблицы (одиночный клик или выбор первой строки после обновления).
- **rowDblClick** (`row: any`) — двойной клик по строке. В конструкторе форм соответствует стандартной команде `standard.row_dbl_click` (обычно открывает форму редактирования).

## Публичные методы (через `ref`)

Внутри программируемого компонента к таблице удобно обращаться через `this.$refs.<name>`:

| Метод               | Назначение                                                                                                                                                                |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `refresh(data?)`    | Перезагружает данные. Если передан `data`, обновляет внутренний контекст (используется в режиме запроса).                                                                  |
| `setData(data)`     | Обновляет внутренний контекст без перезагрузки. Применяется перед `refresh()`, когда компонент монтируется программно (например, через `constructorFormRenderer`).         |
| `clearFilters()`    | Сбрасывает фильтры в исходное состояние и перезагружает данные.                                                                                                            |
| `getDataSource()`   | Возвращает текущее значение `dataSource`.                                                                                                                                  |
| `getQueryPayload()` | Возвращает объект параметров запроса в том же формате, что использует `TableViewProvider.query` (поля `dataSource`, `first`, `rows`, `sortField`, `sortOrder`, `filters`). |

## Колонки

Колонка таблицы описывается моделью `TableViewColumnViewModel` со следующими ключевыми полями (наследуется от `TableColumnViewModelBase`):

- **name** (`string`) — имя поля строки, отображаемого в колонке. Для колонок со ссылочными значениями принято имя `<field>_display` — оно используется для отображения, а для фильтра автоматически берётся «срезанное» имя `<field>` (см. `valueName`).
- **title** (`string`) — заголовок колонки.
- **width** (`string`) — ширина (`"120px"`, `"auto"`, `"50%"` и т. п.). Если задана, проставляется одновременно в `min-width`/`max-width`.
- **sortable** (`boolean`) — разрешить сортировку по колонке.
- **frozen** (`boolean`) — «прилипшая» колонка (фиксируется при горизонтальной прокрутке).
- **format** (`string`) — формат отображения значения. Поддерживаются:

| Значение `format`            | Отображение                                                                                                                              |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| пусто (по умолчанию)         | Текст «как есть» с обрезкой по ширине и тултипом.                                                                                        |
| `number`                     | Число с количеством знаков после точки из `numberDigits`. Колонка автоматически выравнивается по правому краю.                            |
| `date`                       | Дата (`ValuesFormatter.formatDate`).                                                                                                     |
| `dateTime`                   | Дата и время (`ValuesFormatter.formatDateTime`).                                                                                         |
| `boolean`                    | Иконка из `iconClass` (по умолчанию `pi pi-check text-primary`), если значение истинно.                                                  |
| `tag`                        | [PrimeVue Tag](https://v3.primevue.org/tag/) с конфигурацией из `tagConfig` / `defaultTagConfig` / `tagSeverityMap`.                     |
| `badge.<severity>`           | [PrimeVue Badge](https://v3.primevue.org/badge/) с указанным `severity` (`badge.info`, `badge.success` и т. п.).                          |

- **numberDigits** (`number`) — количество знаков после запятой для `format: 'number'`.
- **filterKind** (`'none' | 'string' | 'number' | 'boolean' | 'date' | 'dateTime' | 'objectReference' | 'multiSelect'`) — тип фильтра. От него зависит контрол в меню фильтра и режим сопоставления.
- **filterSource** (`string`) — для `filterKind: 'objectReference'` и `'multiSelect'`: UID объекта метаданных, из которого выбираются значения.
- **iconClass** (`string`) — CSS-класс иконки для `format: 'boolean'` (по умолчанию `pi pi-check text-primary`).
- **tagConfig** / **defaultTagConfig** / **tagSeverityMap** — конфигурация раскраски тегов для `format: 'tag'`. `tagSeverityMap` — упрощённый вариант (`{ <value>: <severity-or-colorClass> }`); `tagConfig` — полный (`{ <value>: { severity, display, colorClass, class, style } }`); `defaultTagConfig` — конфигурация по умолчанию, если значение не нашлось в `tagConfig`.

## Свойства по умолчанию в конструкторе форм

При добавлении компонента через визуальный конструктор форм (`FormElementBuilder.createTableView`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию |
| --------------- | --------------------- |
| `ComponentName` | `bs-table-view`       |
| `CssClass`      | пусто                 |
| `Style`         | пусто                 |
| `name`          | `list`                |
| `dataSource`    | пустая строка         |
| `height`        | `stretch`             |
| `@RowSelect`    | пустая строка         |
| `@RowDblClick`  | пустая строка         |

Свойство `name` используется командами вида `standard.refresh:<name>` и `standard.clear_filters:<name>` для адресации таблицы. По умолчанию стандартные кнопки формы списка обращаются к таблице с именем `list`.

Колонки добавляются как дочерние элементы — `FormElementBuilder.createTableViewColumn` создаёт `bs-table-view-column` со свойствами `title` (пусто), `name` (пусто) и `width` (`auto`). Остальные свойства колонки (`sortable`, `frozen`, `format`, `filterKind`, `filterSource` и т. д.) задаются вручную или автоматическим помощником `TableViewBuilder`.

## Примеры использования

### Использование в форме-конструкторе (JSON)

Таблица формы списка `operation.task` с фиксированными колонками: номер и дата (заморожены), статус и автор (фильтр по объектной ссылке), тема (строковый фильтр), срок (фильтр по дате-времени).

```json
{
  "Id": "bs-table-view-likZiU",
  "ComponentName": "bs-table-view",
  "CssClass": "",
  "Style": "",
  "Properties": [
    { "Name": "dataSource", "Value": "operation.task" },
    { "Name": "@RowSelect", "Value": "standard.row_select" },
    { "Name": "@RowDblClick", "Value": "standard.row_dbl_click" },
    { "Name": "name", "Value": "list" },
    { "Name": "height", "Value": "stretch" }
  ],
  "Items": [
    {
      "Id": "bs-table-view-column-sk5Ij3",
      "ComponentName": "bs-table-view-column",
      "Properties": [
        { "Name": "title", "Value": "Номер" },
        { "Name": "frozen", "Value": "true" },
        { "Name": "name", "Value": "number" },
        { "Name": "width", "Value": "120px" },
        { "Name": "sortable", "Value": "true" },
        { "Name": "filterKind", "Value": "number" }
      ],
      "Items": []
    },
    {
      "Id": "bs-table-view-column-zKKAP0",
      "ComponentName": "bs-table-view-column",
      "Properties": [
        { "Name": "title", "Value": "Дата" },
        { "Name": "frozen", "Value": "true" },
        { "Name": "name", "Value": "date" },
        { "Name": "sortable", "Value": "true" },
        { "Name": "width", "Value": "160px" },
        { "Name": "format", "Value": "dateTime" },
        { "Name": "filterKind", "Value": "dateTime" }
      ],
      "Items": []
    },
    {
      "Id": "bs-table-view-column-GsNTrI",
      "ComponentName": "bs-table-view-column",
      "Properties": [
        { "Name": "title", "Value": "Статус" },
        { "Name": "name", "Value": "status_display" },
        { "Name": "width", "Value": "200px" },
        { "Name": "filterKind", "Value": "objectReference" },
        { "Name": "filterSource", "Value": "049e5e0f-eb73-4a21-ae91-91505d1ee048" }
      ],
      "Items": []
    },
    {
      "Id": "bs-table-view-column-lQaA4O",
      "ComponentName": "bs-table-view-column",
      "Properties": [
        { "Name": "title", "Value": "Тема" },
        { "Name": "name", "Value": "topic" },
        { "Name": "width", "Value": "300px" },
        { "Name": "sortable", "Value": "true" },
        { "Name": "filterKind", "Value": "string" }
      ],
      "Items": []
    }
  ]
}
```

### Использование в шаблоне (программируемый компонент)

Две одинаковые таблицы с разными фиксированными фильтрами («Задачи мне» и «Задачи от меня» — упрощённый фрагмент `operation.task.form.my_tasks_script.vue`). Колонки строятся в `beforeMount` через инжектированный конструктор `TableViewColumnViewModel`, статус выводится тегом с раскраской из `tagConfig`, после загрузки строки дорабатываются `rowTransformer`-функцией.

```javascript
export default {
  inject: ['TableViewColumnViewModel'],
  data() {
    return {
      currentUserId: 0,
      tagConfig: {},
      myTasksColumns: [],
      context: { currentRow: { userId: 0 } },
    };
  },
  computed: {
    filterMyTasks() {
      return {
        responsible: {
          value: this.currentUserId,
          matchMode: 'equals',
          columnDataType: 'number',
        },
      };
    },
  },
  methods: {
    async calculateAfterLoad(rows) {
      rows.forEach((row) => {
        if (row.deadline === '0001-01-01T00:00:00') row.deadline = '';
      });
      return rows;
    },
    onMyTasksRowSelect(row) { this.myTasksCurrentRow = row; },
    onMyTasksRowDblClick() { /* … */ },
  },
  async beforeMount() {
    this.myTasksColumns = [
      new this.TableViewColumnViewModel({
        name: 'number', title: 'Номер', width: '100px', frozen: true, filterKind: 'number',
      }),
      new this.TableViewColumnViewModel({
        name: 'date', title: 'Дата', width: '140px', format: 'dateTime', filterKind: 'dateTime', frozen: true,
      }),
      new this.TableViewColumnViewModel({
        name: 'status', title: 'Статус', width: '140px',
        format: 'tag', tagConfig: this.tagConfig,
        filterKind: 'multiSelect',
        filterSource: '049e5e0f-eb73-4a21-ae91-91505d1ee048',
      }),
      new this.TableViewColumnViewModel({
        name: 'topic', title: 'Тема', width: 'auto', filterKind: 'string',
      }),
    ];
  },
};
```

```html
<BsTableViewComponent
  ref="myTasksTableView"
  name="my-tasks"
  dataSource="operation.task"
  height="stretch"
  size="compact"
  :fixed-filters="filterMyTasks"
  :columns="myTasksColumns"
  :data="context"
  :row-transformer="calculateAfterLoad"
  @rowSelect="onMyTasksRowSelect"
  @rowDblClick="onMyTasksRowDblClick" />
```

### Перезагрузка и сброс фильтров через `ref`

```javascript
this.$refs.myTasksTableView?.refresh();
this.$refs.myTasksTableView?.clearFilters();
```

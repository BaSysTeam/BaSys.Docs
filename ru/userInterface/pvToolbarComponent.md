# Компонент `Toolbar` / `pv-toolbar`

Панель инструментов из библиотеки PrimeVue. В **BaSYS** применяется как контейнер для надписей, бейджей и кнопок над списками, табличными частями и в шапках карточек: содержимое размещается в именованных слотах `start` и `end`. Полный список props, событий и слотов приведён в [PrimeVue 3 — Toolbar](https://v3.primevue.org/toolbar/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-toolbar` (реестр `ConstructorComponents`).
- В программируемых компонентах — под именем `Toolbar` (без регистрации, см. перечень в [Программируемые компоненты](programmableComponents.md)).

## Часто используемые в BaSYS props

У `Toolbar` собственных props в **BaSYS**-формах практически не задают: компонент используется как пустая обёртка, а его внешний вид настраивается через `CssClass`/`Style` и распределение содержимого по слотам.

- **CssClass** / **Style** — отступы и flex-раскладка панели. В типовых формах применяется inline-стиль вида `display: flex; flex-wrap: wrap; gap: 5px;` либо более узкие правила (`padding`, `margin-bottom`).
- **Слоты `start` и `end`** — основной способ управления компоновкой. У дочерних элементов в JSON-форме обязательно указывается свойство `slot` со значением `start` (левая часть панели) или `end` (правая часть). Подробнее о том, как конструктор форм собирает named slots из детей по этому свойству, см. раздел [Привязки данных и события](formConstructor.md#виды-свойств) в `formConstructor.md`.

## Свойства по умолчанию в конструкторе форм

При добавлении панели инструментов через визуальный конструктор форм (`FormElementBuilder.createToolbar`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию                         |
| --------------- | --------------------------------------------- |
| `ComponentName` | `pv-toolbar`                                  |
| `CssClass`      | пусто                                         |
| `Style`         | `display: flex; flex-wrap: wrap; gap: 5px;`   |
| `Properties`    | пусто                                         |

Дочерние элементы (кнопки, бейджи, тексты) добавляются вручную, и им проставляется свойство `slot` со значением `start` или `end`.

## Примеры использования

### В программируемом компоненте (template)

Панель инструментов над таблицей задач (из `operation.task.form.my_tasks_script.vue`). Кнопки действий вынесены в слот `start`, кнопки фильтрации/обновления — в слот `end`. Для самого `Toolbar` задан inline-стиль, уменьшающий стандартные отступы:

```html
<Toolbar style="margin:0; padding:5px;">
  <template #start>
    <Button label="Редактировать" size="small" severity="primary"
            icon="pi pi-external-link" text @click="onMyTaskEditClick" />
    <Button label="На основании" v-tooltip="'Ввод на основании'"
            size="small" severity="primary" icon="pi pi-file-import"
            class="ml-1" text @click="onMyTaskCreateFromClick" />
    <Button label="Дерево задач" size="small" severity="primary"
            icon="pi pi-sitemap" class="ml-1" text @click="onMyTaskTreeClick" />
  </template>
  <template #end>
    <Button v-tooltip="'Очистить фильтры'" size="small" severity="secondary"
            icon="pi pi-filter-slash" class="ml-1" text
            @click="onMyTaskClearFiltersClick" />
    <Button label="Обновить" size="small" severity="primary"
            icon="pi pi-refresh" class="ml-1" text @click="onMyTaskRefreshClick" />
  </template>
</Toolbar>
```

### В форме-конструкторе (JSON)

Минимальный случай — заголовок над таблицей списка «Мои задачи» (`operation.task.form.my_tasks.json`). Панель содержит единственный дочерний элемент `bs-text`, размещённый в слоте `end`:

```json
{
  "id": "pv-toolbar-DRhQLt",
  "componentName": "pv-toolbar",
  "cssClass": "",
  "style": "display: flex; flex-wrap: wrap; gap: 5px;",
  "properties": [],
  "items": [
    {
      "id": "bs-text-sQnSOG",
      "componentName": "bs-text",
      "cssClass": "w-full",
      "style": "",
      "properties": [
        { "name": "text", "value": "Мои задачи" },
        { "name": "slot", "value": "end" }
      ],
      "items": []
    }
  ]
}
```

Панель сообщений из формы редактирования задачи (`operation.task.form.edit_PM2RXN.json`) — пример с обоими слотами. Кнопка обновления стоит в слоте `start`, а условно отображаемая кнопка-команда — в слоте `end`:

```json
{
  "id": "pv-toolbar-So32GH",
  "componentName": "pv-toolbar",
  "cssClass": "",
  "style": "display: flex; flex-wrap: wrap; gap: 5px; padding: 3px;",
  "properties": [],
  "items": [
    {
      "id": "pv-button-lj78VU",
      "componentName": "pv-button",
      "properties": [
        { "name": "label", "value": "Обновить" },
        { "name": "icon", "value": "pi pi-refresh" },
        { "name": "@Click", "value": "messages_refresh" },
        { "name": "severity", "value": "primary" },
        { "name": "size", "value": "small" },
        { "name": "slot", "value": "start" },
        { "name": "text", "value": "" }
      ],
      "items": []
    },
    {
      "id": "pv-button-UacQEU",
      "componentName": "pv-button",
      "properties": [
        { "name": "label", "value": "Сообщения мной прочитаны" },
        { "name": "icon", "value": "pi pi-check" },
        { "name": "@Click", "value": "mark_messages" },
        { "name": "severity", "value": "danger" },
        { "name": "size", "value": "small" },
        { "name": "slot", "value": "end" },
        { "name": "vIf", "value": "$h.is_task_user" }
      ],
      "items": []
    }
  ]
}
```

Панель команд табличной части `время` из формы редактирования наряда (`operation.монтаж_площадка.form.edit_KgWjmu.json`) — нестандартный случай, в котором слот `start` собирает кнопки и `pv-split-button`-ы со списком действий, а слот `end` — заголовок и счётчик строк (`pv-badge` с привязкой `:value` к выражению):

```json
{
  "id": "pv-toolbar-JVyHFb",
  "componentName": "pv-toolbar",
  "cssClass": "",
  "style": "padding: 0.2rem; margin-bottom: 0.2rem",
  "properties": [],
  "items": [
    {
      "id": "pv-button-Px6yls",
      "componentName": "pv-button",
      "properties": [
        { "name": "icon", "value": "pi pi-plus" },
        { "name": "@Click", "value": "standard.table_add:время" },
        { "name": "severity", "value": "primary" },
        { "name": "size", "value": "small" },
        { "name": "text", "value": "" },
        { "name": "slot", "value": "start" }
      ],
      "items": []
    },
    {
      "id": "bs-text-ZCPAci",
      "componentName": "bs-text",
      "properties": [
        { "name": "text", "value": "Учет отработанного времени" },
        { "name": "slot", "value": "end" }
      ],
      "items": []
    },
    {
      "id": "pv-badge-gQIbPu",
      "componentName": "pv-badge",
      "cssClass": "ml-1",
      "properties": [
        { "name": ":value", "value": "data.tables.время?.count()" },
        { "name": "severity", "value": "primary" },
        { "name": "size", "value": "small" },
        { "name": "slot", "value": "end" }
      ],
      "items": []
    }
  ]
}
```

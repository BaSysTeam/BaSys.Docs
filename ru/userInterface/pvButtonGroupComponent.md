# Компонент `ButtonGroup` / `pv-button-group`

Структурный контейнер из библиотеки PrimeVue, объединяющий вложенные кнопки [`pv-button`](pvButtonComponent.md) в единую группу с общим скруглением углов и без зазоров между кнопками. Используется в **BaSYS** для группировки кнопок управления в панелях инструментов (`pv-toolbar`) на формах списков и редактирования (типовой набор «Добавить / Редактировать / Удалить» в списках и «Вернуться / Сохранить&Закрыть / Сохранить» в формах редактирования). Подробное описание см. в [PrimeVue 3 — Button (раздел Button Group)](https://v3.primevue.org/button/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-button-group` (реестр `ConstructorComponents`).
- В программируемых компонентах — **не входит** в список компонентов, доступных программируемому компоненту по умолчанию (отсутствует в `dynamicComponentLoader.ts`). При необходимости группа кнопок в шаблоне программируемого компонента собирается из обычных `Button` внутри контейнера с CSS-классом PrimeVue `p-buttonset` либо регистрируется явно в `components` самой формы.

## Часто используемые в BaSYS props

`ButtonGroup` — чисто структурный контейнер; собственных props не имеет. Поведение и внешний вид группы целиком определяются вложенными `pv-button`. На уровне самого узла в **BaSYS** дополнительно можно использовать:

- **CssClass** — общий CSS-класс группы (например, для отступов или выравнивания внутри тулбара).
- **Style** — inline-стиль контейнера.
- **Items** — обязательный список дочерних `FormElement` с `ComponentName: "pv-button"`. Другие типы компонентов в группе не размещаются.

Свойства, влияющие на состояние отдельной кнопки (`label`, `icon`, `severity`, `size`, `outlined`, `@Click`, `:disabled`, `vIf` и т. п.), задаются на дочерних узлах — см. [Компонент `Button` / `pv-button`](pvButtonComponent.md).

## Свойства по умолчанию в конструкторе форм

При добавлении группы кнопок через визуальный конструктор форм (`FormElementBuilder.createButtonGroup`) создаётся пустой `FormElement` без `Properties`. Кнопки в группу добавляются отдельно — каждая поверх `pv-button-group` как дочерний элемент.

| Поле            | Значение по умолчанию |
| --------------- | --------------------- |
| `ComponentName` | `pv-button-group`     |
| `CssClass`      | пусто                 |
| `Style`         | пусто                 |
| `Properties`    | `[]` (пустой массив)  |
| `Items`         | `[]` (пустой массив; кнопки добавляются отдельно) |

## Примеры использования

### В форме-конструкторе (JSON)

Типовая группа кнопок управления записями в форме списка задач (`operation.task.form.list_veNfSv.json`) — «Добавить» и «Редактировать», привязанные к стандартным командам `standard.add` и `standard.edit`:

```json
{
  "id": "pv-button-group-8fGhAo",
  "componentName": "pv-button-group",
  "cssClass": "",
  "style": "",
  "properties": [],
  "items": [
    {
      "id": "pv-button-KLeHPa",
      "componentName": "pv-button",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Добавить" },
        { "name": "icon", "value": "pi pi-plus" },
        { "name": "@Click", "value": "standard.add" },
        { "name": "severity", "value": "primary" },
        { "name": "size", "value": "small" },
        { "name": "outlined", "value": "outlined" }
      ],
      "items": []
    },
    {
      "id": "pv-button-fZH9A5",
      "componentName": "pv-button",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Редактировать" },
        { "name": "icon", "value": "pi pi-pencil" },
        { "name": "@Click", "value": "standard.edit" },
        { "name": "severity", "value": "primary" },
        { "name": "size", "value": "small" },
        { "name": "outlined", "value": "outlined" }
      ],
      "items": []
    }
  ]
}
```

Группа кнопок в форме редактирования задачи (`operation.task.form.edit_PM2RXN.json`) — «Вернуться / Сохранить&Закрыть / Сохранить». Сохранение блокируется через привязку `:disabled` к выражению на основе `formState`-хелпера:

```json
{
  "id": "pv-button-group-uM22qD",
  "componentName": "pv-button-group",
  "cssClass": "",
  "style": "",
  "properties": [],
  "items": [
    {
      "id": "pv-button-YLSvAe",
      "componentName": "pv-button",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Вернуться" },
        { "name": "icon", "value": "pi pi-arrow-left" },
        { "name": "@Click", "value": "standard.return" },
        { "name": "severity", "value": "primary" },
        { "name": "size", "value": "small" },
        { "name": "outlined", "value": "outlined" }
      ],
      "items": []
    },
    {
      "id": "pv-button-9akpYT",
      "componentName": "pv-button",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Сохранить&Закрыть" },
        { "name": "icon", "value": "pi pi-save" },
        { "name": "@Click", "value": "standard.save_close" },
        { "name": "severity", "value": "primary" },
        { "name": "size", "value": "small" },
        { "name": "outlined", "value": "outlined" },
        { "name": ":disabled", "value": "!$h.is_task_user" }
      ],
      "items": []
    },
    {
      "id": "pv-button-OlwYc7",
      "componentName": "pv-button",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Сохранить" },
        { "name": "icon", "value": "pi pi-save" },
        { "name": "@Click", "value": "standard.save" },
        { "name": "severity", "value": "primary" },
        { "name": "size", "value": "small" },
        { "name": "outlined", "value": "outlined" },
        { "name": ":disabled", "value": "!$h.is_task_user" }
      ],
      "items": []
    }
  ]
}
```

Группа обычно размещается внутри панели инструментов `pv-toolbar` (типовой контейнер для верхнего ряда кнопок формы). Кнопки, которые должны стоять рядом с группой, но визуально не сливаться с ней (например, «Создать на основании», «Перейти к движениям», «Файлы»), выносятся за пределы `pv-button-group` отдельными `pv-button` с CSS-классом `ml-1` — см. примеры в статье [`Button` / `pv-button`](pvButtonComponent.md).

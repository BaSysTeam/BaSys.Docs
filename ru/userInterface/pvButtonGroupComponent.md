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
  "Id": "pv-button-group-8fGhAo",
  "ComponentName": "pv-button-group",
  "CssClass": "",
  "Style": "",
  "Properties": [],
  "Items": [
    {
      "Id": "pv-button-KLeHPa",
      "ComponentName": "pv-button",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Добавить" },
        { "Name": "icon", "Value": "pi pi-plus" },
        { "Name": "@Click", "Value": "standard.add" },
        { "Name": "severity", "Value": "primary" },
        { "Name": "size", "Value": "small" },
        { "Name": "outlined", "Value": "outlined" }
      ],
      "Items": []
    },
    {
      "Id": "pv-button-fZH9A5",
      "ComponentName": "pv-button",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Редактировать" },
        { "Name": "icon", "Value": "pi pi-pencil" },
        { "Name": "@Click", "Value": "standard.edit" },
        { "Name": "severity", "Value": "primary" },
        { "Name": "size", "Value": "small" },
        { "Name": "outlined", "Value": "outlined" }
      ],
      "Items": []
    }
  ]
}
```

Группа кнопок в форме редактирования задачи (`operation.task.form.edit_PM2RXN.json`) — «Вернуться / Сохранить&Закрыть / Сохранить». Сохранение блокируется через привязку `:disabled` к выражению на основе `formState`-хелпера:

```json
{
  "Id": "pv-button-group-uM22qD",
  "ComponentName": "pv-button-group",
  "CssClass": "",
  "Style": "",
  "Properties": [],
  "Items": [
    {
      "Id": "pv-button-YLSvAe",
      "ComponentName": "pv-button",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Вернуться" },
        { "Name": "icon", "Value": "pi pi-arrow-left" },
        { "Name": "@Click", "Value": "standard.return" },
        { "Name": "severity", "Value": "primary" },
        { "Name": "size", "Value": "small" },
        { "Name": "outlined", "Value": "outlined" }
      ],
      "Items": []
    },
    {
      "Id": "pv-button-9akpYT",
      "ComponentName": "pv-button",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Сохранить&Закрыть" },
        { "Name": "icon", "Value": "pi pi-save" },
        { "Name": "@Click", "Value": "standard.save_close" },
        { "Name": "severity", "Value": "primary" },
        { "Name": "size", "Value": "small" },
        { "Name": "outlined", "Value": "outlined" },
        { "Name": ":disabled", "Value": "!$h.is_task_user" }
      ],
      "Items": []
    },
    {
      "Id": "pv-button-OlwYc7",
      "ComponentName": "pv-button",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Сохранить" },
        { "Name": "icon", "Value": "pi pi-save" },
        { "Name": "@Click", "Value": "standard.save" },
        { "Name": "severity", "Value": "primary" },
        { "Name": "size", "Value": "small" },
        { "Name": "outlined", "Value": "outlined" },
        { "Name": ":disabled", "Value": "!$h.is_task_user" }
      ],
      "Items": []
    }
  ]
}
```

Группа обычно размещается внутри панели инструментов `pv-toolbar` (типовой контейнер для верхнего ряда кнопок формы). Кнопки, которые должны стоять рядом с группой, но визуально не сливаться с ней (например, «Создать на основании», «Перейти к движениям», «Файлы»), выносятся за пределы `pv-button-group` отдельными `pv-button` с CSS-классом `ml-1` — см. примеры в статье [`Button` / `pv-button`](pvButtonComponent.md).

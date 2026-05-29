# Компонент `SplitButton` / `pv-split-button`

Кнопка с выпадающим меню из библиотеки PrimeVue. В **BaSYS** применяется в типовых формах списка и редактирования для группировки набора дополнительных команд (как правило, под подписью «Действия» или «Печать»): «Обновить», «Очистить фильтры», «Excel», «Создать/Удалить записи», «Журнал расчётов», «Пересчитать» и т. п. Полная документация и список свойств приведены в [PrimeVue 3 — SplitButton](https://v3.primevue.org/splitbutton/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-split-button` (реестр `ConstructorComponents`).
- В программируемых компонентах — не входит в список компонентов, доступных программируемому компоненту по умолчанию (см. перечень в [Программируемые компоненты](programmableComponents.md)). При необходимости использовать `Button` или `Menu` напрямую.

## Часто используемые в BaSYS props

- **label** (`string`) — подпись на основной (левой) кнопке. В типовых формах **BaSYS** — «Действия» или «Печать».
- **icon** (`string`) — CSS-класс иконки из набора [PrimeIcons](https://primevue.org/icons/) для основной кнопки. В большинстве форм не задаётся.
- **severity** (`'primary' | 'secondary' | 'success' | 'info' | 'warning' | 'help' | 'danger' | 'contrast'`) — семантический цвет. В типовых формах **BaSYS** используется `primary`.
- **size** (`'small' | 'large'`) — размер. В типовых формах — `small`.
- **outlined** (`boolean`) — вариант с рамкой без заливки. По умолчанию выставляется в конструкторе форм.
- **@Click** — обработчик нажатия на основную (левую) кнопку. Значение — имя команды (см. раздел [Команды](formConstructor.md#команды) в `formConstructor.md`). В реальных формах **BaSYS** обычно не задаётся: основная кнопка используется только как «якорь» для меню, а команды размещены в пунктах меню.

Пункты выпадающего меню задаются дочерними элементами `pv-split-button-item` в массиве `Items`. Их свойства (`label`, `icon`, `command`) собираются рендерером в массив `model`, который и передаётся в PrimeVue-`SplitButton`. Подробнее — в [pvSplitButtonItemComponent.md](pvSplitButtonItemComponent.md).

## Свойства по умолчанию в конструкторе форм

При добавлении компонента через визуальный конструктор форм (`FormElementBuilder.createSplitButton`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию |
| --------------- | --------------------- |
| `ComponentName` | `pv-split-button`     |
| `CssClass`      | пусто                 |
| `Style`         | пусто                 |
| `label`         | пустая строка         |
| `size`          | `small`               |
| `severity`      | `primary`             |
| `outlined`      | `outlined`            |

Свойство `icon`, обработчик `@Click` и сами пункты меню (`pv-split-button-item`) по умолчанию не создаются — добавляются вручную в конструкторе или в JSON формы.

## Примеры использования

### В форме-конструкторе (JSON)

Кнопка «Действия» с одним пунктом меню «Журнал расчётов» — типичный случай для формы редактирования (`operation.task.form.edit_PM2RXN.json`). Сама кнопка обработчика `@Click` не имеет; команда привязана к единственному пункту меню:

```json
{
  "id": "pv-split-button-e2fvUY",
  "componentName": "pv-split-button",
  "cssClass": "ml-1",
  "style": "",
  "properties": [
    { "name": "label", "value": "Действия" },
    { "name": "severity", "value": "primary" },
    { "name": "size", "value": "small" },
    { "name": "outlined", "value": "outlined" }
  ],
  "items": [
    {
      "id": "pv-split-button-item-DXaf6N",
      "componentName": "pv-split-button-item",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Лог вычислений" },
        { "name": "icon", "value": "pi pi-list" },
        { "name": "command", "value": "standard.open_log" }
      ],
      "items": []
    }
  ]
}
```

Кнопка «Действия» в форме списка (`operation.task.form.list_veNfSv.json`) c четырьмя стандартными командами — «Создать записи», «Удалить записи», «Обновить» и «Очистить фильтры». Команды «Обновить» и «Очистить фильтры» вызываются с параметром после двоеточия (имя табличной части `list`):

```json
{
  "id": "pv-split-button-u0LAOa",
  "componentName": "pv-split-button",
  "cssClass": "ml-1",
  "style": "",
  "properties": [
    { "name": "label", "value": "Действия" },
    { "name": "severity", "value": "primary" },
    { "name": "size", "value": "small" },
    { "name": "outlined", "value": "outlined" }
  ],
  "items": [
    {
      "id": "pv-split-button-item-VaAnSX",
      "componentName": "pv-split-button-item",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Создать записи" },
        { "name": "icon", "value": "pi pi-check" },
        { "name": "command", "value": "standard.create_records" }
      ],
      "items": []
    },
    {
      "id": "pv-split-button-item-DeTm1M",
      "componentName": "pv-split-button-item",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Удалить записи" },
        { "name": "icon", "value": "pi pi-times" },
        { "name": "command", "value": "standard.delete_records" }
      ],
      "items": []
    },
    {
      "id": "pv-split-button-item-dSdtWl",
      "componentName": "pv-split-button-item",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Обновить" },
        { "name": "icon", "value": "pi pi-refresh" },
        { "name": "command", "value": "standard.refresh:list" }
      ],
      "items": []
    },
    {
      "id": "pv-split-button-item-TJkC1o",
      "componentName": "pv-split-button-item",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Очистить фильтры" },
        { "name": "icon", "value": "pi pi-filter-slash" },
        { "name": "command", "value": "standard.clear_filters:list" }
      ],
      "items": []
    }
  ]
}
```

Развёрнутый набор пунктов меню — типичен для формы редактирования объекта со связанными записями (`operation.проект_договора.form.edit_72xnlc.json`). Помимо «Журнала расчётов» и «Пересчитать», в меню добавлены команды для управления связанными записями и команда экспорта в Excel:

```json
{
  "id": "pv-split-button-ntucHA",
  "componentName": "pv-split-button",
  "cssClass": "ml-1",
  "style": "",
  "properties": [
    { "name": "label", "value": "Действия" },
    { "name": "severity", "value": "primary" },
    { "name": "size", "value": "small" },
    { "name": "outlined", "value": "outlined" }
  ],
  "items": [
    {
      "id": "pv-split-button-item-iDpPmb",
      "componentName": "pv-split-button-item",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Лог вычислений" },
        { "name": "icon", "value": "pi pi-list" },
        { "name": "command", "value": "standard.open_log" }
      ],
      "items": []
    },
    {
      "id": "pv-split-button-item-9SQNQr",
      "componentName": "pv-split-button-item",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Пересчитать" },
        { "name": "icon", "value": "pi pi-refresh" },
        { "name": "command", "value": "standard.recalculate" }
      ],
      "items": []
    },
    {
      "id": "pv-split-button-item-u59SPi",
      "componentName": "pv-split-button-item",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Создать записи" },
        { "name": "icon", "value": "pi pi-check" },
        { "name": "command", "value": "standard.create_records" }
      ],
      "items": []
    },
    {
      "id": "pv-split-button-item-r6hOkZ",
      "componentName": "pv-split-button-item",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Удалить записи" },
        { "name": "icon", "value": "pi pi-times" },
        { "name": "command", "value": "standard.delete_records" }
      ],
      "items": []
    },
    {
      "id": "pv-split-button-item-tV09Ly",
      "componentName": "pv-split-button-item",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "label", "value": "Excel" },
        { "name": "icon", "value": "pi pi-file-excel" },
        { "name": "command", "value": "standard.export_excel" }
      ],
      "items": []
    }
  ]
}
```

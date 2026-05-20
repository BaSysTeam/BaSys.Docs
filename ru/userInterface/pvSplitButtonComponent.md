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
  "Id": "pv-split-button-e2fvUY",
  "ComponentName": "pv-split-button",
  "CssClass": "ml-1",
  "Style": "",
  "Properties": [
    { "Name": "label", "Value": "Действия" },
    { "Name": "severity", "Value": "primary" },
    { "Name": "size", "Value": "small" },
    { "Name": "outlined", "Value": "outlined" }
  ],
  "Items": [
    {
      "Id": "pv-split-button-item-DXaf6N",
      "ComponentName": "pv-split-button-item",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Лог вычислений" },
        { "Name": "icon", "Value": "pi pi-list" },
        { "Name": "command", "Value": "standard.open_log" }
      ],
      "Items": []
    }
  ]
}
```

Кнопка «Действия» в форме списка (`operation.task.form.list_veNfSv.json`) c четырьмя стандартными командами — «Создать записи», «Удалить записи», «Обновить» и «Очистить фильтры». Команды «Обновить» и «Очистить фильтры» вызываются с параметром после двоеточия (имя табличной части `list`):

```json
{
  "Id": "pv-split-button-u0LAOa",
  "ComponentName": "pv-split-button",
  "CssClass": "ml-1",
  "Style": "",
  "Properties": [
    { "Name": "label", "Value": "Действия" },
    { "Name": "severity", "Value": "primary" },
    { "Name": "size", "Value": "small" },
    { "Name": "outlined", "Value": "outlined" }
  ],
  "Items": [
    {
      "Id": "pv-split-button-item-VaAnSX",
      "ComponentName": "pv-split-button-item",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Создать записи" },
        { "Name": "icon", "Value": "pi pi-check" },
        { "Name": "command", "Value": "standard.create_records" }
      ],
      "Items": []
    },
    {
      "Id": "pv-split-button-item-DeTm1M",
      "ComponentName": "pv-split-button-item",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Удалить записи" },
        { "Name": "icon", "Value": "pi pi-times" },
        { "Name": "command", "Value": "standard.delete_records" }
      ],
      "Items": []
    },
    {
      "Id": "pv-split-button-item-dSdtWl",
      "ComponentName": "pv-split-button-item",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Обновить" },
        { "Name": "icon", "Value": "pi pi-refresh" },
        { "Name": "command", "Value": "standard.refresh:list" }
      ],
      "Items": []
    },
    {
      "Id": "pv-split-button-item-TJkC1o",
      "ComponentName": "pv-split-button-item",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Очистить фильтры" },
        { "Name": "icon", "Value": "pi pi-filter-slash" },
        { "Name": "command", "Value": "standard.clear_filters:list" }
      ],
      "Items": []
    }
  ]
}
```

Развёрнутый набор пунктов меню — типичен для формы редактирования объекта со связанными записями (`operation.проект_договора.form.edit_72xnlc.json`). Помимо «Журнала расчётов» и «Пересчитать», в меню добавлены команды для управления связанными записями и команда экспорта в Excel:

```json
{
  "Id": "pv-split-button-ntucHA",
  "ComponentName": "pv-split-button",
  "CssClass": "ml-1",
  "Style": "",
  "Properties": [
    { "Name": "label", "Value": "Действия" },
    { "Name": "severity", "Value": "primary" },
    { "Name": "size", "Value": "small" },
    { "Name": "outlined", "Value": "outlined" }
  ],
  "Items": [
    {
      "Id": "pv-split-button-item-iDpPmb",
      "ComponentName": "pv-split-button-item",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Лог вычислений" },
        { "Name": "icon", "Value": "pi pi-list" },
        { "Name": "command", "Value": "standard.open_log" }
      ],
      "Items": []
    },
    {
      "Id": "pv-split-button-item-9SQNQr",
      "ComponentName": "pv-split-button-item",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Пересчитать" },
        { "Name": "icon", "Value": "pi pi-refresh" },
        { "Name": "command", "Value": "standard.recalculate" }
      ],
      "Items": []
    },
    {
      "Id": "pv-split-button-item-u59SPi",
      "ComponentName": "pv-split-button-item",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Создать записи" },
        { "Name": "icon", "Value": "pi pi-check" },
        { "Name": "command", "Value": "standard.create_records" }
      ],
      "Items": []
    },
    {
      "Id": "pv-split-button-item-r6hOkZ",
      "ComponentName": "pv-split-button-item",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Удалить записи" },
        { "Name": "icon", "Value": "pi pi-times" },
        { "Name": "command", "Value": "standard.delete_records" }
      ],
      "Items": []
    },
    {
      "Id": "pv-split-button-item-tV09Ly",
      "ComponentName": "pv-split-button-item",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "label", "Value": "Excel" },
        { "Name": "icon", "Value": "pi pi-file-excel" },
        { "Name": "command", "Value": "standard.export_excel" }
      ],
      "Items": []
    }
  ]
}
```

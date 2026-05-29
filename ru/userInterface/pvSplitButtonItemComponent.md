# Компонент `SplitButtonItem` / `pv-split-button-item`

Псевдокомпонент **BaSYS**, описывающий один пункт меню кнопки [`pv-split-button`](pvSplitButtonComponent.md). В чистом [PrimeVue 3](https://v3.primevue.org/splitbutton/) отдельного `SplitButtonItem` нет — пункты меню задаются объектами массива `model` родительской кнопки (поля `label`, `icon`, `command`, `items`). Конструктор форм **BaSYS** оборачивает каждый такой объект в отдельный `FormElement`, чтобы его можно было визуально добавлять, удалять и переставлять в дереве формы; при рендеринге список дочерних `pv-split-button-item` собирается в массив `model` для `pv-split-button`.

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-split-button-item` (перечисление `ConstructorComponents`). Используется только как дочерний элемент `pv-split-button`.
- В программируемых компонентах не используется — пункты меню задаются напрямую массивом `model` компонента `SplitButton` (см. примеры на [странице PrimeVue](https://v3.primevue.org/splitbutton/)).

## Часто используемые в BaSYS props

В JSON-узле `pv-split-button-item` практически всегда задаются три свойства из объекта пункта меню PrimeVue:

- **label** (`string`) — подпись пункта меню (например, `Создать записи`, `Обновить`, `Очистить фильтры`, `Лог вычислений`).
- **icon** (`string`) — CSS-класс иконки из набора [PrimeIcons](https://primevue.org/icons/) (`pi pi-check`, `pi pi-times`, `pi pi-refresh`, `pi pi-filter-slash`, `pi pi-list` и т. п.). Допускается пустая строка, если пункт меню без иконки.
- **command** (`string`) — имя команды, выполняемой при выборе пункта меню. В **BaSYS** обработчик задаётся не через `@Click`, а строковым свойством `command`, значение которого — имя команды формата `группа.имя[:параметр]` (см. раздел [Команды](formConstructor.md#команды) в `formConstructor.md`). Используются стандартные команды (`standard.create_records`, `standard.delete_records`, `standard.refresh:list`, `standard.clear_filters:list`, `standard.open_log`, `standard.recalculate`, `standard.export_excel` и др.) или пользовательские команды объекта метаданных.

Свойства `CssClass` и `Style` у пункта меню, как правило, не задаются: внешний вид определяется родительским `pv-split-button` и темой PrimeVue. Дочерних элементов `pv-split-button-item` обычно не содержит — вложенное меню в формах **BaSYS** на момент написания статьи не используется.

## Свойства по умолчанию в конструкторе форм

При добавлении пункта меню через визуальный конструктор форм (`FormElementBuilder.createSplitButtonItem`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию    |
| --------------- | ------------------------ |
| `ComponentName` | `pv-split-button-item`   |
| `CssClass`      | пусто                    |
| `Style`         | пусто                    |
| `label`         | пустая строка            |
| `icon`          | пустая строка            |
| `command`       | пустая строка            |

## Примеры использования

### В форме-конструкторе (JSON)

Один пункт меню «Лог вычислений» внутри `pv-split-button` «Действия» формы редактирования задачи (`operation.task.form.edit_PM2RXN.json`). Привязан к стандартной команде `standard.open_log`:

```json
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
```

Набор пунктов меню `pv-split-button` «Действия» в форме списка задач (`operation.task.form.list_veNfSv.json`). У части команд через двоеточие передан параметр — имя таблицы, к которой применяется команда (`standard.refresh:list`, `standard.clear_filters:list`):

```json
{
  "id": "pv-split-button-MO5w4r",
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

Пункт меню без иконки и с пользовательской командой объекта метаданных (`operation.учет_раб_времени_цех.form.edit_1I7HDt.json`) — здесь значение `command` совпадает с именем команды, заданной у объекта метаданных:

```json
{
  "id": "pv-split-button-item-ExxEPS",
  "componentName": "pv-split-button-item",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "label", "value": "Сотрудники Киселёва А.А." },
    { "name": "icon", "value": "" },
    { "name": "command", "value": "сотрудники" }
  ],
  "items": []
}
```

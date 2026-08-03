# Компонент `Checkbox` / `pv-checkbox`

Стандартный флажок из библиотеки PrimeVue. В **BaSYS** применяется для редактирования булевых полей шапки документа/справочника, признаков и опций формы (создавать записи, фиксации, признаки «уволен», «актуальные» и т. п.). Полный список props, событий и слотов приведён в [PrimeVue 3 — Checkbox](https://v3.primevue.org/checkbox/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-checkbox` (реестр `ConstructorComponents`).
- В программируемых компонентах — не входит в список компонентов, доступных программируемому компоненту по умолчанию (см. перечень в [Программируемые компоненты](programmableComponents.md)). При необходимости компонент регистрируется вручную в опциях программируемого компонента или используется альтернатива `pv-input-switch` / `TriStateCheckbox`.

Для булевых полей шапки/таблицы помощники `EditFormBuilder` и `DetailsTableBuilder` автоматически подбирают элемент ввода — `pv-checkbox` либо `pv-input-switch` (см. примечание о типах элементов в [Конструкторе форм](formConstructor.md#быстрое-создание-формы-списка-и-формы-редактирования)). В сгенерированной форме элемент при необходимости можно вручную заменить на любой из этих двух вариантов.

## Часто используемые в BaSYS props

- **vModel** (`boolean | any[]`) — связанное значение. В **BaSYS** в подавляющем большинстве случаев привязывается к булеву полю шапки/строки таблицы (`$h.<имя>`, `$r.<имя>`) и используется вместе с `binary: true`.
- **binary** (`boolean`) — режим одиночного булева флажка. В типовых формах **BaSYS** всегда выставляется в `true`; без него `vModel` ожидает массив выбранных значений (групповой режим из документации PrimeVue).
- **disabled** (`boolean`) — отключение флажка. В JSON-формах удобно задавать привязкой `:disabled` к выражению на основе `formState`; виртуальные поля помощник `EditFormBuilder` помечает `disabled` автоматически.
- **invalid** (`boolean`) — индикация ошибки валидации (рамка ошибки). В **BaSYS** применяется редко — обязательность поля задаётся на уровне `bs-form-field` (`required`) и проверяется при сохранении.

Собственного клик-обработчика для запуска команд платформы у флажка обычно нет — изменение значения отражается через `vModel`; при необходимости можно навесить `@Change` с именем команды (см. раздел [Команды](formConstructor.md#команды) в `formConstructor.md`).

Подпись к флажку в **BaSYS** не задаётся свойством компонента — для подписи используется обёртка [`bs-form-field`](bsFormFieldComponent.md) со свойствами `text` и `labelWidth` (в ранее сохранённых формах вместо `labelWidth` встречается `labelCols`, см. примеры ниже).

## Свойства по умолчанию в конструкторе форм

При добавлении флажка через визуальный конструктор форм (`FormElementBuilder.createCheckbox`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию |
| --------------- | --------------------- |
| `ComponentName` | `pv-checkbox`         |
| `CssClass`      | пусто                 |
| `Style`         | пусто                 |
| `vModel`        | пустая строка         |
| `binary`        | `true`                |

## Примеры использования

### В форме-конструкторе (JSON)

Типовой флажок «Создать записи» в шапке формы редактирования, обёрнутый в `bs-form-field` с подписью (`operation.штатное_расписание.form.edit_SeUfKV.json`):

```json
{
  "id": "bs-form-field-PqokLV",
  "dataUid": "593754f9-3ae3-abfb-7165-4cf8be8cb659",
  "componentName": "bs-form-field",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "text", "value": "Создать записи" },
    { "name": "labelCols", "value": "4" }
  ],
  "items": [
    {
      "id": "pv-checkbox-ARIRtq",
      "dataUid": "593754f9-3ae3-abfb-7165-4cf8be8cb659",
      "componentName": "pv-checkbox",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "vModel", "value": "$h.create_records" },
        { "name": "binary", "value": "true" }
      ],
      "items": []
    }
  ]
}
```

Флажок признака «Уволен!» в форме редактирования (`operation.штатное_расписание.form.edit_SeUfKV.json`) — привязка к полю шапки `$h.уволен`:

```json
{
  "id": "pv-checkbox-Oe21hu",
  "dataUid": "973b5610-86b3-c991-76a8-907e7f5d39e8",
  "componentName": "pv-checkbox",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "vModel", "value": "$h.уволен" },
    { "name": "binary", "value": "true" }
  ],
  "items": []
}
```

Флажок с привязкой через полный путь `data.header.<имя>` вместо сокращения `$h.<имя>` (`operation.монтаж_площадка.form.edit_KgWjmu.json`) — поведение конструктора при таком написании эквивалентно:

```json
{
  "id": "pv-checkbox-DQghS6",
  "dataUid": "15af10b0-eb96-3aa0-ff24-f2c78948b3a8",
  "componentName": "pv-checkbox",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "vModel", "value": "data.header.create_records" },
    { "name": "binary", "value": "true" }
  ],
  "items": []
}
```

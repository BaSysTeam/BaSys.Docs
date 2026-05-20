# Компонент `BsCollapsibleGroup`

Компонент-обёртка для сворачиваемой группы элементов формы. Содержит кликабельный заголовок с шевроном (`pi pi-chevron-right` / `pi pi-chevron-down`) и контентную область, которая показывается или скрывается по клику. В конструкторе форм компонент доступен под именем `bs-collapsible-group` и используется, в частности, помощником `EditFormBuilder` для группировки полей шапки документа.

Типовые сценарии использования:

- группировка полей шапки документа/справочника в форме редактирования (раздел «Основные» и подобные);
- сворачиваемые вспомогательные блоки внутри вкладок (например, «Добавление визы вручную» в форме согласования);
- любые декоративные группы, которые пользователь должен иметь возможность сворачивать, чтобы освободить место на форме.

## Props

- **title** (`string`, обязательный) — текст заголовка группы. Отображается в кликабельной строке заголовка с CSS-классом `bs-group-title`.
- **open** (`boolean`, необязательный, по умолчанию `true`) — начальное состояние группы (раскрыта/свёрнута). Значение реактивно: при изменении prop извне внутреннее состояние группы синхронизируется автоматически.
- **textAlign** (`'left' | 'right'`, необязательный, по умолчанию `'right'`) — расположение заголовка и шеврона внутри строки заголовка. Управляет CSS-классом корневого элемента `bs-text-align-{textAlign}` и порядком элементов:

| Значение `textAlign` | CSS-класс корня         | Расположение               |
| -------------------- | ----------------------- | -------------------------- |
| `right`              | `bs-text-align-right`   | Заголовок и шеврон у правого края, шеврон справа от заголовка. |
| `left`               | `bs-text-align-left`    | Заголовок и шеврон у левого края, шеврон слева от заголовка.   |

- **cssClass** (`string`, необязательный) — дополнительный CSS-класс корневого элемента. Объединяется с базовыми классами `bs-collapsible-group` и `bs-text-align-{textAlign}`.
- **style** (`Record<string, any>`, необязательный) — inline-стиль корневого элемента.

Содержимое группы передаётся через слот по умолчанию. Когда группа свёрнута, контентный `<div class="bs-group-content">` не рендерится — это означает, что компоненты в слоте размонтируются и их состояние не сохраняется между сворачиваниями.

## Свойства по умолчанию в конструкторе форм

При добавлении компонента через визуальный конструктор форм (`FormElementBuilder.createCollapsibleGroup`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию  |
| --------------- | ---------------------- |
| `ComponentName` | `bs-collapsible-group` |
| `CssClass`      | пусто                  |
| `Style`         | пусто                  |
| `title`         | пустая строка          |
| `open`          | `true`                 |
| `text-align`    | `right`                |

Внутри группы конструктор допускает только строки сетки (`bs-row`) — это контролируется методом `move` класса `ConstructorFormSettings`. Соответственно, поля шапки и любые другие компоненты следует размещать внутри `bs-row` / `bs-col`, помещённых в `bs-collapsible-group`.

## Примеры использования

### Использование в шаблоне (программируемый компонент)

Группа с двумя полями ввода и заголовком слева:

```html
<BsCollapsibleGroup title="Основные параметры"
                    text-align="left"
                    :open="false">
  <div class="grid">
    <div class="col-6">
      <BsFormFieldComponent text="Название">
        <InputText v-model="title" size="small" class="w-full" />
      </BsFormFieldComponent>
    </div>
    <div class="col-6">
      <BsFormFieldComponent text="Код">
        <InputText v-model="code" size="small" class="w-full" />
      </BsFormFieldComponent>
    </div>
  </div>
</BsCollapsibleGroup>
```

### Использование в форме-конструкторе (JSON)

Группа «Основные» с полями шапки документа (типовой результат работы `EditFormBuilder` — взято из формы редактирования объекта `operation.task`):

```json
{
  "Id": "bs-collapsible-group-JlldTC",
  "ComponentName": "bs-collapsible-group",
  "CssClass": "",
  "Style": "",
  "Properties": [
    { "Name": "title", "Value": "Основные" },
    { "Name": "open", "Value": "true" },
    { "Name": "text-align", "Value": "right" }
  ],
  "Items": [
    {
      "Id": "bs-row-ZEW0KN",
      "ComponentName": "bs-row",
      "CssClass": "grid",
      "Style": "",
      "Properties": [],
      "Items": [
        {
          "Id": "bs-col-...",
          "ComponentName": "bs-col",
          "CssClass": "col",
          "Style": "",
          "Properties": [],
          "Items": []
        }
      ]
    }
  ]
}
```

Вспомогательная сворачиваемая группа внутри вкладки (взято из формы редактирования объекта `operation.проект_договора`, блок «Добавление визы вручную»):

```json
{
  "Id": "bs-collapsible-group-FaaqRO",
  "ComponentName": "bs-collapsible-group",
  "CssClass": "",
  "Style": "",
  "Properties": [
    { "Name": "title", "Value": "Добавление визы вручную" },
    { "Name": "open", "Value": "true" },
    { "Name": "text-align", "Value": "right" }
  ],
  "Items": [
    {
      "Id": "bs-row-NCVXvU",
      "ComponentName": "bs-row",
      "CssClass": "grid",
      "Style": "",
      "Properties": [],
      "Items": []
    }
  ]
}
```

Если требуется привязать начальное состояние группы к выражению на основе данных формы, имя свойства следует записать как `:open` (одностороннее `v-bind`) — например, чтобы группа раскрывалась только для новых документов:

```json
{
  "Id": "bs-collapsible-group-extra",
  "ComponentName": "bs-collapsible-group",
  "CssClass": "",
  "Style": "",
  "Properties": [
    { "Name": "title", "Value": "Дополнительные параметры" },
    { "Name": ":open", "Value": "$h.is_new" },
    { "Name": "text-align", "Value": "left" }
  ],
  "Items": []
}
```

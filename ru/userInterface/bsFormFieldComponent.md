# Компонент `BsFormFieldComponent`

Компонент предназначен для размещения элемента ввода в составе формы вместе с его подписью. Использует 12-колоночную сетку [PrimeFlex](https://primeflex.org/) и автоматически делит строку на колонку подписи (`labelCols`) и колонку самого поля ввода (`12 - labelCols`). Поддерживает признак обязательности (CSS-класс `bs-required` на подписи) и горизонтальное выравнивание текста подписи внутри её колонки. В конструкторе форм компонент доступен под именем `bs-form-field`.

Типовые сценарии использования:

- стандартное поле редактирования в шапке документа/справочника (подпись слева, элемент ввода справа);
- поле обязательного ввода с пометкой звёздочкой (`required`);
- широкое поле для длинного значения (узкая подпись через `labelCols="1"`, например, для тем и комментариев);
- поле без подписи (`labelCols="0"`) — рендерится только элемент ввода, удобно для выравнивания со «штатными» полями формы.

## Props

- **text** (`string`, обязательный) — текст подписи, выводимый в `<label>`.
- **labelCols** (`number`, необязательный, по умолчанию `4`) — количество колонок PrimeFlex-сетки (от 0 до 12), отводимых под подпись. Остаток (`12 - labelCols`) занимает слот с элементом ввода. При значении `0` подпись не рендерится — выводится только содержимое слота.
- **labelFor** (`string`, необязательный) — значение HTML-атрибута `for` подписи; идентификатор связанного элемента ввода.
- **id** (`string`, необязательный) — идентификатор корневого DOM-элемента подписи (`<label>`).
- **style** (`Record<string, any>`, необязательный) — inline-стиль подписи. Объединяется со стилем выравнивания, формируемым из `labelAlign`.
- **cssClass** (`string`, необязательный) — дополнительный CSS-класс подписи. К корневому элементу подписи всегда добавляются базовые классы `bs-label` и `col-{labelCols}`; значение `cssClass` мержится с ними.
- **required** (`boolean`, необязательный, по умолчанию `false`) — при значении `true` к подписи добавляется CSS-класс `bs-required` (визуальный признак обязательного поля).
- **labelAlign** (`'left' | 'right' | 'center'`, необязательный, по умолчанию `'left'`) — горизонтальное выравнивание текста подписи внутри её колонки. Реализуется через CSS-свойство `justify-content`:

| Значение `labelAlign` | `justify-content` |
| --------------------- | ----------------- |
| `left`                | `flex-start`      |
| `center`              | `center`          |
| `right`               | `flex-end`        |

Элемент ввода передаётся в компонент через слот по умолчанию. Корневой элемент компонента имеет CSS-классы `field grid align-items-center`.

## Свойства по умолчанию в конструкторе форм

При добавлении компонента через визуальный конструктор форм (`FormElementBuilder.createFormField`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию |
| --------------- | --------------------- |
| `ComponentName` | `bs-form-field`       |
| `CssClass`      | `w-full`              |
| `Style`         | пусто                 |
| `text`          | пустая строка         |
| `labelFor`      | пустая строка         |
| `labelCols`     | `4`                   |
| `labelAlign`    | `left`                |

Значения `id`, `style`, `required` и собственный `cssClass` по умолчанию не заполняются — их следует задавать вручную при необходимости. Сам элемент ввода (`pv-input-text`, `pv-input-number`, `pv-calendar`, `pv-checkbox`, `bs-object-reference-select` и т. п.) добавляется как дочерний элемент `bs-form-field` и автоматически попадает в слот по умолчанию.

## Примеры использования

### Использование в шаблоне (программируемый компонент)

Поле обязательного ввода с подписью и текстовым полем:

```html
<BsFormFieldComponent text="Тема"
                      labelFor="topic-input"
                      :labelCols="3"
                      :required="true">
  <InputText id="topic-input" v-model="topic" size="small" class="w-full" />
</BsFormFieldComponent>
```

### Использование в форме-конструкторе (JSON)

Обычное поле выбора объекта метаданных (взято из формы редактирования объекта `operation.task`, поле «Основание»):

```json
{
  "Id": "bs-form-field-iSizb4",
  "DataUid": "3714ed97-925b-a834-ca63-46efe62c3bbf",
  "ComponentName": "bs-form-field",
  "CssClass": "",
  "Style": "",
  "Properties": [
    { "Name": "text", "Value": "Основание" },
    { "Name": "labelCols", "Value": "4" }
  ],
  "Items": [
    {
      "Id": "bs-object-reference-select-nTEgyr",
      "ComponentName": "bs-object-reference-select",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "vModel", "Value": "$h.base_task" }
      ],
      "Items": []
    }
  ]
}
```

Обязательное поле с пометкой звёздочкой (поле «Автор» там же):

```json
{
  "Id": "bs-form-field-wAf07A",
  "DataUid": "0d9fc741-1867-e208-b0db-4d1bc02b44d2",
  "ComponentName": "bs-form-field",
  "CssClass": "",
  "Style": "",
  "Properties": [
    { "Name": "text", "Value": "Автор" },
    { "Name": "labelCols", "Value": "4" },
    { "Name": "required", "Value": "" }
  ],
  "Items": [
    {
      "Id": "bs-object-reference-select-author",
      "ComponentName": "bs-object-reference-select",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "vModel", "Value": "$h.author" }
      ],
      "Items": []
    }
  ]
}
```

Широкое поле для длинного текста (узкая подпись, `labelCols="1"` — поле «Описание»):

```json
{
  "Id": "bs-form-field-8gON6Q",
  "DataUid": "0ebdd22a-3383-1b6f-95db-97ea72c38796",
  "ComponentName": "bs-form-field",
  "CssClass": "",
  "Style": "",
  "Properties": [
    { "Name": "text", "Value": "Описание" },
    { "Name": "labelCols", "Value": "1" }
  ],
  "Items": [
    {
      "Id": "pv-input-textarea-gfL3h1",
      "ComponentName": "pv-input-textarea",
      "CssClass": "w-full",
      "Style": "",
      "Properties": [
        { "Name": "vModel", "Value": "$h.description" },
        { "Name": "rows", "Value": "5" },
        { "Name": "size", "Value": "small" }
      ],
      "Items": []
    }
  ]
}
```

Поле с выравниванием подписи по правому краю и условным рендерингом (взято из формы редактирования объекта `operation.проект_договора`):

```json
{
  "Id": "bs-form-field-osnDog",
  "DataUid": "523d6000-6839-7e1b-84c6-eb33b1eccb64",
  "ComponentName": "bs-form-field",
  "CssClass": "",
  "Style": "",
  "Properties": [
    { "Name": "text", "Value": "Основной договор" },
    { "Name": "labelCols", "Value": "4" },
    { "Name": "v-if", "Value": "$h.договор_допсоглашение == 'доп_соглашение'" },
    { "Name": "labelAlign", "Value": "right" }
  ],
  "Items": [
    {
      "Id": "bs-object-reference-select-ZpoFnx",
      "ComponentName": "bs-object-reference-select",
      "CssClass": "",
      "Style": "",
      "Properties": [
        { "Name": "vModel", "Value": "$h.основной_договор" },
        { "Name": ":text", "Value": "$h.основной_договор_display" },
        { "Name": "size", "Value": "small" }
      ],
      "Items": []
    }
  ]
}
```

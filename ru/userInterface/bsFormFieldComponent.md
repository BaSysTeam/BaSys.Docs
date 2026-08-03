# Компонент `BsFormFieldComponent`

Компонент предназначен для размещения элемента ввода в составе формы вместе с его подписью. Поддерживает признак обязательности (CSS-класс `bs-required` на подписи) и горизонтальное выравнивание текста подписи. В конструкторе форм компонент доступен под именем `bs-form-field`.

Типовые сценарии использования:

- стандартное поле редактирования в шапке документа/справочника (подпись слева, элемент ввода справа);
- поле обязательного ввода с пометкой звёздочкой (`required`);
- широкое поле для длинного значения (узкая подпись через `labelWidth`, например, для тем и комментариев);
- поле без подписи (`labelCols="0"`) — рендерится только элемент ввода, удобно для выравнивания со «штатными» полями формы.

## Режимы раскладки

Компонент умеет делить строку на подпись и элемент ввода двумя способами. Режим выбирается свойством `labelWidth`, которое имеет приоритет над `labelCols`.

### Фиксированная ширина подписи (режим по умолчанию)

Строка строится настоящей CSS-сеткой: первая колонка равна заданной ширине подписи, вторая — `minmax(0, 1fr)`, между ними задаётся промежуток `column-gap`. Элемент ввода «резиновый» и занимает всю оставшуюся ширину ячейки, длинная подпись переносится по словам.

```css
display: grid;
grid-template-columns: 10.6875rem minmax(0, 1fr);
column-gap: 0.6875rem;
```

Значения по умолчанию берутся из общих токенов дизайна в `shared/src/styles/basys_custom.css`:

| Токен                       | Значение     | Назначение                        |
| --------------------------- | ------------ | --------------------------------- |
| `--basys-form-label-width`  | `10.6875rem` | ширина колонки подписи            |
| `--basys-form-label-gap`    | `0.6875rem`  | промежуток между подписью и полем |

Корневой элемент в этом режиме имеет классы `field bs-form-field--fixed`, размеры колонок передаются inline-стилем, слот с элементом ввода оборачивается в `div.bs-form-field__control`.

### Колоночная раскладка PrimeFlex (устаревший режим)

Включается только явным значением `labelWidth="cols"`. Строка делится по 12-колоночной сетке [PrimeFlex](https://primeflex.org/): подпись занимает `labelCols` колонок, слот с элементом ввода — остаток (`12 - labelCols`). Корневой элемент получает классы `field grid align-items-center`, подпись — класс `col-{labelCols}`, слот — `col-{12 - labelCols}`.

## Props

- **text** (`string`, обязательный) — текст подписи, выводимый в `<label>`.
- **labelWidth** (`string | number`, необязательный, по умолчанию `default`) — ширина колонки подписи; определяет режим раскладки. Допустимые значения:

| Значение `labelWidth`      | Результат                                                                |
| -------------------------- | ------------------------------------------------------------------------ |
| `default` или пустая строка | ширина из токена `--basys-form-label-width` (`10.6875rem`)               |
| CSS-длина, например `12rem` | указанная ширина «как есть»                                              |
| число, например `170`      | ширина в пикселях (`170px`); строка `"170"` трактуется так же            |
| `cols`                     | переход на колоночную раскладку PrimeFlex, ширина берётся из `labelCols` |

- **columnGap** (`string | number`, необязательный, по умолчанию значение токена `--basys-form-label-gap`) — промежуток между подписью и элементом ввода. Задаётся так же, как `labelWidth`: CSS-длина или число пикселей; пустое значение означает промежуток по умолчанию. Учитывается только в режиме фиксированной ширины — в колоночной раскладке отступы задаёт PrimeFlex.
- **labelCols** (`number`, необязательный, по умолчанию `4`) — количество колонок PrimeFlex-сетки (от 0 до 12), отводимых под подпись; используется только в режиме `labelWidth="cols"`. При значении `0` подпись не рендерится в любом режиме — выводится только содержимое слота (в режиме фиксированной ширины колонка подписи при этом убирается из сетки).
- **labelFor** (`string`, необязательный) — значение HTML-атрибута `for` подписи; идентификатор связанного элемента ввода.
- **id** (`string`, необязательный) — идентификатор корневого DOM-элемента подписи (`<label>`).
- **style** (`Record<string, any>`, необязательный) — inline-стиль подписи. Объединяется со стилем выравнивания, формируемым из `labelAlign`.
- **cssClass** (`string`, необязательный) — дополнительный CSS-класс подписи. К подписи всегда добавляется базовый класс `bs-label` (а в колоночной раскладке ещё и `col-{labelCols}`); значение `cssClass` мержится с ними.
- **required** (`boolean`, необязательный, по умолчанию `false`) — при значении `true` к подписи добавляется CSS-класс `bs-required` (визуальный признак обязательного поля).
- **labelAlign** (`'left' | 'right' | 'center'`, необязательный, по умолчанию `'left'`) — горизонтальное выравнивание текста подписи внутри её колонки. В режиме фиксированной ширины реализуется через `text-align`, в колоночной раскладке — через исторически сложившееся `justify-content`:

| Значение `labelAlign` | `text-align` (фикс. ширина) | `justify-content` (колонки) |
| --------------------- | --------------------------- | --------------------------- |
| `left`                | `left`                      | `flex-start`                |
| `center`              | `center`                    | `center`                    |
| `right`               | `right`                     | `flex-end`                  |

Элемент ввода передаётся в компонент через слот по умолчанию.

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
| `labelWidth`    | `default`             |
| `columnGap`     | пустая строка         |
| `labelAlign`    | `left`                |

Значения `id`, `style`, `required` и собственный `cssClass` по умолчанию не заполняются — их следует задавать вручную при необходимости. Сам элемент ввода (`pv-input-text`, `pv-input-number`, `pv-calendar`, `pv-checkbox`, `bs-object-reference-select` и т. п.) добавляется как дочерний элемент `bs-form-field` и автоматически попадает в слот по умолчанию.

## Примеры использования

### Использование в шаблоне (программируемый компонент)

Поле обязательного ввода с подписью и текстовым полем (ширина подписи по умолчанию):

```html
<BsFormFieldComponent text="Тема"
                      labelFor="topic-input"
                      :required="true">
  <InputText id="topic-input" v-model="topic" size="small" class="w-full" />
</BsFormFieldComponent>
```

Поле с узкой подписью и увеличенным промежутком (ширина задаётся явно):

```html
<BsFormFieldComponent text="Тема"
                      labelFor="topic-input"
                      labelWidth="6rem"
                      columnGap="1rem">
  <InputText id="topic-input" v-model="topic" size="small" class="w-full" />
</BsFormFieldComponent>
```

Поле со старой колоночной раскладкой (подпись — 3 колонки из 12):

```html
<BsFormFieldComponent text="Тема"
                      labelFor="topic-input"
                      labelWidth="cols"
                      :labelCols="3">
  <InputText id="topic-input" v-model="topic" size="small" class="w-full" />
</BsFormFieldComponent>
```

### Использование в форме-конструкторе (JSON)

Обычное поле выбора объекта метаданных (взято из формы редактирования объекта `operation.task`, поле «Основание»):

```json
{
  "id": "bs-form-field-iSizb4",
  "dataUid": "3714ed97-925b-a834-ca63-46efe62c3bbf",
  "componentName": "bs-form-field",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "text", "value": "Основание" },
    { "name": "labelCols", "value": "4" }
  ],
  "items": [
    {
      "id": "bs-object-reference-select-nTEgyr",
      "componentName": "bs-object-reference-select",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "vModel", "value": "$h.base_task" }
      ],
      "items": []
    }
  ]
}
```

Обязательное поле с пометкой звёздочкой (поле «Автор» там же):

```json
{
  "id": "bs-form-field-wAf07A",
  "dataUid": "0d9fc741-1867-e208-b0db-4d1bc02b44d2",
  "componentName": "bs-form-field",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "text", "value": "Автор" },
    { "name": "labelCols", "value": "4" },
    { "name": "required", "value": "" }
  ],
  "items": [
    {
      "id": "bs-object-reference-select-author",
      "componentName": "bs-object-reference-select",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "vModel", "value": "$h.author" }
      ],
      "items": []
    }
  ]
}
```

Широкое поле для длинного текста (узкая подпись, `labelWidth="5rem"` — поле «Описание»). В ранее сохранённых формах в этом месте встречается `labelCols="1"`: такие формы продолжают открываться, но ширину подписи теперь задаёт `labelWidth`, поэтому для узкой подписи свойство нужно добавить (или указать `labelWidth="cols"`, чтобы вернуть колоночную раскладку):

```json
{
  "id": "bs-form-field-8gON6Q",
  "dataUid": "0ebdd22a-3383-1b6f-95db-97ea72c38796",
  "componentName": "bs-form-field",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "text", "value": "Описание" },
    { "name": "labelWidth", "value": "5rem" }
  ],
  "items": [
    {
      "id": "pv-input-textarea-gfL3h1",
      "componentName": "pv-input-textarea",
      "cssClass": "w-full",
      "style": "",
      "properties": [
        { "name": "vModel", "value": "$h.description" },
        { "name": "rows", "value": "5" },
        { "name": "size", "value": "small" }
      ],
      "items": []
    }
  ]
}
```

Поле с выравниванием подписи по правому краю и условным рендерингом (взято из формы редактирования объекта `operation.проект_договора`):

```json
{
  "id": "bs-form-field-osnDog",
  "dataUid": "523d6000-6839-7e1b-84c6-eb33b1eccb64",
  "componentName": "bs-form-field",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "text", "value": "Основной договор" },
    { "name": "labelCols", "value": "4" },
    { "name": "v-if", "value": "$h.договор_допсоглашение == 'доп_соглашение'" },
    { "name": "labelAlign", "value": "right" }
  ],
  "items": [
    {
      "id": "bs-object-reference-select-ZpoFnx",
      "componentName": "bs-object-reference-select",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "vModel", "value": "$h.основной_договор" },
        { "name": ":text", "value": "$h.основной_договор_display" },
        { "name": "size", "value": "small" }
      ],
      "items": []
    }
  ]
}
```

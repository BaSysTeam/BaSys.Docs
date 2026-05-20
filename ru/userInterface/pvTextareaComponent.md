# Компонент `Textarea` / `pv-input-textarea`

Многострочное текстовое поле ввода из библиотеки PrimeVue. В **BaSYS** применяется для длинных текстовых полей форм — описаний, комментариев, и других многострочных значений. В исходниках PrimeVue компонент импортируется как `Textarea` (`import Textarea from 'primevue/textarea'`); в реестре конструктора форм **BaSYS** ему присвоено kebab-имя `pv-input-textarea` — с префиксом `input-`, общим для всех ввода-полей (`pv-input-text`, `pv-input-number`, `pv-input-switch`), поэтому в PrimeVue имя короче, а в JSON-формах **BaSYS** длиннее. Полный список props, событий и слотов приведён в [PrimeVue 3 — Textarea](https://v3.primevue.org/textarea/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-input-textarea` (реестр `ConstructorComponents`).
- В программируемых компонентах — под именем `Textarea` (без регистрации, см. перечень в [Программируемые компоненты](programmableComponents.md)).

## Часто используемые в BaSYS props

- **vModel** (`string`) — двусторонняя привязка значения. В JSON-формах задаётся именем `vModel` со ссылкой на переменную формы (`$h.<имя>`) или поле текущей записи (`$r.<имя>`). В программируемом компоненте используется обычная директива `v-model`.
- **rows** (`number`) — число видимых строк. По умолчанию в конструкторе форм выставляется `5`; на компактных формах часто понижают до `3`.
- **cols** (`number`) — ширина в символах. В **BaSYS** обычно не задаётся: ширина управляется CSS-классом (`w-full`) и сеткой родительской колонки.
- **size** (`'small' | 'large'`) — размер поля. В типовых формах **BaSYS** используется `small`.
- **autoResize** (`boolean`) — авторастягивание по высоте вместо появления скроллбара. Удобно для полей с переменной длиной текста.
- **autocomplete** (`string`) — стандартный HTML-атрибут автозаполнения; в формах **BaSYS** обычно ставится `off`.
- **disabled** (`boolean`) — блокировка поля. В JSON-формах удобно задавать через привязку `:disabled` к выражению на основе помощников формы (например, `$h.edit_disabled`, `!$h.is_user_author`).
- **invalid** (`boolean`) — индикация ошибки валидации.
- **placeholder** (`string`) — подсказка в пустом поле.

Дополнительный CSS-класс корневого элемента задаётся полем `CssClass` (стандартно — `w-full` для растягивания по ширине колонки сетки).

## Свойства по умолчанию в конструкторе форм

При добавлении компонента через визуальный конструктор форм (`FormElementBuilder.createTextArea`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию |
| --------------- | --------------------- |
| `ComponentName` | `pv-input-textarea`   |
| `CssClass`      | `w-full`              |
| `Style`         | пусто                 |
| `vModel`        | пустая строка         |
| `size`          | `small`               |
| `rows`          | `5`                   |
| `autocomplete`  | `off`                 |

Остальные props (`cols`, `autoResize`, `placeholder`, `disabled`, `invalid` и т. п.) по умолчанию не заполняются — их следует задавать вручную при необходимости.

## Примеры использования

### В программируемом компоненте (template)

Многострочное поле описания задачи в диалоге создания (`operation.task.form.create_task.vue`). Привязка через обычный `v-model` к локальной переменной формы, ширина — по колонке сетки:

```html
<Textarea id="description"
          v-model="description"
          autocomplete="off"
          size="small"
          rows="5"
          class="w-full" />
```

### В форме-конструкторе (JSON)

Поле описания задачи в форме редактирования (`operation.task.form.edit_PM2RXN.json`). Привязка к переменной формы через `vModel`, блокировка по условию через `:disabled`:

```json
{
  "Id": "pv-input-textarea-gfL3h1",
  "ComponentName": "pv-input-textarea",
  "CssClass": "w-full",
  "Style": "",
  "Properties": [
    { "Name": "vModel", "Value": "$h.description" },
    { "Name": "size", "Value": "small" },
    { "Name": "rows", "Value": "3" },
    { "Name": "autocomplete", "Value": "off" },
    { "Name": ":disabled", "Value": "!$h.is_user_author" }
  ],
  "Items": []
}
```

Поле «Предмет договора» в форме редактирования проекта договора (`operation.проект_договора.form.edit_72xnlc.json`). Компактный вариант с `rows: 3` и блокировкой через общий флаг `$h.edit_disabled`:

```json
{
  "Id": "pv-input-textarea-h73byU",
  "ComponentName": "pv-input-textarea",
  "CssClass": "w-full",
  "Style": "",
  "Properties": [
    { "Name": "vModel", "Value": "$h.предмет_договора" },
    { "Name": "size", "Value": "small" },
    { "Name": "rows", "Value": "3" },
    { "Name": "autocomplete", "Value": "off" },
    { "Name": ":disabled", "Value": "$h.edit_disabled" }
  ],
  "Items": []
}
```

Поле «Комментарий» в той же форме — типовой вариант с `rows: 5` (как по умолчанию в конструкторе):

```json
{
  "Id": "pv-input-textarea-IqZkUl",
  "ComponentName": "pv-input-textarea",
  "CssClass": "w-full",
  "Style": "",
  "Properties": [
    { "Name": "vModel", "Value": "$h.комментарий" },
    { "Name": "size", "Value": "small" },
    { "Name": "rows", "Value": "5" },
    { "Name": "autocomplete", "Value": "off" },
    { "Name": ":disabled", "Value": "$h.edit_disabled" }
  ],
  "Items": []
}
```

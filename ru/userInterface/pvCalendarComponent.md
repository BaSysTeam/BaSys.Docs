# Компонент `Calendar` / `pv-calendar`

Поле ввода даты и/или времени из библиотеки PrimeVue (известно также как DatePicker). В **BaSYS** применяется для ввода полей даты и даты-времени в шапке документов и справочников (как через автоматический подбор типа поля при сборке формы редактирования, так и при ручной разметке), а также в программируемых компонентах — например, в диалогах создания записей. Полный список props, событий и слотов приведён в [PrimeVue 3 — Calendar](https://v3.primevue.org/calendar/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-calendar` (реестр `ConstructorComponents`).
- В программируемых компонентах — под именем `Calendar` (без регистрации, см. перечень в [Программируемые компоненты](programmableComponents.md)).

## Часто используемые в BaSYS props

- **vModel** (`Date | Date[] | null`) — двусторонняя привязка к значению. В JSON-формах задаётся выражением пути вида `$h.<имя>` (поле шапки) или `$r.<имя>` (поле текущей строки таблицы) — например, `$h.date`, `$h.deadline`, `$h.дата_уволен`. В программируемых компонентах используется обычный `v-model="<имя_свойства_data>"`.
- **dateFormat** (`string`) — формат отображаемой даты. В **BaSYS** по умолчанию используется `dd.mm.yy` (полные правила формата — в [PrimeVue 3 — Calendar / Format](https://v3.primevue.org/calendar/#format)).
- **showTime** (`boolean`) — отображать поле выбора времени рядом с датой. Для поля только-дата задаётся `false` или просто опускается. Для дата-время удобно задавать через привязку `:showTime="true"` (см. JSON-примеры ниже).
- **timeOnly** (`boolean`) — отображать только выбор времени без даты. В конструкторе форм по умолчанию выставляется `:timeOnly="false"`.
- **showIcon** (`boolean`) — отображать иконку календаря рядом с полем. В типовых формах **BaSYS** включена.
- **iconDisplay** (`'input' | 'button'`) — место отображения иконки. В **BaSYS** используется значение `input` (иконка отрисовывается внутри поля).
- **showButtonBar** (`boolean`) — отображать в подвале выпадающей панели кнопки «Сегодня» и «Очистить». В типовых формах **BaSYS** включена.
- **size** (`'small' | 'large'`) — размер поля. В типовых формах **BaSYS** — `small`.
- **disabled** (`boolean`) — блокировка ввода. В JSON-формах удобно задавать привязкой `:disabled` к выражению на основе `formState` (например, `!$h.is_user_author`).
- **hideOnDateTimeSelect** (`boolean`) — закрывать выпадающую панель сразу после выбора значения в режиме `showTime`. Применяется в программируемых компонентах.

Для размещения поля в слоте родителя (например, в слоте `start`/`end` `pv-toolbar`) задаётся свойство `slot`.

## Свойства по умолчанию в конструкторе форм

При добавлении компонента через визуальный конструктор форм (`FormElementBuilder.createCalendar`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию |
| --------------- | --------------------- |
| `ComponentName` | `pv-calendar`         |
| `CssClass`      | `w-full`              |
| `Style`         | пусто                 |
| `vModel`        | пустая строка         |
| `showIcon`      | пустая строка         |
| `showTime`      | пустая строка         |
| `showButtonBar` | пустая строка         |
| `:timeOnly`     | `false`               |
| `iconDisplay`   | `input`               |
| `size`          | `small`               |
| `dateFormat`    | `dd.mm.yy`            |

Также `pv-calendar` автоматически подставляется в формы редактирования, собираемые `EditFormBuilder`, для полей с типом данных «дата» (см. [Конструктор форм](formConstructor.md#быстрое-создание-формы-списка-и-формы-редактирования)).

## Примеры использования

### В программируемом компоненте (template)

Поле выбора даты-времени дедлайна в диалоге создания задачи (`operation.task.form.create_task.vue`). Используется `showTime`, `showIcon`, `iconDisplay="input"`, ширина растягивается до контейнера через `class="w-full"`, выпадающая панель закрывается сразу после выбора через `:hideOnDateTimeSelect="true"`:

```html
<Calendar id="deadline"
          v-model="deadline"
          size="small"
          showTime
          showIcon
          :hideOnDateTimeSelect="true"
          iconDisplay="input"
          class="w-full" />
```

### В форме-конструкторе (JSON)

Поле только-дата для шапки документа `штатное_расписание` (`operation.штатное_расписание.form.edit_SeUfKV.json`) — `vModel` привязан к `$h.дата_уволен`, выбор времени отключён через привязку `:showTime="false"`:

```json
{
  "id": "pv-calendar-y2hdJ3",
  "dataUid": "b6491015-ca52-5008-b0a4-2b01b50d379b",
  "componentName": "pv-calendar",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "vModel", "value": "$h.дата_уволен" },
    { "name": "size", "value": "small" },
    { "name": "showIcon", "value": "" },
    { "name": "showButtonBar", "value": "" },
    { "name": "iconDisplay", "value": "input" },
    { "name": ":showTime", "value": "false" },
    { "name": "dateFormat", "value": "dd.mm.yy" }
  ],
  "items": []
}
```

Поле даты-времени с условной блокировкой в форме редактирования задачи (`operation.task.form.edit_PM2RXN.json`) — `vModel` привязан к `$h.deadline`, выбор времени включён через `:showTime="true"`, ввод заблокирован, если текущий пользователь не автор задачи (`:disabled` со значением `!$h.is_user_author`):

```json
{
  "id": "pv-calendar-YKavgY",
  "dataUid": "28ae968a-973d-bd39-28fa-7ec91799c2ad",
  "componentName": "pv-calendar",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "vModel", "value": "$h.deadline" },
    { "name": "size", "value": "small" },
    { "name": "showIcon", "value": "" },
    { "name": "showButtonBar", "value": "" },
    { "name": "iconDisplay", "value": "input" },
    { "name": ":showTime", "value": "true" },
    { "name": "dateFormat", "value": "dd.mm.yy" },
    { "name": ":disabled", "value": "!$h.is_user_author" }
  ],
  "items": []
}
```

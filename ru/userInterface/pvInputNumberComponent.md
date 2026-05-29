# Компонент `InputNumber` / `pv-input-number`

Поле ввода числовых значений из библиотеки PrimeVue. В **BaSYS** применяется для редактирования сумм, ставок, рабочих часов, количественных и порядковых реквизитов формы (заработная плата, суммы договоров, контрольные суммы, плановые и фактические часы и т. п.). Полный список props, событий и слотов приведён в [PrimeVue 3 — InputNumber](https://v3.primevue.org/inputnumber/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-input-number` (реестр `ConstructorComponents`).
- В программируемых компонентах — под именем `InputNumber` (без регистрации, см. перечень в [Программируемые компоненты](programmableComponents.md)).

## Часто используемые в BaSYS props

- **vModel** (`number | null`) — двусторонняя привязка значения. В JSON-формах значение указывается выражением пути в `formState`, обычно с сокращениями `$h.<имя>` (поле шапки) или `$r.<имя>` (поле текущей строки таблицы); полные формы — `data.header.<имя>` и `data.currentRow.<имя>` (см. раздел [Сокращения путей](formConstructor.md#сокращения-путей) в `formConstructor.md`). В программируемых компонентах используется обычная директива `v-model`.
- **size** (`'small' | 'large'`) — размер поля. В типовых формах **BaSYS** используется `small`.
- **minFractionDigits** (`number`) — минимальное количество знаков после запятой. Типовые значения: `2` для денежных сумм и ставок, `1`/`2` для рабочих часов, `0` для целочисленных полей (порядковые номера и т. п.).
- **maxFractionDigits** (`number`) — максимальное количество знаков после запятой. Чаще всего совпадает с `minFractionDigits`, чтобы зафиксировать формат отображения.
- **min** (`number`) — минимально допустимое значение. Применяется, например, для запрета отрицательных часов (`:min="0"`).
- **useGrouping** (`boolean`) — включение/отключение разделителя разрядов. По умолчанию `true`; для идентификаторов, табельных и порядковых номеров отключается через `:useGrouping="false"`.
- **disabled** (`boolean`) — блокировка поля. В JSON удобно задавать привязкой `:disabled` к выражению на основе данных формы (`$h.edit_disabled`, `!$h.is_user_author` и т. п.); без выражения — пустая строка в обычном prop `disabled`.
- **invalid** (`boolean`) — состояние ошибки валидации. Используется в программируемых компонентах вместе с собственной проверкой (`:invalid="errors.<field>"`); альтернативно ошибочное состояние оформляется CSS-классом `p-invalid`.

## Свойства по умолчанию в конструкторе форм

При добавлении поля через визуальный конструктор форм (`FormElementBuilder.createInputNumber`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию |
| --------------- | --------------------- |
| `ComponentName` | `pv-input-number`     |
| `CssClass`      | `w-full`              |
| `Style`         | пусто                 |
| `vModel`        | пустая строка         |
| `size`          | `small`               |

Параметры `minFractionDigits`, `maxFractionDigits`, `min`, `useGrouping`, `disabled` и прочие по умолчанию не заполняются — их следует задавать вручную в зависимости от типа величины. При построении формы редактирования помощником `EditFormBuilder` числовые поля шапки и строк таблицы автоматически получают `vModel` вида `$h.<имя>` или `$r.<имя>`, а виртуальные поля — `disabled` (см. раздел [Быстрое создание формы списка и формы редактирования](formConstructor.md#быстрое-создание-формы-списка-и-формы-редактирования) в `formConstructor.md`).

## Примеры использования

### В программируемом компоненте (template)

Поле ввода планового количества часов в диалоге создания задачи (`operation.task.form.create_task.vue`). Поле занимает всю ширину колонки, не допускает отрицательных значений, отключает группировку разрядов и фиксирует один знак после запятой; состояние ошибки управляется собственной валидацией формы:

```html
<InputNumber id="hours_plan"
             v-model="hours_plan"
             v-tooltip="'Количество часов на выполнение задачи по плану'"
             :invalid="errors.hours_plan"
             :min="0"
             :useGrouping="false"
             :minFractionDigits="1"
             :maxFractionDigits="1"
             size="small"
             class="w-full" />
```

Узкое поле ввода фактических часов в строке смены статуса задачи (`operation.task.form.my_tasks_script.vue`). Ширина задаётся inline-стилем, ошибочное состояние оформляется CSS-классом `p-invalid`; при получении фокуса значение автоматически выделяется для быстрой замены:

```html
<InputNumber v-model="selectedTask.hours_fact"
             :style="{ width: '70px' }"
             :class="{ 'p-invalid': hoursFactValidationError }"
             size="small"
             v-tooltip.top="'Количество часов, факт'"
             :min="0"
             :minFractionDigits="1"
             :maxFractionDigits="1"
             @focus="$event.target.select()" />
```

### В форме-конструкторе (JSON)

Типовое денежное поле — сумма договора в форме редактирования проекта договора (`operation.проект_договора.form.edit_…json`). Привязано к полю шапки `сумма_договора`, ограничено двумя знаками после запятой и блокируется в режиме «только для чтения» через выражение `$h.edit_disabled`:

```json
{
  "id": "pv-input-number-3oSoC7",
  "componentName": "pv-input-number",
  "cssClass": "w-full",
  "style": "",
  "properties": [
    { "name": "vModel", "value": "$h.сумма_договора" },
    { "name": "size", "value": "small" },
    { "name": ":minFractionDigits", "value": "2" },
    { "name": ":maxFractionDigits", "value": "2" },
    { "name": ":disabled", "value": "$h.edit_disabled" }
  ],
  "items": []
}
```

Поле плановых часов в форме редактирования задачи (`operation.task.form.edit_…json`). Привязано к `$h.hours_plan`, использует один знак после запятой и блокируется для пользователей, не являющихся автором задачи:

```json
{
  "id": "pv-input-number-r7zY3W",
  "componentName": "pv-input-number",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "vModel", "value": "$h.hours_plan" },
    { "name": "size", "value": "small" },
    { "name": ":minFractionDigits", "value": "1" },
    { "name": ":maxFractionDigits", "value": "1" },
    { "name": ":disabled", "value": "!$h.is_user_author" }
  ],
  "items": []
}
```

Целочисленное поле — порядковый номер ручной визы в форме проекта договора (`operation.проект_договора.form.edit_…json`). Знаки после запятой принудительно отключены установкой `:minFractionDigits = 0` и `:maxFractionDigits = 0`:

```json
{
  "id": "pv-input-number-OaUzyl",
  "componentName": "pv-input-number",
  "cssClass": "w-full",
  "style": "",
  "properties": [
    { "name": "vModel", "value": "$h.visa_manual_order" },
    { "name": "size", "value": "small" },
    { "name": ":minFractionDigits", "value": "0" },
    { "name": ":maxFractionDigits", "value": "0" }
  ],
  "items": []
}
```

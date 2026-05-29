# Компонент `InputSwitch` / `pv-input-switch`

Стандартный переключатель из библиотеки PrimeVue, предназначенный для редактирования булева значения в виде «тумблера». В **BaSYS** применяется как альтернатива `pv-checkbox` для bool-полей шапки и табличных частей — выбор между ними при автоматической генерации формы редактирования определяется параметром `RenderSettings.ControlKindUid` поля (`PrimeVueSwitchInput` → `pv-input-switch`, иначе `pv-checkbox`; см. раздел «Быстрое создание формы списка и формы редактирования» в [Конструкторе форм](formConstructor.md)). Полный список props, событий и слотов приведён в [PrimeVue 3 — InputSwitch](https://v3.primevue.org/inputswitch/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-input-switch` (реестр `ConstructorComponents`).
- В программируемых компонентах — под именем `InputSwitch` (без регистрации, см. перечень в [Программируемые компоненты](programmableComponents.md)).

## Часто используемые в BaSYS props

- **vModel** / **v-model** (`boolean`) — двусторонняя привязка к bool-значению. В формах-конструкторах задаётся как `vModel` со значением пути `$h.<имя>` (поле шапки) или `$r.<имя>` (поле строки таблицы). В программируемых компонентах — обычный `v-model="..."`.
- **disabled** (`boolean`) — отключение переключателя. В JSON-формах удобно задавать привязкой `:disabled` к выражению на основе `formState` (например, `:disabled="$h.is_closed"`); в программируемых компонентах — `:disabled="..."`.
- **invalid** (`boolean`) — индикатор ошибки валидации, рисуется красной обводкой. Применяется в программируемых формах при ручной проверке значений.
- **inputId** (`string`) — `id` скрытого нативного `<input>`. Используется, чтобы связать переключатель с подписью `bs-label` / `bs-form-field` через `labelFor`.
- **@Change** — событие изменения значения. В формах-конструкторах значение — имя команды (см. раздел [Команды](formConstructor.md#команды) в `formConstructor.md`); в программируемых компонентах — обычный обработчик `@change="..."`.

## Свойства по умолчанию в конструкторе форм

При добавлении переключателя через визуальный конструктор форм (`FormElementBuilder.createInputSwitch`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию |
| --------------- | --------------------- |
| `ComponentName` | `pv-input-switch`     |
| `CssClass`      | пусто                 |
| `Style`         | пусто                 |
| `vModel`        | пустая строка         |

При автоматической генерации формы редактирования (`EditFormBuilder` + `FormElementFactory.CreateInputSwitch`) дополнительно проставляются `vModel` вида `$h.<имя_поля>` и `size: small`.

## Примеры использования

> В формах из `Experimental/Elevator/operation` готовых примеров с `pv-input-switch` нет — bool-поля по умолчанию рендерятся как `pv-checkbox`. Ниже приведены типовые шаблоны на основе значений по умолчанию из `FormElementBuilder` / `FormElementFactory` и того, как переключатель используется во внутренних формах платформы (`DataObjectHeaderFieldEditComponent.vue`).

### В программируемом компоненте (template)

Минимальная двусторонняя привязка к полю шапки с обработчиком изменения и блокировкой по другому полю:

```html
<InputSwitch :id="column.uid"
             v-model="item.header[column.name]"
             :disabled="column.disabled"
             @change="onChange(column.name)" />
```

### В форме-конструкторе (JSON)

Типовой переключатель, созданный конструктором форм для bool-поля шапки `is_active`. Привязан к модели через `vModel`:

```json
{
  "id": "pv-input-switch-3xQwLa",
  "componentName": "pv-input-switch",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "vModel", "value": "$h.is_active" },
    { "name": "size", "value": "small" }
  ],
  "items": []
}
```

Переключатель с подписью через `bs-form-field` и условной блокировкой: значение поля шапки `is_published` редактируется только пока документ не закрыт. `inputId` связывает переключатель с подписью поля формы, `@Change` запускает пользовательскую команду пересчёта зависимых полей:

```json
{
  "id": "pv-input-switch-K7uMeT",
  "componentName": "pv-input-switch",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "vModel", "value": "$h.is_published" },
    { "name": "inputId", "value": "fld-is-published" },
    { "name": ":disabled", "value": "$h.is_closed" },
    { "name": "@Change", "value": "recalc_publication" }
  ],
  "items": []
}
```

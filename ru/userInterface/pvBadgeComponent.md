# Компонент `Badge` / `pv-badge`

Индикатор **Badge** из библиотеки PrimeVue, отображающий короткое числовое или текстовое значение рядом с другим элементом. В **BaSYS** применяется в основном как счётчик строк табличных частей в заголовках вкладок и как индикатор количества/состояния в шапках групп и тулбарах программируемых компонентов. Полный список props, событий и слотов приведён в [PrimeVue 3 — Badge](https://v3.primevue.org/badge/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-badge` (реестр `ConstructorComponents`).
- В программируемых компонентах — под именем `Badge` (без регистрации, см. перечень в [Программируемые компоненты](programmableComponents.md)).

## Часто используемые в BaSYS props

- **value** (`string | number`) — отображаемое значение. В типовых формах **BaSYS** задаётся привязкой `:value` к выражению, считающему количество строк табличной части: `$t.table_1?.count()`, `data.tables.<имя>?.count()`, или к произвольному реактивному свойству программируемого компонента.
- **severity** (`'primary' | 'secondary' | 'success' | 'info' | 'warning' | 'danger' | 'contrast'`) — семантический цвет значка. По соглашению **BaSYS**: `primary` — нейтральный счётчик (по умолчанию в конструкторе форм), `info` — информационный счётчик (например, количество непрочитанных сообщений), `success` — положительный итог, `warning` — требующее внимания, `danger` — критическое количество или ошибки.
- **size** (`'large' | 'xlarge'`) — размер значка (см. [PrimeVue 3 — Badge / Size](https://v3.primevue.org/badge/#size)). По умолчанию значок отображается в обычном размере; в `FormElementBuilder` поле `size` создаётся со значением `small`, которое в типовых формах **BaSYS** остаётся неизменным.
- **slot** (`string`) — имя слота родительского компонента, в который вставляется значок. В **BaSYS** обычно используется в связке с `pv-toolbar` (слот `end`), чтобы значок выводился справа от заголовка табличной части.

Привязка к данным выполняется через стандартный для **BaSYS** префикс `:` в имени свойства (`:value`). Обработчиков событий компонент не имеет.

## Свойства по умолчанию в конструкторе форм

При добавлении значка через визуальный конструктор форм (`FormElementBuilder.createBadge`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию                                                          |
| --------------- | ------------------------------------------------------------------------------ |
| `ComponentName` | `pv-badge`                                                                     |
| `CssClass`      | пусто                                                                          |
| `Style`         | пусто                                                                          |
| `value`         | пустая строка                                                                  |
| `severity`      | `primary`                                                                      |
| `size`          | `small`                                                                        |
| `slot`          | не задаётся; добавляется, если значок размещён в слоте родителя (например, `end` панели инструментов) |

## Примеры использования

### В программируемом компоненте (template)

Группа значков-счётчиков в шапке формы списка задач (`operation.task.form.my_tasks_script.vue`). Каждый значок вложен в `Tag` рядом с подписью и привязан к реактивному свойству компонента; цвет задаётся через `severity`:

```html
<Tag class="mt-2" severity="secondary">
  <span>Задачи мне (новые)</span>
  <Badge :value="myTasksCount" class="ml-1" severity="danger" />
</Tag>

<Tag class="ml-2 mt-2" severity="secondary">
  <span>Сообщений в "Задачах мне"</span>
  <Badge :value="myTasksUnreadMessageCount" class="ml-1" severity="info" />
</Tag>

<Tag class="ml-2 mt-2" severity="secondary">
  <span>Выполненные задачи</span>
  <Badge :value="tasksFromMeCompletedCount" class="ml-1" severity="success" />
</Tag>
```

### В форме-конструкторе (JSON)

Типовой счётчик строк табличной части в слоте `end` тулбара вкладки. Значение пересчитывается выражением `$t.table_2?.count()` (`operation.учет_раб_времени_цех.form.edit_1I7HDt.json`):

```json
{
  "id": "pv-badge-DjS4sm",
  "componentName": "pv-badge",
  "cssClass": "ml-1",
  "style": "",
  "properties": [
    { "name": ":value", "value": "$t.table_2?.count()" },
    { "name": "severity", "value": "primary" },
    { "name": "size", "value": "small" },
    { "name": "slot", "value": "end" }
  ],
  "items": []
}
```

Тот же сценарий с привязкой к табличной части по явному имени через `data.tables.<имя>` и более крупным отступом (`operation.монтаж_площадка.form.edit_KgWjmu.json` и `operation.заказ_поставщик_мтс.form.edit_IfQWXx.json`):

```json
{
  "id": "pv-badge-gQIbPu",
  "componentName": "pv-badge",
  "cssClass": "ml-3",
  "style": "",
  "properties": [
    { "name": ":value", "value": "data.tables.время?.count()" },
    { "name": "severity", "value": "primary" },
    { "name": "size", "value": "small" },
    { "name": "slot", "value": "end" }
  ],
  "items": []
}
```

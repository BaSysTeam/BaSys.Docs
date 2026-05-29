# Компонент `Button` / `pv-button`

Стандартная кнопка из библиотеки PrimeVue. В **BaSYS** применяется для запуска команд из тулбаров, форм списков и редактирования (стандартные «Сохранить», «Сохранить&Закрыть», «Вернуться», «Добавить», «Редактировать», «Удалить» и т. п.), а также для кастомных действий в программируемых компонентах. Полный список props, событий и слотов приведён в [PrimeVue 3 — Button](https://v3.primevue.org/button/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-button` (реестр `ConstructorComponents`).
- В программируемых компонентах — под именем `Button` (без регистрации, см. перечень в [Программируемые компоненты](programmableComponents.md)).

## Часто используемые в BaSYS props

- **label** (`string`) — подпись на кнопке. Допускается пустая строка, если кнопка иконочная.
- **icon** (`string`) — CSS-класс иконки из набора [PrimeIcons](https://primevue.org/icons/) (`pi pi-save`, `pi pi-plus`, `pi pi-times`, `pi pi-refresh` и т. п.).
- **severity** (`'primary' | 'secondary' | 'success' | 'info' | 'warning' | 'help' | 'danger' | 'contrast'`) — семантический цвет кнопки. По соглашению **BaSYS**: `primary` — основное действие, `secondary` — нейтральное, `success` — подтверждение/завершение, `warning` — изменение состояния, `danger` — удаление и отмена.
- **size** (`'small' | 'large'`) — размер кнопки. В типовых формах **BaSYS** используется `small`.
- **outlined** (`boolean`) — вариант с рамкой без заливки. По умолчанию выставляется в конструкторе форм для типовых кнопок тулбара.
- **text** (`boolean`) — текстовая кнопка без фона и рамки. Часто используется в `pv-toolbar` (слоты `start`/`end`) и в программируемых компонентах.
- **loading** (`boolean`) — индикатор ожидания. В программируемых компонентах привязывается через `:loading="isWaiting"` для блокировки кнопки на время асинхронной операции.
- **disabled** (`boolean`) — отключение кнопки. В JSON-формах удобно задавать привязкой `:disabled` к выражению на основе `formState`.
- **@Click** — обработчик нажатия. В формах-конструкторах значение — имя команды (см. раздел [Команды](formConstructor.md#команды) в `formConstructor.md`), стандартные команды: `standard.save`, `standard.save_close`, `standard.return`, `standard.add`, `standard.edit`, `standard.delete`, `standard.copy`, `standard.refresh`, `standard.open_records`, `standard.open_files`, `standard.create_from` и др. В программируемых компонентах — обычный обработчик `@click="onClick"`.

Для размещения кнопки в слоте родителя (например, `start` или `end` панели `pv-toolbar`) в JSON задаётся свойство `slot` со значением имени слота.

## Свойства по умолчанию в конструкторе форм

При добавлении кнопки через визуальный конструктор форм (`FormElementBuilder.createButton`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию                                                          |
| --------------- | ------------------------------------------------------------------------------ |
| `ComponentName` | `pv-button`                                                                    |
| `CssClass`      | пусто                                                                          |
| `Style`         | пусто                                                                          |
| `label`         | пустая строка                                                                  |
| `icon`          | пустая строка                                                                  |
| `@Click`        | пустая строка                                                                  |
| `severity`      | `primary`                                                                      |
| `size`          | `small`                                                                        |
| `outlined`      | `outlined`                                                                     |
| `slot`          | не задаётся; добавляется, если кнопка размещена в слоте родителя (например, `start`/`end` панели инструментов) |

## Примеры использования

### В программируемом компоненте (template)

Пара кнопок «Отмена» / «Создать» внизу диалога создания задачи (`operation.task.form.create_task.vue`). На кнопке «Создать» через `:loading="isWaiting"` отображается индикатор ожидания на время серверного вызова:

```html
<Button label="Отмена"
        size="small"
        severity="secondary"
        outlined
        @click="onCancelClick" />
<Button label="Создать"
        size="small"
        class="ml-1"
        severity="primary"
        :loading="isWaiting"
        outlined
        @click="onCreateClick" />
```

Текстовые кнопки тулбара в слоте `end` (`operation.task.form.my_tasks_script.vue`) — иконочная кнопка с подсказкой через `v-tooltip` и кнопка-команда:

```html
<Button v-tooltip="'Очистить фильтры'"
        size="small"
        severity="secondary"
        icon="pi pi-filter-slash"
        class="ml-1"
        text
        @click="onMyTaskClearFiltersClick" />
<Button label="Обновить"
        size="small"
        severity="primary"
        icon="pi pi-refresh"
        class="ml-1"
        text
        @click="onMyTaskRefreshClick" />
```

### В форме-конструкторе (JSON)

Типовая кнопка «Добавить» внутри группы кнопок (`pv-button-group`) формы списка задач (`operation.task.form.list_…json`). Привязана к стандартной команде `standard.add`:

```json
{
  "id": "pv-button-KLeHPa",
  "componentName": "pv-button",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "label", "value": "Добавить" },
    { "name": "icon", "value": "pi pi-plus" },
    { "name": "@Click", "value": "standard.add" },
    { "name": "severity", "value": "primary" },
    { "name": "size", "value": "small" },
    { "name": "outlined", "value": "outlined" }
  ],
  "items": []
}
```

Иконочная кнопка вне группы — стоит в строке кнопок формы списка с отступом `ml-1`, без подписи, с командой `standard.create_from`:

```json
{
  "id": "pv-button-0rpJeY",
  "componentName": "pv-button",
  "cssClass": "ml-1",
  "style": "",
  "properties": [
    { "name": "label", "value": "" },
    { "name": "icon", "value": "pi pi-file-import" },
    { "name": "@Click", "value": "standard.create_from" },
    { "name": "severity", "value": "primary" },
    { "name": "size", "value": "small" },
    { "name": "outlined", "value": "outlined" }
  ],
  "items": []
}
```

Кнопка в слоте `end` панели инструментов с условным рендерингом через `vIf` — выводится в табличной части сообщений и привязана к пользовательской команде `mark_messages`:

```json
{
  "id": "pv-button-UacQEU",
  "componentName": "pv-button",
  "cssClass": "",
  "style": "",
  "properties": [
    { "name": "label", "value": "Сообщения мной прочитаны" },
    { "name": "icon", "value": "pi pi-check" },
    { "name": "@Click", "value": "mark_messages" },
    { "name": "severity", "value": "danger" },
    { "name": "size", "value": "small" },
    { "name": "slot", "value": "end" },
    { "name": "vIf", "value": "$h.is_task_user" }
  ],
  "items": []
}
```

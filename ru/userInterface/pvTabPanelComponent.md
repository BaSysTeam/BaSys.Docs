# Компонент `TabPanel` / `pv-tab-panel`

Отдельная вкладка из библиотеки PrimeVue. В **BaSYS** применяется исключительно как дочерний элемент [`pv-tab-view`](pvTabViewComponent.md) — каждый `pv-tab-panel` соответствует одной вкладке набора и содержит её собственное наполнение (поля шапки, табличные части, тулбары и т. п.). Полный список props, событий и слотов приведён в [PrimeVue 3 — TabView](https://v3.primevue.org/tabview/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-tab-panel` (реестр `ConstructorComponents`).
- В программируемых компонентах — под именем `TabPanel` (без регистрации, см. перечень в [Программируемые компоненты](programmableComponents.md)).

## Часто используемые в BaSYS props

- **header** (`string`) — текст заголовка вкладки. Единственный обязательный по смыслу атрибут; именно по нему пользователь выбирает вкладку (`«Данные»`, `«Согласование»`, `«Задачи»`, `«Тарифы»`, `«Сообщения»` и т. п.). В типовых формах редактирования заполняется автоматически помощником `EditFormBuilder` по названию табличной части (см. [Конструктор форм](formConstructor.md#быстрое-создание-формы-списка-и-формы-редактирования)).
- **disabled** (`boolean`) — отключение вкладки (заголовок становится неактивным). В JSON-формах удобно задавать привязкой `:disabled` к выражению на основе `formState`.
- **:key** — стандартный механизм Vue для принудительного пересоздания вкладки при изменении значения. В JSON задаётся как свойство с именем `:key` и выражением в `Value` (например, `$h.статус`); используется, когда содержимое вкладки должно полностью перерисовываться при смене состояния документа.
- **:pt** — PassThrough-объект PrimeVue для тонкой стилизации заголовка и контента (`header`, `headerAction`, `content` и т. д.). В **BaSYS** применяется только в программируемых компонентах, когда стандартного оформления вкладок недостаточно.

Свойства `cssClass` и `style` к самому `pv-tab-panel` не применяются — внешний вид вкладок и активного заголовка задаётся либо классами на родительском `pv-tab-view`, либо `:pt` на самом `TabPanel`. Содержимое вкладки нужно оформлять стандартными обёртками **BaSYS** — `bs-row` / `bs-col` (см. [Конструктор форм](formConstructor.md#структура-верстки)).

## Свойства по умолчанию в конструкторе форм

При добавлении вкладки через визуальный конструктор форм (`FormElementBuilder.createTabPanel`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию                                                                |
| --------------- | ------------------------------------------------------------------------------------ |
| `ComponentName` | `pv-tab-panel`                                                                       |
| `CssClass`      | пусто                                                                                |
| `Style`         | пусто                                                                                |
| `header`        | `New Tab` (или название табличной части, если вкладка создаётся помощником `EditFormBuilder`) |

Вместе с самой вкладкой автоматически создаётся вложенная сетка `bs-row` (`CssClass = "grid"`) с одной колонкой `bs-col` (`CssClass = "col"`) — это место для размещения наполнения вкладки. При создании набора вкладок через `FormElementBuilder.createTabView` в `pv-tab-view` сразу добавляются две заготовки `pv-tab-panel` с заголовками `Tab 1` и `Tab 2`.

## Примеры использования

### В программируемом компоненте (template)

Две вкладки в форме списка задач (`operation.task.form.my_tasks_script.vue`). Внутри каждой вкладки — собственный `pv-toolbar` с командами и таблица `BsTableViewComponent`; через `:pt.headerAction` индивидуально стилизуется заголовок (цвет, размер шрифта, рамка) в зависимости от активности (`context.active`):

```html
<TabView :pt="tabViewPT">
  <TabPanel header="Задачи мне"
            :pt="{
              headerAction: ({ context }) => ({
                style: {
                  fontSize: '1.2rem',
                  fontWeight: '600',
                  padding: '0.8rem 1.5rem',
                  backgroundColor: context.active ? '#dc3545' : '#e0e0e0',
                  color: context.active ? '#ffffff' : '#333333',
                  borderColor: context.active ? '#dc3545' : '#e0e0e0'
                }
              })
            }">
    <!-- Toolbar и BsTableViewComponent для текущей вкладки -->
  </TabPanel>
  <TabPanel header="Задачи от меня"
            :pt="{ /* аналогичная стилизация */ }">
    <!-- Toolbar и BsTableViewComponent для другой вкладки -->
  </TabPanel>
</TabView>
```

### В форме-конструкторе (JSON)

Минимальный типовой случай — вкладка с табличной частью документа «Тарифы дорожные» (`operation.тарифы_дорожные.form.edit_QLItN2.json`). Свойство `DataUid` ссылается на UID табличной части в метаданных, наполнение размещается во вложенной сетке `bs-row` / `bs-col`:

```json
{
  "Id": "pv-tab-panel-YccvYN",
  "DataUid": "dc29dbd1-2a91-c867-3e20-4b9354d33c29",
  "ComponentName": "pv-tab-panel",
  "CssClass": "",
  "Style": "",
  "Properties": [
    { "Name": "header", "Value": "Тарифы" }
  ],
  "Items": [
    {
      "Id": "bs-row-EhFAQP",
      "ComponentName": "bs-row",
      "CssClass": "grid",
      "Style": "",
      "Properties": [],
      "Items": [ /* bs-col → pv-toolbar / bs-details-table / ... */ ]
    }
  ]
}
```

Вкладка с принудительной перерисовкой содержимого через `:key` и условным рендерингом дочерней строки — из формы редактирования проекта договора (`operation.проект_договора.form.edit_72xnlc.json`). Вкладка перерисовывается при изменении поля шапки `статус`, а её внутренняя строка отображается только при наличии флага `status_approve_remarks`:

```json
{
  "Id": "pv-tab-panel-hTtZOk",
  "DataUid": "7234d758-7b24-3f0f-c17e-e86d8e8f9f04",
  "ComponentName": "pv-tab-panel",
  "CssClass": "",
  "Style": "",
  "Properties": [
    { "Name": "header", "Value": "Согласование замечаний" },
    { "Name": ":key", "Value": "$h.статус" }
  ],
  "Items": [
    {
      "Id": "bs-row-qG6zJ0",
      "ComponentName": "bs-row",
      "CssClass": "grid",
      "Style": "",
      "Properties": [
        { "Name": "v-if", "Value": "$h.status_approve_remarks" }
      ],
      "Items": [ /* содержимое вкладки */ ]
    }
  ]
}
```

# Компонент `TabView` / `pv-tab-view`

Контейнер для группировки содержимого по вкладкам. В **BaSYS** применяется в формах редактирования с несколькими табличными частями (вкладка на каждую табличную часть) и в программируемых формах для разделения логически разных представлений в пределах одной страницы. Дочерними элементами выступают [`pv-tab-panel`](pvTabPanelComponent.md) — отдельные вкладки. Полный список props, событий и слотов приведён в [PrimeVue 3 — TabView](https://v3.primevue.org/tabview/).

## Доступность в BaSYS

- В конструкторе форм — под именем `pv-tab-view` (реестр `ConstructorComponents`). Допустимыми потомками являются `pv-tab-panel`.
- В программируемых компонентах — под именем `TabView` (без регистрации, см. перечень в [Программируемые компоненты](programmableComponents.md)). Используется в паре с `TabPanel`.

## Часто используемые в BaSYS props

В типовых формах-конструкторах **BaSYS** компонент применяется как «прозрачный» контейнер — собственные props на `pv-tab-view` обычно не задаются, всё оформление концентрируется на вкладках (`pv-tab-panel`). В программируемых компонентах из библиотеки PrimeVue чаще востребованы:

- **activeIndex** (`number`) — индекс активной вкладки. Для интерактивного переключения из скрипта используется двусторонняя привязка `v-model:activeIndex="active"`.
- **scrollable** (`boolean`) — включает горизонтальную прокрутку шапки вкладок со стрелками. Полезно, когда вкладок много и они не помещаются в строку.
- **lazy** (`boolean`) — отложенный рендер содержимого: вкладка монтируется только при первой активации (имеет смысл при тяжёлых таблицах внутри вкладок).
- **pt** (`object`) — passthrough-объект PrimeVue для тонкой настройки внешнего вида вкладок (например, изменение фона, отступов, рамок заголовка). В программируемых компонентах **BaSYS** это основной способ кастомизации внешнего вида табов, поскольку scoped-стили не поддерживают `:deep()` (см. раздел «Стили» в [Программируемые компоненты](programmableComponents.md)).
- **@TabChange** — событие смены активной вкладки. В форме-конструкторе задаётся как имя команды (см. раздел [Команды](formConstructor.md#команды) в `formConstructor.md`); в программируемом компоненте — обычный обработчик `@tab-change="onTabChange"`.

Свойства, относящиеся к одной вкладке (`header`, `disabled`, `header` через `#header`-слот), задаются на `pv-tab-panel`, а не на `pv-tab-view`.

## Свойства по умолчанию в конструкторе форм

При добавлении компонента через визуальный конструктор форм (`FormElementBuilder.createTabView`) создаётся `FormElement` со следующими значениями:

| Поле            | Значение по умолчанию                                                                |
| --------------- | ------------------------------------------------------------------------------------ |
| `ComponentName` | `pv-tab-view`                                                                        |
| `CssClass`      | пусто                                                                                |
| `Style`         | пусто                                                                                |
| `Properties`    | не задаются                                                                          |
| `Items`         | две дочерние вкладки `pv-tab-panel` с заголовками `Tab 1` и `Tab 2` (каждая содержит пустую сетку `bs-row` → `bs-col`) |

Дополнительно: при автосоздании формы редактирования помощник `EditFormBuilder` оборачивает табличные части документа/справочника в `pv-tab-view` и добавляет по одной `pv-tab-panel` на каждую табличную часть (см. раздел [Быстрое создание формы списка и формы редактирования](formConstructor.md#быстрое-создание-формы-списка-и-формы-редактирования)).

## Примеры использования

### В программируемом компоненте (template)

Кастомизированный `TabView` с двумя вкладками из формы «Мои задачи» (`operation.task.form.my_tasks_script.vue`). Внешний вид всей навигации задаётся объектом `tabViewPT` через `:pt`, а оформление активного/неактивного заголовка отдельной вкладки — собственным `:pt` на `TabPanel`:

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
    <!-- Toolbar + BsTableViewComponent -->
  </TabPanel>

  <TabPanel header="Задачи от меня" :pt="{ /* ... */ }">
    <!-- Toolbar + BsTableViewComponent -->
  </TabPanel>
</TabView>
```

Сам `tabViewPT` определён в `data()` компонента (фрагмент):

```javascript
data() {
  return {
    tabViewPT: {
      nav: { style: { borderBottom: 'none' } },
      navContainer: { style: { gap: '0.5rem' } },
      // ...
    }
  };
}
```

### В форме-конструкторе (JSON)

Минимальный случай — `pv-tab-view` как контейнер вкладок без собственных свойств, дочерние `pv-tab-panel` отвечают за заголовок и наполнение. Фрагмент формы редактирования задачи (`operation.task.form.edit_PM2RXN.json`) — внутри `bs-col` размещён `pv-tab-view` с вкладкой «Сообщения»:

```json
{
  "id": "pv-tab-view-B1fsLk",
  "componentName": "pv-tab-view",
  "cssClass": "",
  "style": "",
  "properties": [],
  "items": [
    {
      "id": "pv-tab-panel-BVwXOB",
      "componentName": "pv-tab-panel",
      "cssClass": "",
      "style": "",
      "properties": [
        { "name": "header", "value": "Сообщения" }
      ],
      "items": [
        // bs-row → bs-col → pv-toolbar + bs-details-table ...
      ]
    }
  ]
}
```

Вариант с растяжкой по ширине через класс PrimeFlex — `pv-tab-view` в форме проекта договора (`operation.проект_договора.form.edit_72xnlc.json`). Класс `w-full` задаётся на самом `pv-tab-view`, что удобно, когда родительская колонка шире содержимого первой вкладки:

```json
{
  "id": "pv-tab-view-UFNeID",
  "dataUid": "eded8edc-8d6b-4e2f-a068-52d401d1caaa",
  "componentName": "pv-tab-view",
  "cssClass": "w-full",
  "style": "",
  "properties": [],
  "items": [
    {
      "id": "pv-tab-panel-wwgBLp",
      "componentName": "pv-tab-panel",
      "properties": [
        { "name": "header", "value": "Обязательные поля" }
      ],
      "items": [ /* поля шапки */ ]
    }
    // ... другие pv-tab-panel
  ]
}
```

При необходимости управлять активной вкладкой из скрипта в `Properties` добавляется привязка `:activeIndex` к выражению на основе `formState`, а на событие смены — `@TabChange` с именем команды (см. раздел [Привязки данных и события](formConstructor.md#привязки-данных-и-события)).

# Программируемые компоненты

Программируемые компоненты используются для реализации сложных пользовательских интерфейсов, которые невозможно собрать с помощью автоматических форм или конструктора форм. Они разрабатываются на базе [Vue.js 3](https://vuejs.org/) с использованием **Options API** и библиотеки [PrimeVue 3](https://v3.primevue.org/).

Такие компоненты динамически встраиваются в интерфейс **BaSYS** и могут использовать как стандартные компоненты PrimeVue 3, так и внутренние компоненты системы. Полный перечень доступных компонентов и API приведён ниже.

Если требуется построить интерфейсы, не связанные с основной оболочкой **BaSYS** (другие фреймворки, библиотеки или дизайн-системы) — рекомендуется использовать механизм UI-плагинов (в разработке) либо вынести функциональность во внешнее frontend-приложение, взаимодействующее с **BaSYS** через API.

## Как это работает

Программируемый компонент сохраняется в метаданных как набор полей: `script`, `template`, `styles`, список дочерних компонентов (`childComponents`) и метаданные (`alias`, `name`, `title`, `formKind`, `metaObjectUid`, `isStylesGlobal`).

При открытии формы клиентское приложение получает эти поля и динамически собирает Vue-компонент:

1. Скрипт компилируется через `new Function('Vue', ...contextKeys, 'return ' + script)` — то есть запись в редакторе должна быть **выражением, возвращающим объект Options API** (`export default { ... }` в `.vue` ниже эквивалентно объекту, который попадает в редактор).
2. Шаблон компилируется в render-функцию через `@vue/compiler-dom` (`compile(template, { mode: 'function' })`).
3. К компоненту подмешивается стандартный набор зарегистрированных компонентов (PrimeVue + внутренние **BaSYS**) и пользовательские дочерние компоненты по их алиасам.
4. При наличии стилей они либо инжектируются в `<head>` глобально, либо «скоупятся» — все селекторы префиксуются классом `bs-dynamic-component-{alias}`, который автоматически добавляется к корневому элементу шаблона.
5. Компонент монтируется в обёртку `ProgrammableFormRenderer` и получает фиксированный набор props, описанный ниже.

Поскольку используется **Options API**, для разработки не требуются сборщики (WebPack, Vite и т. п.) и любой компонент может редактироваться «прямо в системе».

> Composition API (`<script setup>`, `defineComponent`-функции, `<script>` с импортами) в редакторе не поддерживается — скрипт должен возвращать обычный объект Options API.

## Создание программируемого компонента

Чтобы программируемые компоненты стали доступны для объекта метаданных, в виде метаданных, к которому относится этот объект, должен быть установлен флаг **Использует формы**. После этого в карточке объекта появляется раздел **Формы**.

Для добавления нового компонента откройте раздел **Формы**, в подменю **Добавить** выберите пункт **Программируемый компонент** — откроется форма редактирования.

Форма редактирования содержит вкладки:

- **Основные** — общая информация о компоненте. Особое внимание следует уделить полю **Имя** (alias): по этому имени компонент подключается как дочерний в других программируемых компонентах. Здесь же расположен флаг **Глобальные стили** (`IsStylesGlobal`).
- **Скрипт** — тело Options API (`data`, `props`, `computed`, `methods`, `watch`, `mounted`, `beforeMount` и т. д.).
- **Шаблон** — Vue-шаблон компонента (`<template>`).
- **Стили** — CSS компонента. По умолчанию стили scoped: см. раздел [Стили](#стили).
- **Дочерние компоненты** — позволяет подключать другие программируемые компоненты по их **Имени** для использования внутри текущего.

Новый компонент создаётся с минимальными работающими заготовками скрипта и шаблона. Их можно использовать как точку старта или удалить и начать с чистого листа.

## Props, передаваемые рендерером

Главный программируемый компонент монтируется через [`ProgrammableFormRenderer.vue`](https://vuejs.org/guide/built-ins/component.html) и автоматически получает следующие props:

| Prop | Тип | Назначение |
| --- | --- | --- |
| `title` | `string` | Заголовок формы из метаданных. |
| `uid` | `string` | Идентификатор формы. |
| `metaObjectUid` | `string` | Идентификатор объекта метаданных, к которому относится форма. |
| `formKind` | `number` | Вид формы (для программируемых форм — `0`). |

Чтобы использовать эти значения, объявите их в `props` компонента:

```javascript
export default {
  props: ['title', 'uid', 'metaObjectUid', 'formKind'],
  // ...
}
```

Когда форма открыта как диалог через `openDialog({ parameters: { ... } })`, в качестве props передаются также все ключи объекта `parameters`. Например, при вызове `openDialog({ formName: 'create_task', parameters: { author, current_task, regime } })` соответствующий компонент должен объявить `props: ['author', 'current_task', 'regime']`.

## Скрипт-контекст: глобальные функции

В скрипт компонента инжектируется набор функций и хелперов, доступных по имени **без импорта**:

| Имя | Назначение |
| --- | --- |
| `from(source)` | Создаёт `SelectQueryBuilder` для запроса к источнику метаданных (`from('operation.task').select([...]).where(...).query()`). |
| `runWorkflow(name, entryPoint, parameters)` | Запуск серверного workflow. `parameters` — массив объектов `{ name, dataType, value }`. |
| `openDialog(config)` | Открытие другой программируемой формы как модального диалога. См. ниже. |
| `isEmpty(value)` / `isNotEmpty(value)` | Проверка значения на «пустоту» с учётом семантики **BaSYS**. |
| `iif(cond, a, b)` | Тернарный аналог. |
| `ifs(...pairs, defaultValue)` | Цепочка условий. |
| `createTable(...)` | Создание объекта `DataTable` в коде. |
| `parseNumber(value)` | Преобразование значения в число с учётом локали. |
| `dateTimeNow()` | Текущая дата/время. |
| `dateDifference(a, b, unit)` | Разница между датами. |

Пример типового запроса (из `operation.task.form.tasks_tree.vue`):

```javascript
async getTasksByNumber(number) {
  return await from('operation.task')
    .select(['number', 'date', 'author', 'topic', 'status', 'base_task', 'responsible'])
    .getDisplays()
    .where('opr_task.number = @number')
    .parameter('number', number, 11)
    .query();
}
```

### Открытие формы как диалога

Функция `openDialog` принимает конфигурацию следующего вида:

```javascript
openDialog({
  kind: 'operation',
  name: 'task',
  formName: 'create_task',
  title: 'Новая задача',
  width: '40rem',
  parameters: {
    author: this.currentUserId,
    current_task: this.tasksFromMeCurrentRow,
    regime: 'new'
  },
  onClose: async (result) => {
    this.onTasksFromMeRefreshClick();
  }
});
```

| Поле | Назначение |
| --- | --- |
| `kind`, `name` | Вид и имя объекта метаданных, к которому привязана форма. |
| `formName` | Имя (alias) программируемой формы, которую нужно открыть. |
| `title`, `width` | Параметры окна диалога. |
| `parameters` | Объект, поля которого попадают в `props` открываемого компонента. |
| `onClose(result)` | Колбэк, вызываемый при закрытии диалога (компонент закрывает себя через `this.$emit('close')`). |

## Доступ к сервисам через `inject`

Часть инфраструктурных объектов предоставляется родительской формой и доступна программируемому компоненту через `inject`:

```javascript
export default {
  inject: ['axios', 'DataTable', 'TableViewColumnViewModel',
           'FilterItem', 'FilterSettingsItem', 'userSettings'],
  // ...
}
```

| Ключ | Описание |
| --- | --- |
| `axios` | Преднастроенный экземпляр [axios](https://axios-http.com/) для прямых HTTP-вызовов к API **BaSYS**. |
| `DataTable` | Класс `DataTable` из `@basysteam/basys-fx`. |
| `TableViewColumnViewModel` | Конструктор колонок для `BsTableViewComponent`. |
| `FilterItem`, `FilterSettingsItem` | Модели фильтров отчётов. |
| `userSettings` | Текущие настройки пользователя (в т. ч. `userName`). |

## Доступные компоненты

Внутри программируемого компонента следующие компоненты можно использовать без регистрации.

### Компоненты PrimeVue 3

| Название            | Имя                                                              | Назначение                                                       |
| ------------------- | ---------------------------------------------------------------- | ---------------------------------------------------------------- |
| Метка               | [`Badge`](https://v3.primevue.org/badge/)                        | Бейдж (счётчик, статус).                                         |
| Кнопка              | [`Button`](https://v3.primevue.org/button/)                      | Кнопка.                                                          |
| Поле даты           | [`Calendar`](https://v3.primevue.org/calendar/)                  | Поле ввода даты/времени.                                         |
| Карточка            | [`Card`](https://v3.primevue.org/card/)                          | Контейнер-карточка.                                              |
| График              | [`Chart`](https://v3.primevue.org/chart/)                        | Графики и диаграммы.                                             |
| Колонка             | [`Column`](https://v3.primevue.org/column/)                      | Колонка таблицы `DataTable`.                                     |
| Таблица             | [`DataTable`](https://v3.primevue.org/datatable/)                | Таблица с фильтрами, сортировкой и постраничной загрузкой.       |
| Диалог              | [`Dialog`](https://v3.primevue.org/dialog/)                      | Модальный диалог.                                                |
| Разделитель         | [`Divider`](https://v3.primevue.org/divider/)                    | Горизонтальный/вертикальный разделитель.                         |
| Выпадающий список   | [`Dropdown`](https://v3.primevue.org/dropdown/)                  | Выпадающий список.                                               |
| Числовое поле       | [`InputNumber`](https://v3.primevue.org/inputnumber/)            | Поле ввода числа.                                                |
| Переключатель       | [`InputSwitch`](https://v3.primevue.org/inputswitch/)            | Переключатель.                                                   |
| Текстовое поле      | [`InputText`](https://v3.primevue.org/inputtext/)                | Однострочное текстовое поле.                                     |
| Орг. структура      | [`OrganizationChart`](https://v3.primevue.org/organizationchart/) | Дерево организационной структуры.                               |
| Группа кнопок выбора | [`SelectButton`](https://v3.primevue.org/selectbutton/)         | Группа кнопок одиночного/множественного выбора.                  |
| Боковая панель      | [`Sidebar`](https://v3.primevue.org/sidebar/)                    | Выезжающая боковая панель.                                       |
| Закладки            | [`TabView`](https://v3.primevue.org/tabview/)                    | Набор вкладок.                                                   |
| Закладка            | [`TabPanel`](https://v3.primevue.org/tabview/)                   | Вкладка внутри `TabView`.                                        |
| Тег                 | [`Tag`](https://v3.primevue.org/tag/)                            | Метка-тег.                                                       |
| Многострочное поле  | [`Textarea`](https://v3.primevue.org/textarea/)                  | Многострочное текстовое поле.                                    |
| Панель инструментов | [`Toolbar`](https://v3.primevue.org/toolbar/)                    | Панель инструментов со слотами `start`/`end`.                    |
| Тройной флажок      | [`TriStateCheckbox`](https://v3.primevue.org/tristatecheckbox/)  | Флажок с тремя состояниями.                                      |

### Компоненты системы BaSYS

| Название                | Имя                                                  | Назначение                                                                                                          |
| ----------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Заголовок страницы      | [`BsViewTitle`](bsViewTitleComponent.md)             | Заголовок страницы с индикатором ожидания/модификации.                                                              |
| Табличное представление | `BsTableViewComponent`                               | Табличное представление с фильтрами, сортировкой и постраничной загрузкой; колонки описываются через `TableViewColumnViewModel`. |
| Текст                   | `BsTextComponent`                                    | Отображение текста с учётом семантики (severity, форматирование).                                                   |
| Выбор элемента          | `BsObjectReferenceSelect`                            | Выпадающий список со ссылкой на объект метаданных.                                                                  |
| Множественный выбор     | `BsObjectReferenceMultiSelect`                       | Выпадающий список с множественным выбором ссылок на объекты метаданных.                                             |
| Выбор периода           | `BsPeriodSelector`                                   | Выбор периода (для отчётов и фильтров).                                                                             |
| Строка фильтра          | `BsFilterRow`                                        | Строка фильтра для отчётов.                                                                                         |

> Для использования значка PrimeIcons (`pi pi-…`) дополнительные действия не требуются — иконки подключены глобально.

## Стили

Стили программируемого компонента инжектируются в `<head>` страницы при монтировании и удаляются при размонтировании. Реализованы два режима, переключаемые флагом **Глобальные стили** (`IsStylesGlobal`):

- **Scoped (по умолчанию)** — все селекторы автоматически префиксуются классом `bs-dynamic-component-{alias}`, который добавляется к корневому элементу шаблона. Это изолирует стили компонента от остальной части приложения.
- **Глобальные** — стили подключаются «как есть». Используйте только когда требуется повлиять на оформление за пределами компонента.

> Скоупинг выполняется простым префиксованием — конструкции `:deep()`, `:slotted()` и подобные из Vue SFC не поддерживаются. Если нужно стилизовать вложенные компоненты, используйте `pt` (passthrough) PrimeVue или глобальные стили.

## Дочерние компоненты

Любой ранее созданный программируемый компонент можно подключить как дочерний во вкладке **Дочерние компоненты**. Внутри родителя он становится доступен по своему **Имени** (`alias`), например:

```html
<my-task-card :task="row" @open="onOpen" />
```

Дочерние компоненты компилируются и загружаются вместе с родительским; они получают тот же `scriptContext` и `inject`. Глубина вложенности не ограничена, но циклические ссылки недопустимы.

## Использование внешних ресурсов

Если для решения задачи требуются внешние скрипты или стили, их можно подключить на уровне вида метаданных. При установленном флаге **Использует формы** в виде метаданных доступен раздел **Внешние ресурсы**, в котором можно указать ссылки на скрипты и стили (например, в CDN).

Эти ресурсы динамически инжектируются в `<head>` страницы при отрисовке форм соответствующего объекта в порядке, заданном в разделе. Подключённые ресурсы автоматически становятся доступными и в программируемых компонентах.

> Динамически подключаемые зависимости могут конфликтовать со скриптами и стилями самого приложения **BaSYS**. В таких случаях предпочтительнее использовать UI-плагины или внешние frontend-приложения.

## Полный пример

Ниже — упрощённая форма создания задачи (из объекта `operation.task`). Демонстрирует объявление props, доступ к `scriptContext` (`from`, `runWorkflow`), `data`/`watch`/`methods`, использование PrimeVue-компонентов и эмит события закрытия диалога.

```html
<script>
export default {
  props: ['author', 'current_task', 'regime'],
  data() {
    return {
      isWaiting: false,
      responsible: 0,
      responsibleOptions: [],
      topic: '',
      hours_plan: 0,
      errors: { responsible: false, topic: false, hours_plan: false }
    };
  },
  watch: {
    topic(value) {
      if (this.errors.topic && value && value.trim() !== '') {
        this.errors.topic = false;
      }
    }
  },
  methods: {
    onCancelClick() {
      this.$emit('close');
    },
    async onCreateClick() {
      // Валидация
      this.errors.responsible = !this.responsible;
      this.errors.topic = !this.topic || this.topic.trim() === '';
      this.errors.hours_plan = !this.hours_plan || this.hours_plan <= 0;
      if (this.errors.responsible || this.errors.topic || this.errors.hours_plan) return;

      const parameters = [
        { name: 'author', dataType: 'integer', value: this.author },
        { name: 'responsible', dataType: 'integer', value: this.responsible },
        { name: 'topic', dataType: 'string', value: this.topic },
        { name: 'hours_plan', dataType: 'number', value: this.hours_plan },
      ];

      this.isWaiting = true;
      try {
        await runWorkflow('create_task', 'operation_create', parameters);
        this.$emit('close');
      } finally {
        this.isWaiting = false;
      }
    }
  },
  async beforeMount() {
    const persons = await from('catalog.person')
      .select(['id as value', 'title as text'])
      .orderBy('title')
      .query();
    this.responsibleOptions = persons.rows;
  }
}
</script>

<template>
  <div class="grid align-items-center">
    <div class="col-12 md:col-3 font-bold">
      <label class="bs-required">Тема</label>
    </div>
    <div class="col-12 md:col-9">
      <InputText v-model="topic" :invalid="errors.topic" size="small" class="w-full" />
      <small v-if="errors.topic" class="p-error">Поле обязательно для заполнения</small>
    </div>
  </div>

  <div class="grid align-items-center">
    <div class="col-12 md:col-3 font-bold">
      <label class="bs-required">Ответственный</label>
    </div>
    <div class="col-12 md:col-9">
      <Dropdown v-model="responsible"
                :options="responsibleOptions"
                optionLabel="text" optionValue="value"
                :invalid="errors.responsible"
                filter size="small" class="w-full" />
    </div>
  </div>

  <div class="grid">
    <div class="col text-right">
      <Button label="Отмена" severity="secondary" outlined size="small" @click="onCancelClick" />
      <Button label="Создать" severity="primary" outlined size="small"
              class="ml-1" :loading="isWaiting" @click="onCreateClick" />
    </div>
  </div>
</template>
```

## Рекомендации

- Используйте программируемые компоненты только там, где автоматических форм и конструктора форм действительно недостаточно (см. [Введение](introduction.md)).
- Всю бизнес-логику, связанную с записью данных и проверкой инвариантов, размещайте в workflow на сервере; компонент должен оставаться «тонким».
- Для повторно используемых блоков выделяйте отдельные программируемые компоненты и подключайте их через раздел **Дочерние компоненты**.
- Не отключайте scoped-стили без необходимости — глобальные стили легко конфликтуют с оболочкой **BaSYS**.
- Если задача требует сложной разработки (нестандартный UI, отдельные сборщики, тестирование) — рассмотрите UI-плагины или внешние приложения.

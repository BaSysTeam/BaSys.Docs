---
name: write-primevue-component-doc
description: Generates a Russian documentation article for a standard PrimeVue 3 component as it is used in BaSYS (in the form constructor and/or programmable components). The article is short and links to the official PrimeVue 3 documentation instead of duplicating it; it covers BaSYS-relevant props, defaults from the form constructor and HTML/JSON usage examples. The article is saved under ru/userInterface/pvXxxComponent.md and cross-linked from formConstructor.md and programmableComponents.md. Use when the user gives a link to a PrimeVue 3 component page (https://v3.primevue.org/...) and asks to write an article / статью / документацию по компоненту PrimeVue.
---

# Документация по PrimeVue-компоненту в BaSYS

Скилл создаёт короткую статью в `ru/userInterface/` по стандартному компоненту [PrimeVue 3](https://v3.primevue.org/), описывающую, как он используется внутри **BaSYS** (в конструкторе форм и/или в программируемых компонентах), и обновляет ссылки в обзорных статьях.

Принципиальное отличие от скилла `write-component-doc` (для собственных `BsXxxComponent.vue`): статья не дублирует официальную документацию PrimeVue — она кратко объясняет назначение компонента, ссылается на оригинал и подробно описывает только то, что специфично для **BaSYS**.

## Когда применять

- Пользователь дал ссылку на страницу компонента PrimeVue 3 (`https://v3.primevue.org/<component>/`).
- Пользователь просит написать статью / документацию / описание по PrimeVue-компоненту в контексте **BaSYS**.
- Скилл применяется и при формулировках типа «оформи аналогичную статью», «добавь статью по компоненту», если речь идёт о компоненте из библиотеки PrimeVue 3.

## Источники контекста (читать обязательно)

Прежде чем писать статью, прочитать:

1. **Официальная страница PrimeVue 3** — URL, указанный пользователем. Цель — понять назначение, ключевые props, события и слоты. Содержимое страницы в статью **не копировать**.
2. **Реестры BaSYS-регистрации компонента** (определяют доступность в конструкторе форм и в программируемых формах):
   - `D:\_WORK\BaSysProject\REPO\BaSysApp\BaSys.Front\shared\src\enums\constructorComponents.ts` — перечисление `ConstructorComponents`. Если компонент есть здесь, его kebab-имя в конструкторе форм имеет префикс `pv-` (например, `pv-calendar`, `pv-input-text`).
   - `D:\_WORK\BaSysProject\REPO\BaSysApp\BaSys.Front\shared\src\models\forms\componentRegistry.ts` — таблица соответствия kebab-имени и реальной реализации.
   - `D:\_WORK\BaSysProject\REPO\BaSysApp\BaSys.Front\app\src\helpers\dynamicComponentLoader.ts` — список компонентов, доступных в программируемых компонентах без регистрации (PascalCase, например `Calendar`, `DataTable`, `Dropdown`). Смотреть массив `componentOptions.components` и список `import` сверху файла.

   На основании этих файлов зафиксировать для статьи:
   - **Где доступен**: только в конструкторе форм, только в программируемых компонентах или и там, и там.
   - **Как называется**: kebab-имя в конструкторе (`pv-xxx`) и/или PascalCase-имя в шаблоне программируемого компонента (`Xxx`).

3. **`formElementBuilder.ts`** — `D:\_WORK\BaSysProject\REPO\BaSysApp\BaSys.Front\shared\src\models\forms\formElementBuilder.ts`. Найти метод `createXxx()` для текущего компонента и зафиксировать значения по умолчанию (`componentName`, `cssClass`, `style`, набор `addProperty`). Если соответствующего метода нет — указать это в статье явно.

4. **Эталонные статьи** для стиля и оформления:
   - `ru/userInterface/bsTextComponent.md` — общий стиль (тон, таблицы, JSON-примеры).
   - `ru/userInterface/formConstructor.md` — раздел «Пример кастомизации компонента» (формат JSON-узла `FormElement`).

5. **Примеры использования** — папка `D:\_WORK\BaSysProject\Experimental\Elevator\operation`.
   - Для конструктора форм искать в `.json`-формах по `"ComponentName": "pv-xxx"` или подстроке `pv-xxx`.
   - Для программируемых компонентов искать в `.vue`-формах по PascalCase-тегу (`<Calendar`, `<DataTable` и т. п.).

   Использовать Grep по обоим именам параллельно, отбирать репрезентативные (разные) примеры: минимальный типовой случай, нестандартный (со стилем/привязкой), пример из программируемой формы (`.vue`).

## Структура статьи

Файл сохранять как `ru/userInterface/pvXxxComponent.md`, где `Xxx` — PascalCase-имя из PrimeVue (например, `pvCalendarComponent.md`, `pvButtonGroupComponent.md`, `pvDataTableComponent.md`, `pvInputTextComponent.md`). Если у компонента в **BaSYS** есть только kebab-имя без PascalCase-аналога (например, `pv-button-group`), всё равно использовать PascalCase из исходного PrimeVue-импорта (`ButtonGroup`) для имени файла.

Обязательные разделы (порядок и заголовки сохранять):

````markdown
# Компонент `<PascalCaseName>` / `<pv-kebab-name>`

<Краткое назначение в 1–3 предложениях. Без перечисления всех возможностей.>
Полная документация и список свойств приведены в [PrimeVue 3 — <PascalCaseName>](<официальный URL>).

## Доступность в BaSYS

<Маркированный список из 1–2 пунктов:>
- В конструкторе форм — под именем `pv-xxx`. <Если нет — «не зарегистрирован в конструкторе форм».>
- В программируемых компонентах — под именем `<PascalCaseName>` (без регистрации). <Если нет — «не входит в список компонентов, доступных программируемому компоненту по умолчанию».>

## Часто используемые в BaSYS props

<Краткий маркированный список — только те props, которые реально применяются в BaSYS-формах (по примерам из `Experimental/Elevator/operation` и значениям по умолчанию из `FormElementBuilder`). Не дублировать весь список из PrimeVue.>

- **propName** (`type`) — назначение. <Пример типового значения в BaSYS.>
- ...

<Если есть событие, которое в BaSYS привязывается через `@Click`/`@Change` и т. п. с командами — упомянуть одной строкой.>

## Свойства по умолчанию в конструкторе форм

<Из `FormElementBuilder.createXxx()`. Если метода нет — написать: «Компонент не создаётся через визуальный конструктор; добавляется вручную в JSON формы или используется только в программируемых компонентах» и пропустить таблицу.>

| Поле            | Значение по умолчанию |
| --------------- | --------------------- |
| `ComponentName` | `pv-xxx`              |
| `CssClass`      | `…`                   |
| ...             | ...                   |

## Примеры использования

### В программируемом компоненте (template)

<Если компонент НЕ доступен в программируемых формах — раздел убрать.>

```html
<PascalCaseName ... />
```

### В форме-конструкторе (JSON)

<Если компонент НЕ зарегистрирован в конструкторе форм — раздел убрать.>

```json
{
  "id": "pv-xxx-…",
  "componentName": "pv-xxx",
  ...
}
```

<По возможности приводить 2 контрастных JSON-примера: минимальный и нестандартный (со стилем/severity/привязкой через `:` или `vModel`, командой через `@Click`).>
````

Правила оформления:

- Тон и стиль — как в `bsTextComponent.md` (нейтральный, технический, на «вы» не обращаться).
- Не переписывать справочник PrimeVue — для подробностей всегда отсылать к официальной странице.
- Не дублировать общие сведения о конструкторе форм или программируемых компонентах — давать ссылки на `formConstructor.md` и `programmableComponents.md`.
- В JSON-примерах сохранять реальные `Id`, имена свойств (`Name`/`Value`) и порядок, как в исходных формах из `Experimental/Elevator/operation`.
- Для props с привязкой к данным в JSON показывать вариант `Name` вида `:propName` или `vModel`, как это принято в BaSYS.
- В разделе «Часто используемые в BaSYS props» включать обработчики событий формата `@EventName` (`@Click`, `@Change`) с указанием, что значение — имя команды (см. раздел «Команды» в `formConstructor.md`).

## Кросс-ссылки

После создания статьи обновить обзорные таблицы:

1. **`ru/userInterface/formConstructor.md`**, раздел **«Компоненты PrimeVue 3»** — заменить внешнюю ссылку (`[`pv-xxx`](https://v3.primevue.org/...)`) на ссылку на локальную статью (`[`pv-xxx`](pvXxxComponent.md)`). Внешний URL остаётся внутри самой статьи, в обзорной таблице он больше не нужен. Сохранить ширину колонок таблицы.

2. **`ru/userInterface/programmableComponents.md`**, раздел **«Компоненты PrimeVue 3»** — аналогично заменить ссылку (`[`PascalCaseName`](https://v3.primevue.org/...)`) на (`[`PascalCaseName`](pvXxxComponent.md)`). Сохранить ширину колонок.

Если в одной из таблиц компонент отсутствует, но по реестрам (`ConstructorComponents` / `dynamicComponentLoader.ts`) он там должен быть — добавить новую строку с соблюдением существующего порядка (как правило алфавитного по русскому названию) и сообщить пользователю, что строка добавлена. Если компонент по реестрам в этой таблице быть не должен — таблицу не трогать.

После правок проверить, что обе ссылки указывают на созданный файл, например через Grep по имени файла `pvXxxComponent.md`.

## Рабочий процесс

1. Уточнить ссылку на компонент PrimeVue 3 (если не указана) и при необходимости — желаемое имя статьи.
2. Параллельно прочитать: страницу PrimeVue 3 (через WebFetch), `constructorComponents.ts`, `dynamicComponentLoader.ts`, `formElementBuilder.ts`, `bsTextComponent.md`, `formConstructor.md`, `programmableComponents.md`.
3. Параллельно Grep'ом найти примеры использования в `D:\_WORK\BaSysProject\Experimental\Elevator\operation` по kebab- и PascalCase-имени.
4. Написать статью по шаблону выше.
5. Обновить кросс-ссылки в `formConstructor.md` и `programmableComponents.md`.
6. Кратко отчитаться: где компонент доступен в **BaSYS** (конструктор/программируемые/оба), какие props и примеры включены (имя исходной формы), какие файлы обновлены.

## Что НЕ делать

- Не создавать статью на английском — все статьи в `ru/userInterface/` ведутся на русском.
- Не переносить полный перечень props/событий/слотов из PrimeVue — только то, что фактически используется в BaSYS. На полное описание давать ссылку.
- Не описывать установку, темизацию и сборку PrimeVue — это не относится к **BaSYS**.
- Не добавлять разделы «Установка», «Импорты», «История изменений».
- Не выдумывать значения по умолчанию для конструктора форм — если в `FormElementBuilder` метода нет, явно отметить это.
- Не менять `index.md` и не править таблицы обзорных статей в разделах, не относящихся к PrimeVue.

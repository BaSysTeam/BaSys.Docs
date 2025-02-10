# Построение и выполнение запросов

## Примеры

# Пример 1

Получение курса валюты из журнала currency_rates.

```javascript
(await from("register.currency_rates")
  .where("currency = @currency")
  .parameter("currency", $h.currency)
  .orderBy("period desc")
  .query()
).rows[0].rate
```

# Пример 2

Получение валюты кассы из справочника cash_box

```javascript
(await from('catalog.cash_box')
 .where('id = @id')
 .parameter('id', $h.cash_box)
 .query()
)
.rows[0].currency;
```
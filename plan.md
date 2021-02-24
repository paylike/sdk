# Payment plans (`plan`)

A payment plan represents an agreement between customer and merchant about
future payments such a subscription, or installments. They are defined using the
`plan` field.

Please note that the payment plan is _not automatically executed_. This field
merely represents the agreement made between customer and merchant as of this
payment order.

Be aware that there is no guarantee for a future payment to succeed despite the
payment plan. Read our section on
[recurring payments](https://github.com/paylike/api-docs#recurring-payments) and
plan accordingly.

Each component represents an amount (`amount`) to be due at a future date either
once (`scheduled`) or repeated.

Repeating components begin at `repeat.first` (defaulting to "now") and repeat
indefinitely or `repeat.count` times at a fixed interval (`repeat.interval`).
Subsequent payments occur each `repeat.interval.unit * repeat.interval.value`
after `repeat.first`. `repeat.interval.value` defaults to `1`.

## Limitations

- `repeat.count` is optional only for the last component
- `scheduled` and `repeat.first` must be chronologically later than the previous
  component

## Examples

### Monthly subscription of €9

```js
const plan = [
  {
    amount: {currency: 'EUR', value: 900, exponent: 2},
    repeat: {
      interval: {unit: 'month'},
    },
  },
]
```

The starting date, and first payment, is assumed to be "now" and thus the next
payment is exactly one month from now. The `amount` for the payment should be €9
to immediately authorize the first amount.

### Monthly subscription of €79 with a 14 days trial

```js
const plan = [
  {
    amount: {currency: 'EUR', value: 7900, exponent: 2},
    repeat: {
      first: new Date(Date.now() + 14 * 24 * 60 * 60 * 1000),
      interval: {unit: 'month'},
    },
  },
]
```

The payment's `amount` would simply be omitted unless the trial has an upfront
price.

### Biweekly (every other week) subscription of €9 charged on Fridays

```js
const plan = [
  {
    amount: {currency: 'EUR', value: 900, exponent: 2},
    repeat: {
      first: new Date('2021-01-22'), // next upcoming Friday
      interval: {
        unit: 'week',
        value: 2,
      },
    },
  },
]
```

The payment's `amount` can be used to charge a prorated amount for the time
between "now" and the first subscription payment if desirable.

### Three months at a reduced rate

```js
const plan = [
  {
    amount: {currency: 'EUR', value: 999, exponent: 2},
    repeat: {
      interval: {unit: 'month'},
      count: 3,
    },
  },
  {
    amount: {currency: 'EUR', value: 1999, exponent: 2},
    repeat: {
      interval: {unit: 'month'},
    },
  },
]
```

### Pay a €1100 debt at €400 monthly (installment)

```js
const plan = [
  {
    amount: {currency: 'EUR', value: 400, exponent: 2},
    scheduled: new Date('2022-02-01'),
  },
  {
    amount: {currency: 'EUR', value: 400, exponent: 2},
    scheduled: new Date('2022-03-01'),
  },
  {
    amount: {currency: 'EUR', value: 300, exponent: 2},
    scheduled: new Date('2022-04-01'),
  },
]
```

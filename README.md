# Paylike Web SDK

Payments for the web.

[Sign up for a merchant account (it's free and instant)](https://paylike.io)

Use the issue tracker to file any bug reports or feature requests.

Make sure to use a key from a test account and set the `test` parameter, which
will also allow you to test without https (e.g. locally).

Need help? Reach out on email: [hello@paylike.io](https://paylike.io/contact).

## Examples

```html
<script src="https://sdk.paylike.io/10.js"></script>
<script>
  const paylike = Paylike({key: '57d23ce1-b651-4b37-8bfb-d4077c3bbf38'})
  paylike.pay(
    {
      test: true,
      amount: {currency: 'EUR', exponent: 2, value: 1499},
      title: 'The Any Tool Shop',
      description: '2x your favorite tool',
      custom: {orderId: 1234},
      text: 'Any Tool Shop*1234',
    },
    (err, result) => {
      if (err) return console.log(err)
      console.log(result.transaction.id)
    }
  )
</script>
```

- [Popup](examples/popup-custom.html)
- [Popup with custom fields](examples/popup-donation.html)
- [Popup for donations](examples/popup-minimal.html)

See also the [standard flow example](standard-flow.md) which includes the
following backend API calls.

## Reference

```js
const paylike = Paylike({key: String})
paylike.pay(config, (err, r) => console.log(err, r))

// config
{
  amount: {
    currency: String, // ISO 4217 (e.g. "EUR")
    exponent: Number, // the number of fractional digits
    value: Number, // integer
  },
  // the amount to be paid now, can be omitted for "save card" (see "Plans"
  // and "unplanned")
  // example: {currency: 'EUR', exponent: 2, value: 1499} (EUR 14.99)

  test: Boolean, // MUST be set if using a test key
  title: String, // title text to show in popup
  description: String, // descriptive text to show in popup

  locale: String, // pin the popup to a locale (e.g. en_US or en)
  // defaults to that of the browser

  text: String,
  // text on customer bank statement (SEE NOTES BELOW)

  custom: Object,
  // data to pass along (objects, nested objects, arrays and primitives)
  // visible in your dashboard
  // can be extracted using the API
  // keep it below 50K

  fields: Array,
  // see "Additional fields" section below

  plan: Array, // see "Plans" below
  unplanned: {
    // see "Unplanned" below
    merchant: Boolean,
    customer: Boolean,
  },

  key: String,
  // override key from factory function
}

// err
'superseded' // the payment popup was closed by the user
'closed' // another payment popup was opened, closing this one
Error // in case of unexpected events
```

All configuration fields are optional, but either `amount` or one of `plan` and
`unplanned` should be provided. All three may also be present (e.g. a gym
subscription with an upfront payment, monthly charge, and pay-as-you-go
classes).

`key` must be provided for either the factory function or `.pay`.

The callback is called in "node-style": `callback(error, response)`.

The response will look like this:

```js
{
  transaction: {
    id: String,
  },

  custom: { ... },
}
```

If the user closes the popup the `error` variable will have a value of `closed`.

If another popup is opened in the meantime `error` variable will have a value of
`superseded`.

### Currency (`amount.currency`)

All supported currencies are listed at https://github.com/paylike/currencies.

### Text (`text`)

The field is optional. If none is provided, the merchant's "descriptor" is used.

The maximum length is 1024 characters. For payment cards (such as Visa,
Mastercard, etc.) the length must be at least 2 characters, it will be truncated
to 22 characters, and only characters matching `/[\x20-\x7E]/` are accepted
(others are replaced).

The text is forwarded to be shown on the customer's bank statement or similar.
There is no guarantee as to how the issuer of the customer's payment instrument
(e.g. their bank) will use or show this text.

### Additional fields (`fields`)

```js
paylike.pay(
  {
    amount: 1000,
    currency: 'DKK',
    fields: [
      // simple custom field
      'name',

      // elaborate custom field
      {
        name: 'email',
        label: 'E-mail', // same as `name` if not provided
        type: 'email',
        placeholder: 'user@example.com',
        required: true,
        value: email, // provide a default value
      },
    ],
  },
  cb
)
```

If you add a field with a name of "amount" it will allow users to dynamically
choose the transaction amount.
[See this example](https://sdk.paylike.io/examples/popup-donation.html).

### Test scenarioes (`test`)

The `test` parameter may be used to trigger specific payment flows and
scenarios. The format is described in
[the API reference documentation for the payments API](https://github.com/paylike/api-reference/blob/master/payments/index.md#test).

### Save (tokenize) a card for later use (subscriptions, installments, pay-as-you-go etc.)

To reuse a card for later transactions from your server, specify at least one of
`plan` and `unplanned`.

You can omit `amount` entirely if no amount is due immedately, but it can
validly be combined with `plan` and `unplanned` for a purchase.

Later on create a transaction from your server using our API:
[create a transaction using a previous transaction](https://github.com/paylike/api-docs#using-a-previous-transaction).

Make sure to read our section about
[recurring payments](https://github.com/paylike/api-docs#recurring-payments).

#### Plans (`plan`)

Please see [this document](plan.md).

This is required for planned subsequent payments to ensure compliance and high
approval rates.

##### Example

```js
{
  plan: [
    {
      amount: {currency: 'EUR', value: 999, exponent: 2},
      repeat: {
        interval: {unit: 'month'},
      },
    }
  ],
}
```

A monthly subscription of EUR 9.99.

#### Unplanned (`unplanned`)

Flag the types (one or more) of unplanned payments the card could be used for in
the future. The supported types are:

- `customer` (initiated by the customer from your website/application)
- `merchant` (initiated by the merchant or an off-site customer)

This is required for unplanned subsequent payments to ensure compliance and high
approval rates.

##### Example

You wish to automatically charge the card for each ride in a roller coaster:

```js
{
  // ...
  unplanned: {merchant: true},
}
```

You allow customers to save their card for faster checkout in the future:

```js
{
  // ...
  unplanned: {customer: true},
}
```

The difference between the two scenarios is whether the customer is triggering
the purchase from a device capable of authenticating the payment.

## Custom payment form

If you are looking to build a custom payment form, make sure to visit these
tools to ease the development:

- [JavaScript (web) client](https://github.com/paylike/js-client)
- [Card form helpers](https://github.com/paylike/js-card-form-tools)

In the
[payments API reference](https://github.com/paylike/api-reference/blob/master/payments/index.md)
you can find the relevant details for submitting payments.

## Browser support

The SDK is tested in all regular browsers capable of conducting modern secure
communication as required for payments.

If you have issues with an environment, please open an issue or drop us an email
at hello@paylike.io.

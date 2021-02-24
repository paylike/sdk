# Paylike Web SDK

Payments for the web.

[Sign up for a merchant account (it's free and instant)](https://paylike.io)

Use the issue tracker to file any bug reports or feature requests.

Make sure to use a key from a test account, which will also allow you to test
without https (e.g. locally).

Need help? Reach out on email: [hello@paylike.io](https://paylike.io/contact).

## Examples

- [Popup](https://sdk.paylike.io/examples/popup-minimal.html)
- [Popup with custom fields](https://sdk.paylike.io/examples/popup-custom.html)
- [Popup for donations](https://sdk.paylike.io/examples/popup-donation.html)
- [Simple but complete webshop](https://github.com/paylike/webshop-example)

## Popup for a transaction

```js
var paylike = Paylike('your key')

paylike.popup(config, callback)
```

```js
// config
{
	currency: String,		// ISO 4217 (e.g. "USD")
	amount: Number,			// minor units (e.g. "200" is $ 2)

	title: String,			// title text to show in popup
	description: String,	// descriptive text to show in popup

	locale: String,			// pin the popup to a locale (e.g. en_US or en)
							// defaults to that of the browser

	descriptor: String,		// text on customer bank statement (SEE RESTRICTIONS BELOW)

	// data to pass along (objects, nested objects, arrays and primitives)
	// visible in your dashboard
	// can be extracted using the API
	// hard limits: keys < 100, depth < 10, key length < 100
	custom: Object,

	// See "Additional fields" section below
	fields: Array,
}
```

Only the amount and a currency is required for transactions.

See this example of using `custom` and `fields`:
https://sdk.paylike.io/examples/popup-custom.html

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

If the user closes the popup the `error` variable will have a value of
`closed`.

### Currencies

All supported currencies are listed at https://github.com/paylike/currencies

### Descriptor

Passing a descriptor you need to be absolutely sure to follow the format to
avoid losing transactions due to rejection by the popup.

See https://github.com/paylike/descriptor for format and restrictions.

#### Additional fields

```js
paylike.popup({
	amount: 1000,
	currency: 'DKK',
	fields: [
		// simple custom field
		'name',

		// elaborate custom field
		{
			name: 'email',
			label: 'E-mail',	// same as `name` if not provided
			type: 'email',
			placeholder: 'user@example.com',
			required: true,
			value: email,		// provide a default value
		},
	],
}, cb)
```

If you add a field with a name of "amount" it will allow users to dynamically
choose the transaction amount.
[See this example](https://sdk.paylike.io/examples/popup-donation.html).

### Example

```html
<script src="//sdk.paylike.io/7.js"></script>
<script>
	var paylike = Paylike('your key')

	paylike.popup({
		currency: 'DKK',
		amount: 1000,
	}, (err, res) => {
			if (err)
				return console.log(err)

		console.log(res.transaction.id)

		alert('Thank you!')
	})
	</script>
```

## Popup to save (tokenize) a card for later use

To store a card for later use from the server, you open the popup in the exact
same manner as described above, but strictly without specifying either
`amount` or `currency`. This will change the popup target from a transaction
to a card token.

### Example

```js
paylike.popup({
	title: 'Add card',
	description: 'Please enter your card details',
}, (err, r) => {
	if (err)
		return console.warn(err)

	console.log(r)	// { card: { id: ... } }
})
```

Later on create a transaction from your server using our API:
https://github.com/paylike/api-docs#from-a-saved-card.

Make sure to read our section about
[recurring payments](https://github.com/paylike/api-docs#recurring-payments).

## Embedded form for transactions

This is the method if you want to design your own payment form.

You should be aware that the work required compared to the popup is
substantial. We recommend all customers to implement the popup first then
experiment with an embedded form in a later iteration.

**Do not give the card number, expiry and cvc fields a name attribute, to
ensure they are never sent to your server**

```js
var paylike = Paylike('your key')

paylike.pay(selector, config, cb)
```

`selector` is the css selector of your form element, it will accept a DOM node
as well.

Your form is required to have input fields with the classes:

- `card-number`
- `card-expiry` (alternatively `card-expiry-month` and `card-expiry-year`)
- `card-code`

You probably want to call `paylike.pay` when the submit event fires on the
form. Make sure to do error checking and show some sort of loading state while
we are processing the payment.

See [this example of a minimal form](examples/embedded-minimal.html) and [this
of a more elaborate use](examples/embedded-complete.html).

[See the list of processing errors](https://github.com/paylike/processing-errors).

## Embedded form for tokenization

Follow the guidelines from the section above but use the `tokenize` method
instead of `pay`.

```js
var paylike = Paylike('your key')

paylike.tokenize(selector, config, cb)
```

### Utilities

To speed up your development we expose some of the utility functions we make
use of ourselves.

```js
Paylike.assistNumber(domNode)
```

- prevent anything else than digits
- add a `visa` or `mastercard` class as soon as possible
- split the number after each run of four digits

```js
Paylike.assistExpiry(domNode)
```

- prevent anything else than digits
- enforce a valid (1-12) month and year (two or four digits)
- assist by inserting "  /  " between month and year

### 3-D Secure

The popup supports 3-D Secure out of the box.

If you wish to implement 3-D Secure support for an embedded form or just know
more about the protocol please read this
[introduction and implementation guide](3dsecure/index.md).

## Browser support

The SDK uses XMLHttpRequest and CORS which is supported by all modern
browsers.

The current SDK has been tested successfully with:

- Firefox 35+
- Chrome 31+
- Chrome for Android 41+
- Opera 27+
- Safari 6.1+ (incl. iPhone 4S with iOS 6)
- IE 10+ (incl. Windows Phone)

It probably works just fine on much older versions of Chrome, Firefox and
Opera even though we have not tested that.

If you have issues with an environment or need specific support for a browser,
please just open an issue and we will have a look.

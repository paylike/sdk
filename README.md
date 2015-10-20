# Paylike SDK

[![Join the chat at https://gitter.im/paylike/sdk](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/paylike/sdk?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Given your public key (found in the dashboard) the job of this SDK is to
create a transaction from your customers card details.

[Sign up for a free merchant account (testing and live, instantly)](https://paylike.io)

Use the issue tracker here to file any bug reports or feature requests, we are
happy to grow with your needs.

## Examples

- https://sdk.paylike.io/examples/popup-minimal.html
- https://sdk.paylike.io/examples/popup-custom.html
- https://sdk.paylike.io/examples/popup-donation.html

You can find the source code of all above examples in this repository.

```html
<script src="//sdk.paylike.io/2.js"></script>
<script>
	var paylike = Paylike('your key');

	paylike.popup({
		currency: 'DKK',
		amount: 1000,
	}, function( err, res ){
		if (err)
			return console.log(err);

		console.log(res.transaction.id);

		alert('Thank you!');
	});
</script>
```

## Popup

```js
var paylike = Paylike('your key');
paylike.popup(config, callback);
```

```js
{
	locale: String,			// pin the popup to a locale (e.g. en_US or en)

	currency: String,		// ISO 4217 (e.g. "USD")
	amount: Number,			// minor units (e.g. "200" is $ 2)

	title: String,			// title text to show in popup
	description: String,	// descriptive text to show in popup

	descriptor: String,		// text on customers bank statement

	// data to pass along (objects, nested objects, arrays and primitives)
	// this will be visible in your dashboard
	// the limits are: keys < 100, depth < 10, key length < 100
	custom: Object,

	// additional fields to display in popup
	fields: Array,
}
```

Most configuration options are optional. Only an amount and a currency are
required.

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

See https://github.com/paylike/descriptor for format and restrictions.

#### Custom fields

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
			type: 'email',
			placeholder: 'user@example.com',
			required: true,
		},
	],
}, cb);
```

You can even add a field with the class "amount", and it will allow users to
set the amount.

## Embedded form

This is the method if you want to design your own payment form.

**Do not give the card number, expiry and cvc fields a name attribute, to
ensure they are never sent to your server**

```js
var paylike = Paylike('your key');

paylike.pay(selector, config, cb);
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

#### Utilities

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

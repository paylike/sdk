# Paylike SDK

[![Join the chat at https://gitter.im/paylike/sdk](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/paylike/sdk?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

https://sdk.paylike.io/1.js

Less than 7 KB (20 KB uncompressed).

Given your public key (found in the dashboard) the job of this SDK is to
create a transaction from your customers card details.

Use the issue tracker here to file any bug reports or feature requests, we are
happy to grow with your needs.

## Example

```html
<script src="//sdk.paylike.io/1.js"></script>
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

Check out all examples in the examples folder.

## API

All methods accept a `callback` which they call in the style: `callback(error,
response)`.

The response will look like this:

```js
{
	transaction: {
		id: String,
	},
}
```

All methods accept a `config` like the following:

```js
{
	currency: String,	// ISO 4217 (e.g. "USD")
	amount: Number,		// minor units (e.g. "200" is $ 2)

	// data to pass along (objects, nested objects, arrays and primitives)
	// this will be visible in your dashboard
	// the limits are: keys < 100, depth < 10, key length < 100
	custom: Object,
}
```

### Popup

```js
var paylike = Paylike('your key');

paylike.popup(config, cb);
```

This will open up a payment window and charge the user 10,00 kr. Once it is
completed, the callback will be fired. If the user closes the window, the
callback is called with the error "closed".

#### Custom fields

With the popup method the config object supports an additional key called
`fields`.

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
			class: 'email',
			placeholder: 'user@example.com',
			required: true,
		},
	],
}, cb);
```

You can even add a field with the class "amount", and it will allow users to
set the amount. A field with the class "currency" works the same way.

### Pay (embedded form)

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
use of ourself.

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

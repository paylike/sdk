# Paylike SDK

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
	Paylike('your key').popup({
		currency: 'DKK',
		amount: 1000,
	}, function( err, res ){
		if (err)
			console.log(err);

		console.log(res.transaction.id);

		alert('Thank you!');
	});
</script>
```

Check out all examples in the examples folder.

## API

All methods take a `callback` which they call in the style: `callback(error,
response)`.

The response will look like this:

```js
{
	transaction: {
		id: String,
	},
}
```

Both methods take a `config` like the following:

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

The config object supports an additional key "fields".

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

### Form (a custom embedded form)

This is the method if you want to design your own payment form.

**Do not give the card number, expiry and cvc fields a name attribute, to
ensure they are never sent to your server**

```js
var paylike = Paylike('your key');

paylike.form(selector, config, cb);
```

`selector` is the css selector of you form element.

Your form is required to have input fields with the classes:

- `card-number`
- `card-expiry` (or `card-expiry-month` and `card-expiry-year`)
- `card-code`

as well as an element with the class `error` for feedback.

When attached the SDK will:

- format the card number when typing
- format the expiry date when typing
- dispatch the required requests upon submit
- run the callback on success

All input fields inside the same `form` block with a `name` attribute (even
`type="hidden"`) will be added to the custom object (indexed by the name) and
be viewable in your dashboard.

## Browser support

The SDK uses XMLHttpRequest and CORS (no iframes from the dark side of the
00's) and supports all modern browsers.

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

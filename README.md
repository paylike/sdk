# Paylike SDK

https://sdk.paylike.io/1.js

Less than 7 KB (20 KB uncompressed).

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

When attached the SDK will:

- put any error message into an element with the class `error` inside the form
- format the card number when typing
- format the expiry date when typing
- dispatch the required requests upon submit
- run the callback on success

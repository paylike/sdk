# Example transaction flow


1. During checkout (frontend)

	1. Open the popup ([examples and documentation](https://github.com/paylike/sdk)). Include at least an order reference (or a hash) in the custom data field which uniquely identifies the order.

	2. The callback for the popup will receive a transaction ID on a succesful payment. Send it to the server using a hidden form, a redirect (query) or an AJAX call.

	```html
	<script src="//sdk.paylike.io/6.js"></script>
	<script>
		var paylike = Paylike('public key')

		paylike.popup({
			currency: 'EUR',
			amount: 10000,
			custom: {
				orderId: '<?=$orderId?>',
			},
		}, (err, res) => {
			if (err)
				return console.log(err)

			location.href = '/complete-order.php?orderId=<?=$orderId?>&transactionId='+res.transaction.id
		})
	</script>
	```

2. During checkout (backend)

	Verify the payment by fetching the transaction:

	```bash
	curl https://api.paylike.io/transactions/<transaction-id> -u :<secret-app-key>
	```

	Check that the currency, amount and unique order reference (or hash) match
	what you have on record and complete the order. If something does not
	match, the payment should fail as the data could have been tampered.

3. When shipping/delivering

	Once you ship the product, capture the transaction:

	```bash
	curl https://api.paylike.io/transactions/<transaction-id>/captures -u :<secret-app-key> -d currency=EUR -d amount=10000
	```

	Amount being EUR 100,00.

In the case of digital products, preorders, vouchers, tickets or other
"instant delivery"-scenarios where you would want to employ "instant capture",
simply do the capture immediately after the verification in step 2.

Resources:

- [Frontend SDK documentation](https://github.com/paylike/sdk)
- [Full API reference and clients](https://github.com/paylike/api-docs)

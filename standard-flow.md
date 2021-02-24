# Example transaction flow

1. During checkout (frontend)

   1. Open the payment popup. Include at least an order reference (or a hash) in
      the custom data field which uniquely identifies the order.

   2. The callback for the popup will receive a transaction ID on a succesful
      payment. Send it to the server using a hidden form, a redirect (query) or
      an AJAX call.

      ```html
      <script src="//sdk.paylike.io/10.js"></script>
      <script>
      	const paylike = Paylike({key: '57d23ce1-b651-4b37-8bfb-d4077c3bbf38'})
      	paylike.pay(
      		{
      			test: true,
      			amount: {currency: 'EUR', exponent: 2, value: 1499},
      			title: 'The Any Tool Shop',
      			description: '2x your favorite tool',
      			custom: {orderId: '<?=$orderId?>'},
      			text: 'Any Tool Shop 1234',
      		},
      		(err, result) => {
      			if (err) return console.log(err)
      			location.href =
      				'/complete-order.php?orderId=<?=$orderId?>&transactionId=' +
      				result.transaction.id
      		}
      	)
      </script>
      ```

2. During checkout (backend)

   Verify the payment by fetching the transaction:

   ```bash
   curl https://api.paylike.io/transactions/<transaction-id> -u :<secret-app-key>
   ```

   Check that the currency, amount and unique order reference match what you
   have on record and complete the order. If something does not match, the
   payment should fail as the data could have been tampered.

   A cryptographic hash of the critical fields, including any internal
   references, could also be added to the `custom` object and checked against a
   re-computed hash.

3. When shipping/delivering

   Once you ship the product, capture the transaction:

   ```bash
   curl https://api.paylike.io/transactions/<transaction-id>/captures -u :<secret-app-key> -d currency=EUR -d amount=1499
   ```

   Amount being EUR 14,99.

In the case of digital products, preorders, vouchers, tickets or other "instant
delivery"-scenarios where you would want to employ "instant capture", simply do
the capture immediately after the verification in step 2.

See also the
[full API reference and clients](https://github.com/paylike/api-docs) for
backend integration.

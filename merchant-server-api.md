# BUX.digital Payment Protocol Merchant Server

An API for the creation and fulfillment of BUX eToken invoices utilizing the [Simple Ledger Payment Protocol](https://github.com/vinarmani/slp-specifications/blob/payment-protocol/slp-payment-protocol.md)

## Invoice Creation

### Request

A GET request is made to `https://bux.digital/pay/`.

GET is used (as opposed to POST) in this case to allow the invoice creation request URL itself to be used as a standalone "buy" link on web pages, in emails, or as a QR code.

#### GET Query Data
The following GET query parameters are used **(all values must be URI encoded)**:
* `merchant_name` - The name of the entity requesting the payment
* `invoice` - The external identifier of the invoice (ie. the invoice number displayed to the buyer)
* `order_key` - The internal identifier of the invoice (ie. the primary key of the invoice row in the merchant's database)
* `merchant_addr` - a single eToken address or a JSON-encoded array of destination addresses
* `amount` - a single number representing some amount of BUX or a JSON-encoded array of numbers representing destination amounts
* `success_url` - (optional) The URL that the buyer's wallet will redirect to upon successful payment
* `cancel_url` - (optional) The URL that the buyer's wallet will redirect to if payment is cancelled
* `ipn_url` - (optional) The URL to which IPN data will be posted upon successful payment
* `return_json` - (optional) If `return_json` is used and set to *true*, then this call will return a JSON object as defined below.

**Note: if sending to multiple recipients, the address at a given index in the** `merchant_addr` **array corresponds with the number at the same index in the** `amount` **array. Both arrays must, therefore, be of the same length or an error will be returned.**

#### Headers

* `Content-Type` should be set to `application/json`.

### Response
The default behavior of this call is to redirect the buyer to a non-custodial web wallet to pay the invoice. This is currently [wallet.badger.cash](https://wallet.badger.cash).

If the `return_json` option is used, the response will be a JSON format payload quite similar to the [Simple Ledger Payment Protocol](https://github.com/simpleledger/slp-specifications/blob/master/slp-payment-protocol.md) format.

#### Body
* `network` - Which network is this request for (main / test / regtest)
* `currency` - Will be `etoken`
* `requiredFeeRate` - Required miner fee (in satoshis per byte). *BUX.digital currently subsidizes this fee for wallets supporting Postage Protocol. In such cases this value can be ignored*
* `outputs` - What output(s) your transaction must include in order to be accepted
* `time` - ISO Date format of when the invoice was generated
* `expires` - ISO Date format of when the invoice will expire
* `status` - The status of the invoice `open` or `paid`
* `memo` - A plain text description of the payment request, can be displayed to the user / kept for records
* `paymentUrl` - The url where the payment should be sent
* `paymentId` - The invoice ID, can be kept for records
* `callback` - An object representing redirect URLs and the IPN data that will be posted to the `ipn_url` upon successful payment

#### Response Body Example
```
{
    "network": "main",
    "currency": "etoken",
    "requiredFeeRate": 1,
    "outputs": [
        {
            "script": "6a04534c500001010453454e44207e7dacd72dcdb14e00a03dd3aff47f019ed51a6f1f4e4f532ae50692f62bc4e50800000000000347d808000000000004bed8",
            "amount": 0
        },
        {
            "amount": 546,
            "script": "76a914e565e4e8ad08d520fde6890fe1b2f8e9210f29d688ac"
        },
        {
            "amount": 546,
            "script": "76a914e565e4e8ad08d520fde6890fe1b2f8e9210f29d688ac"
        }
    ],
    "time": "2022-10-29T03:36:33.270Z",
    "expires": "2022-10-29T03:51:33.270Z",
    "memo": "Payment to Some Merchant for order #asd123",
    "paymentUrl": "https://pay.badger.cash/i/4TW2V",
    "paymentId": "4TW2V",
    "status": "open",
    "callback": {
        "success_url": "https://example.com/success?id=123",
        "cancel_url": "https://example.com/cancel?id=123",
        "ipn_url": "https://example.com/ipn?id=123",
        "ipn_body": {
            "merchant": [
                "etoken:qrjkte8g45yd2g8au6yslcdjlr5jzref6c89q0p5ds",
                "etoken:qrjkte8g45yd2g8au6yslcdjlr5jzref6c89q0p5ds"
            ],
            "invoice": "asd123",
            "custom": "FFWSD",
            "ipn_type": "simple",
            "currency1": "USD",
            "amount1": [
                21.5,
                31.1
            ]
        }
    }
}
```

### Curl Example
```
curl -v -L -H 'Content-Type: application/json' 'http://bux.digital/v1/pay?merchant_name=Some%20Merchant&invoice=asd123&order_key=FFWSD&amount=%5B21.5%2C31.1%5D&merchant_addr=%5B%22etoken%3Aqrjkte8g45yd2g8au6yslcdjlr5jzref6c89q0p5ds%22%2C%22etoken%3Aqrjkte8g45yd2g8au6yslcdjlr5jzref6c89q0p5ds%22%5D&success_url=https%3A%2F%2Fexample.com%2Fsuccess%3Fid%3D123&cancel_url=https%3A%2F%2Fexample.com%2Fcancel%3Fid%3D123&ipn_url=https%3A%2F%2Fexample.com%2Fipn%3Fid%3D123&return_json=true'

* TCP_NODELAY set
* Connected to bux.digital (216.238.107.167) port 80 (#0)
> Host: bux.digital
> User-Agent: curl/7.68.0
> Accept: */*
>
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Access-Control-Allow-Origin: *
< Content-Type: application/json; charset=utf-8
< Content-Length: 935
< ETag: W/"3a7-M/YJlKaN5vjzC72cjO+CmC3ip04"
< Date: Sat, 29 Oct 2022 04:21:53 GMT
< Connection: keep-alive
< Keep-Alive: timeout=5
<
* Connection #1 to host bux.digital left intact
{"network":"main","currency":"etoken","requiredFeeRate":1,"outputs":[{"script":"6a04534c500001010453454e44207e7dacd72dcdb14e00a03dd3aff47f019ed51a6f1f4e4f532ae50692f62bc4e50800000000000347d808000000000004bed8","amount":0},{"amount":546,"script":"76a914e565e4e8ad08d520fde6890fe1b2f8e9210f29d688ac"},{"amount":546,"script":"76a914e565e4e8ad08d520fde6890fe1b2f8e9210f29d688ac"}],"time":"2022-10-29T04:21:53.278Z","expires":"2022-10-29T04:36:53.278Z","memo":"Payment to Some Merchant for order #asd123","paymentUrl":"https://pay.badger.cash/i/TSWJ5","paymentId":"TSWJ5","status":"open","callback":{"success_url":"https://example.com/success?id=123","cancel_url":"https://example.com/cancel?id=123","ipn_url":"https://example.com/ipn?id=123","ipn_body":{"merchant":["1MuwofKDBfyyNLmMybseUoRqNTkhKg6s17","1MuwofKDBfyyNLmMybseUoRqNTkhKg6s17"],"invoice":"asd123","custom":"FFWSD","ipn_type":"simple","currency1":"USD","amount1":[21.5,31.1]}}}
```

## Payment

The payment process follows the [SLP Payment Protocol](https://github.com/vinarmani/slp-specifications/blob/payment-protocol/slp-payment-protocol.md)


## API Endpoints

### Check Status

#### Request Status

A GET request is made to `paymentUrl` with a header of `Accept: application/payment-request`

#### Request Status (Websocket)

A Websocket Secure (WSS) request is made to `wss://` and the origin and path of `paymentUrl` (eg. `wss://pay.badger.cash/i/TSWJ5`)

When connection is established, a string consisting solely of `paymentId` (eg. `TSWJ5`) must be sent as the first message. If any other message is sent, or if `paymentId` is invalid, a JSON string representing an empty object will be returned and the connection will close.

#### Response

Server will respond with a JSON response of the same type as the response during invoice creation. 

If websocket is used and `status` is `open`, connection will remain open until closed by client or until `status` changes to `paid`, at which time a new object, reflecting the updated state, will be returned and the connection will close. If the invoice expires, a JSON string representing an empty object will be returned and the connection will close.

### Invoice Payment Via Web Wallet

Visiting `paymentUrl` in a browser (no additional headers) will redirect to to a non-custodial web wallet to pay the invoice.

### QR Code

#### Request QR code

A GET request is made to `paymentUrl` with a header of `Accept: image/png`.

#### Response

A QR code representation of a URI formatted to [Simple Ledger Payment Protocol URI specification](https://github.com/simpleledger/slp-specifications/blob/master/slp-payment-protocol.md#uri-specification) (`etoken:` prefix, non-backwards compatible) is returned as a PNG image.

## Instant Payment Notification (IPN)

### Request

Upon successful payment, an IPN request is sent to `ipn_url`, originating from the payment server identified by `paymentUrl` in the invoice creation response. The request will have, as its JSON body, the data in the `callback.ipn_body` property of the invoice creation response.

* `Content-Type` will be `application/json`.

#### IPN POST Data
* `payment_id` - corresponds to `paymentId` in invoice creation response
* `txn_id` - transaction hash of payment. **This value can (and should) be used by the merchant to independently verify the existence of the transaction on the blockchain before fulfilling the corresponding order**
* `merchant` - a single eToken address or a JSON-encoded array of destination addresses
* `amount1` - a single number representing some amount of BUX or a JSON-encoded array of numbers representing destination amounts
* `invoice` - The external identifier of the invoice (ie. the invoice number displayed to the buyer)
* `custom` - Corresponds to `order_key`. The internal identifier of the invoice (ie. the primary key of the invoice row in the merchant's database)
* `ipn_type` - currently will always be set to `simple`
* `currency1` - currently will always be set to `USD`
* `status` - Set to the number `100`, representing completed payment
* `status_text` - status message (ie. `Status paid`)
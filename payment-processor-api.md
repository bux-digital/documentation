# BUX.digital Payment Processor Model

This specification describes the general model used for interaction between the BUX authorization code generation infrastructure and a payment/credit card processor. Most payment processors already have an API and have no need for this document. For processors without an existing API who desire to work with BUX, this framework can serve as a starting point for development.

## Payment Processing Flow

1. User client (wallet) sends a request to the payment processor with all necessary information to effectuate a sale

2. Payment processor validates payment and, if successful, makes a request to the BUX server to receive authorization code

3. BUX server makes a request back to the payment processor server verifying the details of the transaction, generates an authorization code based on the transaction data, and responds with the authorization code

4. The payment processor responds to the user witht he authorization code which is then used by the client/wallet to generate the self-mint transaction

## Client Payment

This section describes the request and response between the user client and the payment processor

### Purchase Request

A POST request is made to the payment processor endpoint, ie. `https://processor.com/purchase/`.

#### POST Body Data
The following properties are in the object sent as JSON POST body:
* `client_id` - an identifier specific to BUX transactions
* `item_id` - an identifier (SKU) specific to a BUX authorization code
* `credit_card_number` - card number being used to pay
* `full_name` - full name of the payee as it appears on the card
* `expiry` - card expiry in format MMYY (or MM/YY)
* `cvc` - CVC number on back of card
* `amount` - the amount of the sale in USD
* `country` - two-letter cardholder country (as registered on card)
* `postcode` - postcode for card
* `email` - cardholder email (can be used in email validation)
* `phone` - phone number associated with cardholder (can be used in SMS validation)
* `metadata` - metadata for BUX validation, in hexadecimal format. 255 bytes maximum

#### Headers

* `Content-Type` should be set to `application/json`.

#### Response Body
* `authcode` - The authorization code in base64
* `description` - Text that is generally shown to a user in their wallet (this is generally not needed in this model)

#### Response Body Example
```
{
   "authcode":"AqyxwR5lHU15X/9va3xacyUCLFLbkQ3ZVCI8aHzz1SEkACFyhhJWlJtF1dI7RDI+cYkBPRzFwi6Xtf2CXoaqzG+4ALaGyjYF3Y51puYgx+bwkXcFz/mlyRLCpbHaz3M/AawFADPDq7+or2OZ+vVPtyOH1mKp7hyDXT9Cyl3Xj8gw0j+8ABS0hx1SulMaS90FvNlqsPydJsy0ZrJyyR4gCtiygtXsAPWWItFzgT+xNZbJltNDRFbb+WPx7uzuupG8v1CcjN+qAA1uXzG1VVEt6LDCL1vsBc6qE/JovcEgBFbmya72oLHoALUyyhgadlzjojLpkAwfL5J96D5W6On0Mf/id2eDHRqqRjBEAiB0hLl8oU3G/dtfCFUMtg6836kzqGXJdYIMr/me5tZ/rwIgVh4UTqyeFVu1FWx9kc5LTMP2VaMo0DBDTkv54VeliVBUUlQ3RgDZL14recsdC3/g7W8Phb8qnIe8ar9hqscJnsT/o8vjp5yMlRxuQ9QHa2Owv9OPNOq76gk3FgAAAAAAAAAAAAAAQGoEU0xQAAECBE1JTlQgQHVFngrIQfI0vHP8T+Rv5UkL5O2YvIyj+biYRDpaOBoIAAAAAAAAJxAIAAAAAAAAA+giAgAAAAAAABl2qRQU/iTRGN78yLo3VZQ5CBy8qvoLMIisIgIAAAAAAAAZdqkUrnick8kEBVsa2IscZFZF2fBFF4WIrA==",
   "description":"Copy and paste this (base64) auth code into your wallet"
}
```

### Curl Example
```
curl -XPOST -H -v -L "Content-type: application/json" -d '{"client_id":123456,"item_id":"bux1","credit_card_number":5555444433332222,"full_name":"John Doe","expiry":"0327","cvc":777,"amount":56.99,"country":"US","postcode":"12212","email":"buyer@email.com","phone":"+17775551212","metadata":"38bb76ed09cb953808481242a04703ba5203ba12b386157e6d7d64cc52f88077"}' 'https://processor.com/purchase/'

* TCP_NODELAY set
* Connected to processor.com (111.111.111.111) port 80 (#0)
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
* Connection #1 to host processor.com left intact
{"authcode":"AqyxwR5lHU15X/9va3xacyUCLFLbkQ3ZVCI8aHzz1SEkACFyhhJWlJtF1dI7RDI+cYkBPRzFwi6Xtf2CXoaqzG+4ALaGyjYF3Y51puYgx+bwkXcFz/mlyRLCpbHaz3M/AawFADPDq7+or2OZ+vVPtyOH1mKp7hyDXT9Cyl3Xj8gw0j+8ABS0hx1SulMaS90FvNlqsPydJsy0ZrJyyR4gCtiygtXsAPWWItFzgT+xNZbJltNDRFbb+WPx7uzuupG8v1CcjN+qAA1uXzG1VVEt6LDCL1vsBc6qE/JovcEgBFbmya72oLHoALUyyhgadlzjojLpkAwfL5J96D5W6On0Mf/id2eDHRqqRjBEAiB0hLl8oU3G/dtfCFUMtg6836kzqGXJdYIMr/me5tZ/rwIgVh4UTqyeFVu1FWx9kc5LTMP2VaMo0DBDTkv54VeliVBUUlQ3RgDZL14recsdC3/g7W8Phb8qnIe8ar9hqscJnsT/o8vjp5yMlRxuQ9QHa2Owv9OPNOq76gk3FgAAAAAAAAAAAAAAQGoEU0xQAAECBE1JTlQgQHVFngrIQfI0vHP8T+Rv5UkL5O2YvIyj+biYRDpaOBoIAAAAAAAAJxAIAAAAAAAAA+giAgAAAAAAABl2qRQU/iTRGN78yLo3VZQ5CBy8qvoLMIisIgIAAAAAAAAZdqkUrnick8kEBVsa2IscZFZF2fBFF4WIrA==","description":"Copy and paste this (base64) auth code into your wallet"}
```

## Authorization Code Generation

This section describes the requests and responses between the payment processor and the BUX API

#### BUX API Endpoints

* `https://bux.digital/v2/success/` - Production
* `https://dev-api.bux.digital/v2/success/` - Sandbox

### Request

When payment processor determines that the payment from the user is valid, a GET request is made to the relevant BUX endpoint

#### GET Query Data
The following GET query parameters are used **(all values must be URI encoded)**:
* `processorId` - identifies payment processor (agreed upon by BUX and processor before integration begins)
* `paymentId` - An identifier assigned to the transaction in question by the payment processor.

#### CURL Example
```
curl -v -XGET -H "Content-type: application/json" 'https://dev-api.bux.digital/v2/success/?processorId=cardguys&paymentId=aabb1'
```

#### Validation and Generation

When the above request is received by the BUX API, a request will be from the BUX API to the payment processor API fetching the status of the transaction based on the provided `paymentId`. Payment processor responds with the original POST body data provided by the user/client with two additional properties (added by the payment processor):

* `payment_id` - coincides with `paymentId`
* `status` - complete | falied | pending 

#### Curl Example
```
curl -v -XGET -H "Content-type: application/json" 'https://processor.com/order/?paymentId=aabb1'

* TCP_NODELAY set
* Connected to processor.com (111.111.111.111) port 80 (#0)
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
* Connection #1 to host processor.com left intact
{"payment_id":"aabb1","status":"complete","client_id":123456,"item_id":"bux1","credit_card_number":5555444433332222,"full_name":"John Doe","expiry":"0327","cvc":777,"amount":56.99,"country":"US","postcode":"12212","email":"buyer@email.com","phone":"+17775551212","metadata":"38bb76ed09cb953808481242a04703ba5203ba12b386157e6d7d64cc52f88077"}
```

In the case that `status` is `complete`, the response from the BUX API to the payment processor is the authcode response body example from the earlier section.


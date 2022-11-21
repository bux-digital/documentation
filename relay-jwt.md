# Relay JWT Specification

A specification for BUX API access tokens using the [JSON Web Token (JWT)](https://www.rfc-editor.org/rfc/rfc7519) standard.

#### Version: 1.0
#### Date published: November 21, 2022

## Purpose

This specification describes a means of using a JSON Web Token (JWT) to facilitate highly-scalable permissionless affiliate relationships for [BUX API](./merchant-server-api.md) relay servers. Using JWT's as described in this document, chains of affiliates can both provide access to their respective infrastructures and also allow the reliable calculation of total transaction amounts by any of their end-users.

## Background

The BUX API allows for the creation and verification of payment requests. These payment requests enable peer-to-peer commerce using BUX tokens. Entities, known as "relay servers," proxy the invoice creation requests that are added to the BUX system. For providing such proxy services, relay servers can (should they so choose) add an additional non-custodial fee as an output in the payment request. The BUX API also adds such a non-custodial fee. These fees can be fixed or can be a percentage of the total output amount requested from the respective relay server - the total amount represented by all of the outputs when the request reaches a given relay server.

A solution is needed to 

1. Regulate access to each relay server
2. Allow any entity in the relay chain to calculate the fee that should be added to the payment request
3. Allow the initial creator of the request, the merchant, to reliably calculate the total amount the customer will pay after all relay servers have added their respective fees

This document describes a means of using a JWT with a specific payload format as a solution the above listed problems.

## Specification

### The JSON Web Token

Each relay server will issue to users (including other relays consuming that relay server's API) a JWT to use, in the standard manner, for access. The JWT issued will meet the following requirements

1. Signature algorithm will be ECDSA using P-256 curve and SHA-256 hash algorithm (ES256)
2. Payload will include the `iat` (Issued At) and `sub` (Subject) [claims](https://www.rfc-editor.org/rfc/rfc7519#section-4.1.2)
3. The subject claim will follow the standard specified in this document

So long as these requirements are met, issuers may extend their JWT token in any way they choose and the token will still be considered valid under this precification

#### Subject Claim

The subject claim consists of a chain of cryptographic certificates that, when decoded, describe the route by which fees will be added to a payment request. The process by which this chain is created is as follows

1. The BUX API (Relay 0) issues a JWT to an end user. This JWT contains a specially formatted `sub` claim prefixed by a signature and using fee-related metadata (described below) as the msg that is signed.
2. Should the end user decide to act as a relay server themselves (Relay 1), then they would issue to their own respective end user a JWT token. The message, in this case, is the fee-related metadata for Relay 1 with the previous `sub` from Relay 0 appended to the metadata. This entire message is signed and the signature is prepended to the message and is used as the new `sub` for the JWT issued by Relay 1
3. This process is repeated down the chain of relay servers, such that any JWT contains within it signed fee-related metadata for every "link" in the relay chain

The `sub` is a Base64-encoded string representing serialized (left to right) bytes in the following order

1. (2 bytes, big endian) *integer* - **signature length**
2. (variable) *bytes* - **signature**: a signature, using the ES256 algorithm, that signs the serialized bytes of the values described in #3 through #8, using the private key associated with the public key in #4
3. (1 byte) *integer* - **public key length**
4. (variable) *bytes* - **public key**: if this is the most recent certificate in the chain, the public key must match the public key for the JWT itself
5. (1 byte) *integer* - **version**: `1` is the only valid value currently
6. (1 byte) *integer* - **fee type**: `0` for a percentage based fee (the value represents `x/1000` so 5% would be represented as `50`) or `1` for a fixed fee
7. (8 bytes, big endian) *integer* - **amount**: between `0` and `1000` for percentage or the number of base units to be charged for fixed
8. (variable) *bytes* - **previous subject**: the entire subject representing the chain of certificates "upstream" of the issuer

#### Certificate Validation

The first serialized signature (reading from the left) must be validated and the associated public key (#4 above) must also be the valid public key for the signature used in the overall JWT.

As the chain is decoded, signatures must also be valid for the entire remainder of data that follows.

This might be visualised as "nested" data, with each signature signing a message that consists of all data that follows the signature in the order of serialization.

#### Example Subject Certificate Chain

JWT `sub` claim as issued by BUX API (Relay 0)

![Relay 0](images/relay0.png)

JWT `sub` claim as issued by Relay 1

![Relay 1](images/relay1.png)

JWT `sub` claim as issued by Relay 2

![Relay 2](images/relay2.png)

## Reference Implementations

### Servers
[BUX Merchant Server API](./merchant-server-api.md)

### Clients
None Currently

### Libraries
[relay-jwt](https://github.com/bux-digital/relay-jwt)
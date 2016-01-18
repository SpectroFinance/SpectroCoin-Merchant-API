SpectroCoin Merchant API
========================
This document describes [SpectroCoin](https://spectrocoin.com) merchant service API specification.

# Contents

* [Requirements](#requirements)
* [API](#api)
* [Signature](#signature)
* [Example applications](#example-applications)

# Requirements

* Must have a SpectroCoin account ([Sign Up!](https://spectrocoin.com/en/signup.html)) to setup and test API. For merchant API usage in production you must approve your account.
* Must [generate](#merchant-key-pair) private and public key pairs.
* Must [setup](https://spectrocoin.com/en/merchant/api/create.html) merchant API instance on SpectroCoin API configuration page.

# API

* [Interactive SpectroCoin merchant API operation list](https://spectrocoin.com/api/index.html)
* [Root Merchant REST based API address](https://spectrocoin.com/api/merchant/1/)

## /createOrder

Merchant who wants his customer order to be paid creates order at SpectroCoin with payment details and presents result to his customer. Merchant can also redirect his customer to returned redirect url where customer will be presented with interactive payment window.

### Request

Request URL: https://spectrocoin.com/api/merchant/1/createOrder

For this method server accept only **POST** of **"application/x-www-form-urlencoded"** media type.

Seq No. | Field | Type | Required | Example
--------|-------|------|----------|--------
1. | merchantId | Long | + | 12345
2. | apiId | Long | + | 1
3. | orderId | String | - | ABC001, **Must be unique** for all merchant API orders. If not provided, order request id will be assigned
4. | payCurrency | String | + | BTC
5. | payAmount | Double | + or receiveAmount | 123.45, 1.23456789. Amount merchant clients will pay for order in provided pay currency
6. | receiveCurrency | String | + | EUR, USD, GBP, SEK, NOK, DKK, PLN, CZK, BGN, GEL, RON, HRK, TRY, CHF
7. | receiveAmount | Double | + or payAmount | 123.45, 1.23456789. Value of provided receive currency which merchant want to receive to configured receive account at merchant API setup window
8. | description | String | - | Order ABC001 for User 123
9. | culture | String | - | en, lt, ru. Language for response
10. | callbackUrl | String | - | https://merchant.com/orderCallback?user=123
11. | successUrl | String | - | https://merchant.com/success?user=123
12. | failureUrl | String | - | https://merchant.com/failure?user=123
13. | sign | String | + | Generated order request signature


Example HTTP request:

```http
POST https://spectrocoin.com/api/merchant/1/createOrder HTTP/1.1
Content-Length: 708
Content-Type: application/x-www-form-urlencoded
Host: spectrocoin.com
Connection: Keep-Alive
Accept-Encoding: gzip,deflate

merchantId=87&apiId=1&orderId=&payCurrency=BTC&payAmount=0.0&receiveCurrency=USD&receiveAmount=15.99&description=Payment+for+Order-2547&culture=&callbackUrl=https%3A%2F%2Fmerchant.com%2Fcallback%3Fuser%3D123&successUrl=https%3A%2F%2Fmerchant.com%2Fsucess%3Fuser%3D123&failureUrl=https%3A%2F%2Fmerchant.com%2Ffailure%3Fuser%3D123&sign=SHlHIo3Q%2FKs7GE08jbp3WNKMq5h%2BJDOaSU%2BOfLq1bdXbA9cF5TrBhM32%2B%2BeiF%2FXLnITYzX8n0TSvWDYKj5ItJXi7Dg7PoU9FtIEUaPIMHx681PhgMh1fNBQVyzn9wOj%2BbVNcEpIzIeziTIkXZbdC%2FDvMlIN85n7jU%2FRhe8INeQxaEWtggkuvkpW34D9fy0syn3lsBn5ev6J09huy2PuOT4qAY28NpIXxhq9WdKJBTp%2FhZv7QAVmq9qcNexOlgd3P1GSlBvMpuF7HoliNC7EKRL%2B5bp8f1P8S0ecqYrc0SFmtQOhhFU%2BkvcEdOi3oA6xgKxgc28CeXGfHXO%2F1BEDnbg%3D%3D
```

### Response

**JSON** formatted response of complex object `Order`.

Field | Type | Example
------|------|--------
orderRequestId | Long | Order request id, SpectroCoin Id to track orders
orderId | String | ABC001
payCurrency | String | BTC
payAmount | Double | 123.45, 1.23456789 (BTC)
receiveCurrency | String | BTC, EUR, ...
receiveAmount | Double | 123.45, 1.23456789 (BTC)
depositAddress | String | 1HZcE7ZbwnEKHYcKkva1uZoxZbFvRyK3fm
redirectUrl | String | https://spectrocoin.com/en/pay/order/18-5fD2HgMK.html
validUntil | Long | Timestamp (how many milliseconds have passed since January 1, 1970, 00:00:00 GMT) until order is valid.


```json
{
"depositAddress":"1HZcE7ZbwnEKHYcKkva1uZoxZbFvRyK3fm",
"orderId":"18",
"orderRequestId":18,
"payAmount":0.00246995,
"payCurrency":"BTC",
"receiveAmount":1.0,
"receiveCurrency":"EUR",
"redirectUrl":"https://spectrocoin.com/en/pay/order/18-5fD2HgMK.html",
"validUntil":1401191587663
}
```

### Callback

**POST** request to merchant provided order callback url. Request provides information about current order status. Usually there will be several callbacks for a successful order (pending, paid).
Merchant page must return HTTP Response **200** with content:** *ok* **for SpectroCoin API to confirm callback as sent successfully.

Complex object `OrderCallback`.

Seq No. | Field | Type | Example
--------|-------|------|--------
1. | merchantId | Long | 12345
2. | apiId | Long | 1
3. | orderId | String | ABC001
4. | payCurrency | String | BTC
5. | payAmount | Double | 123.45, 1.23456789 (BTC)
6. | receiveCurrency | String | BTC, EUR, ...
7. | receiveAmount | Double | 123.45, 1.23456789 (BTC)
8. | receivedAmount | Double | 123.45, 1.23456789 (BTC)
9. | description | String | Order ABC001 for User 123
10. | orderRequestId | Long | Order request id, SpectroCoin Id to track orders
11. | status | Short | Order status
12. | sign | String | Generated order callback signature

Order status table

Status Code | Order status | Description
------------|--------------|------------
1 | New | Start state when order is registered in SpectroCoin system
2 | Pending | Payment (or part of it) was received but still not confirmed
3 | Paid | Order is complete
4 | Failed | Some error occurred
5 | Expired | Payment was not received in time
6 | Test | Test order

Example HTTP request:

```http
POST https://merchant.com/callback HTTP/1.1
Accept-Encoding: gzip,deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 578
Host: merchant.com
Connection: Keep-Alive

merchantId=169&apiId=1&orderId=Order%20123&payCurrency=BTC&payAmount=1.23456789&receiveCurrency=EUR&receiveAmount=50.0&receivedAmount=0.0&description=My%20special%20order&orderRequestId=123&status=1&sign=r6NuZ0JCBSAGCpPlYG5eXLkSRyOqSncWj8j7LfLIiiWeZdeH0Yy2nZ4Osn0JJY9cqWnoT%2Fvn%2B0KjY7f9tdYQjk67XTFY%2Bn1yg41FkzfbDxF8LQWYinLpnBBCf5AlFACJJ26yBHXMxsPFn67khxOV55AGRJeJcw03anH%2FobjevHiOGkV9%2BjTVLwV553U6Y9Ud995D66f45QpPR54IgBBDhA%2BHNlwockLcEyzCMbwMPs0pnsfVO65x0if8TJ2MtwTCa5c%2B%2FuCAq%2BoafTRV9%2Bd9meTRbC%2BFTDf%2FAMyU9SiltpdIqoZPRypB7faBDZ5YVsQrRPIfD6Wy%2FtS6fb8MiFBnOw%3D%3D
```

## Error result

When calling any of API operation, it may result an error.
Errors will be returned **JSON** formatted with **203** http status code.

```json
[
  {
    "code": 1,
    "message": "apiId is required"
  },
  {
    "code": 1,
    "message": "merchantId is required"
  }
]
```

Error codes

Error code | Error message
-----------|--------------
1 | Validation errors
2 | Bad signature
3 | Not supported currency
4 | Can't create order, please check your merchant account
5 | Merchant order id exist
6 | Check your merchantId and apiId
97 | Unsupported Media Type
99 | Please check your request
100 | Unexpected error


# Signature

All API requests must be signed using merchant private key so they can be validated by SpectroCoin using merchants public key. Merchant should provide his public key at SpectroCoin merchant API configuration.

Some API request may result in callback from SpectroCoin, such request are also signed by SpectroCoin and must be validated by merchant using [SpectroCoin Merchant Public Key](https://spectrocoin.com/files/merchant.public.pem).

## Merchant key pair

You should create key pair (private and public keys) ([Wiki](http://en.wikipedia.org/wiki/Public-key_cryptography)) for your requests to be signed and validated by SpectroCoin.

**Private key** must be kept safely by merchant without any disclosure.

**Public key** must be inserted into configuration of specific SpectroCoin API (Create/Edit form of API details). This key will be used to check signature validity of any merchant API request signed by specific merchant and API.

### Private and public key generation using SpectroCoin

Open [merchant API setup](https://spectrocoin.com/en/merchant/api/create.html) window.
Click Refresh button near "Public Key", new key pair will be generated.
Download private key and keep it safe.
Save merchant API with newly generated public key.

### Private key generation with OpenSSL

```shell
# generate a 2048-bit RSA private key
openssl genrsa -out "C:\private.pem" 2048

# convert private Key to PKCS#8 format (if you use Java)
openssl pkcs8 -topk8 -inform PEM -outform DER -in "C:\private.pem" -out "C:\private.der" -nocrypt
```

### Public key generation with OpenSSL

```shell
# output public key portion in PEM format
openssl rsa -in "C:\private.pem" -pubout -outform PEM -out "C:\public.pem"
```

## Signing request

Request to be signed must be converted to **UTF-8 URL encoded concatenated parameters**, consisting of one string line, including parameter names, which are ordered in specific sequence specified in documentation.

Request data which will be signed must be:

* URL encoded
* Spaces must be encoded with "**+**" sign (not "%20"):
```
&description=Some+string
```
* Numbers must be formatted with **0.0#######** number format:
```
null => 0.0
0 => 0.0
0.0 => 0.0
1.1 => 1.1
1.123 => 1.123
```
* **All** request fields should be included for signature generation even if they don't have value set.

Example request data to be signed:
```
merchantId=25&apiId=74&orderId=NO-2514&payCurrency=BTC&payAmount=20.0&receiveCurrency=EUR&receiveAmount=0.0&description=Super+payment&culture=&callbackUrl=http%3A%2F%2FmySite.com%2Fcallback&successUrl=&failureUrl=
```

Signature must be **Base64 encoded**. Example:
```
QKpoaBL/P2tcFttFm/TVcn0utkgaOzEAOsZbSSOa+zntcxyJUijaM5egewRoRu68d4CoswTpkdOqaKdGrLWPNqQGujPHIX3q8Q0lK/C8GqN7MYHFrLxu+rpY0G4srIaDzXww4uOTkBIBFWn3TVI4AZAGm0/APlZZeCrhwIIkImYc8ab69zeqikyaMXRK0XMAD/8Fz9b+rUR342hMjFR+epZnNmWQpFtQLvB/SxlCZIZ+u1k2WLJYa7CChDePmdXNHgutvt1mQxLMpJmeDNjD2aOzF9+DPIqsOEkJ9RLJ8F0kQXnn9W02Av/a3GMVC7A/u/kxnKo3LRfkkkAAYkCKug==
```

### Java signing

```java
String formValue = "merchantId=25&apiId=74&orderId=NO-2514&payCurrency=BTC&payAmount=20.0&receiveCurrency=EUR&receiveAmount=0.0&description=Super+payment&culture=&callbackUrl=http%3A%2F%2FmySite.com%2Fcallback&successUrl=&failureUrl=";
Signature ourSign = Signature.getInstance("SHA1withRSA");
ourSign.initSign(privateKey);
ourSign.update(formValue.getBytes());
return new BASE64Encoder().encode(ourSign.sign());
```

### PHP signing

```php
$data = "merchantId=25&apiId=74&orderId=NO-2514&payCurrency=BTC&payAmount=20.0&receiveCurrency=EUR&receiveAmount=0.0&description=Super+payment&culture=&callbackUrl=http%3A%2F%2FmySite.com%2Fcallback&successUrl=&failureUrl=";
// fetch private key from file and ready it
$private_pem_key = openssl_pkey_get_private($path_to_private_key);
// compute signature
openssl_sign($data, $signature, $private_pem_key, OPENSSL_ALGO_SHA1);
$encodedSignature = base64_encode($signature);
```

## Validating callbacks

Requests coming from SpectroCoin to merchant pages are also signed. Merchant must validate signature of the request.
Request signature to be validated must be **Base64 decoded**.
Request must be converted to **UTF-8 URL encoded concatenated parameters** of one string line including parameter names and ordered in specific sequence specified in documentation.

### Java validation

```java
String parameters = "merchantId=25&apiId=25&orderId=L254S&payCurrency=BTC&payAmount=25.0&receiveCurrency=EUR&receiveAmount=245.0&description=Some+sting+with+symbols+%25%3D%26&orderRequestId=11&status=1";
String responseSign = "qGy2ablxcWtoGVAS2YufkYVWT0jLSilaUtMkz6Z8P6Mn6qInewEfK5Bsn4BFRxg1suENJJF8LJGyJV6vZt3XLmAHoTJwRjWLij2FROdFthuVt/U4Ima6uFm6hkjseeNLvJtdLFYWSAyKkt7wpeLPA2QUspQbG0asOhwd8EeP+mZDSfvOwTv2OFvGWcVEPR6DOWKEaw5wW6ilM8yZKowQzhrqoCUyJN4pxK02PLTKGIb6YDu1nESrN6ebp7ugskYwcmynLWNOY8Tu1bdg0fWBF2uCgJkWpc9yy2UYbtpVO7sCjIe+dmojJzqCwS5/7Ny04Mf+ouj6oEchxHfUq7VpaA==";
Signature verifier = Signature.getInstance("SHA1withRSA");
verifier.initVerify(publicKey);
verifier.update(parameters.getBytes());
byte[] bytes = new BASE64Decoder().decodeBuffer(responseSign);
return verifier.verify(bytes);
```

### PHP validation

```php
$data = "merchantId=25&apiId=25&orderId=L254S&payCurrency=BTC&payAmount=25.0&receiveCurrency=EUR&receiveAmount=245.0&description=Some+sting+with+symbols+%25%3D%26&orderRequestId=11&status=1";
$encodedSignature = "QKpoaBL/P2tcFttFm/TVcn0utkgaOzEAOsZbSSOa+zntcxyJUijaM5egewRoRu68d4CoswTpkdOqaKdGrLWPNqQGujPHIX3q8Q0lK/C8GqN7MYHFrLxu+rpY0G4srIaDzXww4uOTkBIBFWn3TVI4AZAGm0/APlZZeCrhwIIkImYc8ab69zeqikyaMXRK0XMAD/8Fz9b+rUR342hMjFR+epZnNmWQpFtQLvB/SxlCZIZ+u1k2WLJYa7CChDePmdXNHgutvt1mQxLMpJmeDNjD2aOzF9+DPIqsOEkJ9RLJ8F0kQXnn9W02Av/a3GMVC7A/u/kxnKo3LRfkkkAAYkCKug=="
$public_key_pem = openssl_pkey_get_public($path_to_public_spectro_coin_key);
$responseDecodedSign = base64_decode($encodedSignature);
$validity = openssl_verify($data, $responseDecodedSign, $public_key_pem, OPENSSL_ALGO_SHA1);
```

# Example applications

There are several sample SpectroCoin merchant API client applications. You should customize them for your needs.

## Java

Sample Java application could be found [here](https://github.com/SpectroFinance/SpectroCoin-Merchant-Java).

## PHP

Sample PHP application could be found [here](https://github.com/SpectroFinance/SpectroCoin-Merchant-PHP).

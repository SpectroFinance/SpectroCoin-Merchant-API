SpectroCoin Merchant API
========================
This document describes [Spectro Coin](https://spectrocoin.com) merchat service API specification.

# Requirements

* Must have a SpectroCoin account ([Sign Up!](https://spectrocoin.com/en/signup.html)) to setup and test API. For merchant API usage in production you must approve your account.
* Must [generate](README.md#merchant-keypair) private and public key pairs.

# API

## Error result

## createOrder
### Request
### Response
### Callback

# Merchant keypair

You should create keypair (private and public keys) ([Wiki](http://en.wikipedia.org/wiki/Public-key_cryptography)) for your requests to be signed and validated by Spectro Coin.
**Private key** must be kept safely by merchant without any disclosure.
**Public key** must be inserted into configuration of specific Spectro Coin API (Create/Edit form of API details). This key will be used to check signature validity of any merchant API request signed by specific merchant and API.

## Generate with OpenSSL

### Private key generation

```
# generate a 2048-bit RSA private key
openssl genrsa -out "C:\private.pem" 2048

# convert private Key to PKCS#8 format (if you use Java)
openssl pkcs8 -topk8 -inform PEM -outform DER -in "C:\private.pem" -out "C:\private.der" -nocrypt
```

### Public key generation

```
# output public key portion in PEM format
openssl rsa -in "C:\private.pem" -pubout -outform PEM -out "C:\public.pem"
```

# Signature

All API requests must be signed with merchant private key so they can be validated with merchant public key (provided in API configuration). Some API request may result in callback from Spectro Coin, such request are also signed by Spectro Coin and must be validated by merchant using Spectro Coin public key.

## Signing

Request to be signed must be converted to **UTF-8 URL encoded concatinated parameters** of one string line with parameter name and in specific order specified in documentation.
Singature must be **Base64 encoded**.

Example URL encoded concatinated parameters
```
merchantId=169&apiId=1&orderId=L254S&payCurrency=BTC&payAmount=0.0&receiveAmount=20.0&description=Some+string+with+symbols+%25%3D%26&callbackUrl=http%3A%2F%2Ftestas.lt%2Fapi%2Fcheck
```

### Java signing (using sample application)

```java
String formValue = "merchantId=169&apiId=1&orderId=L254S&payCurrency=BTC&payAmount=0.0&receiveAmount=20.0&description=Some+string+with+symbols+%25%3D%26&callbackUrl=http%3A%2F%2Ftestas.lt%2Fapi%2Fcheck";
String orderSign = SignUtil.sign(formValue, path_to_private_key);
```

### PHP signing

```php
$data = "merchantId=169&apiId=1&orderId=L254S&payCurrency=BTC&payAmount=0.0&receiveAmount=20.0&description=Some+string+with+symbols+%25%3D%26&callbackUrl=http%3A%2F%2Ftestas.lt%2Fapi%2Fcheck";
// fetch private key from file and ready it
$private_pem_key = openssl_pkey_get_private($path_to_private_key);
// compute signature
openssl_sign($data, $signature, $private_pem_key, OPENSSL_ALGO_SHA1);
$encodedSignature = base64_encode($signature);
```

## Validating callbacks

Requests comming from Spectro Coin to merchant pages are also signed. Merchant must validate signature of the request.
Request signature to be validated must be **Base64 decoded**.
Request must be converted to **UTF-8 URL encoded concatinated parameters** of one string line with parameter name and in specific order specified in documentation.

### Java validation (using sample application)

```java
String parameters = "merchantId=25&apiId=25&orderId=L254S&payCurrency=BTC&payAmount=25.0&receiveCurrency=EUR&receiveAmount=245.0&description=Some+sting+with+symbols+%25%3D%26&orderRequestId=11&status=1";
String responseSign = "qGy2ablxcWtoGVAS2YufkYVWT0jLSilaUtMkz6Z8P6Mn6qInewEfK5Bsn4BFRxg1suENJJF8LJGyJV6vZt3XLmAHoTJwRjWLij2FROdFthuVt/U4Ima6uFm6hkjseeNLvJtdLFYWSAyKkt7wpeLPA2QUspQbG0asOhwd8EeP+mZDSfvOwTv2OFvGWcVEPR6DOWKEaw5wW6ilM8yZKowQzhrqoCUyJN4pxK02PLTKGIb6YDu1nESrN6ebp7ugskYwcmynLWNOY8Tu1bdg0fWBF2uCgJkWpc9yy2UYbtpVO7sCjIe+dmojJzqCwS5/7Ny04Mf+ouj6oEchxHfUq7VpaA==";
boolean callbackSign = CheckSignUtil.checkSign(parameters, responseSign, path_to_public_spectro_coin_key);
```

### PHP validation

```php
$data = "merchantId=25&apiId=25&orderId=L254S&payCurrency=BTC&payAmount=25.0&receiveCurrency=EUR&receiveAmount=245.0&description=Some+sting+with+symbols+%25%3D%26&orderRequestId=11&status=1";
$encodedSignature = "QKpoaBL/P2tcFttFm/TVcn0utkgaOzEAOsZbSSOa+zntcxyJUijaM5egewRoRu68d4CoswTpkdOqaKdGrLWPNqQGujPHIX3q8Q0lK/C8GqN7MYHFrLxu+rpY0G4srIaDzXww4uOTkBIBFWn3TVI4AZAGm0/APlZZeCrhwIIkImYc8ab69zeqikyaMXRK0XMAD/8Fz9b+rUR342hMjFR+epZnNmWQpFtQLvB/SxlCZIZ+u1k2WLJYa7CChDePmdXNHgutvt1mQxLMpJmeDNjD2aOzF9+DPIqsOEkJ9RLJ8F0kQXnn9W02Av/a3GMVC7A/u/kxnKo3LRfkkkAAYkCKug=="
$public_key_pem = openssl_pkey_get_public($path_to_public_spectro_coin_key);
$responseDecodedSign = base64_decode($encodedSignature);
$validity = openssl_verify($data, $responseDecodedSign, $public_key_pem, OPENSSL_ALGO_SHA1);
```


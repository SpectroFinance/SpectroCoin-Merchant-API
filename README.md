SpectroCoin Merchant API
========================
This document describes [Spectro Coin](https://spectrocoin.com) merchat service API specification.

# Requirements

* Must have a SpectroCoin account ([Sign Up!](https://spectrocoin.com/en/signup.html)) to setup and test API. For merchant API usage in production you must approve your account.
* Must [generate](README.md#Keypair) private and public key pairs.

# API

## Error result

## createOrder
### Request
### Response
### Callback

# Keypair

You should create keypair (private and public keys) for your requests to be signed and validated by Spectro Coin.
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

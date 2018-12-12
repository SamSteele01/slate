---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - json

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---


# Vendible Payment Gateway API specs

<!-- These are some of the ones that we will need:

## Key

not done

- done

group = customer || merchant || vendible

currency = btc || bch || ltc || pivx || iop

## Public

/customer/register

- /merchant/register

- /login

/request-password-reset

- /(currency)/get-new-address

- /(currency)/get-received-by-address

/(currency)/monitor-txn

### Authenticated

/customer/{id}/*

/merchant/{id}/*

/vendible/*

- /merchant/{id}/(currency)/get-account-address

- /merchant/{id}/(currency)/get-addresses-by-accounts

- /merchant/{id}/(currency)/get-balance

- /merchant/{id}/(currency)/get-received-by-account

- /merchant/{id}/(currency)/list-transactions

- /merchant/{id}/(currency)/move

- /merchant/{id}/(currency)/send-from

/customer/{id}/new-user

/merchant/{id}/new-user

- /vendible/create-initial-user

- /vendible/get-account-address

/vendible/new-user

/vendible/get-merchants

/vendible/get-customers

/vendible/transactions

/vendible/(currency)/send-from

/logout

/reset-password -->



## Dev Usage

Create a merchant. Login. Get new account for the currency that is being tested. Create an address to send funds to. Send the funds with your wallet ( outside the payment gateway ).
Use the merchantId in the url when making requests to the currency ( ex: pivx ) endpoints.

<!-- Creating a merchant requires an email address. This should send an email with a url containing a token that will allow the User to create a user account associated with that merchant. -->


# Authentication

This server uses tokens (JWT) as the way to control access to routes. The token is received at login.

```javascript
{
    "token": "JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1NDA2MjQ3NzcsInVzZXJuYW1lIjoiU3RldmUifQ.XAZISgrpE1gAv3fZK3csraAgp0Yxdn1W8s3rt2Zv4R4",
    "expires": "2018-10-27T03:19:37-04:00",
    "userId": "5bca8e024701c49073e894ea", // testing
    "merchantId": "5bca8e024701c49073e894e9" // testing
}
```

It can be passed in either the url as a query parameter.

`...?token=eyJ0eXAiOiJKV1...`

Or in the request header.

`{ Authorization: JWT eyJ0eXAiOiJKV1... }`

All routes with a `/<group>/{id}/` in the url require authentication.


# Merchant

## Create new merchant

this creates a merchant and a user with "owner" role.

### HTTP Request

POST `<rootUrl>/api/v1/merchant/register`

### Body Parameters

<!-- send: -->

<!-- ```javascript
{
    "merchantName": <string>,
    "username": <string>,
    "password": <string>,
    "email": <string>,
    "role": <string> // "owner", "admin", or "user"
}
``` -->

Parameter | Type | Note | Description
--------- | ---- | ---- | -----------
merchantName | string | must be unique | The name of the merchant
username | string | must be unique |
password | string |
email | string |
role | string | "owner", "admin", or "user" | determines level of access 


```javascript
can return: (example)
{
    "message": "Merchant created.",
    "merchantId": "5bca8e024701c49073e894e9",
    "user": {
        "_id": "5bca8e024701c49073e894ea",
        "username": "Steve",
        "email": "steve@example.com",
        "role": "owner",
        "belongsTo": {
            "parentGroup": "merchant",
            "id": "5bca8e024701c49073e894e9"
        },
        "created_at": "2018-10-20T02:08:02.488Z",
        "updated_at": "2018-10-20T02:08:02.488Z",
        "__v": 0
    }
}
```


```javascript
but for now:
{
  "message": "Merchant created.",
  "merchantId": "5bca921661238e9119a0faea",
  "user": "5bca921661238e9119a0faeb"
}
```

<aside class="success">
Note — this must be done to obtain an API key!
</aside>

## Add additional users

POST `<rootUrl>/api/v1/merchants/{id}/merchUsers?access_token=$ACCESS_TOKEN`

that would be the link sent in an auto email for a two step initial

send: `{ "email": "<string>", "password": "<string>" }`


### Login

POST `<rootUrl>/api/v1/login`

send:

```javascript
{
    "username": <string>,
    "password": <string>
}
```

returns:

```javascript
{
    "token": "JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1NDA2MDk0ODMsInVzZXJuYW1lIjoiU3RldmUifQ.Pgq3QvBNlj0X0PvZF34NFXSYPt0rEuIacIyrbfvYLfY",
    "expires": "2018-10-26T23:04:43-04:00",
    "userId": "5bca8e024701c49073e894ea"
}
```

## Confirm email

A email with a link will be sent when a user registers. Getting from this URL will update the DB for that user with `emailVerified: true`.

GET `<rootUrl>/api/v1/users/confirm-email?token={token}`

### Request Password Reset

The User will be looked up by their username. An email will be sent to the address on file.

POST `<rootUrl>/api/v1/request-password-reset`

send:

```javascript
{
	"username": "Cypress"
}
```

returns:

```javascript
{
    "message": "Check your email."
}
```

### Change Password

The token is sent in an email in a link to the Admin Panel, which takes a new password and POSTs to this URL.

POST `<rootUrl>/api/v1/change-password?token={token}`

send:

```javascript
{
	"password": <new password>
}
```

returns:
```javascript
{
    "message": "Password changed!"
}
```

-----

### Get an Account

The next step is to create an account for each currency that the merchant wants to accept. These will be protected routes. ( token in the headers )

POST `<rootUrl>/api/v1/merchant/{id}/{currency}/get-account-address`

returns:
```javascript
{
    "message": "Created new pivx accounts.",
    "accounts": [
        {
            "accountName": "Mariah Test receiving",
            "accountNumber": "xzwV4oKTr28pEorm2u5Rvfj2CWyEYHs8QZ",
            "currency": "pivx",
            "type": "receiving"
        },
        {
            "accountName": "Mariah Test available",
            "accountNumber": "yFvYfuzHye8AURhNtkytkwd9MeHUuL266d",
            "currency": "pivx",
            "type": "available"
        }
    ]
}
```


<!-- ### Get Transactions

Transactions stored in the DB.

GET `<rootUrl>/api/v1/merchants/{id}/transactions`

optionally filtered with query string

GET `<rootUrl>/api/v1/merchants/{id}/transactions?count=20&from=0`

or with parameters in the request body

POST `<rootUrl>/api/v1/merchants/{id}/transactions`

send:
```javascript
{

}
```

returns:
```javascript
{

}
``` -->

-----

## Customer

Same API as merchant

-----

## Bitcoin RPC currencies

Any currency forked or copied from Bitcoin shares the same API. For this payment gateway those are: Bitcoin (btc), Bitcoin cash (bch), Litecoin (ltc), Pivx (pivx), and IoP (iop).


[Typical uses](https://github.com/PIVX-Project/PIVX/wiki/Accounts-Explained#Typical_Uses)

The publicly accessible end points will be:

`<rootUrl>/api/v1/{currency}/<method>`

This requires the merchantId to be sent in the req.body. Later we can make the merchantId be passed in the url.


### Create new account

POST `<rootUrl>/api/v1/merchant/{id}/{currency}/get-account-address`

Makes an account with the same name as merchantName. Calling multiple times will return the address already created.

returns:
```javascript
{
    "message": "Created new <currency> accounts.", // only appears if the accounts were just created
    "accounts": [
        {
            "_id": "5bd6aadb6a29f10633554930",
            "accountName": "Coffee House receiving",
            "accountNumber": "y5Sc5zcpHbjQwj8cRsGjSFhBjBBWWnigiP",
            "currency": "pivx",
            "type": "receiving"
        },
        {
            "_id": "5bd6aadb6a29f10633554931",
            "accountName": "Coffee House available",
            "accountNumber": "y4jtHoXhEorbNXYCpjVdUREJDF4JqwMbiP",
            "currency": "pivx",
            "type": "available"
        }
    ]
}
```

### Create Address

The first step needed for a transaction to be started is to create an address.

POST `<rootUrl>/api/v1/{currency}/get-new-address`

send: (required)
```javascript
{
    "merchantId": "5bca921661238e9119a0faea"
}
```

returns:
```javascript
{
    "address": "xxXWxYLpCBR5v6jKVAJ8sjV91vynV6q8YW"
}
```

or, if the merchant hasn't created an account for that currency yet:
```javascript
{
    "message": "That currency is not accepted by the merchant"
}
```

This creates an address in the merchant's receiving account.
Returns that address to the merchant UI. The merchant can display a QR code or string for the customer.

### Send amount

Send funds to the address returned from 'get-new-address'. For an unauthenticated user this transaction occurs outside of the payment gateway. ( We need some way to detect when this goes through. I'm thinking of pushing created addresses to an array and spawning a process whenever .length > 0 that calls getReceivedByAddress at a set interval. )

An authenticated user can 'move' funds from their Account to a Merchant's account without fees ( same wallet ). They could also use the API to 'sendFrom'.


### Get Received By Address

This returns 0 until the transaction gets confirmed. Calling this with the front end is a way to check

POST `<rootUrl>/api/v1/{currency}/get-received-by-address`

send:

```javascript
{
    "address": "<address>",
    "minconf": "<number>" // optional
}
```

minconf can be 0 - 6. 0 is unconfirmed and therefore instant. Default is 1.

receive:

```javascript
{
    "amount": <float>
}
ex:
{
    "amount": 0.002
}
```


### Monitor Transaction

`<rootUrl>/api/v1/{currency}/monitor-txn`

should be used to set up a webhook with the client/merchant site, and start a listening process on the server.


### Get Addresses By Account

POST `/merchant/{id}/{currency}/get-addresses-by-accounts`

send:
```javascript
{
    "type": "receiving" // or "available"
}
```

returns:
```javascript
{
    "addresses": [
        "yKXCtksdxL1K9ZiCzB58Mj4MkbZoWPoBo2",
        "yAKDK9HtqQ93EcVTRJPpJSe4equ1oHenqB",
        "xxXWxYLpCBR5v6jKVAJ8sjV91vynV6q8YW",
        "y8VARigKaLgT5Mkynfandqpe9XbQYHhQCb"
    ]
}
```


### Get Balance

This returns the current balance of the account.

send:
```javascript
{
	"type": "receiving" // or "available"
}
```

returns:
```javascript
{
    "accountBalance": 0.0004
}
```

### Get Received By Account

This returns the total of all "sends" to this address. It does not include any "move"s, "sendFrom"s or represent the current total.

send:
```javascript
{
	"type": "receiving" // or "available"
}
```

returns:
```javascript
{
    "fundsReceived": 0.004
}
```

### List Transactions

POST `/merchant/{id}/{currency}/list-transactions`

send:
```javascript
{
    "type": "receiving"
}
```

returns:
```javascript
{
    "transactions": [
        {
            "account": "Doug Test receiving",
            "address": "xzSYVE6PMDPW9jc8QvBMGXBSwiuyMkRZUz",
            "category": "receive",
            "amount": 0.003,
            "vout": 1,
            "confirmations": 2569,
            "bcconfirmations": 2569,
            "blockhash": "64c62cb74e76d1dcac0cccc962bce467cb12fe8db3edb7f971fef00d5e7d7d9d",
            "blockindex": 2,
            "blocktime": 1541096299,
            "txid": "010322caf9cf5b220d7c1d053d9928bd24bc78cffff78660fa6c147430757d0f",
            "walletconflicts": [],
            "time": 1541096235,
            "timereceived": 1541096235
        },
        {
            "account": "Doug Test receiving",
            "address": "QcmTTdMuU5Yxhdw243XmTVpvyN9sctegKv",
            "category": "receive",
            "amount": 0.001,
            "label": "Doug Test receiving", // not in PIVX
            "vout": 1,
            "confirmations": 168,
            "blockhash": "91fdfdd0b886317731203fdefeb51fd5aec571b76d5968105e6a0c8969dcc1a1",
            "blockindex": 2,
            "blocktime": 1543478713,
            "txid": "e8283d469dd65e4f979b419cc074d3c7ac5250bc64feb6750b923b4d5a5d623c",
            "walletconflicts": [],
            "time": 1543478625,
            "timereceived": 1543478625,
            "bip125-replaceable": "no"
        },
        {
            "account": "Doug Test receiving",
            "category": "move",
            "time": 1543478774,
            "amount": -0.000005,
            "otheraccount": "vendible fee",
            "comment": "process fees"
        },
        {
            "account": "Doug Test receiving",
            "category": "move",
            "time": 1543478774,
            "amount": -0.000995,
            "otheraccount": "Doug Test available",
            "comment": "process net payments"
        }
    ]
}
```

### Move

This moves funds from one account to another without processing fees. It does not get mined. Only works for accounts in the same wallet. Currently exposed just for testing.

POST `/merchant/{id}/{currency}/move`

send:
```javascript
{
    "type": "available", // or "receiving"
    "toAccount": "Doug Test available", // has to be the string accountName
    "amount": "0.0023",
    "minconf": "1",
    "comment": "This is a test of the move function"
}
```

returns:
```javascript
{
    "move": true
}
```

### Process Payments

This takes the balance of the 'receiving' account, calculates and moves the fee to Vendible's Fee account, and moves the remainder to the merchant's 'available' account.

POST `/merchant/{id}/:currency/process-payment`

returns:
```javascript
{
  "message": "Payments processed"
  // or
  "message": "The account balance is 0. Nothing to process."
}
```

### Send From

How to get funds out of a Vendable account. The "toAddress" is from a wallet outside the Vendable system.

POST `<rootUrl>/api/v1/customer/{id}/{currency}/send-from`

send:
```javascript
{
	"toAddress": "y1C3iJwUfQQbK4b4CGbzpaUPM8CuECrzUC",
	"amount": "0.0006",
	"minconf": "1",
	"comment": "This is a test of the sendFrom function",
  "commentTo": "Message to receiver"
}
```

returns:
```javascript
{
  "transactionId": "f3cc1c52ac13abffbe012065b4abddc89daa1d9690a27623c95b2707fb682f01"
}
```

-----

## PIVX

Pivx has additional methods in its API

-----

## Vendible

### Create initial user

This creates the first user for Vendible. Should be deactivated after doing so as it is an unprotected route.

POST `/vendible/create-initial-user`

send:
```javascript
{
    "merchantName": <string>,
    "username": <string>,
    "password": <string>,
    "email": <string>,
    "role": <string> // "owner", "admin", or "user"
}
```

returns:
```
{
    "message": "Vendible user account created.",
    "userId": "5bfcdecf2065127eefc87954"
}
```

### Create Accounts

Fee accounts need to be created for each currency to receive fees. This needs to be done before the Payment Gateway can be used. Other types of accounts could be created.

POST `/vendible/get-account-address`

send:
```javascript
{
	"currency": "ltc",
	"type": "fee"
}
```

returns:
```
{
    "message": "Created new ltc account.",
    "vendableAccount": {
        "_id": "5bfce62fb2f0227fa47b6aef",
        "accountName": "vendible fee",
        "defaultAddress": "QbmuQxREc4Qhj3TUBaWGfvub9aVzgPFWBE",
        "currency": "ltc",
        "type": "fee",
        "__v": 0
    }
}
```

***@dev*** I think the following section is out of location -Aaron

Response attributes:

*transactions* : Array of transactions.

Example response

```
[{
    "transactionID": "akld923982983dkdfjfjlsf",
    "status": "pending"
},
{
    "transactionID": "akld92398sdf2983dkdfjfjlsf",
    "status": "pending"
}
{
    "transactionID": "ld923982983dkdfjfjlsf",
    "status": "success"
}
{
    "transactionID": "294ld923982983dkdfjfjlsf",
    "status": "success"
}]
```
[Back to README](README.md)

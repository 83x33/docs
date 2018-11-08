# General

## Server

<table class="table" style="width: auto;">
  <tbody>
    <tr>
      <th>url</th>
      <th>context</th>
    </tr>
    <tr>
      <td>https://api.8333.io</td>
      <td>mainnet</td>
    </tr>
    <tr>
      <td>https://testnet.8333.io</td>
      <td>testnet</td>
    </tr>
  </tbody>
</table>


## Authentication
You can authenticate with 8333.io API via your `apikey` which you can find in the settings tab of your dashboard.

apikey's should be passed in the `authorization: apikey *yourApiKey*` request header, but can also be passed in the `body` or `query`, always under the parameter name `apikey`.


## Rate Limit
Some endpoints are rate limited. You'll find `X-RateLimit-Limit` and `X-RateLimit-Remaining` headers in the reponse.






# Wallets

Wallets on 8333.io are **read only** access of your **existing** Bitcoin wallets.  
Each wallet is bind to a unique **bip32 extented public key** , stored in the `xpub` field.  
Many Bitcoin wallets support **bip32**, including Electrum, Trezor, Ledger, Samourai...  

All the [slip0132](https://github.com/satoshilabs/slips/blob/master/slip-0132.md) `pubkey`'s format are supported :

<table class="table" style="width: auto;">
  <tbody>
    <tr>
      <th>format</th>
      <th>context</th>
    </tr>
    <tr>
      <td>xpub, ypub, zpub</td>
      <td>mainnet</td>
    </tr>
    <tr>
      <td>tpub, upub, vpub</td>
      <td>testnet</td>
    </tr>
  </tbody>
</table>


## Create / Update

**endpoints**
- create: POST `/wallets`
- update: PUT  `/wallets/:id`


<table class="table">
  <tbody>
    <tr>
      <th>param</th>
      <th>meta</th>
      <th class="w-50 d-none d-sm-table-cell">info</th>
    </tr>
    <tr>
      <td><code>name</code></td>
      <td>string / required</td>
      <td class="w-50 d-none d-sm-table-cell"></td>
    </tr>
    <tr>
      <td><code>xpub</code></td>
      <td>string / required</td>
      <td class="w-50 d-none d-sm-table-cell"></td>
    </tr>
    <tr>
      <td><code>ticker</code></td>
      <td>string</td>
      <td class="w-50 d-none d-sm-table-cell">Which ticker is used to display the invoice price. You can either use a <bold>Fiat</bold> or <bold>BTC</bold> ticker.<br/><br/>Fiat ticker => <code>usd</code>,<code>eur</code>,<code>jpy</code>,<code>gbp</code>,<code>cad</code><br/>BTC ticker => <code>btc</code>,<code>mbtc</code>,<code>satoshis</code></td>
    </tr>
    <tr>
      <td><code>fiat</code></td>
      <td>string</td>
      <td class="w-50 d-none d-sm-table-cell">Fiat ticker to check prices against.<br/><br/>Fiat ticker => <code>usd</code>,<code>eur</code>,<code>jpy</code>,<code>gbp</code>,<code>cad</code></td>
    </tr>
    <tr>
      <td><code>expires</code></td>
      <td>integer</td>
      <td class="w-50 d-none d-sm-table-cell">Number of minutes without receiving payment before the invoice change it's status to <code>expired</code></td>
    </tr>
    <tr>
      <td><code>confs</code></td>
      <td>integer</td>
      <td class="w-50 d-none d-sm-table-cell">Number of block confirmations before the invoice change it's status to <code>confirmed</code></td>
    </tr>
  </tbody>
</table>

**Request**

```bash
curl -X POST "https://api.8333.io/wallets" \
  -H 'content-type: application/json' \
  -H 'authorization: apikey *yourApiKey*' \
  -d '{ "name": "The Moon ☾ Wallet", "xpub": "xpub661MyMw.....Mr7XscfK" }'
```
<br/>

**Response**

```json
{
  "id": "HyL0n1eBz",
  "name": "The Moon ☾ Wallet",
  "xpub": "xpub661MyMw.....Mr7XscfK",
  "ticker": "mbtc",
  "fiat": "usd",
  "expires": 20,
  "confs": 0
}
```






# Invoices  

Invoices are linked to a wallet via the `wid` field.  
Each invoice you create is automatically assigned to a bitcoin address, derived from your wallet extended pubkey. By default there is no address reuse, unless you specify the option in your wallet.  

Note that "address reuse" only concern addresses which are tied to **expired** invoices. Addresses bound to **confirmed** invoices will never be reuse. Address reuse aims to keep your addresses gap limit low.

Specifying an invoice `amount` is not required. The `amount` parameter is not stored in the invoice, instead it is transformed to a **price snapshot** stored in the field `price`. If no amount is specified, the `price` will be set when someone will pay the invoice.

## Status

Invoices can have 4 different `status`.
 - **pending** : Default. set at invoice creation.
 - **received** : Soon as the transaction is received by the network.
 - **confirmed** : Depend on the `block confirmation` setting you choose.
 - **expired** : No bitcoins received before invoice expiration time

**Important note** : the `confirmed` status does not necessearly reflect the transaction settlement on the blockchain. This will depend on the `block confirmation` setting you choose (defined in your wallet settings). Block confirmation can be set to `0`, in that case invoice status will switch from `pending` directly to `confirmed` as soon as the transaction has been received by the network (and not yet included in a block). Depending on your business, you might want to set invoice status to `confirmed` only when the transaction has been included in block. To do so, set the block confirmation setting to the number of block you wish to wait for.


## Create

**endpoints**
- create: POST `/invoices`
- update: PUT  `/invoices/:id`

<table class="table">
  <tbody>
    <tr>
      <th>param</th>
      <th>meta</th>
      <th class="w-50 d-none d-sm-table-cell">info</th>
    </tr>
    <tr>
      <td><code>wid</code></td>
      <td>string</td>
      <td class="w-50 d-none d-sm-table-cell">wallet id. If not provided, it will bind to your first wallet.</td>
    </tr>
    <tr>
      <td><code>amount</code></td>
      <td><pre><code>{
    "value": 0.2
  , "ticker": "btc"
  , "fiat_ticker": "usd"
}</code></pre></td>
      <td class="w-50 d-none d-sm-table-cell"></td>
    </tr>
  </tbody>
</table>

**Request**

```bash
curl -X POST "https://api.8333.io/invoices" \
  -H 'content-type: application/json' \
  -H 'authorization: apikey *yourApiKey*' \
  -d '{ "wid": "HyL0n1eBz", "amount": { "value": 0.2, "ticker": "btc" } }'
```
<br/>

**Response**

```json
{
  "id": "zyVgt1lHf",
  "wid": "HyL0n1eBz",
  "address": "1Eztd9KpVXsbqFKeTDQ67NuXxh7Mq71uYp",
  "status": "pending",
  "eat": "2018-01-19T22:20:32.275Z",
  "price": {
    "satoshis": 20000000,
    "btc": 0.2,
    "mbtc": 200,
    "fiat": 233147,
    "unit": 1165735,
    "fiat_formated": "$2,331.47",
    "unit_formated": "$11,657.35",
    "ticker": "btc",
    "fiat_ticker": "usd",
    "ts": 1516399880173
  }
}
```

## Public link

Each invoice comes out of the box with it's public url:

```bash
https://www.8333.io/invoice/{invoiceId}
``` 










# Price utils

All princing utilities uses a live `price index` calculated on the fly from multiple sources. The number of sources used at calculation can vary because not all sources provides the currency/pair requested.  

Current sources: `bitfinex`, `bitstamp`, `coinbase`, `okcoin`   

There is 2 types of tickers :  
- BTC tickers  => `btc`, `mbtc`, `satoshis`
- Fiat tickers => `usd`, `eur`, `jpy`, `gbp`, `cad`


## Snapshots

Return prices informations given an `amount` & a `ticker`. Informations consist of the amount priced in bitcoin and fiat, it also return the timestamp of the snapshot.

**endpoints**
- create: GET `/prices/:amount/:ticker`

<table class="table">
  <tbody>
    <tr>
      <th>param</th>
      <th>meta</th>
      <th class="w-50 d-none d-sm-table-cell">info</th>
    </tr>
    <tr>
      <td><code>amount</code></td>
      <td>float / required</td>
      <td class="w-50 d-none d-sm-table-cell"></td>
    </tr>
    <tr>
      <td><code>ticker</code></td>
      <td>string / required</td>
      <td class="w-50 d-none d-sm-table-cell">Fiat or Btc ticker.<br/><br/>Note: when using a BTC ticker, you can also append a Fiat ticker to check against like this <code>btcTicker-fiatTicker</code><br/><br/>ex: <code>/prices/340000/satoshis-jpy</code></td>
    </tr>
  </tbody>
</table>

**Request**

```bash
curl "https://api.8333.io/prices/0.2/btc"
```
<br/>

**Response**

```json
{
  "satoshis": 20000000,
  "btc": 0.2,
  "mbtc": 200,
  "fiat": 233147,
  "unit": 1165735,
  "fiat_formated": "$2,331.47",
  "unit_formated": "$11,657.35",
  "ticker": "btc",
  "fiat_ticker": "usd",
  "ts": 1516399880173
}
```

## Bitcoin price

Get current Bitcoin price value in fiat.

**endpoints**
- create: GET `/prices/:fiat-ticker`

**Request**

```bash
curl "https://api.8333.io/prices/jpy"
```
<br/>

**Response**

```json
{
  "fiat_ticker": "jpy",
  "fiat": 70503600,
  "fiat_formated": "￥705,036.00",
  "ts": 1539253258941
}
```









# Pubkey Utils

## Address derivation

**endpoints**
- GET  `/utils/derive/:pubkey`

<table class="table">
  <tbody>
    <tr>
      <th>param</th>
      <th>meta</th>
      <th class="w-50 d-none d-sm-table-cell">info</th>
    </tr>
    <tr>
      <td><code>start</code></td>
      <td>integer</td>
      <td class="w-50 d-none d-sm-table-cell"></td>
    </tr>
    <tr>
      <td><code>end</code></td>
      <td>integer</td>
      <td class="w-50 d-none d-sm-table-cell"></td>
    </tr>
  </tbody>
</table>

**Request**

```bash
curl -X POST "https://api.8333.io/utils/derive/xpub6CUGRUonZSQ4TWtTMmzXdrXDtypWKiKrhko4egpiMZbpiaQL2jkwSB1icqYh2cfDfVxdx4df189oLKnC5fSwqPfgyP3hooxujYzAu3fDVmz" \
  -H 'content-type: application/json' \
  -H 'authorization: apikey *yourApiKey*'
```
<br/>

**Response**

return an array with 2 values. First is an array conatining the derived addresses, second is an object contening informations about the process.

```json
[
  [
    "1EfgV2Hr5CDjXPavHDpDMjmU33BA2veHy6",
    "12iNxzdF6KFZ14UyRTYCRuptxkKSSVHzqF",
    "1CcEugXu9Yf9Qw5cpB8gHUK4X9683WyghM",
    "15xANZb5vJv5RGL263NFuh8UGgHT7noXeZ",
    "1PJMBXKBYEBMRDmpAoBRbDff26gHJrawSp"
  ],
  {
    "network": "mainnet",
    "pubkeyType": "p2pkh",
    "prefix": "xpub"
  }
]
```

## Wallet balance

This endpoint is very handy if you are building a wallet. Given a `pubkey`, you get back all the utxos associated. The sum of all those utxos constitute your wallet `balance`. The gap limit used during the scan is `5` unused addresses. Utxos are ordered by `receiving` and `change` address type.

**endpoints**
- GET  `/utils/wallet/:pubkey`

**Request**

```bash
curl -X POST "https://api.8333.io/utils/wallet/xpub6CUGRUonZSQ4TWtTMmzXdrXDtypWKiKrhko4egpiMZbpiaQL2jkwSB1icqYh2cfDfVxdx4df189oLKnC5fSwqPfgyP3hooxujYzAu3fDVmz" \
  -H 'content-type: application/json' \
  -H 'authorization: apikey *yourApiKey*'
```
<br/>

**Response**

```json
{
    "balance": 489068,
    "balance_btc": 0.00489068,
    "utxos_count": 7,
    "utxos": {
      "received": {
        "balance": 489068,
        "utxos": [
          {
            "tx_hash": "d398308c6b1123d53f0c669146ad9772164dacdf4df4c79d26e657264f20250f",
            "tx_pos": 0,
            "height": 425673,
            "value": 8747,
            "address": "1CcEugXu9Yf9Qw5cpB8gHUK4X9683WyghM"
          },
          {
            "tx_hash": "9922f59186cb133769bee41876bd8c0186dee21a005823d4e28ddaf6e324805c",
            "tx_pos": 1,
            "height": 472199,
            "value": 100000,
            "address": "15xANZb5vJv5RGL263NFuh8UGgHT7noXeZ"
          },
          ...
        ],
        "address": "1MS6eGqD4iUGyJPbEsjqmoNaRhApgtmF8J",
        "cursor": 10,
        "exec_time": "435 ms",
        "balance_btc": 0.00489068
      },
      "change": {
        "balance": 0,
        "utxos": [],
        "address": "1muF2Eq9iR4ttJKpc4zZkoTmu3E41Ab9v",
        "cursor": 0,
        "exec_time": "121 ms",
        "balance_btc": 0
      }
    }
}
```


## Utxos by address

Retreive utxos for a given address. you can specify multiple addresses to scan, just separate them with a `,`.

**endpoints**
- GET  `/utils/utxos/:addresses`


**Request**

```bash
curl -X POST "https://api.8333.io/utils/utxos/1CcEugXu9Yf9Qw5cpB8gHUK4X9683WyghM" \
  -H 'content-type: application/json' \
  -H 'authorization: apikey *yourApiKey*'
```
<br/>

**Response**

```json
{
  "1CcEugXu9Yf9Qw5cpB8gHUK4X9683WyghM": [
    {
      "tx_hash": "d398308c6b1123d53f0c669146ad9772164dacdf4df4c79d26e657264f20250f",
      "tx_pos": 0,
      "height": 425673,
      "value": 8747,
      "address": "1CcEugXu9Yf9Qw5cpB8gHUK4X9683WyghM"
    }
  ]
}
```



# SDK

You can consume the 8333.io platform trhough our Sotware Development Kit's.

 - NodeJS/Browser - [https://github.com/83x33/sdk.js](https://github.com/83x33/sdk.js)
 - IOS : soon
 - Android : soon


## NodeJS/Browser

The NodeJS/Browser SDK is open source and available on github [https://github.com/83x33/sdk.js](https://github.com/83x33/sdk.js)


**Node**

```js
// install
npm install sdk33 --save

// ES module
import sdk33 from 'sdk33'

// or UMD lib
import sdk33 from 'sdk33/dist/sdk.umd.js'
```
   

**Browser**

```html
<!-- include SDK dependencies -->
<script type="text/javascript" src="https://unpkg.com/axios@0.18.0/dist/axios.min.js"></script>
<script type="text/javascript" src="https://unpkg.com/socket.io-client@2.1.1/dist/socket.io.js"></script>

<!-- include SDK -->
<script type="text/javascript" src="https://unpkg.com/sdk33@latest"></script>
```

## Widget

The widget is an out of the box visual interface for invoices. You can choose between 2 functions to initiate the widget. Use the one that fit your business/project requirements.

> Note: you have total control over invoice creation (wether you create them server side, or client side without an apikey. By default, the API block invoice creation if no secret `apikey` is provided. To allow unauthenticated invoice creation, just disable the option in your wallet settings. 

- **payInvoice()**: Create invoices server side, using your apikey, and forward the invoice ID client side, then initiate the widget by calling the `payInvoice()` SDK function, with the invoice ID as first parameter. To make workflow simple, create a wrapper function (`PayWithBitcoin()` in the exemple below), and use it wherever it make sense (attached to an `onclick` event of a payment link or button for exemple).

  ```html
  <html>
    <head>
      <script type="text/javascript" src="https://unpkg.com/axios@0.18.0/dist/axios.min.js"></script>
      <script type="text/javascript" src="https://unpkg.com/socket.io-client@2.1.1/dist/socket.io.js"></script>
      <script type="text/javascript" src="https://unpkg.com/sdk33@latest"></script>
    </head>
    <body>
      <script type="text/javascript">
        function PayWithBitcoin(){
          sdk33.payInvoice('invoiceID', (status, widget) => {
            // custom logic on status change
            if(status == 'success') doSomething()
          })
        }
      </script>

      <a onclick="PayWithBitcoin()">
        Click to Pay
      </a>
    </body>
  </html>
  ```
   

- **payNewInvoice()**: Allows you to generate invoices on the fly, no need for an invoice id (the function takes care of creating the invoice). It's the same logic as above, except the function takes an invoice object as first argument. The invoice object must contain a `wid` key specifing which wallet ID the newly created invoice will be ratached to. The `amount` is optionnal. If you are using a `testnet` wallet, don't forget to add a `network` key to the invoice object, with a value of `testnet`.

  ```html
  <script type="text/javascript">
    function PayWithBitcoin(amount){
      sdk33.payNewInvoice({ wid: 'walletID', amount }, (status, widget) => {
        // custom logic on status change
        if(status == 'success') doSomething()
      })
    }
  </script>

  <a onclick="PayWithBitcoin('20-eur')">
    Click to Pay
  </a>
  ```
   
> Note: if you use the `payNewInvoice()` client side (without an apikey), in order to successfuly generate an invoice you must disable the "authenticated only invoice creation" option in your wallet settings.


## Handler

Both `payInvoice()` & `payNewInvoice()` takes a handler function as their last argument.The handler is where you can write some custom logic based on the current widget status, it get's called everytime the status changes. You can receive up to 3 different `status` upon the payment cycle (depending on your "confirmation blocks" setting).

<table class="table">
  <tbody>
    <tr>
      <th>status</th>
      <th style="width: 80%" class="d-none d-sm-table-cell">info</th>
    </tr>
    <tr>
      <td><code>received</code></td>
      <td class="d-none d-sm-table-cell">This status is sent when the transaction get's received by the bitcoin network, only if the invoice uses a value > 0 for confirmation blocks settings. The transaction is just received, it's not yet included in a block.<br/><br/>When you receive this status, the user is the "Settling Payment" step of the widget.</td>
    </tr>
    <tr>
      <td><code>confirmed</code></td>
      <td class="d-none d-sm-table-cell">If the invoice uses 0 confirmation blocks (aka instant/zero confs) then technically, you receive the status at the time the transaction get's received by the bitcoin network, but just received, not yet included in a block. <br/><br/>On the other hand, an invoice setting with > 0 (1 to n) confirmation blocks will receive this status at the time the n'th block gets added over the block transaction. A block is created about every 10min on average.<br/><br/>When you receive this status, the user is in the "Payment Success" step of the widget, he can still interact with widget, even though the payment is confirmed. (he might want to forward the receipt to his email)</td>
    </tr>
    <tr>
      <td><code>success</code></td>
      <td class="w-50 d-none d-sm-table-cell">This status is sent (only if the invoice status is confirmed) when the user is done interacting with the widget. This is the appropriate time and status to hook up your custom logic about what happens once payment process is done.</td>
    </tr>
  </tbody>
</table>

**Basic Handler**

Ex: After the payment process is done, redirect to another page.

```js
(status) => {
  if(status == 'success') windows.location = '/thankyou.html'
}
```
   

**Advanced Handler**

Asserting custom logic against the `success` status is probably the payment scenario that will fit the most usecases. Know that it's very simple to customize the UX using the 2 params `status` & `widget`.

Ex: Let's says you're building an online game where users can buy credits, lives, kittens or whatever. You want to make sure transactions get's included under at least 6 blocks before you credit the user (so you set 6 blocks confs in your wallet setting). You might want to close the widget as soon as the transaction is received, so the user can get back to the game righ away, and only trigger the credit once the 6 blocks has passed, which correspond to the `confirmed` status :

```js
(status, widget) => {
  if(status == 'received') widget.close()
  if(status == 'confirmed') creditKittens()
}
```
   

Handlers takes a widget instance as a second parameter. Note that in this case, because we close the widget programmatically (by calling `widget.close()`) before the user had the chance to manually close it, the handler will then never received a `success` status, which is only sent when yhe user manually close the widget of a confirmed invoice.

**Callback hook**

Additionnaly, you can handle the status changes server side by specifying a callback hook url in you wallet settings. The hook url get's called on every new status, via a POST request. It sends the json invoice, containing the current status, and other invoice informations, such as invoide ID, price etc..

```js
{ "id":"invoiceID", "status":"confirmed" }
```

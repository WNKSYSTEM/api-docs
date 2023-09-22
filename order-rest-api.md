# Order Rest API for Exchange 
# General API Information
* All endpoints return either a JSON object or array.
* Data is returned in **ascending** order. Oldest first, newest last.
* All time and timestamp related fields are in milliseconds.
* HTTP `4XX` return codes are used for for malformed requests;
  the issue is on the sender's side.
* HTTP `5XX` return codes are used for internal errors; the issue is on Exchange's side.
  It is important to **NOT** treat this as a failure operation; the execution status is
  **UNKNOWN** and could have been a success.
* Any endpoint can return an ERROR; the error payload is as follows:
```javascript
{
  "code": 500,
  "msg": "Invalid symbol."
}
```

* Specific error codes and messages defined in another document.
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a
  `query string` or in the `request body` with content type
  `application/x-www-form-urlencoded` or `application/json`.

# Endpoint security type
* Each endpoint has a security type that determines the how you will
  interact with it.
* API-keys are passed into the Rest API via the `X-BIFRAX-APIKEY`
  header.
* API-keys and secret-keys **are case sensitive**.
* API-keys can be configured to only access certain types of secure endpoints.
* By default, API-keys can access all secure routes.

# SIGNED (Private) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string` or `request body`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* `totalParams` is must be sorted `alphabetically` for signatures.
* The `signature` is **not case sensitive**.
* `totalParams` is defined as the `query string` concatenated with the
  `request body` include `application/json`.

## Orders endpoints

### Infomation

* ExchangeType

`LOCAL`: local currency exchanges(piat currency)


### New order (TRADE)
```
POST /v1/order  (HMAC SHA256)
```
Send in a new order.

**Parameters:**


Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
exchangeType | STRING | YES | 
price | DECIMAL | NO |
quantity | DECIMAL | YES |
side | ENUM | YES | `BUY`, `SELL`
symbol | STRING | YES |
type | ENUM | YES | `MARKET`, `LIMIT`
signature | STRING | YES |


Additional mandatory parameters based on `type`:

Type | Additional mandatory parameters
------------ | ------------
LIMIT | `quantity`, `price`
MARKET | `quantity`


### Example
* **`totalParams` is must be sorted `alphabetically` for signatures.**

* **requestBody:** 
  
  exchangeType=LOCAL&price=0.1&quantity=1&side=BUY&symbol=LTC/BTC&type=LIMIT
  
  `or`
  
  {"exchangeType":"LOCAL", "price":"0.1", "quantity":"1", "side":"BUY", "symbol":"LTC/BTC", "type":"LIMIT"}
  
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "exchangeType=LOCAL&price=0.1&quantity=1&side=BUY&symbol=LTC/BTC&type=LIMIT" | openssl dgst -sha256 -hmac "testSecretKey"
    (stdin)= aa406abc91c8a05f9e3e5d8aa31a9c6ef6ef015ec008f9c6334faa8fdc9c1b96
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-BIFRAX-APIKEY: testApiKey" -X POST 'https://api.bispex.com:3000/v1/order' -d 'exchangeType=LOCAL&price=0.1&quantity=1&side=BUY&symbol=LTC/BTC&type=LIMIT'


**Response RESULT:**
```javascript
{
    "orderId": "344287",
    "clientOrderId": "201903100228344287",
    "symbol": "BTC/KRW",
    "quantity": "0.003"
}
```
### Cancel order (TRADE)
```
POST /v1/order/cancel  (HMAC SHA256)
```
Cancel an active order.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
exchangeType | STRING | YES | 
origClientOrderId | STRING | YES |
origOrderId | STRING | YES |
symbol | STRING | YES |
signature | STRING | YES |


### Example
* **`totalParams` is must be sorted `alphabetically` for signatures.**

* **requestBody:** 
  
  exchangeType=LOCAL&origClientOrderId=201903100614344832&origOrderId=344832&symbol=BTC/KRW
  
  or
  
  {"exchangeType":"LOCAL", "origClientOrderId":"201903100614344832", "origOrderId":"344832", "symbol":"BTC/KRW"}
  
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "exchangeType=LOCAL&origClientOrderId=201903100614344832&origOrderId=344832&symbol=BTC/KRW" | openssl dgst -sha256 -hmac "testSecretKey"
    (stdin)= 00af9b8b207d4ffa303ac145167ac86d84e46b504397518f1d581b27cebd6ede
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-BIFRAX-APIKEY: testApiKey" -X POST 'https://api.bispex.com:3000/v1/order/cancel' -d 'exchangeType=LOCAL&origClientOrderId=201903100614344832&origOrderId=344832&symbol=BTC/KRW'


**Response:**
```javascript
{
    "orderId": "344833",
    "clientOrderId": "201903100614344832",
    "symbol": "BTC/KRW"
    "origOrderId": "344832",
    "origClientOrderId": "201903100614344832"
}
```

### Current open all orders 
```
GET /v1/orders  (HMAC SHA256)
```
Get all open orders on a symbol. 


**Parameters:**


Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
exchangeType | STRING | YES | 
symbol | STRING | YES |
signature | STRING | YES |

### Example
* **`totalParams` is must be sorted `alphabetically` for signatures.**
* **queryString:** exchangeType=LOCAL&symbol=BTC/KRW
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "exchangeType=LOCAL&symbol=BTC/KRW" | openssl dgst -sha256 -hmac "testSecretKey"
    (stdin)= 710e769778d4e9d84611dc78200316f302f36e027e78843989fa7b14b7902957
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-BIFRAX-APIKEY: testApiKey" -X GET 'https://api.bispex.com:3000/v1/orders?exchangeType=LOCAL&symbol=BTC/KRW&signature=710e769778d4e9d84611dc78200316f302f36e027e78843989fa7b14b7902957'
    ```

**Response:**
```javascript
{
    "orders": [
        {
            "orderId": "194671",
            "origOrderId": "0",
            "clientOrderId": "201903090636194671",
            "symbol": "BTC/KRW",
            "side": "BUY",
            "type": "LIMIT",
            "price": "3755000",
            "origQty": "0.003",
            "executedQty": "0",
            "remainedQty": "0.003",
            "txTime": 1552113370000
        },
        {
            "orderId": "194517",
            "origOrderId": "0",
            "clientOrderId": "201903090633194517",
            "symbol": "BTC/KRW",
            "side": "BUY",
            "type": "LIMIT",
            "price": "3755000",
            "origQty": "0.003",
            "executedQty": "0",
            "remainedQty": "0.003",
            "txTime": 1552113194000
        }
    ]
}
```

### all trades
```
GET /v1/trades (HMAC SHA256)
```
Get all orders; canceled, or filled.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
endTime | LONG | NO |Actually system works by day.(exclude hours, minutes, seconds), ex) 1552089600000
exchangeType | STRING | YES | 
nextKey | STRING | NO | for Next Page
startTime | LONG | NO | Actually system works by day.(exclude hours, minutes, seconds) , ex) 1552089600000
symbol | STRING | YES |
signature | STRING | YES |

### Example
* **`totalParams` is must be sorted `alphabetically` for signatures.**
* **queryString:** endTime=1552133565000&exchangeType=LOCAL&startTime=1552118360000&symbol=BTC/KRW
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "endTime=1552133565000&exchangeType=LOCAL&startTime=1552118360000&symbol=BTC/KRW" | openssl dgst -sha256 -hmac "testSecretKey"
    (stdin)= d0ac97f56cb4dfe29cd40a8532639d05525313ff895dff69137f0adc6a1bbf19
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-BIFRAX-APIKEY: testApiKey" -X GET 'https://api.bispex.com:3000/v1/trades?endTime=1552133565000&exchangeType=LOCAL&startTime=1552118360000&symbol=BTC/KRW&signature=d0ac97f56cb4dfe29cd40a8532639d05525313ff895dff69137f0adc6a1bbf19'
    ```

**Response:**
```javascript
{
    "trades": [
        {
            "orderId": "229590",
            "origOrderId": "0",
            "clientOrderId": "201903091212229590",
            "origClientOrderId": "00",
            "symbol": "BTC/KRW",
            "tradeType": "NEW_ORDER",
            "tradeStatus": "OK",
            "rejectReason": "0",
            "side": "BUY",
            "type": "MARKET",
            "price": "0",
            "origQty": "0.003",
            "executedPrice": "4339000",
            "executedQty": "0.003",
            "remainedQty": "0",
            "executedPriceSum": "13017",
            "tradeFee": "13",
            "txTime": 1552133565000
        },
        {
            "orderId": "205344",
            "origOrderId": "194962",
            "clientOrderId": "201903090759205344",
            "origClientOrderId": "201903090639194962",
            "symbol": "BTC/KRW",
            "tradeType": "CANCEL",
            "tradeStatus": "COMPLETE",
            "rejectReason": "0",
            "side": "BUY",
            "type": "LIMIT",
            "price": "0",
            "origQty": "0.003",
            "executedPrice": "0",
            "executedQty": "0",
            "remainedQty": "0",
            "executedPriceSum": "0",
            "tradeFee": "0",
            "txTime": 1552118360000
        },
    ]
}
```
### trade status
```
GET /v1/tradeStatus (HMAC SHA256)
```
Get one order status. 

orderStatus: Submitted, Replaced, Cancelled, PatiallyFilled, Filled, Rejected

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
ClientOrderId | STRING | YES |
OrderId | STRING | YES |
exchangeType | STRING | YES | 
symbol | STRING | YES |
signature | STRING | YES |

### Example
* **`totalParams` is must be sorted `alphabetically` for signatures.**
* **queryString:** ClientOrderId=201903091212229590&OrderId=229590&exchangeType=LOCAL&symbol=BTC/KRW
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "ClientOrderId=201903091212229590&OrderId=229590&exchangeType=LOCAL&symbol=BTC/KRW" | openssl dgst -sha256 -hmac "testSecretKey"
    (stdin)= f65a7c7fadef0c06699f7361ae01653e117883f3c734f2b00ac576f7b1a6da76
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-BIFRAX-APIKEY: testApiKey" -X GET 'https://api.bispex.com:3000/v1/tradeStatus?ClientOrderId=201903091212229590&OrderId=229590&exchangeType=LOCAL&symbol=BTC/KRW&signature=f65a7c7fadef0c06699f7361ae01653e117883f3c734f2b00ac576f7b1a6da76'
    ```

**Response:**
```javascript
{
    "tradeStatus": [
        {
            "orderId": "229590",
            "origOrderId": "0",
            "clientOrderId": "201903091212229590",
            "origClientOrderId": "00",
            "symbol": "BTC/KRW",
            "tradeType": "NEW_ORDER",
            "tradeStatus": "OK",
            "rejectReason": "0",
            "side": "BUY",
            "type": "MARKET",
            "price": "0",
            "origQty": "0.003",
            "executedPrice": "4339000",
            "executedQty": "0.003",
            "remainedQty": "0",
            "executedPriceSum": "13017",
            "tradeFee": "13",
            "txTime": 1552133565000,
            "orderStatus": "Filled"
        }
    ]
}
```

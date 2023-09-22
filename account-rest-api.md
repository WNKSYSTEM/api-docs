# Account Rest API for Exchange 
# General API Information
* All endpoints return either a JSON object or array.
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

## Account endpoints

### Infomation

* ExchangeType

`LOCAL`: local currency exchanges(piat currency)

### Balance information (USER_DATA)
```
GET /v1/balances (HMAC SHA256)
```
Get current balance information.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
currency | STRING | NO | if not set, all coins
exchangeType | STRING | YES |
signature | STRING | YES |

### Example
* **`totalParams` is must be sorted `alphabetically` for signatures.**
* **queryString:** currency=PHP&exchangeType=LOCAL
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "currency=PHP&exchangeType=LOCAL" | openssl dgst -sha256 -hmac "testSecretKey"
    (stdin)= 95fe30a84085c3ca6c2f387bd98bb21053e52e58b8fe14dd6ba533bed805fcab
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-BIFRAX-APIKEY: testApiKey" -X GET 'https://api.bispex.com:3000/v1/balances?currency=PHP&exchangeType=LOCAL&signature=95fe30a84085c3ca6c2f387bd98bb21053e52e58b8fe14dd6ba533bed805fcab'
    ```

**Response:**
```
  "currency": "PHP",
  "totalAmount": "9999978134.79999924",
  "amount": "9999957126.5",
  "evaluationAmount": "10000000144.79999924",
  "balances": [
    {
      "symbol": "BTC/PHP",
      "quantity": "0.1",
      "avgPrice": "218302.8301886",
      "currentPrice": "220100",
      "porfitOrLoss": "179.71698114",
      "polRate": "0.82",
      "usingQuantity": "0",
      "marginQuantity": "0",
      "marginPrice": "0",
      "evaluationPrice": "22010",
      "side": "BUY"
    }
  ]
}
```

### Local Currency withdrawl, deposit list
```
GET /v1/ioList (HMAC SHA256)
```
Get currency history(piat currency)

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
endTime | LONG | NO |Actually system works by day.(exclude hours, minutes, seconds), ex) 1552089600000
exchangeType | STRING | YES |
startTime | LONG | NO | Actually system works by day.(exclude hours, minutes, seconds) , ex) 1552089600000
signature | STRING | YES |

additional Response parameters infomation on `status`

Type | Additional mandatory response parameters
------------ | ------------
`NEW` | initionalized 
`DOING` | doing
`DONE`  | finished
`CANCEL` | canceled

### Example
* **`totalParams` is must be sorted `alphabetically` for signatures.**
* **queryString:** endTime=1552089600000&exchangeType=LOCAL&startTime=1552089600000
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "endTime=1552089600000&exchangeType=LOCAL&startTime=1552089600000" | openssl dgst -sha256 -hmac "testSecretKey"
    (stdin)= 2bf2511d60f0bd85385881de1c42270e9ab5a1d6e792b108fc6004c0c7c60f5d
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-BIFRAX-APIKEY: testApiKey" -X GET 'https://api.bispex.com:3000/v1/ioList?endTime=1552089600000&exchangeType=LOCAL&startTime=1552089600000&signature=2bf2511d60f0bd85385881de1c42270e9ab5a1d6e792b108fc6004c0c7c60f5d'
    ```

**Response:**
```
  "ioList": [
    {
      "timeznoe": "UTC",
      "currency": "",
      "requestDate": "",
       "reqeustTime": "",
       "ioType": "",
       "amount": "",
       "fee": "",
       "approveDate": "",
       "approveTime": "",
       "status": "",
       "widrawalBankCode": "",
       "widrawalBankName": "",
       "widrawalBankAccount": "",
       "widrawalBankAccountName": "",
       "exchangeRatio": "",
       "utcTime": "",
    }
  ]
}
```

### Coin Currency withdrawl, deposit list
```
GET /v1/coinIoList (HMAC SHA256)
```
Get currency history(piat currency)

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
endTime | LONG | NO |Actually system works by day.(exclude hours, minutes, seconds), ex) 1552089600000
exchangeType | STRING | YES |
startTime | LONG | NO | Actually system works by day.(exclude hours, minutes, seconds) , ex) 1552089600000
signature | STRING | YES |

additional Response parameters infomation on `status`

Type | Additional mandatory response parameters
------------ | ------------
`NEW` | initionalized 
`DOING` | doing
`DONE`  | finished
`CANCEL` | canceled

### Example
* **`totalParams` is must be sorted `alphabetically` for signatures.**
* **queryString:** endTime=1552089600000&exchangeType=LOCAL&startTime=1552089600000
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "endTime=1552089600000&exchangeType=LOCAL&startTime=1552089600000" | openssl dgst -sha256 -hmac "testSecretKey"
    (stdin)= 2bf2511d60f0bd85385881de1c42270e9ab5a1d6e792b108fc6004c0c7c60f5d
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-BIFRAX-APIKEY: testApiKey" -X GET 'https://api.bispex.com:3000/v1/coinIoList?endTime=1552089600000&exchangeType=LOCAL&startTime=1552089600000&signature=2bf2511d60f0bd85385881de1c42270e9ab5a1d6e792b108fc6004c0c7c60f5d'
    ```

**Response:**
```
  "ioList": [
    {
      "timeznoe": "UTC",
      "currency": "",
      "requestDate": "",
       "reqeustTime": "",
       "ioType": "",
       "amount": "",
       "fee": "",
       "approveDate": "",
       "approveTime": "",
       "status": "",
       "txId": "",
    }
  ]
}
```

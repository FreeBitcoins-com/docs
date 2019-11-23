<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Public Rest API for Xchange (2019-11-22)](#public-rest-api-for-xchange-2019-11-22)
  - [General API Information](#general-api-information)
  - [HTTP Return Codes](#http-return-codes)
  - [Error Codes](#error-codes)
  - [General Information on Endpoints](#general-information-on-endpoints)
- [Endpoint security type](#endpoint-security-type)
- [SIGNED (TRADE and USER_DATA) Endpoint security](#signed-trade-and-user_data-endpoint-security)
  - [Timing security](#timing-security)
  - [SIGNED Endpoint Examples for POST /api/v1/order](#signed-endpoint-examples-for-post-apiv1order)
    - [Example: Query string request](#example-query-string-request)
- [Public API Endpoints](#public-api-endpoints)
  - [Terminology](#terminology)
  - [General endpoints](#general-endpoints)
    - [Test connectivity](#test-connectivity)
    - [Check server time](#check-server-time)
    - [Exchange information](#exchange-information)
  - [Market Data endpoints](#market-data-endpoints)
    - [Order book](#order-book)
    - [Recent trades list](#recent-trades-list)
    - [Kline/Candlestick data](#klinecandlestick-data)
    - [24hr ticker price change statistics](#24hr-ticker-price-change-statistics)
    - [Market price ticker](#market-price-ticker)
    - [Market order book ticker](#market-order-book-ticker)
  - [Account endpoints](#account-endpoints)
    - [New order  (TRADE)](#new-order--trade)
    - [Query order (USER_DATA)](#query-order-user_data)
    - [Cancel order (TRADE)](#cancel-order-trade)
    - [Current open orders (USER_DATA)](#current-open-orders-user_data)
    - [All orders (USER_DATA)](#all-orders-user_data)
    - [Account information (USER_DATA)](#account-information-user_data)
    - [Withdraw (USER_DATA)](#withdraw-user_data)
    - [Account trade list (USER_DATA)](#account-trade-list-user_data)
    - [Deposit history (USER_DATA)](#deposit-history-user_data)
    - [Withdraw history (USER_DATA)](#withdraw-history-user_data)
    - [Deposit address (USER_DATA)](#deposit-address-user_data)
    - [New deposit address (USER_DATA)](#new-deposit-address-user_data)
- [Filters](#filters)
  - [Market filters](#market-filters)
    - [PRICE_FILTER](#price_filter)
    - [LOT_SIZE](#lot_size)
    - [MARKET_QUANTITY_INPUT](#market_quantity_input)
    - [MAX_NUM_ORDERS](#max_num_orders)
    - [MIN_ORDER_SIZE](#min_order_size)
  - [Exchange Filters](#exchange-filters)
    - [EXCHANGE_MAX_NUM_ORDERS](#exchange_max_num_orders)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Public Rest API for Xchange (2019-11-22)

## General API Information
* The base endpoint is: **https://freebitcoins.com/xchange/**
* All endpoints return either a JSON object or array.
* Data is returned in **descending** order. Newest first, oldest last.
* All time and timestamp related fields are in **milliseconds**.

## HTTP Return Codes

* HTTP `4XX` return codes are used for malformed requests;
  the issue is on the sender's side.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `5XX` return codes are used for internal errors; the issue is on
  Xchange's side.

## Error Codes
* Any endpoint can return an ERROR

Sample Payload below:
```javascript
{
  "code": -1104,
  "msg": "Invalid market."
}
```
* Specific error codes and messages are defined in [Errors Codes](./errors.md).

## General Information on Endpoints
* For `GET` and `POST`  and `DELETE` endpoints, parameters must be sent as a `query string`.
* Parameters may be sent in any order.

# Endpoint security type
* Each endpoint has a security type that determines the how you will
  interact with it. This is stated next to the NAME of the endpoint.
    * If no security type is stated, assume the security type is NONE.
* API-keys are passed into the Rest API via the `X-MBX-APIKEY`
  header.
* API-keys and secret-keys **are case sensitive**.
* By default, API-keys can access all secure routes.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
TRADE | Endpoint requires sending a valid API-Key and signature.
USER_DATA | Endpoint requires sending a valid API-Key and signature.

* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.

# SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **not case sensitive**.

## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the millisecond timestamp of when the request was created and sent.
* An additional parameter, `recvWindow`, may be sent to specify the number of
  milliseconds after `timestamp` the request is valid for. If `recvWindow`
  is not sent, **it defaults to 5000**.
* The logic is as follows:
  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**It is recommended to use a small recvWindow of 5000 or less! The max cannot go beyond 60,000!**


## SIGNED Endpoint Examples for POST /api/v1/order
Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


Parameter | Value
------------ | ------------
market | BTC_CLAM
side | buy
type | limit
quantity | 1
price | 0.1
recvWindow | 5000
timestamp | 1499827319559

### Example: Query string request
* **queryString:** market=BTC_CLAM&side=buy&type=limit&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "market=BTC_CLAM&side=buy&type=limit&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://freebitcoins.com/xchange/api/v1/order?market=BTC_CLAM&side=buy&type=limit&quantity=1&price=0.1&recvWindow=5000&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

# Public API Endpoints
## Terminology
* `base symbol` refers to the symbol that is the `quantity` of a market.
* `quote symbol` refers to the symbol that is the `price` of a market.


**Order status (status):**

* new
* partial
* filled
* canceled
* rejected
* booked

**Order types (orderTypes, type):**

* limit
* market

**Order side (side):**

* buy
* sell

**Kline/Candlestick chart intervals:**

m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

* 1m
* 3m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 8h
* 12h
* 1d
* 3d
* 7d
* 1M

**Rate limit intervals (interval)**

* SECOND
* MINUTE
* DAY

## General endpoints
### Test connectivity
```
GET /api/v1/ping
```
Test connectivity to the Rest API.


**Parameters:**
NONE

**Response:**
```javascript
{}
```

### Check server time
```
GET /api/v1/time
```
Test connectivity to the Rest API and get the current server time.

**Parameters:**
NONE

**Response:**
```javascript
{
    "timezone": "UTC"
    "serverTime": 1573509210273
}
```

### Exchange information
```
GET /api/v1/exchangeInfo
```
Current exchange trading rules and market information

**Parameters:**
NONE

**Response:**
```javascript
{
    "timezone": "UTC"m
    "serverTime": 1573509098104, 
    "exchangeFilters": [
        {
            "filterType": "MAX_NUM_ORDERS",
            "maxNumOrders": 100
        },
        {
            "filterType": "EXCHANGE_MAX_NUM_ORDERS",
            "maxNumOrdersExchange": 1000
        }
    ],
    "markets": [
        {
            "id": 1,
            "quote": "BTC",
            "base": "CLAM",
            "basePrecision": 2,
            "quotePrecision": 6,
            "orderTypes": [
                "limit"
            ],
            "filters": [
                {
                    "filterType": "PRICE_FILTER",
                    "minPrice": "0.00000100",
                    "maxPrice": "10000.00000000",
                    "tickSize": "0.00000100"
                },
                {
                    "filterType": "LOT_SIZE",
                    "minQty": "0.01000000",
                    "maxQty": "100000.00000000",
                    "stepSize": "0.01000000"
                }
            ]
        }
    ]
}
```

## Market Data endpoints
### Order book
```
GET /api/v1/depth
```

Limit | Weight
------------ | ------------
5, 10, 20, 50, 100 | 1
500 | 5
1000 | 10
5000| 50


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
limit | INT | NO | Default 100; max 5000. Valid limits:[5, 10, 20, 50, 100, 500, 1000, 5000]

**Response:**
```javascript
{
    "bids": [
        {
            "price": "0.0012",
            "quantity": "10.0000"
        },
    "asks": [
        {
            "price": "0.0013",
            "quantity": "10.0000"
        }
    ],
    "sequence": 15
}
```


### Recent trades list
```
GET /api/v1/trades
```
Get recent trades (up to last 500).


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
limit | INT | NO | Default 500; max 1000.

**Response:**
```javascript
[
    {
        "id": 3,
        "market": "BTC_ETH",
        "side": "sell",
        "quantity": "10.00000000",
        "price": "0.00400000",
        "timestamp": 1573500901940
    }
]
```

### Kline/Candlestick data
```
GET /api/v1/klines
```
Kline/candlestick bars for a market.
Klines are uniquely identified by their open time.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
interval | ENUM | YES |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.

* If startTime and endTime are not sent, the most recent klines are returned.

**Response:**
```javascript
[
  [
    1573246800000,      // Open time
    "0.0013",           // Open
    "0.0013",           // High
    "0.0013",           // Low
    "0.0013",           // Close
    "20.0000",          // Volume
    1573257600000,      // Close time
    2,                  // Number of trades
  ]
]
```



### 24hr ticker price change statistics
```
GET /api/v1/ticker/24hr
```
24 hour rolling window price change statistics.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | NO |

* If the market is not sent, tickers for all markets will be returned in an array.

**Response:**
```javascript
 {
    "market": "BTC_CLAM",
    "priceChange": "0.000000",
    "priceChangePercent": "0",
    "weightedAvgPrice": "0.000000",
    "prevClosePrice": "0.000000",
    "lastPrice": "0.000110",
    "lastQty": "0.00",
    "bidPrice": "0.000100",
    "askPrice": "0.000120",
    "openPrice": "0.000110",
    "highPrice": "0.000000",
    "lowPrice": "0.000000",
    "volume": "0.00",
    "quoteVolume": "0.00000000",
    "count": 0,
    "openTime": "2019-11-06T00:00:00Z",
    "closeTime": "2019-11-07T00:00:00Z"
}
```
OR
```javascript
[
    {
        "market": "BTC_CLAM",
        "priceChange": "0.000000",
        "priceChangePercent": "0",
        "weightedAvgPrice": "0.000000",
        "prevClosePrice": "0.000000",
        "lastPrice": "0.000110",
        "lastQty": "0.00",
        "bidPrice": "0.000100",
        "askPrice": "0.000120",
        "openPrice": "0.000110",
        "highPrice": "0.000000",
        "lowPrice": "0.000000",
        "volume": "0.00",
        "quoteVolume": "0.00000000",
        "count": 0,
        "openTime": "2019-11-06T00:00:00Z",
        "closeTime": "2019-11-07T00:00:00Z"
    }
]
```


### Market price ticker
```
GET /api/v1/ticker/price
```
Latest price for a market or markets.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | NO |

* If the market is not sent, prices for all markets will be returned in an array.

**Response:**
```javascript
{
    "market": "BTC_CLAM",
    "price": "0.000110"
}
```
OR
```javascript
[
    {
        "market": "BTC_CLAM",
        "price": "0.000110"
    },
    {
        "market": "BTC_ETH",
        "price": "0.0040"
    }
]
```

### Market order book ticker
```
GET /api/v1/ticker/bookTicker
```
Best price/qty on the order book for a market or markets.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | NO |

* If the market is not sent, bookTickers for all markets will be returned in an array.

**Response:**
```javascript
{
  "market": "BTC_CLAM",
  "bidPrice": "4.00000000",
  "bidQty": "431.00000000",
  "askPrice": "4.00000200",
  "askQty": "9.00000000"
}
```
OR
```javascript
[
  {
    "market": "BTC_CLAM",
    "bidPrice": "4.00000000",
    "bidQty": "431.00000000",
    "askPrice": "4.00000200",
    "askQty": "9.00000000"
  },
  {
    "market": "BTC_LTC",
    "bidPrice": "0.07946700",
    "bidQty": "9.00000000",
    "askPrice": "100000.00000000",
    "askQty": "1000.00000000"
  }
]
```

## Account endpoints
### New order  (TRADE)
```
POST /api/v1/order  (HMAC SHA256)
```
Send in a new order.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
side | ENUM | YES |
type | ENUM | YES |
quantity | DECIMAL | YES |
price | DECIMAL | NO |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

Additional mandatory parameters based on `type`:

Type | Additional mandatory parameters
------------ | ------------
`LIMIT` | `quantity`, `price`
`MARKET` | `quantity` ( qty in `base symbol` for sell, `quote symbol` for buy)


**Response:**
```javascript
{
  "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406',
  "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a',
  "price": "0.00000100",
  "quantity": "2.00000000",
  "market": "BTC_CLAM",
  "side": "sell",
  "order_type": "market",
  "order_state": "filled",
  "trades":
   [ { "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406",
       "user_id": "c1932526-8885-45af-a91a-c35aa4a2caf7",
       "trade_id": 9086,
       "market": "BTC_CLAM",
       "side": "sell",
       "price": "0.00011000",
       "quantity": "0.91000000",
       "maker": false,
       "total": "0.00010010",
       "commission": "0.00000051",
       "commission_asset": "BTC",
       "date": "2019-11-21T18:25:38.780258882Z" },
     { "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406",
       "user_id": "c1932526-8885-45af-a91a-c35aa4a2caf7",
       "trade_id": 9087,
       "market": "BTC_CLAM",
       "side": "sell",
       "price": "0.00011000",
       "quantity": "1.09000000",
       "maker": false,
       "total": "0.00011990",
       "commission": "0.00000060",
       "commission_asset": "BTC",
       "date": "2019-11-21T18:25:38.819797122Z" } 
    ]
}
```


### Query order (USER_DATA)
```
GET /api/v1/order (HMAC SHA256)
```
Check an order's status.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
orderId | LONG | NO |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

Notes:
* Either `orderId` or `origClientOrderId` must be sent.

**Response:**
```javascript
{
  "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406',
  "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a',
  "price": "0.00000100",
  "quantity": "2.00000000",
  "market": "BTC_CLAM",
  "side": "sell",
  "order_type": "market",
  "order_state": "filled",
}
```

### Cancel order (TRADE)
```
DELETE /api/v1/order  (HMAC SHA256)
```
Cancel an active order.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
orderId | LONG | NO |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

Either `orderId` or `origClientOrderId` must be sent.

**Response:**
```javascript
{
  "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406',
  "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a',
  "price": "0.00110000",
  "quantity": "2.00000000",
  "market": "BTC_CLAM",
  "remaining": "2.00000000",
  "side": "sell",
  "order_type": "limit",
  "order_state": "cancelled",
}
```

### Current open orders (USER_DATA)
```
GET /api/v1/openOrders  (HMAC SHA256)
```
Get all open orders on a market.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Response:**
```javascript
[
    { 
        "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406",
        "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
        "price": "0.00110000",
        "quantity": "10.00000000",
        "market": "BTC_ETH",
        "side": "buy",
        "remaining": "10.00000000",
        "order_type": "limit",
        "order_state": "booked",
        "created_at": "2019-11-08T19:44:06.981056Z" 
    }
]
```

### All orders (USER_DATA)
```
GET /api/v1/allOrders (HMAC SHA256)
```
Get all account orders; active, canceled, or filled.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
orderId | LONG | NO |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Notes:**
* If `orderId` is set, it will get orders >= that `orderId`. Otherwise most recent orders are returned.

**Response:**
```javascript
[
    {
       "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406",
        "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
        "price": "0.00110000",
        "quantity": "10.00000000",
        "market": "BTC_ETH",
        "side": "buy",
        "remaining": "10.00000000",
        "order_type": "limit",
        "order_state": "booked",
        "created_at": "2019-11-08T19:44:06.981056Z" 
    }
]
```


### Account information (USER_DATA)
```
GET /api/v1/account (HMAC SHA256)
```
Get current account information.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Response:**
```javascript
{ balances:
    canTrade: false,
    canWithdraw: false,
    [ 
        { 
            "asset": "BTC",
            "free": "999.80160609",
            "pending": "0.00000000",
            "locked": "0.04140000" 
        },
        { 
            "asset": "CLAM",
            "free": "998.88888890",
            "pending": "0.00000000",
            "locked": "0.00000000" 
        }
    ]
}
```

### Withdraw (USER_DATA)
```
POST /api/v1/withdraw  (HMAC SHA256)
```
Request a withdraw for a specific market.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
destination | string | NO |
quantity | DECIMAL | NO |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Response:**
```javascript
[
  {
    "id": 28457,
    "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
    "symbol": "CLAM",
    "destination": "xX3gahy5kjozWwaPKViDDiF4vkGfUhMiyU",
    "amount": "10.0000",
    "txid": "e3748cbc9bd38edbd1a63d6c2f3f6044e9294bbdebaab2a2d416e1a78abf1030",
    "state": "complete",
    "complete": true,
    "fee": "0.00100000",
  }
]
```

### Account trade list (USER_DATA)
```
GET /api/v1/myTrades  (HMAC SHA256)
```
Get trades for a specific account and market.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
fromId | LONG | NO | TradeId to fetch from. Default gets most recent trades.
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Notes:**
* If `fromId` is set, it will get orders >= that `fromId`.
Otherwise most recent orders are returned.

**Response:**
```javascript
[ 
    { 
        "order_id": "9574c6c5-cd3d-41d3-88b5-abfa836ac111",
        "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
        "trade_id": 3,
        "market": "BTC_ETH",
        "side": "buy",
        "price": "0.00400000",
        "quantity": "10.00000000",
        "maker": true,
        "total": "0.04000000",
        "commission": "0.05000000",
        "commission_asset": "ETH",
        "date": "2019-11-11T19:35:01.897252Z" 
    }
]
```

### Deposit history (USER_DATA)
```
GET /api/v1/depositHistory  (HMAC SHA256)
```
Get deposits for a specific account and market.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |


**Response:**
```javascript
[
    { 
        "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
        "symbol": "CLAM",
        "address": "xLpyii7piMV6Y4jx1UKr5QfxyswkALniD3",
        "amount": "0.01",
        "txid": "00bbfec31dc6f2c0ff917524768786258bb057c471a536eaa960999c4e334c26",
        "confirmations": 30,
        "confirmed": true,
        "complete": false,
        "state": "complete",
        "created_at": 1571159819902,
        "updated_at": 1571161613058 
    }
]
```


### Withdraw history (USER_DATA)
```
GET /api/v1/withdrawHistory  (HMAC SHA256)
```
Get withdrawals for a specific account and symbol.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
limit | INT | NO | Default 500; max 1000.
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |


**Response:**
```javascript
[
    { 
        "id": 52,
        "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
        "symbol": "CLAM",
        "destination": "x9Ph1xyrN7z4vfzYuNCdujgrX2Ly2SQsPs",
        "amount": "10.00100000",
        "txid": "6d5069d30a87bcd4c7631890e20375119444adcb52b97bb77a8183fe960e437a",
        "state": "complete",
        "fee": "0.00100000",
        "complete": true,
        "failed": false,
        "cancelled": false,
        "rejected": false,
        "reject_reason": "",
        "updated_at": 1572482164000
     }
]
```

### Deposit address (USER_DATA)
```
GET /api/v1/deposit  (HMAC SHA256)
```
Get current account deposit address(s) for a specific symbol or all if no symbol given.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |

**Response:**
```javascript
[
    { 
        address: 'xLpyii7piMV6Y4jx1UKr5QfxyswkALniD3',
        symbol: 'CLAM' 
    }
]
```

### New deposit address (USER_DATA)
```
POST /api/v1/deposit  (HMAC SHA256)
```
Get new deposit address for a specific symbol.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
recvWindow | LONG | NO | The value cannot be greater than ```60000```
timestamp | LONG | YES |


**Response:**
```javascript
[
  {
        "address": "xLpyii7piMV6Y4jx1UKr5QfxyswkALniD3",
        "symbol": "CLAM"
  }
]
```



# Filters
Filters define trading rules on a market or an exchange.
Filters come in two forms: `market filters` and `exchange filters`.

## Market filters
### PRICE_FILTER
The `PRICE_FILTER` defines the `price` rules for a market. There are 3 parts:

* `minPrice` defines the minimum `price` allowed; disabled on `minPrice` == 0.
* `maxPrice` defines the maximum `price` allowed; disabled on `maxPrice` == 0.
* `tickSize` defines the intervals that a `price` can be increased/decreased by; disabled on `tickSize` == 0.

Any of the above variables can be set to 0, which disables that rule in the `price filter`. In order to pass the `price filter`, the following must be true for `price` of the enabled rules:

* `price` >= `minPrice`
* `price` <= `maxPrice`
* (`price`-`minPrice`) % `tickSize` == 0

**/exchangeInfo format:**
```javascript
{
  "filterType": "PRICE_FILTER",
  "minPrice": "0.00000100",
  "maxPrice": "100000.00000000",
  "tickSize": "0.00000100"
}
```

### LOT_SIZE
The `LOT_SIZE` filter defines the `quantity` (aka "lots" in auction terms) rules for a market. There are 3 parts:

* `minQty` defines the minimum `quantity` allowed.
* `maxQty` defines the maximum `quantity`allowed.
* `stepSize` defines the intervals that a `quantity` can be increased/decreased by.

In order to pass the `lot size`, the following must be true for `quantity`:

* `quantity` >= `minQty`
* `quantity` <= `maxQty`
* (`quantity`-`minQty`) % `stepSize` == 0

**/exchangeInfo format:**
```javascript
{
  "filterType": "LOT_SIZE",
  "minQty": "0.00100000",
  "maxQty": "100000.00000000",
  "stepSize": "0.00100000"
}
```

### MARKET_QUANTITY_INPUT
The `MARKET_QUANTITY_INPUT` filter defines the symbol used for quantity input when making a `market` order on the given order side. Any market orders submitted quantity will be assumed to be in the symbol defined in the `MARKET_QUANTITY_INPUT` filter.

**/exchangeInfo format:**
```javascript
{
  "filterType": "MARKET_QUANTITY_INPUT",
  "buy": "quote",
  "sell": "base",
}
```

### MAX_NUM_ORDERS
The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on a market.
Note that both "algo" orders and normal orders are counted for this filter.

**/exchangeInfo format:**
```javascript
{
  "filterType": "MAX_NUM_ORDERS",
  "limit": 25
}
```

### MIN_ORDER_SIZE
The `MIN_ORDER_SIZE` filter defines the minimum value allowed for an order on a market.
An order's value is the `price` * `quantity`.

**/exchangeInfo format:**
```javascript
{
  "filterType": "MIN_NOTIONAL",
  "minNotional": "0.00001000",
}
```

## Exchange Filters
### EXCHANGE_MAX_NUM_ORDERS
The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on the exchange.
Note that both "algo" orders and normal orders are counted for this filter.

**/exchangeInfo format:**
```javascript
{
  "filterType": "EXCHANGE_MAX_NUM_ORDERS",
  "maxNumOrders": 1000
}
```

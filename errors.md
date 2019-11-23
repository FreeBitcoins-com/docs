# Error codes for Xchange (2019-11-06)
Errors consist of two parts: an error code and a message. Codes are universal,
 but messages can vary. Here is the error JSON payload:
```javascript
{
  "code":-1104,
  "msg":"Invalid market."
}
```


## 10xx - General Server or Network issues
#### -1000 UNKNOWN
 * An unknown error occured while processing the request.

#### -1001 UNAUTHORIZED
 * You are not authorized to execute this request.

#### -1003 TOO_MANY_REQUESTS
 * Too many requests queued.

#### -1005 TOO_MANY_ORDERS
 * Too many new orders.
 * Too many new orders; current limit is %s orders per %s.

#### -1008 EXCHANGE_NOT_ACTIVE
 * This service is no longer available.

#### -1009 INVALID_SIGNATURE
 * Signature for this request is not valid.

#### -1010 INVALID_TIMESTAMP
 * Timestamp for this request is outside of the recvWindow.
 * Timestamp for this request was ahead of the server's time.





#### -1100 INVALID_QUANTITY
 * Invalid order quantity.

#### -1101 INVALID_PRICE
 * Invalid order price.

#### -1102 INVALID_ORDER_SIDE
 * Invalid side.

#### -1103 INVALID_ORDER_TYPE
 * Invalid orderType.

#### -1104 INVALID_MARKET
 * Invalid orderType.

#### -1105 INVALID_INTERVAL
 * Invalid interval.

#### -1106 INVALID_LIMIT
 * Invalid limit.

#### -1107 MARKET_CLOSED
 * Market is not accepting orders.

#### -1108 INVALID_SYMBOL
 * Invalid symbol.

#### -1109 ORDER_TOO_SMALL
 * 

 #### -1110 INVALID_PRECISION
 * Precision is over the maximum defined for this asset.


#### -2000 CREATE_ORDER_ERROR
 * Error creating order

 #### -2001 NOT_ENOUGH_BALANCE
 * Balance to low for request

#### -2010 INVALID_API_KEY
 * Invalid API-key, IP, or permissions for action.


 #### -9000 MIN_NOTIONAL
 * Filter failure: Order under the minimium required value.

## API Overview

Ice3x trading platform provides a simple REST API which allows for integration of business applications, payment systems, trading systems, mobile apps, etc.  

All requests use the "application/json" content type and must use https. 

Unless specified all API methods use http POST.

If you have any feedback on the API please post a question in our support center.


## Getting started
Before using API please take some time to understand the authentication mechanism built into the API.  
Some parts of the API require authentication and in those cases you will be required to register with Ice3x and then contact our support team who can generate the required API keys for you. 


## Market Data API
Market data API is accessible through https and parameters are passed using GET-requests. All responses are encoded in JSON. No authentication is required for calling market data.

Currently the APIs are cached for 30 seconds and all service calls are monitored for excessive load so please don’t query more often than once every 1 minute.

Tick:

path: /market/BTC/ZAR/tick

sample response:
```json
{"bestBid":13700000000,"bestAsk":14000000000,"lastPrice":14000000000,"currency":"ZAR","instrument":"BTC","timestamp":1378878117}
```

Orderbook:

path: /market/BTC/ZAR/orderbook

sample response:
```json
{"currency":"ZAR","instrument":"BTC","timestamp":1378941290,"asks":[[14000000000,20000000],[14100000000,10000000],[14200000000,20000000],[14600000000,10000000],[15900000000,50000000],[16000000000,10000000],"bids":[[13700000000,20000000],[12500000000,20000000],[12000010000,100000000],[12000000000,1000000]]}
```

Trades:

path: /market/BTC/ZAR/trades

sample response:
```json
[{"tid":4432702312,"amount":10000000,"price":14000000000,"date":1378878093},{"tid":59861212129,"amount":1000000,"price":12500000000,"date":1377840783}]
```

`since` is an optional parameter for trades (get trades since the trade id). For instance:

/market/BTC/ZAR/trades?since=59868345231



## Authentication

To authenticate a request, you will need to build a string which includes several elements of the http request. You will then use your private API key to calculate a HMAC of that string. 

This will generate a signature string which needs to be added as a parameter of the request by using the syntax described in the following sections. 


### Timestamp/nonce
A valid client timestamp must be used for authenticated requests. the timestamp included with an authenticated request must be within +/- 30 seconds of the server timestamp at the time when the request is received. Failing to submit a timestamp will result in Authentication fail response. 

The intention of these restrictions is to limit the possibility that intercepted requests could be replayed by an adversary.  


### Authentication example
Below is an example of creating signature for order/history API. 

In this example the parameters used to calculate the signature are:

1- URI

``
/order/history
``

2- current timestamp in milliseconds
``
1378818710123
``

2- Request body
```json
{"currency":"ZAR", "instrument":"BTC", "limit":"10"}
```

The string to sign is:

``
'/order/history' + '\n' + '1378818710123' + '\n' + '{"currency":"ZAR", "instrument":"BTC", "limit":"10"}'
``
 
Note: if creating a signature for http GET method then post data will be null and therefore no need to add it to this string. 

Use HmacSHA512 algorithm in order to sign above string with your API private key which results in the following signature:
'bEDtDJnW0y/Ll4YZitxb+D5sTNnEpQKH67EJRCmQCqN9cvGiB8+IHzB7HjsOs3mSlxLmu4aiPDRpe9anuWzylw=='


Now we are ready to build a http request with all the headers and parameters required.


* "Accept": "application/json"
* "Accept-Charset": "UTF-8"
* "Content-Type": "application/json"
* "apikey": "your API key"
* "timestamp": "timestamp used in above process to create the signature"
* "signature": "bEDtDJnW0y/Ll4YZitxb+D5sTNnEpQKH67EJRCmQCqN9cvGiB8+IHzB7HjsOs3mSlxLmu4aiPDRpe9anuWzylw=="



## Trading API

Trading API covers all order placement and trade management. all trading apis require authentication.

### Create an order
path: /order/create

http post

sample request:
```json
{"currency":"ZAR","instrument":"BTC","price":13000000000,"volume":10000000,"orderSide":"Bid","ordertype":"Limit","clientRequestId":"abc-cdf-1000"}
```

sample success response:
```json
{"success":true,"errorCode":null,"errorMessage":null,"id":100,"clientRequestId":"abc-cdf-1000"}
```

sample error response

```json
{"success":false,"errorCode":3,"errorMessage":"Invalid argument.","id":0,"clientRequestId":"abc-cdf-1000"}
```

### Cancel an order
path: /order/cancel

http post

sample request:

```json
{"orderIds":[6840125478]}
```

sample response:
```json
{"success":true,"errorCode":null,"errorMessage":null,"responses":[{"success":false,"errorCode":3,"errorMessage":"order does not exist.","id":6840125478}]}
```

### Order history
path: /order/history

http post

sample request:

```json
{"currency":"ZAR","instrument":"BTC","limit":10,"since":33434568724}
```

sample response:

```json
{"success":true,"errorCode":null,"errorMessage":null,"orders":[{"id":1003245675,"currency":"ZAR","instrument":"BTC","orderSide":"Bid","ordertype":"Limit","creationTime":1378862733366,"status":"Placed","errorMessage":null,"price":13000000000,"volume":10000000,"openVolume":10000000,"clientRequestId":null,"trades":[]},{"id":4345675,"currency":"ZAR","instrument":"BTC","orderSide":"Ask","ordertype":"Limit","creationTime":1378636912705,"status":"Fully Matched","errorMessage":null,"price":13000000000,"volume":10000000,"openVolume":0,"clientRequestId":null,"trades":[{"id":5345677,"creationTime":1378636913151,"description":null,"price":13000000000,"volume":10000000,"fee":100000}]}]}
```

### Open Orders
path: /order/open

http post

This is similar to the order history request and response. The only difference is that this method only returns open orders. 


### Trade History
path: /order/trade/history

http post

sample request:

```json
{"currency":"ZAR","instrument":"BTC","limit":10,"since":33434568724}
```

### Order detail
path: /order/detail

http post

sample request:

```json
{"orderIds":[6840125478]}
```


## Fund transfer API
coming soon

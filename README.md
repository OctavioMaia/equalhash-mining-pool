# EqualHash Mining pool
Highly Efficient Ethereum Classic mining pool

![alt text](https://i.imgur.com/fLBuYTt.png)

### Features

**This pool is reegineered from sammy007 open-ethereum-pool for efficiency and for better payment algorithm. This software is functional and tested and implemented in big Mining farms. Testing and bug submissions are still welcome!**

*	Support for HTTP, SSL, Stratum, Stratum+SSL mining
*	Detailed block stats with luck percentage and full reward
*	Failover node instances: high availability built in (Any number of full nodes can be added to the configurations)
*	Modern beautiful Ember.js frontend for Individual coin statistics vs consolidated coin statistics
*	Separate stats for workers: can highlight timed-out workers so miners can perform maintenance of rigs
*	JSON-API for statistics, (Looking for contributor to build app for the pool)
*	Dynamic PPLNS block reward (*New)

### How it is different from sammy007 version

*	Reengineered most of the code for efficiency and Scaling
*	New Dynamic PPLNS Reward System
*	Integration with the Exchange to get real-time conversion between crypto and fiat.
*	The Network fees can be configured to be withheld for every transfers. 
*	The gas can be set as Auto and made to deduct automatically or can be fixed by the pool operator
*	Many parameters configurable using config file
*	Nicehash support *Not tested
*	SSL Support built-in
*	Very attractive frontend with more detailed statistics

### Architecture

![Architecture](https://raw.githubusercontent.com/techievee/ethash-mining-pool/master/images/Architecture.PNG)


### **Building on Linux**

Dependencies:

  * go >= 1.9 but <1.10
  * geth or parity (I only tested with parity since geth is discontinued)
  * redis-server >= 2.8.0
  * nodejs >= 4 LTS
  * nginx

**I highly recommend to use Ubuntu 16.04 LTS or 18.04 LTS.**

Start by first installing [parity-ethereum](https://github.com/paritytech/parity-ethereum) and syncing the chain.
Follow by installing redis-server and the previous dependencies.

### **Building Frontend**

Check the [equalhash-statistics-api](https://github.com/OctavioMaia/equalhash-statistics-api) for frontend tutorial.

### **DPPLNS ALGORITHM**

The algorithm explanation is as follows

* CALCULATING THE LAST N VALUE
```javascript
Check whether blockchainnode present
If present
	Calulcate the current network difficulty
	Calulate the network difficulty *2
	Calculate the N value by (2* Network difficulty)/ Share difficulty
 	Set the last N value in the statistics hash key
Else
	Fix the predetermined standard ‘N’ value from the block chain	
```
Space : O(1)	
RunTime : O(1)

* ADJUSTING THE ‘N’ SHARES
```javascript
Get the current last N value from the lastNshares Hash
Get the current count of lastNshares list
If lastNShare< count
	Loop(count - lastNshare )
		Pop lastNShare
		Decrement the miners share count from the lastN value
    		Decrement the total shares count
       end
```

Space : O(1)	
RunTime : O(N)

* NEW SHARE SUBMISSION
```javascript
Push the value of miners address into the last shares list
Increment the current round shares by 1
Increment the round shares value by 1
Increment the miners shares by 1
Check if new block found
If found
	Run the New block function
Else
Adjust the value of ‘N’ shares
```

Space : O(1)	
RunTime : O(1)

* REWARD CALUCLATION USING DPPLNS
```javascript
Loop(lastNshare )
		Get the miners address from list
Increment the local miners current round share 
       End
Loop(miners of current round)
		Percentage= minersshare / current round share
End
```

Space : O(N)	
RunTime : O(N)


### **Configuration Documentation**

Configuration is actually simple, just read it twice and think twice before changing defaults.

The example below is for the ***config_api.json*** file, hence the **false** flags except on the api section.

```json
{ 
   "threads": 2,
   "coin": "etc-pplns",
   "name": "main",
   "pplns": 15000,
   "coin-name": "ETC",
   "proxy": { 
      "enabled": false,
      "listen": "0.0.0.0:9999",
      "limitHeadersSize": 1024,
      "limitBodySize": 256,
      "behindReverseProxy": false,
      "blockRefreshInterval": "120ms",
      "stateUpdateInterval": "3s",
      "difficulty": 4000000000,
      "hashrateExpiration": "3h",
      "healthCheck": true,
      "maxFails": 100,
      "stratum": { 
         "enabled": false,
         "listen": "0.0.0.0:9009",
         "timeout": "120s",
         "maxConn": 8192
      },
      "stratum_nice_hash": { 
         "enabled": false,
         "listen": "0.0.0.0:9099",
         "timeout": "120s",
         "maxConn": 8192
      },
      "policy": { 
         "workers": 20,
         "resetInterval": "60m",
         "refreshInterval": "1m",
         "banning": { 
            "enabled": false,
            "ipset": "blacklist",
            "timeout": 300,
            "invalidPercent": 30,
            "checkThreshold": 30,
            "malformedLimit": 50
         },
         "limits": { 
            "enabled": false,
            "limit": 30,
            "grace": "5m",
            "limitJump": 10
         }
      }
   },
   "api": { 
      "enabled": true,
      "purgeOnly": false,
      "purgeInterval": "10m",
      "listen": "0.0.0.0:9091",
      "statsCollectInterval": "5s",
      "hashrateWindow": "1h",
      "hashrateLargeWindow": "6h",
      "luckWindow": [ 
         25,
         50,
         100,
         200,
         400
      ],
      "payments": 30,
      "blocks": 50
   },
   "upstreamCheckInterval": "5s",
   "upstream": [ 
      { 
         "name": "main",
         "url": "http://127.0.0.1:8545",
         "timeout": "10s"
      },
      { 
         "name": "backup",
         "url": "http://127.0.0.2:8545",
         "timeout": "10s"
      }
   ],
   "redis": { 
      "endpoint": "127.0.0.1:6379",
      "poolSize": 300,
      "database": 9,
      "password": ""
   },
   "unlocker": { 
      "enabled": false,
      "poolFee": 0.5,
      "poolFeeAddress": "0x3d5158949c0cbc060ac102a0ee0df8ccf15a1c99",
      "donate": false,
      "depth": 120,
      "immatureDepth": 20,
      "keepTxFees": false,
      "interval": "15m",
      "daemon": "http://127.0.0.1:8545",
      "timeout": "10s"
   },
   "payouts": { 
      "enabled": false,
      "requirePeers": 5,
      "interval": "60m",
      "daemon": "http://127.0.0.1:8545",
      "timeout": "10s",
      "address": "0x3d5158949c0cbc060ac102a0ee0df8ccf15a1c99",
      "gas": "21000",
      "gasPrice": "20000000000",
      "autoGas": true,
      "keepNwFees": true,
      "nwTxGas": "21000",
      "nwTxGasPrice": "20000000000",
      "threshold": 500000000,
      "bgsave": false
   },
   "exchange": { 
      "enabled": true,
      "url": "https://api.coinmarketcap.com/v1/ticker/ethereum-classic/?convert=USD",
      "timeout": "50s",
      "refreshInterval": "1800s"
   },
   "newrelicEnabled": false,
   "newrelicName": "",
   "newrelicKey": "",
   "newrelicVerbose": false
}
```

### **Sample VM Configurations**

![Configuration](https://raw.githubusercontent.com/techievee/ethash-mining-pool/master/images/Configurations.PNG)

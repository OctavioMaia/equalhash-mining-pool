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


### Building on Linux

Dependencies:

  * go >= 1.9 but <1.10
  * geth or parity (I only tested with parity since geth is discontinued)
  * redis-server >= 2.8.0
  * nodejs >= 4 LTS
  * nginx

**I highly recommend to use Ubuntu 16.04 LTS or 18.04 LTS.**

Start by first installing [parity-ethereum](https://github.com/paritytech/parity-ethereum) and syncing the chain.
Follow by installing redis-server and the previous dependencies.

### Building Frontend

Check the [equalhash-statistics-api](https://github.com/OctavioMaia/equalhash-statistics-api) for frontend tutorial.

### DPPLNS ALGORITHM

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


### Configuration Documentation

Configuration is actually simple, just read it twice and think twice before changing defaults.

```javascript
module.exports = function (environment) {
    var ENV = {
        modulePrefix: 'open-ethereum-pool',
        environment: environment,
        rootURL: '/',
        locationType: 'hash',
        EmberENV: {
            FEATURES: {
                // Here you can enable experimental features on an ember canary build
                // e.g. 'with-controller': true
            }
        },

        APP: {
            // API host and port
            ApiUrl: '//equalhash.pt/',

            // HTTP mining endpoint
            HttpHost: 'equalhash.pt',
            HttpPort: 40002,

            // Stratum mining endpoint
            StratumHost: 'equalhash.pt',
            StratumPort: 9009,

            // Stratum SSL mining endpoint
            StratumHost1: 'equalhash.pt',
            StratumPort1: 8008,

            NicehashHost: '',
            NicehashPost: 40004,

            // Fee and payout details
            PoolFee: '0.5%',
            PayoutThreshold: '0.5',
            ShareDifficulty: '4000000000',

            //Current and Localization
            Currency: 'USD',
            CoinName: 'Ethereum Classic',
            CoinShortName: 'ETC',
            PaymentText: 'every hour',
            SupportMail: '',
            SupportHelpdesk: '',
            WebsiteName: 'equalhash.pt',

            //Coin Bases Settings
            ChainAddress : 'https://etcblockexplorer.com/addr/',
	        TransactionAddress : 'https://minergate.com/blockchain/etc/transaction/',
            UncleAddress : 'https://minergate.com/blockchain/etc/block/',
            BlockAddress : 'https://minergate.com/blockchain/etc/block/',

            //Twitter Parameter
            TwitterURL: '',
            TwitterHash: '',


            // For network hashrate (change for your favourite fork)
            BlockTime: 14
        }
    };

    if (environment === 'development') {
        /* Override ApiUrl just for development, while you are customizing
         frontend markup and css theme on your workstation.
         */
        ENV.APP.ApiUrl = 'https://etc.daggerpool.com/'
        // ENV.APP.LOG_RESOLVER = true;
        // ENV.APP.LOG_ACTIVE_GENERATION = true;
        // ENV.APP.LOG_TRANSITIONS = true;
        // ENV.APP.LOG_TRANSITIONS_INTERNAL = true;
        // ENV.APP.LOG_VIEW_LOOKUPS = true;
    }

    if (environment === 'test') {
        // Testem prefers this...
        ENV.locationType = 'none';

        // keep test console output quieter
        ENV.APP.LOG_ACTIVE_GENERATION = false;
        ENV.APP.LOG_VIEW_LOOKUPS = false;

        ENV.APP.rootElement = '#ember-testing';
    }

    if (environment === 'production') {
        ENV.baseURL = '/ember-cli-twitter-feed/';
    }

    ENV.contentSecurityPolicy = {};
    return ENV;
};
```

### Sample VM Configurations

![Configuration](https://raw.githubusercontent.com/techievee/ethash-mining-pool/master/images/Configurations.PNG)

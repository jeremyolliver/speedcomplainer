# speedcomplainer
A python app that will test your internet connection and then complain to your service provider (and log to a data store if you'd like)

## Prerequisites
You will need the following on your machine before you attempt this
* pip (sudo apt-get install python-pip) for Python 2
* sudo pip install daemon
* sudo pip install twitter or sudo pip install python-twitter
* sudo apt-get install speedtest-cli
* sudo ln -s /usr/bin/speedtest-cli /usr/local/bin/speedtest-cli
* optional if you are on a raspberry pi (pip install twython)

## Twitter Account
You will need to create and account with twitter http://twitter.com
you also need to create a new app https://apps.twitter.com/

## Configuration
Configuration is handled by a basic JSON file. Things that can be configured are:
* twitter
 * twitterToken: This is your app access token
 * twitterConsumerKey: This is your Consumer Key (API Key)
 * twitterTokenSecret: This is your Access Token Secret
 * TwitterConsumerSecret: This is your Consumer Secret (API Secret)
* tweetTo: This is a account (or list of accounts) that will be @ mentioned (include the @!)
* internetSpeed: This is the speed (in MB/sec) you're paying for (and presumably not getting).
* tweetThresholds: This is a list of messages that will be tweeted when you hit a threshold of crappiness. Placeholders are:
 * {tweetTo} - The above tweetTo configuration.
 * {internetSpeed} - The above internetSpeed configuration.
 * {downloadResult} - The poor download speed you're getting

Threshold Example (remember to limit your messages to 140 characters or less!):
```
    "tweetThresholds": {
        "5": [
            "Hey {tweetTo} I'm paying for {internetSpeed}Mb/s but getting only {downloadResult} Mb/s?!? Shame.",
            "Oi! {tweetTo} $100+/month for {internetSpeed}Mbit/s and I only get {downloadResult} Mbit/s? How does that seem fair?"
        ],
        "12.5": [
            "Uhh {tweetTo} for $100+/month I expect better than {downloadResult}Mbit/s when I'm paying for {internetSpeed}Mbit/s. Fix your network!",
            "Hey {tweetTo} why am I only getting {downloadResult}Mb/s when I pay for {internetSpeed}Mb/s? $100+/month for this??"
        ],
        "25": [
            "Well {tweetTo} I guess {downloadResult}Mb/s is better than nothing, still not worth $100/mnth when I expect {internetSpeed}Mb/s"
        ]
    }
```

Logging can be done to CSV files, with a log file for ping results and speed test results. 

CSV Logging config example:
```
"log": {
    "type": "csv",
    "files": {
        "ping": "pingresults.csv",
        "speed": "speedrestuls.csv"
    }
}
```

## Usage
> python speedcomplainer.py

Or to run in the background:
> python speedcomplainer.py > /dev/null &


## Known Issues
You might run accross a few issues.
* If you see any issues saying "AttributeError: 'module' object has no attribute 'Api'" you will need to use twython https://twython.readthedocs.io/en/latest/

Replace
``` 
import twitter 
```
with 
```
from twython import Twython
```
Then replace the existing API "twitter.Api(..." with the following
```
    TTS=self.config['twitter']['twitterTokenSecret']
	TCK=self.config['twitter']['twitterConsumerKey']
	TCS=self.config['twitter']['twitterConsumerSecret']
	TT=self.config['twitter']['twitterToken']
			
	api = Twython(TCK, TCS, TT, TTS)
	
	if api:
        api.update_status(status=message)

```

* You might also run into an issue with this not parsing a float
```
float(downloadResult.replace('Download: ', '').replace(' Mbit/s', ''))
```
Change all instances of Mbit/s to Mbits/s

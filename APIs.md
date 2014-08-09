What are the APIs?
----

Good question. We don't have all of them yet. Here are some we have, please add more:

##### Matcher

In
PUT /relationships

```
{
    "pairId": 1,
    "relationships": [
        {
            "correlatorType": "sentence",
            "relations": [
                {
                    "score": 1,
                    "description": "Twitted together two days ago about Putin"
                },
                {
                    "score": 10,
                    "description": "Coded together three days ago in Groovy"
                }
            ]
        },
        {
            "correlatorType": "place",
            "relations": [
                {
                    "score": 1,
                    "description": "Been in Cracow two days ago"
                },
                {
                    "score": 10,
                    "description": "Been in Berling one week ago"
                }
            ]
        }
    ]
}
```


##### Importance Judge

In
PUT /relationships

```
{
    "pairId": 1,
    "correlatorType": "sentence",
    "relationships": [
        {
            "score": 1,
            "description": "Twitted together two days ago about Putin"
        },
        {
            "score": 10,
            "description": "Coded together three days ago in Groovy"
        }
    ]
}
```
Out 202 Accepted

Out 400 Bad Request - Score not with in range 1-10

Out 400 Bad Request - Valid values for correlatorType [sentence, place, topic]

#####  Twitter Collector

```
In 
/tweets/{twitterLogin}/{pairId}

Out
[
        {
            "extraData":{
    
            },
            "id":494727655208808448,
            "text":"@neuvio Hey, just because I don't understand, doesn't mean it's a waste. As far as position names go though, this is as bad as it gets :)",
            "createdAt":1406787239000,
            "fromUser":"jnabrdalik",
            "profileImageUrl":"http://pbs.twimg.com/profile_images/466951940937498625/wPZPKYS8_normal.jpeg",
            "toUserId":15872334,
            "inReplyToStatusId":494712408234283008,
            "inReplyToUserId":15872334,
            "inReplyToScreenName":"neuvio",
            "fromUserId":2547488709,
            "languageCode":"en",
            "source":"<a href=\"http://twitter.com\" rel=\"nofollow\">Twitter Web Client</a>",
            "retweetCount":0,
            "retweeted":false,
            "retweetedStatus":null,
            "favorited":false,
            "favoriteCount":0,
            "entities":{
                "extraData":{
                    "symbols":[
    
                    ]
                },
                "urls":[
    
                ],
                "mentions":[
                    {
                        "extraData":{
                            "id_str":"15872334"
                        },
                        "id":15872334,
                        "screen_name":"neuvio",
                        "name":"sebastian.konkol",
                        "indices":[
                            0,
                            7
                        ]
                    }
                ],
                "media":[
    
                ],
                "tickerSymbols":[
    
                ],
                "hashTags":[
    
                ]
            },
            "user":{
                "extraData":{
                    "profile_background_image_url_https":"https://abs.twimg.com/images/themes/theme1/bg.png",
                    "profile_image_url_https":"https://pbs.twimg.com/profile_images/466951940937498625/wPZPKYS8_normal.jpeg",
                    "entities":{
                        "url":{
                            "urls":[
                                {
                                    "url":"http://t.co/Z2m77GDql3",
                                    "expanded_url":"http://solidcraft.eu",
                                    "display_url":"solidcraft.eu",
                                    "indices":[
                                        0,
                                        22
                                    ]
                                }
                            ]
                        },
                        "description":{
                            "urls":[
    
                            ]
                        }
                    },
                    "default_profile_image":false,
                    "id_str":"2547488709",
                    "default_profile":false,
                    "is_translation_enabled":false
                },
                "id":2547488709,
                "screenName":"jnabrdalik",
                "name":"Jakub Nabrdalik",
                "url":"http://t.co/Z2m77GDql3",
                "profileImageUrl":"http://pbs.twimg.com/profile_images/466951940937498625/wPZPKYS8_normal.jpeg",
                "description":"Software Dev",
                "location":"",
                "createdDate":1400078639000,
                "language":"pl",
                "statusesCount":108,
                "friendsCount":40,
                "followersCount":160,
                "favoritesCount":128,
                "listedCount":1,
                "following":false,
                "followRequestSent":false,
                "notificationsEnabled":false,
                "verified":false,
                "geoEnabled":false,
                "contributorsEnabled":false,
                "translator":false,
                "timeZone":"Athens",
                "utcOffset":10800,
                "sidebarBorderColor":"C0DEED",
                "sidebarFillColor":"DDEEF6",
                "backgroundColor":"C0DEED",
                "backgroundImageUrl":"http://abs.twimg.com/images/themes/theme1/bg.png",
                "backgroundImageTiled":false,
                "textColor":"333333",
                "linkColor":"000000",
                "protected":false,
                "profileUrl":"http://twitter.com/jnabrdalik"
            },
            "unmodifiedText":"@neuvio Hey, just because I don't understand, doesn't mean it's a waste. As far as position names go though, this is as bad as it gets :)",
            "retweet":false
        }
]
```

#####  Twitter Places Analyzer

```
In 
/api/{pairId}
Output from Twitter Collector

Out

{
    “pairId” : 1,
    “places” :
    [
            {
            "place" :
            {
                "name":"Washington",
                "country_code": "US"
            },
            "probability" : "high",
            "origin" : "twitter_place"
            },
            {
            "place" :
            {
                "name":"Warsaw",
                "country_code": "PL"
            },
            "probability" : "high",
            "origin" : "twitter_mention"
            }
    ]
}
```
#### Topics correlator
```
In
{
TBA
}
Out
{
    "pairId": 1,
    "correlatorType": "topic",
    "relationships": [
        {
            "score": 10,
            "description": "Groovy"
        },
        {
            "score": 2,
            "description": "JavaScript"
        }
    ]
}

```
```
#### Places correlator
```
In
{
    “pairId” : 1,
    "origin": "twitter"
    “places” :
    [
            {
            "place" :
            {
                "name":"Washington",
                "country_code": "US"
            },
            "probability" : "high",
            "origin" : "twitter_place"
            },
            {
            "place" :
            {
                "name":"Warsaw",
                "country_code": "PL"
            },
            "probability" : "high",
            "origin" : "twitter_mention"
            }
    ]
}
Out
{
    "pairId": 1,
    "correlatorType": "place",
    "relationships": [
        {
            "score": 10,
            "description": "Were in Warsaw."
        },
        {
            "score": 2,
            "description": "Were in Lublin."
        }
    ]
}

```

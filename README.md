Microservice Hackaton
=====================

Yep, hack a ton.

Tutorial
----

This page should have all the info you need to start hacking. It's not about organization (you'll find it on our web page:  microhackaton.github.io/2014/), it's about the gory technical hardcore. But you are technical hardcore, aren't you?


What we build
----
Distributed system consisting from microservices, of course! Here is the big picture:

Now for details of the flow:

Normal user enters his details in Web GUI (client-side). Web GUI asks User data holder to generate link for this user, and returns the link.

Normal user gives the link to Celebrity.

Celebrity goes to the link (Web GUI) and enters his details (e.g.: twitter account). Web GUI asks User data holder for user data for given link. Web GUI gives both data to Matcher.

Matcher has a web socket session open with Web GUI

Matcher generates PairId (unique id to recognize the pair User + Celebrity). From now on, PairId is included in every call in a header. 

Matcher sends this data to all registered collector caches. If collector caches do not have the data, they ask collectors. When they finally have the data, they send the data to analyzers.

Each analyzer analyzes the data. For example Twitter places analyzer, searches through tweets for places. Then analyzers send those to Collerators.

A Collector get data from many different analyzers. Each time there is a call with a PairId that has already been present, it searches for common items in all sets of data with that PairId. For example Common Place and Date Collerator, searches the data for common places and dates. Then, Collerators create a score from 1-10 (how close the data is) and send the outcomes to Importance Judge.

Importance Judge gathers all the scores and data, saves it locally, and creates a single prioritized list of where the Celebrity may know the normal person from. Then it sends it to Matcher.

Matcher sends that to Web GUI within its open session. Web GUI presents the information to the Celebrity.

The thing to note here is: no service waits for another. Everything is asynchronous. 
If a Correlator gets data with PairId it never had before, it saves that, and does nothing. 
When it gets data for this PairId from a second analyzer, it saves that, correlates, and sends outcomes to Importance Judge. 
When it gets the data for this PairId from a third analyzer, it saves that, correlates data from all three analyzers, and sends outcomes to Importance Judge. 
And so on.

This way, stuff should start appearing on Web GUI as fast as possible, and nothing blocks anything.

Show me the code (examples)!
----

Here you have them

Twitter collector: https://github.com/microhackaton/twitter-collector

Twitter places analyzer: https://github.com/microhackaton/twitter-places-analyzer

UI: https://github.com/microhackaton/you-should-remember-me-ui

But how do I microserivce?
----

That's a valid question. If writing distributed systems was easy, we wouldn't be writing anything else. There is actually a shitload of things to consider. 

#### Service discovery

First of all, when you are going distributed, it helps a lot, when services can find each other, when you can just throw your service to production, tell it what dependencies it has, and not worry about IPs, DNSes, etc. Especially when services die, it would be great if the rest of your application would work. Since you can have several instances of a service, the fact that one dies, should not matter much. Load balance that shit!

We use [ZooKeeper/Curator] for that. It allows us to:
- register service availability
- Locating a single instance of a particular service
- Notifying when the instances of a service change
 
And so we get our URL for you dependencies like that:

```
String analyzerUrl = serviceResolver.getUrl('analyzer').get()
restTemplate.put("$analyzerUrl/{pairId}", createEntity(tweets), pairId)
```

To make it easy on you, we have created a project that takes care of setting this up. Here, it has quite a good tutorial [micro-deps]

TL;DR? you need to add that dependency, and then:
    
    microservice.json - describes your service and its dependencies

Also, if you want to react, when your dependency state changes, you can register you listener here:

    DependencyWatcher.registerDependencyStateChangeListener
    
Now every time any of your dependency changes, you will get a notification with the dependencyName (the key from dependencies) and the state of it.

[ZooKeeper/Curator]:http://curator.apache.org/curator-x-discovery/index.html
[micro-deps]:https://github.com/4finance/micro-deps

#### Configuration for Spring

Winter may be comming, but we have zookeeper configuration for Spring ready. You may love farming in Minecraft, but you don't have to setup the beans yourself. Here is what you need: [micro-deps-spring-config]

[micro-deps-spring-config]:https://github.com/4finance/micro-deps-spring-config

#### Stubs, stubs everywhere

You want to fire your microservice, but it depends on others, right? It's not hard, you can use Wiremock for that in tests. But it's even nicer, when people who create microservices prepare a stub (a.k.a. test double) for you. 

We have that as well, here is the repo you need: [boot-microservice-stub]

If you follow the readme, you gonna have automatically deployable stubs for you microservice. Trust us, your consumers will love you.

[boot-microservice-stub]:https://github.com/4finance/boot-microservice-stub

#### Centralized logging

When you have several microservices running together, grepping a log file doesn't work anymore. You need centralized logging. We have that for you as well.

<TODO: describe it>


#### CorrelationId, healthcheck, ping and other infrastructural stuff

It's nice to have logs, but it's nicer to be able to read them. When end-user hits your system, you create a correlationId, that you should pass in the header, to all the other services you call, and they should propagate it as well. It makes reading logs a blast, you just search centralized logs by the correlationId to understand how the flow went.

And, of course, you don't have to do it yourself. [micro-infra-spring] project has it done for you. If you are not Springy, just take a look at the: com.ofg.infrastructure.web.filter.correlationid.CorrelationIdFilter 

[micro-infra-spring]:https://github.com/4finance/micro-infra-spring

This project has much more, than just correlationId. Take a look at it, and you'll see metrics, healthcheck, ping, CORS filtering, REST Template, Swagger. It's implemented for Spring, but if you use a different technology, it's a good starting/reference point.

We even added a profile verification, so you won't run it with wrong profile accidentaly on production (happened to us).


What are the APIs?
----

Good question. We don't have all of them yet. Here are some we have, please add more:

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
/{pairId}
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
}l
```

#### NOW GO, START HACKING

Remember developer's ABC: Always Be Coding!
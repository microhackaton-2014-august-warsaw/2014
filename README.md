Microservice Hackaton
=====================

Yep, hack a ton.

This page should have all the info you need to start hacking. It's not about organization (you'll find it on our web page:  [microhackaton.github.io/2014/]), it's about the gory technical hardcore. But you are technical hardcore, aren't you?

[microhackaton.github.io/2014/]:microhackaton.github.io/2014/

What we build
----

Do you hate it, when you are a famous internet celebrity with thousands of friends, and somebody wants to add you to his friends/circles/linkedin/whatever, and acts like you should already know him/her, but you look on the picture and name and have absolutely no fucking idea, who the person is?

Yeah, we too.

So we are building a service, which will tell you, who this person is, and where do you know this person from. On one page. No search necessary.

Welcome to YouShouldRememberMe

User story:

As an average human being, I want to add this internet celebrity to add me to friends, so I can act as if the one time we had a beer together means something.
In order to do that:
- I go to the site
- I enter my details (github login, twitter login, google+ login, name, some other stuff)
- The service creates a link for me.
- I send the link to my celebrity

As a celebrity I get a facebook/google+/whatever invitation with a link created by our system.
- I go to the link
- I enter my details (github login, twitter login, google+ login, name, some other stuff)
- The service analyzes (anti)social networks, github, other places, and shows me a short summary of where I know that person from, and who the hell is (s)he.

How we build
----
Distributed system consisting from microservices, of course! Here is the big picture:

![Alt text](https://raw.githubusercontent.com/microhackaton/2014/master/microhackathon_flow_0_1.png "What are we building: flow")

Now for details of the flow:

Normal user enters his details in Web GUI (client-side). Web GUI asks User data holder to generate link for this user, and returns the link.

Normal user gives the link to Celebrity.

Celebrity goes to the link (Web GUI) and enters his details (e.g.: twitter account). Web GUI asks User data holder for user data for given link. Web GUI gives both data to Matcher.

Matcher has a web socket session open with Web GUI

Matcher generates PairId (unique id to recognize the pair User + Celebrity). From now on, PairId is included in every call in a header. 

Matcher sends this data to all registered collector caches. If collector caches do not have the data, they ask collectors. When they finally have the data, they send the data to analyzers.

Each analyzer analyzes the data. For example Twitter places analyzer, searches through tweets for places. Then analyzers send those to Collerators.

A Correlator get data from many different analyzers. Each time there is a call with a PairId that has already been present, it searches for common items in all sets of data with that PairId. For example Common Place and Date Correlator, searches the data for common places and dates. Then, Collerators create a score from 1-10 (how close the data is) and send the outcomes to Importance Judge.

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

But how do I microservice?
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

[boot-microservice-stub]:https://github.com/microhackaton/boot-microservice-stub

#### Works on my machine (with stubs)

Now that you have stubs, you want to fire them on your machine, right?

No problem. At least as long as you're using gradle. You are using gradle, right?

[micro-deps-gradle-plugin]

This project gives you some fancy gradle tasks to run stubs/mocks on your machine (and others, if you want to).

[micro-deps-gradle-plugin]: https://github.com/4finance/micro-deps-gradle-plugin


#### Centralized logging

When you have several microservices running together, grepping a log file doesn't work anymore. You need centralized logging. We have that for you as well.

We use logback reporting to logstash, with Kibana as visualization. If you are using logback, just set this as a log pattern:
    
    String logPattern = "%d{yyyy-MM-dd HH:mm:ss.SSSZ, Europe/Warsaw} | %-5level | %X{correlationId} | %thread | %logger{1} | %m%n"
    
And feed logstash, with for example [logstash-forwarder].

[logstash-forwarder]:https://github.com/elasticsearch/logstash-forwarder


#### CorrelationId, healthcheck, ping and other infrastructural stuff

It's nice to have logs, but it's nicer to be able to read them. When end-user hits your system, you create a correlationId, that you should pass in the header, to all the other services you call, and they should propagate it as well. It makes reading logs a blast, you just search centralized logs by the correlationId to understand how the flow went.

And, of course, you don't have to do it yourself. [micro-infra-spring] project has it done for you. If you are not Springy, just take a look at the: com.ofg.infrastructure.web.filter.correlationid.CorrelationIdFilter 

[micro-infra-spring]:https://github.com/4finance/micro-infra-spring

This project has much more, than just correlationId. Take a look at it, and you'll see metrics, healthcheck, ping, CORS filtering, REST Template, Swagger. It's implemented for Spring, but if you use a different technology, it's a good starting/reference point.

We even added a profile verification, so you won't run it with wrong profile accidentaly on production (happened to us).


What are the APIs?
----

Here: [APIs.md]

[APIs.md]:https://github.com/microhackaton/2014/blob/master/APIs.md

How do I deploy/Zookeep/access DB?
---- 

Want some infrastructure? Of course you want some infrastructure! What good are microservices without infrastructure.

You've got it! [infrastructure-setup]

[infrastructure-setup]: https://github.com/microhackaton/2014/wiki/Infrastructure-setup

FAQs?
----

Still having questions? Shit doesn't build on your computer? Have a look at our [FAQ]

[FAQ]:https://github.com/microhackaton/2014/wiki/FAQ

Is that all I need to know? 
-----

Shit, no! This stuff is complex. Need some theory behind this? Need to consider latency/fault tolerance? Need to stop cascading failures? There is a lot to read, and a lot to do.

Going microservices is much harder in the short run, than a single JVM app. But in the long run, a large team on single, large JVM app, does not work. For several reasons. Read this:

http://martinfowler.com/articles/microservices.html

http://www.infoq.com/minibooks/emag-microservices#idp_register

http://microservices.io/

http://qconlondon.com/dl/qcon-london-2014/slides/AdrianCockcroft_MigratingToMicroservices.pdf

Or maybe a book? You remember books, right?

http://shop.oreilly.com/product/0636920033158.do

http://www.amazon.com/The-Phoenix-Project-Helping-Business/dp/0988262592


#### Have fun!

Your chosen technology turned out to be a piece of crap? You can’t handle Scala UTF-8 one liners? You’ve just found that nodejs is about js and you hate js with passion? Fuck that. Drop it, get another technology and play with it.

Remember developer's ABC: Always Be Coding!

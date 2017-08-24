---
layout: "post"
title: "Why You might be facing Latency issues with Heroku and AWS"
date: "2017-08-10 22:19"
published: true
tags: [aws dynamodb heroku]
categories: aws
permalink: :categories/:title
---

Like most of the rails developers, I prefer to use heroku for hosting applications; especially when the application is in early stage. At work, we use EC2 sometimes, sometimes heroku. Heroku and Ruby is perfact combination.

In a application at work we used AWS DynamoDB along with Heroku Postgres for database. Surprisingly, the endpoints interacting with DynamoDB tables were slow. According to cloudwatch average query latency was around 20-50 ms, which is quite good. However, total time spent querying was found around 1.x seconds.

I was using AWS Ruby SDK v2, and DynamoDB client was initialized in initializer with following Singleton class as below:

{% gist 8parth/65186f32f29488333cbc06389096db73 %}

Hence, dynamodb client was not being created in every request.

Only one thing left to blame was network latency. My DynamoDB table was in `us-west-2` region (US - Oregon). I searched for network latency for different regions from my laptop to understand how much difference it might make and I stumbled upon [clouding.info](http://www.clouding.info) .

![image]()

Latency from my home location (India, from which nearest AWS region is Mumbai) to `us-west-2` was at average 350 ms. So, the next thing to look for was AWS region from which heroku dynos running the application are being operated. By default heroku creates applications in `us` region if no preference is selected while creation. [Heroku Dev Center](https://devcenter.heroku.com/articles/regions#view-the-list-of-available-regions) lists available regions. Running `heroku info` shows region of application.

As heroku uses AWS to run dynos, you can know under which aws region the dyno is being run by doing following curl request.

```ruby
curl -n -X GET https://api.heroku.com/regions/us -H "Accept: application/vnd.heroku+json; version=3"
```














And another website (clouding.co)[https://www.cloudping.co/] showed inter-region latency, which is of around 250.92 ms.

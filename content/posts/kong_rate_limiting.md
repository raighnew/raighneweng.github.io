+++
title = 'How to implement rate limiting by tier in Kong without the Enterprise Edition'
date = 2023-03-29 21:49:33+08:00
tags = ["Kong", "Backend"]
+++


Recently, I came across a requirement where I needed to implement rate limiting based on customer group and HTTP method. For instance, I want to limit the number of calls free users can make to GET routes to 1000 per minute and to POST routes to 500 per minute. I am aware that rate-limiting-advance supports rate limiting by customer group, but it requires the Enterprise version. How can I implement rate limiting by HTTP method?

<!--more-->

Here's my solution:

#### Create the service and routes

Fortunately, Kong allows using the same path to create distinct routes for different HTTP methods. To achieve the required rate-limiting, you can set up two different routes for the same path, one for GET requests and the other for POST requests, and apply the desired rate-limiting plugin to each of them according to the requirements. This approach allows you to limit the number of calls specific to each method and group of customers.

```yaml
services:
  - name: test-service
    url: http://localhost:3000
    routes:
      - name: read-route
        methods:
          - GET
        paths:
          - /v1
      - name: write-route
        methods:
          - POST
          - PUT
          - DELETE
        paths:
          - /v1
```

#### Create consumer plugins

We need to create plugins for each consumer and call the admin API to create or update the consumer and consumer plugins.

```yaml
plugins:
  - config:
      limit_by: consumer
      minute: 1000
      policy: local
    consumer: consumer_id
    enabled: true
    name: rate-limiting
    route: read-route
  - config:
      limit_by: consumer
      minute: 500
      policy: local
    consumer: consumer_id
    enabled: true
    name: rate-limiting
    route: write-route
```

#### Any performance issues?

My solution will create two plugins for each consumer, which means that for 5000 consumers, it will have 10000 plugins in place. This volume of plugins can potentially affect [the performance of Kong](https://docs.konghq.com/gateway/latest/production/sizing-guidelines/).

Let's have a test. I have written a script to create 1000 consumers and 2000 plugins, and I am using JMeter to conduct some stress tests. As a baseline, let's start with 50 RPS. What are the test results?

![](/images/50RPS.png)
![](/images/50RPS-Latencies.png)

Alright, let's start stressing the test. We increased the RPS to 500. Here are the test results:
![](/images/500RPS.png)
![](/images/500RPS-Latencies.png)

Based on the test results, is it considered acceptable?

#### In the long term

As the user count increases, my current solution will do impact the performance of Kong. In that case, we can consider purchasing Kong Enterprise. The total number of plugins will be routes count multiplied by consumer groups count.
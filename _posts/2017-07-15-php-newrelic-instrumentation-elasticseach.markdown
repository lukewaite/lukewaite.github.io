---
layout: post
title:  "NewRelic Instrumentation of the Elasticsearch PHP SDK"
date:   2017-07-15 09:00:00
categories: newrelic php instrumentation elasticsearch
---

We run Elasticsearch in production, fronted by an API which abstracts away complex queries and
presents our APIs which consume the data a consistent interface. It came to my attention
recently that we had no visibility in NewRelic on external transaction time going to ES.

In a nutshell, the problem turned out to be that the elasticsearch-php-sdk uses [RingPHP][ringphp]
as a transport, which NewRelic doesn't support.

I've written a [RingPHP Guzzle Adapter][ringphp_guzzle] which the user can
[pass to elasticsearch][elasticsearch_handler_config] and quickly gain instrumentation.

{% highlight php %}
$guzzle = new GuzzleHttp\Client();
$guzzleHandler  = new LukeWaite\RingPhpGuzzleHandler\GuzzleHandler($guzzle);

$client = Elasticsearch\ClientBuilder::create()
            ->setHandler($guzzleHandler)
            ->build();
{% endhighlight %}

{% include image.html url="/images/2017-07-15-newrelic-elasticsearch/newrelic-instrumentation.png" description="NewRelic Before and After Instrumentation" %}

[ringphp]: https://github.com/guzzle/RingPHP
[ringphp_guzzle]: https://github.com/lukewaite/ringphp-guzzle-handler
[elasticsearch_handler_config]: https://www.elastic.co/guide/en/elasticsearch/client/php-api/current/_configuration.html#_configure_the_http_handler
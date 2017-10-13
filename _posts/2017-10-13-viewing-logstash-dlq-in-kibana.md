---
layout: post
title:  "Viewing the Logstash Dead Letter Queue in Kibana"
date:   2017-10-13 09:00:00
categories: logstash monitoring kibana dead-letter-queue
---

At [Intouch Insight][intouch], our logging infrastructure is our holy
grail. The engineering team that relies on it every day, so we need to keep
it up to snuff. I was lucky enough to be able to update our ELK cluster
this week to 5.6 - a huge upgrade from our previous stack running ES 2.3
and Kibana 4.

One of the features I was most looking forward to was the [dead letter
queue][dlq] that was [introduced in Logstash 5.5][dlq-blog].

## The Problem
The documentation surrounding the usage of the dead letter queue mostly
revolves around [re-processing rejected events][reprocessing]. I wasn't
particularly interested in that use case; I just wanted to be able to
**easily see when events were rejected**.

## The Solution
{% highlight conf linenos %}
input {
    dead_letter_queue {
        path => "/usr/share/logstash/data/dead_letter_queue"
    }
}

filter {
    # First, we must capture the entire event, and write it to a new
    # field; we'll call that field `failed_message`
    ruby {
        code => "event.set('failed_message', event.to_json())"
    }

    # Next, we prune every field off the event except for the one we've
    # just created. Note that this does not prune event metadata.
    prune {
        whitelist_names => [ "^failed_message$" ]
    }

    # Next, convert the metadata timestamp to one we can parse with a
    # date filter. Before conversion, this field is a Logstash::Timestamp.
    # http://www.rubydoc.info/gems/logstash-core/LogStash/Timestamp
    ruby {
        code => "event.set('timestamp', event.get('[@metadata][dead_letter_queue][entry_time]').toString())"
    }

    # Apply the date filter.
    date {
        match => [ "timestamp", "ISO8601" ]
    }

    # Pull useful information out of the event metadata provided by the dead
    # letter queue, and add it to the new event.
    mutate {
        add_field => {
            "message" => "%{[@metadata][dead_letter_queue][reason]}"
            "plugin_id" => "%{[@metadata][dead_letter_queue][plugin_id]}"
            "plugin_type" => "%{[@metadata][dead_letter_queue][plugin_type]}"
        }
    }
}
{% endhighlight %}

### How it looks
{% include image.html url="/images/2017-10-13-dlq-kibana/logs-list.png" description="Dead Letter Queue - Logs" %}
{% include image.html url="/images/2017-10-13-dlq-kibana/logs-detail.png" description="Dead Letter Queue - Detail" %}

[intouch]:      https://www.intouchinsight.com
[dlq]:          https://www.elastic.co/guide/en/logstash/current/dead-letter-queues.html
[dlq-blog]:     https://www.elastic.co/blog/logstash-lines-2017-05-16
[reprocessing]: https://www.elastic.co/guide/en/logstash/5.6/dead-letter-queues.html#dlq-example
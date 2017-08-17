---
layout: post
title:  "Finding Files in S3 (without a known prefix)"
date:   2017-08-03 21:00:00
categories: aws s3 search
---

S3 is a fantastic storage service. We use it all over the place, but sometimes
it can be hard to find what you're looking for in buckets with massive
data sets. Consider the following questions:

> What happens when you know the file name, but perhaps not the full prefix
(path) of the file?

I hit this in production today, which is the motive of the blog post. The
next question is what I thought might be a useful example, when I wanted
to extrapolate what I'd learned to other use cases.

> How do you find files modified on specific dates, regardless of prefix?

<!--more-->

Typically, this is when things start to get difficult. As a starting point,
I use [Transmit][transmit]{:target="_blank"} - it's a fantastic tool. However, it starts to fall
down when you need to either deal with "folders" that contain many tens of
thousands of documents, or when you need to look for things that could be
in multiple folders.

Both of the above questions can be answered relatively easily by using the
[`--query` parameter of the aws cli][query]{:target="_blank"}. You can pass to `--query` any
[JMESPath][jmespath]{:target="_blank"} query.

#### Find file by partial name

For this example, we will search for a file name containing `1018441`. I
have slightly redacted some bits of the path as this is a query of
production data.
{% highlight bash %}
$ aws s3api list-objects --bucket <bucket-name> --query "Contents[?contains(Key, '1018441')]"
{% endhighlight %}

{% highlight json %}
[
    {
        "LastModified": "2017-05-31T20:36:28.000Z",
        "ETag": "\"b861daa5cc3775f38519f5de6566cbe7\"",
        "StorageClass": "STANDARD",
        "Key": "clients/<client name>/programs/0ced4d20939fe16978df9e6d8f8985ad/1018441-synth.json",
        "Owner": {
            "DisplayName": "owner",
            "ID": "123"
        },
        "Size": 27032
    },
    {
        "LastModified": "2017-07-27T19:38:47.000Z",
        "ETag": "\"829ba43ee2bca9e20f7165ef7e7f055b\"",
        "StorageClass": "STANDARD",
        "Key": "clients/<client-name>/programs/8c1bb9a738f64c9219d8eda292f90394/1018441-synth.json",
        "Owner": {
            "DisplayName": "owner",
            "ID": "123"
        },
        "Size": 29381
    },
    {
        "LastModified": "2017-07-27T19:05:55.000Z",
        "ETag": "\"70f187a8bbbd2d5c2c54844c1b6da3a6\"",
        "StorageClass": "STANDARD",
        "Key": "cache/1018441.json",
        "Owner": {
            "DisplayName": "owner",
            "ID": "123"
        },
        "Size": 12718
    }
]
{% endhighlight %}

In this case, we can quickly see that when we were expecting only two files
by this name (a cache, and a processed file) we have two different processed
files! Here's our problem.

#### Find files modified on a given date
{% highlight bash %}
$ aws s3api list-objects --bucket <bucket name> --query "Contents[?contains(LastModified) > '2017-08-03')]"
{% endhighlight %}

#### Find files modified between given times
{% highlight bash %}
$ aws s3api list-objects --bucket <bucket name> --query "Contents[?LastModified > '2017-08-03T23' && LastModified < '2017-08-03T23:15']"
{% endhighlight %}

Technically, the ability to perform a comparison on a date is not part of
the official spec for JMESPath, but I found a GitHub issue which describes
that [many users rely on the ability to do just this][date-issue]{:target="_blank"}.

That issue was closed by [this commit][date-fix]{:target="_blank"} which mentions that it
will be added explicitly to the spec:

> The spec doesn't officially support string types yet, but enough people
are relying on this behavior that it's been added back.  This should
eventually become part of the official spec.

### Caveats
When you use the `--query` option, it's all done as post-processing, on your
local system. Nothing is done remotely by s3, it's essentially the same as
piping the output to `jq`, [`jp`][jp]{:target="_blank"} or another filtering program. This means if
you have many 100s of thousands, or millions of objects, or a slow connection
it will not be a fast process.

[transmit]: https://www.panic.com/transmit/
[query]: http://docs.aws.amazon.com/cli/latest/reference/index.html#options
[jmespath]: http://jmespath.org/
[date-issue]: https://github.com/jmespath/jmespath.py/issues/124
[date-fix]: https://github.com/jamesls/jmespath/commit/b893fc2bdd52fdd227a8cec3b7a26574de813e67
[jp]: https://github.com/jmespath/jp
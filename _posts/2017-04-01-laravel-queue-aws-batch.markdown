---
layout: post
title:  "AWS Batch as a Laravel Queue"
date:   2017-04-01 11:50:00
categories: aws laravel batch queue
permalink: :categories/:year/:month/:day/:title.html
---

We use Laravel for all of our APIs at [Intouch Insight][intouch], so when [AWS Batch][batch] was released, I started
wondering about backing our Laravel Queues with AWS Batch. This seemed like the perfect opportunity to give back,
since I'm sure others are looking at Batch with the same interest that I am. A few evenings of playing around, and
here we are.

<!--more-->

I've just published the initial release of my [laravel-queue-aws-batch][batch_queue] plugin on [packagist][packagist].

Initially the plugin has been built for Laravel 5.1, since we use the LTS releases. ~~Support for 5.4 will be coming, or
quite possibly could already be working, but I need to review what has changed in the underlying queue classes before
making that promise.~~

If you review the [implementation][batch_queue], you'll see that I based my work off the `DatabaseQueue`
class, as we need a place to store jobs before batch can process them. Serializing the entire job payload and passing that
as a string through the job submission to batch seemed like it could be error prone, to say the least.

~~Tests are needed, and will be coming.~~ I'd love to hear feedback, or receive pull requests for improvements.

### Update - Stable Releases are now available
[Stable releases are now available]({{ site.baseurl }}{% post_url 2017-04-09-laravel-queue-aws-batch-stable %}) for Laravel 5.1
through to Laravel 5.4.

### Feedback
How has this worked for you? Found a problem? Let me know and open an [issue][issues]!

[intouch]:         https://www.intouchinsight.com
[batch]:           https://aws.amazon.com/batch/
[batch_queue]:     https://github.com/lukewaite/laravel-queue-aws-batch
[packagist]:       https://packagist.org/packages/lukewaite/laravel-queue-aws-batch
[issues]:          https://github.com/lukewaite/laravel-queue-aws-batch/issues

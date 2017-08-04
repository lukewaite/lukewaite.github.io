---
layout: post
title:  "Laravel Caching for AWS Credentials"
date:   2017-04-29 16:00:00
categories: aws laravel iam
permalink: posts/2017/04/29/laravel-aws-cache-adapter.html
---

We've seen occasionally poor performance on the AWS EC2 Metadata API when using IAM roles at [Intouch][intouch] which got
me thinking. Why does the `aws-pdp-sdk` need to hit the EC2 Metadata API during every request? Well, it turns out, it's
simple. If you don't explicitly give the sdk a cache interface, then it won't use one!

I've just published the initial release of my [laravel-aws-cache-adapter][cache_adapter] plugin on [packagist][packagist].

<!--more-->

If you pass an instance of the [CacheInterface][cache_interface] class through as your `credentials` setting when you
instantiate the aws-sdk, it'll use that interface to cache STS tokens returned via the metadata API when granting your
application permissions via either EC2 IAM Roles or ECS Task IAM Roles. Simple!

### Feedback
How has this worked for you? Found a problem? Let me know and open an [issue][issues]!

[intouch]:         https://www.intouchinsight.com
[cache_interface]: http://docs.aws.amazon.com/aws-sdk-php/v3/api/class-Aws.CacheInterface.html
[cache_adapter]:   https://github.com/lukewaite/laravel-aws-cache-adapter
[packagist]:       https://packagist.org/packages/lukewaite/laravel-aws-cache-adapter
[issues]:          https://github.com/lukewaite/laravel-aws-cache-adapter/issues

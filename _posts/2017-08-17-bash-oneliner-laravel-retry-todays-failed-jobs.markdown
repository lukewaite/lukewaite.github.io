---
layout: post
title:  "Bash One-Liner: Retry Laravel Failed Jobs, Filtered by Date"
date:   2017-08-17 22:00:00
categories: bash-one-liner laravel queue retry
---

We all have those moments when Queue jobs fail. Sometimes it's a bad deploy,
others it's an upstream service that's taken a poop. Sometimes, we need to
retry failed jobs, but can't just `artisan queue retry:all`, because maybe we
haven't done a cleanup of failed jobs lately.

### Oops...

{% highlight console %}
worker@5986f48e91d6:/var/www/app# php artisan queue:failed | grep -v 2017-08-17
+------+------------+--------------------------------------------------+-------------------------+---------------------+
| ID   | Connection | Queue                                            | Class                   | Failed At           |
+------+------------+--------------------------------------------------+-------------------------+---------------------+
| 3094 | sqs        | https://sqs.us-east-1.amazonaws.com/123/my-queue | App\Jobs\MyImportantJob | 2017-08-17 21:14:07 |
...
| 851  | sqs        | https://sqs.us-east-1.amazonaws.com/123/my-queue | App\Jobs\MyImportantJob | 2017-08-17 03:35:08 |
| 850  | sqs        | https://sqs.us-east-1.amazonaws.com/123/my-queue | App\Jobs\MyImportantJob | 2017-07-28 11:35:08 |
| 849  | sqs        | https://sqs.us-east-1.amazonaws.com/123/my-queue | App\Jobs\MyImportantJob | 2017-07-28 11:27:09 |
| 848  | sqs        | https://sqs.us-east-1.amazonaws.com/123/my-queue | App\Jobs\MyImportantJob | 2017-07-28 11:26:37 |
| 847  | sqs        | https://sqs.us-east-1.amazonaws.com/123/my-queue | App\Jobs\MyImportantJob | 2017-07-28 11:20:38 |
...
| 262  | sqs        | https://sqs.us-east-1.amazonaws.com/123/my-queue | App\Jobs\MyImportantJob | 2017-03-21 10:33:19 |
+------+------------+--------------------------------------------------+-------------------------+---------------------+
{% endhighlight %}

<!--more-->

In my case today, I was running some maintenance jobs overnight, when I
overworked a legacy database and caused about ~2500 of 1m+ jobs to timeout.
Not a big problem, but the jobs needed to be retried.

A colleague watching me do it, informed me that last time they did something like this
(I sincerely hope it wasn't 2500 failed jobs) they typed all the numbers in manually!

## So, down to the point, the oneliner:

{% highlight shell %}
$ php artisan queue:retry $(php artisan queue:failed | grep 2017-08-17 | awk '{print $2}')
{% endhighlight %}

### Let's break that down

#### Basics
* `$( ... )`
  * [Command substitution][cmdsub]{:target="_blank"} allows you to insert the results of the contents into your command
* `|`
  * The classic [bash pipe][pipe]{:target="_blank"} sends the output of one command to the input of the next


#### The commands used
* `php artisan queue:retry $( ... )`
  * Run `artisan queue:retry`, we'll end up giving it a list of IDs for the jobs to retry
* `php artisan queue:failed`
  * Get the listing of failed jobs
* `grep 2017-08-17`
  * Search for the date we want to filter by in each line
* `awk '{print $2}'`
  * awk will by default split on spaces, so print id which is in the second captured group (the `|` from the beginning of the table will be in the first position)

## Other uses
Change the `grep`, and you can filter by a lot more than date.

* `grep -v` will inverse filter, so you can run everything _except_ a given date
* `grep <job name>` - filter only specific classes
* `grep <queue name>` - filter specific queues, or connections

[cmdsub]: https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html
[pipe]:   http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-4.html
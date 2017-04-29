---
layout: post
title:  "Improving OpsWorks Boot Times"
date:   2015-02-25 12:15:00
categories: aws opsworks
permalink: :categories/:year/:month/:day/:title.html
---

#### The Problem
Initial setup of OpsWorks instances takes between 15 and 25 minutes depending on the complexity of your chef recipes.

#### Why?
The OpsWorks startup process injects a sequence of updates and package installations via the instance userdata before 
setup can run. To make matters worse, the default Ubuntu 14.04 AMI provided by AWS (at the time of writing) over six
months old! YMMV but I experienced a 11 minute speedup simply in "time to `running_setup`" by introducing a simple 
custom AMI.

<!--more-->

#### How do I get this? 
* Start a new EC2 instance based on the must current Ubuntu 1404 AMI.
* Run the following commands on the instance:
{% highlight bash %}
apt-get update
apt-get install -y ruby ruby-dev libicu-dev libssl-dev libxslt-dev libxml2-dev monit
apt-get dist-upgrade -y
{% endhighlight %}
* Create an AMI based on your EC2 instance http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html
* Use your AMI to boot instances in your stack

#### Can I make it even faster?
Sure! Do you have other packages that are common to your entire stack? For us it's docker - and we can even preload 
some base docker images if we want - for you it may be PHP, Node... Install current versions, then make your AMI. 

**Don't forget to keep your custom AMI relatively up to date, or you'll lose your speed bonus!**

Want to automate? Due to the simplicity, I haven't gone there yet, but you could likey use [aminator][aminator] or even
a dedicated OpsWorks stack and cookbooks. If you go the OpsWorks route, don't forget to delete the instance identity
data as specified in: http://docs.aws.amazon.com/opsworks/latest/userguide/workinginstances-custom-ami.html



[aminator]: https://github.com/Netflix/aminator

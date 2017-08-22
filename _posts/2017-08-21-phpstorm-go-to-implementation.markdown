---
layout: post
title:  "PHPStorm Tips - Go to Implementation"
date:   2017-08-21 22:00:00
categories: phpstorm tip laravel ide
---

Laravel is a great framework; it's easy to use and extend, and makes liberal
use of Interfaces so that you can write your own implementations and provide
them to the IoC Container.

Unfortunately, the downside to this is navigability. Many times, your IDE detects
that a variable has either been type hinted, or has code comments indicating that
it is storing a particular Interface, rather than a concrete class, and this
sometimes makes navigation difficult.

Allow me to introduce you to `Go to Implementation (CMD-OPT-B)` in PHPStorm -
just one additional key press away from the most commonly used `Go to Declaration (CMD-B)`
It'll allow you to select a specific implementation of whatever abstract method
you've got a reference to.

<!--more-->

{% include image.html url="/images/2017-08-21-phpstorm-go-to-implementation/go-to-implementation-filesystem.php.png" description="Go to Implementation - Filesystem.php" %}

### Video Demo

{% include youtube.html video="C0ZExLjMawo" %}

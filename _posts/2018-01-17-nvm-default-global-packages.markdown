---
layout: post
title:  "NVM Default Global Packages"
date:   2018-01-17 10:45:00
categories: nvm nodejs npm development tips
---

Did you know that `nvm` supports a list of [default global packages][default-packages] to include when installing
a new node version?

> ### Default global packages from file while installing
>  
>  If you have a list of default packages you want installed every time you install a new version we support that too. You can add anything npm would accept as a package argument on the command line.
>  
{% highlight conf %}
 # $NVM_DIR/default-packages
 
 rimraf
 object-inspect@1.0.2
 stevemao/left-pad
{% endhighlight %}

Personally, I use this to make sure `gulp-cli` is installed in new node versions.

[default-packages]: https://github.com/creationix/nvm/blob/master/README.md#default-global-packages-from-file-while-installing


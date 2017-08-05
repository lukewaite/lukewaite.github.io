---
layout: post
title:  "GitLab, NPM, and SSHd MaxStartups"
date:   2015-08-17 21:45:00
categories: gitlab npm ssh
permalink: :categories/:year/:month/:day/:title.html
---

Intermittent issues are every developer's best friend. Recently we started hitting an error during the `npm install` 
phase of our CI and CD jenkins jobs. Here's the error:

{% highlight shell %}
npm ERR! git fetch -a origin (ssh://git@git-server.internal/group/project.git) ssh_exchange_identification: read: Connection reset by peer
npm ERR! git fetch -a origin (ssh://git@git-server.internal/group/project.git) fatal: Could not read from remote repository.
npm ERR! git fetch -a origin (ssh://git@git-server.internal/group/project.git)
npm ERR! git fetch -a origin (ssh://git@git-server.internal/group/project.git) Please make sure you have the correct access rights
npm ERR! git fetch -a origin (ssh://git@git-server.internal/group/project.git) and the repository exists.
npm ERR! Linux 3.13.0-54-generic
npm ERR! argv "/usr/bin/node" "/usr/bin/npm" "install"
npm ERR! node v0.12.4
npm ERR! npm  v2.10.1
npm ERR! code 128

npm ERR! Command failed: git fetch -a origin
npm ERR! ssh_exchange_identification: read: Connection reset by peer
npm ERR! fatal: Could not read from remote repository.
npm ERR!
npm ERR! Please make sure you have the correct access rights
npm ERR! and the repository exists.
npm ERR!
npm ERR!
npm ERR! If you need help, you may report this error at:
npm ERR!     <https://github.com/npm/npm/issues>
{% endhighlight %}

In our particular use case, we'd just passed the threshold of having more than 10 internally sourced NPM dependencies,
being sourced by tag directly from our GitLab server.

The solution is updating quite a simple SSH setting; `MaxStartups`. Here's the man page entry from [sshd_config][sshd_man].

> Specifies the maximum number of concurrent unauthenticated
> connections to the SSH daemon.  Additional connections will be
> dropped until authentication succeeds or the LoginGraceTime
> expires for a connection.  The default is 10.

Yes - sshd will throttle your concurrent connections while they authenticate. Increasing `MaxStartups` caused our npm
installation woes to disappear from our CI environment. Huzzah!

[sshd_man]: http://manpages.ubuntu.com/manpages/lucid/en/man5/sshd_config.5.html

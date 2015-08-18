---
layout: post
title:  "GitLab, NPM, and SSHd MaxStartups"
date:   2015-08-17 21:45:00
categories: gitlab npm ssh
---

Intermittent issues are every developer's best friend. Recently we started hitting an error during the `npm install` 
phase of our CI and CD jenkins jobs. The full error can be found in the attached [gist], however here's the important
bits:

{% gist lukewaite/35ba917ed20466cb79f5 short.txt %}

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
[gist]: https://gist.github.com/lukewaite/35ba917ed20466cb79f5

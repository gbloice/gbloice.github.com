---
layout: post
title: "Improving HouseMon installation"
date: 2013-05-07 23:51
comments: true
categories: housemon
published: true
---

The sample instructions on setting up HouseMon are OK for initial testing, but require a bit more work for a permanent installation.  This post describes how I managed it.

<!--more-->

The redis server should be moved to run as a service, I used the [quickstart][1] guide from the redis documentation as described in the section "Installing Redis more properly", modified a little to suit my preferences.  Instead of using the port number to describe the redis instance I used "housemon", the only issue this caused was with the init script that uses the value both as a port number and the instance identifier, so I modified the start of that script a little:

``` diff redis_init_script
--- ../redis-2.6.12/utils/redis_init_script     2013-03-29 16:42:39.000000000 +0000
+++ /etc/init.d/redis_housemon  2013-05-07 23:44:01.042072032 +0100
@@ -4,11 +4,12 @@
 # as it does use of the /proc filesystem.

 REDISPORT=6379
+REDISINST=housemon
 EXEC=/usr/local/bin/redis-server
 CLIEXEC=/usr/local/bin/redis-cli

-PIDFILE=/var/run/redis_${REDISPORT}.pid
-CONF="/etc/redis/${REDISPORT}.conf"
+PIDFILE=/var/run/redis_${REDISINST}.pid
+CONF="/etc/redis/${REDISINST}.conf"

 case "$1" in
     start)
```

I also used [forever][2] to run the housemon server, install it with `sudo npm install -g forever` and then start Housemon with `forever start app.js`.

 [1]: http://redis.io/topics/quickstart
 [2]: https://npmjs.org/package/forever
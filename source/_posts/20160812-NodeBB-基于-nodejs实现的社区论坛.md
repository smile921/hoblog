---
title: NodeBB 基于 nodejs实现的社区论坛
date: 2016-08-12 12:39:00
tags:
  - forum
  - nodejs
---

## Node.js based forum software built for the modern web http://www.nodebb.org

[**NodeBB Forum Software**](https://nodebb.org) is powered by Node.js and built on either a Redis or MongoDB database. It utilizes web sockets for instant interactions and real-time notifications. NodeBB has many modern features out of the box such as social network integration and streaming discussions, while still making sure to be compatible with older browsers.

Additional functionality is enabled through the use of third-party plugins.

* [Get NodeBB](http://www.nodebb.org/ "NodeBB")
* [Demo & Meta Discussion](http://community.nodebb.org)
* [Documentation & Installation Instructions](http://docs.nodebb.org)
* [Help translate NodeBB](https://www.transifex.com/projects/p/nodebb/)
* [NodeBB Blog](http://blog.nodebb.org)
* [Join us on IRC](https://kiwiirc.com/client/irc.freenode.net/nodebb) - #nodebb on Freenode
* [Follow us on Twitter](http://www.twitter.com/NodeBB/ "NodeBB Twitter")
* [Like us on Facebook](http://www.facebook.com/NodeBB/ "NodeBB Facebook")

## NodeBB Dockerfile

```
# The base image is the latest 4.x node (LTS) on jessie (debian)
# -onbuild will install the node dependencies found in the project package.json
# and copy its content in /usr/src/app, its WORKDIR
FROM node:4-onbuild

ENV NODE_ENV=production \
    daemon=false \
    silent=false

# nodebb setup will ask you for connection information to a redis (default), mongodb then run the forum
# nodebb upgrade is not included and might be desired
CMD node app --setup && npm start

# the default port for NodeBB is exposed outside the container
EXPOSE 4567
```

# Example Forum user nodebb
[区块链开发者社区](https://uchain.org/)
![nodebb](images/nodebb-chain.png)

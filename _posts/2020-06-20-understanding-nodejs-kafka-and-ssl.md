---
layout: post
title: "Understanding NodeJS, Kafka, and SSL"
description: "Installing Kafka library for NodeJS might not be as easy as it seems."
date: 2020-06-20
category: Tech
tags: [nodejs, kafka, ssl]
---

A few months back I was tasked in upgrading NodeJS version for a project. This project consumes Kafka messages and connecting to Kafka using SSL.

Seems very simple, right? Well..

<!--more-->

## Why Update?

To set up context, our application was running using NodeJS v8 LTS which ends its support on Dec 2019.

In order to keep up with security patches and bug fixes, it's important to always use the LTS version of NodeJS. When a vulnerability is discovered in NodeJS which is no longer supported, the cost of fixing the application will be more expensive rather than pre-emptively upgrading the version.

And so, our team decided to go with upgrading NodeJS v8 to NodeJS v12 LTS which will be supported until Apr 2022. This is good enough for our team because it is the stable version at the time of this writing.

## The Painful Lesson

Our application is dockerized with RHEL 7 as the underlying OS, so it's a pretty simple process to upgrade.

As part of the business requirement, our application needs to read the messages from Kafka. For this purpose, it's using [node-rdkafka](https://github.com/Blizzard/node-rdkafka) for connecting to Kafka. The connection is secured through SSL.

When the application tried to run Kafka consumer for the first time, we were getting this error:

```bash
Segmentation fault (core dumped)
```

I have to admit that debugging this issue takes the better part of me, simply because I don't understand enough on how NodeJS is shipped with its own OpenSSL version.

## Understanding NodeJS, Librdkafka and C++

What is segmentation fault? Segmentation fault is an error thrown by program written in C / C++ in order to avoid memory corruption.

In our application's case, this happened because of the incompatibility between OpenSSL C++ API that is shipped by NodeJS v12 and the OS.

When we are running `npm install`, `node-rdkafka` will build `librdkafka`, which is a C++ library, using OpenSSL libraries that is found in the OS. Now RHEL 7 is generally shipped with OpenSSL 1.0.2 by default.

Meanwhile, NodeJS v12 is shipped using OpenSSL 1.1.1. This normally can be inspected using `process.versions`

```bash
$ node -p process.versions
{
  node: '12.18.1',
  v8: '7.8.279.23-node.38',
  uv: '1.38.0',
  zlib: '1.2.11',
  brotli: '1.0.7',
  ares: '1.16.0',
  modules: '72',
  nghttp2: '1.41.0',
  napi: '6',
  llhttp: '2.0.4',
  http_parser: '2.9.3',
  openssl: '1.1.1g',
  cldr: '37.0',
  icu: '67.1',
  tz: '2019c',
  unicode: '13.0'
}
```

## Solution

Now that we know the problem is between the two C++ ibraries, we need to ensure that `librdkafka` is also compiled with the same OpenSSL version as node v12. Upgrading OpenSSL version in our docker image works and now the application is able to connect to Kafka using SSL.

## References
* [https://nodejs.org/en/download/releases/](https://nodejs.org/en/download/releases/)
* [https://nodejs.org/api/process.html#process_process_versions](https://nodejs.org/api/process.html#process_process_versions)
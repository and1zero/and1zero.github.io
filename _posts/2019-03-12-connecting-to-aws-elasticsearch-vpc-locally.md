---
layout: post
title: "Connecting to AWS Elasticsearch VPC locally"
description: "Using Ruby Elasticsearch gem for the bulk actions might not be working when we are using the default configuration. This article will try to shed a light on that."
date: 2019-03-12
category: Tech
tags: [aws, elasticsearch, vpc]
---

Generally it is better to have an Elasticsearch cluster in AWS secured using VPC connection instead of open internet, instead of relying on bucket policy when accessing them.

However, connecting them to our localhost is a bit complicated because we need to port-forward the connection. As a pre-requisite, we would need to connect through an EC2 instance in the same VPC with our Elasticsearch cluster.

<!-- more -->

This is what we would need to execute every time we want to port-forward:

```bash
ssh -i ~/.ssh/id_rsa -N 9200:https://your-aws-vpc.us-east-1.es.amazonaws.com:443 ec2-user:your-ec2-instance.us-east-1.amazonaws.com
```

## Solution

I stumbled upon [this post](https://www.jeremydaly.com/access-aws-vpc-based-elasticsearch-cluster-locally/) while trying to search for a more painless way to connect to the Elasticsearch cluster. In a nutshell, it's just as simple as:

1. Add a configuration in your SSH config file (`~/.ssh/config`):
```bash
# ~/.ssh/config
# Elasticsearch Tunnel
Host estunnel
HostName your-ec2-instance.us-east-1.amazonaws.com # your EC2 server's public IP address or host
User ec2-user
IdentitiesOnly yes
IdentityFile ~/.ssh/MY-KEY.pem # private key you're using to connect to the EC2 instance
LocalForward 9200 your-aws-vpc.us-east-1.es.amazonaws.com:443
```

2. Once we have the configuration, port-forwarding is now as simple as:
```bash
$ ssh -N estunnel
```

3. And now we can access the Elasticsearch cluster from **https://localhost:9200** in the browser, ignoring the SSL certificate warning.
We can also access Kibana from the URL, just simply go to **https://localhost:9200/_plugin/kibana**.

Sometimes when we are making any *POST* or *PUT* requests to **https://localhost:9200**, it will be rejected because it's violating the cross-origin.
In order to prevent this, we can mock the Elasticsearch domain in `/etc/hosts` and add the following entry:
```bash
# /etc/hosts
127.0.0.1 your-aws-vpc.us-east-1.es.amazonaws.com
```
And then you can retry the requests to **https://your-aws-vpc.us-east-1.es.amazonaws.com:9200** instead of localhost.

## Reference

[https://www.jeremydaly.com/access-aws-vpc-based-elasticsearch-cluster-locally/](https://www.jeremydaly.com/access-aws-vpc-based-elasticsearch-cluster-locally/)
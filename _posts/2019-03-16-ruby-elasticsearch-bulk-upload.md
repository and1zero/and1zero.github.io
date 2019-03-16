---
layout: post
title: "Bulk Upload using Ruby Elasticsearch Gem"
description: "Using Ruby Elasticsearch gem for the bulk actions might not be working when we are using the default configuration. This article will try to shed a light on that."
date: 2019-03-16
category: Tech
tags: [elasticsearch, jenkins, ruby]
---

## Background

I was tasked with doing some data transforming job which needs to be stored in an Elasticsearch cluster. The gist of the job is very simple:

1. Do a massive query which will return a CSV file roughly 3GB.
2. Import the CSV file in Elasticsearch.
3. Users should be able to query the data in Elasticsearch.

Seems very simple, right? So I am using the [elasticsearch-ruby](https://github.com/elastic/elasticsearch-ruby) gem in order to make life easier for the rest of us.

## Why Using Ruby?

Aside from just uploading the CSV file, we also need to run all kind of odd-jobs in the task, such as:

* Making query to AWS Athena in order to get the humongous file.
* Making sure we are tagging with the right index in Elasticsearch.
* Making sure we deleted any expired indexes in Elasticsearch.

All these are running in `jenkins` using `docker`. The `docker` part will be important in the upcoming section.

## The Problem

Aside from the runs locally, we never observed any successful run when we are using Elasticsearch's [bulk](https://github.com/elastic/elasticsearch-ruby/blob/master/elasticsearch-api/lib/elasticsearch/api/actions/bulk.rb) action inside Jenkins. What's even more frustrating, the error messages are different for every runs, such as:

```ruby
...
Elasticsearch::Transport::Transport::Errors::InternalServerError: [500] {"error":{"root_cause":[{"type":"json_e_o_f_exception","reason":"Unexpected end-of-input in field name\n at [Source: org.elasticsearch.common.bytes.BytesReference$MarkSupportingStreamInputWrapper@2a743cdb; line: 1, column: 27]"}],"type":"json_e_o_f_exception","reason":"Unexpected end-of-input in field name\n at [Source: org.elasticsearch.common.bytes.BytesReference$MarkSupportingStreamInputWrapper@2a743cdb; line: 1, column: 27]"},"status":500}
...
```

Or:

```ruby
...
Elasticsearch::Transport::Transport::Errors::BadRequest: [400] {"error":{"root_cause":[{"type":"illegal_argument_exception","reason":"Malformed action/metadata line [367], expected START_OBJECT or END_OBJECT but found [VALUE_STRING]"}],"type":"illegal_argument_exception","reason":"Malformed action/metadata line [367], expected START_OBJECT or END_OBJECT but found [VALUE_STRING]"},"status":400}
...
```

Without anything in common. It was a very dark period in our office.

## The Beacon of Light

Until I ask one of my colleague on the fated day, he noticed that these build in `jenkins` will fail roughly at the same time. We deduced that it could be to `bulk` operation timeout which will cut the parameters that the gem send to the endpoint.

Upon reading the gem's README more closely, I found this text under **Usage**:

> Keep in mind, that for optimal performance, you should use a HTTP library which supports persistent ("keep-alive") connections, e.g. Patron or Typhoeus. These libraries are not dependencies of the elasticsearch gems, so be sure to define a dependency in your own application.

Oh boy, so that explains why the rake task ran successfully when it was executed out of docker containers.

## The Solution

It's simple. We just need to change the default HTTP library to something else.

```ruby
require 'typhoeus'
require 'typhoeus/adapters/faraday'

client = Elasticsearch::Client.new(
  url: ENV.fetch('ES_URL'),
  retry_on_failure: 3,
  request_timeout: 5*60,
  transport_options: {
    request: {
      open_timeout: 5*60,
      timeout: 5*60,
    },
) do |f|
  f.adapter :typhoeus
end
```

That's it!

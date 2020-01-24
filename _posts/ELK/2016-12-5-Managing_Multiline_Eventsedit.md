---
layout: post
title:  "logstash Manager_multiline_event"
categories: "ELK"
tags: "logstash multi_event"
author: "songzhx"
date:   2016-12-5
---

# logstash Manager_multiline_event

## Timestamps

Activity logs from services such as Elasticsearch typically begin with a timestamp, followed by information on the specific activity, as in this example:

```
[2015-08-24 11:49:14,389][INFO ][env                      ] [Letha] using [1] data paths, mounts [[/
(/dev/disk1)]], net usable_space [34.5gb], net total_space [118.9gb], types [hfs]
```

To consolidate these lines into a single event in Logstash, use the following configuration for the multiline codec:

```
input {
  file {
    path => "/var/log/someapp.log"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601} "
      negate => true
      what => previous
    }
  }
}
```

This configuration uses the `negate` option to specify that any line that does not begin with a timestamp belongs to the previous line.

## 参考文献

- https://www.elastic.co/guide/en/logstash/5.x/multiline.html
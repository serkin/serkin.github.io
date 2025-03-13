---
layout: post
title: Single Logger writes to stdout and Graylog
date: 2024-12-22 12:47:00
description: Figuring out the most convenient way to see logs for my team
tags: logging, python
featured: false
---

First challenge in my current position was to ease the debugging process. We have k8s+argocd set up and all our microservices spin there.

So when something went wrong the only way was to download logs from argocd and grep them. While we didn't have lots of clients it was acceptable but slow. Another problem is that after deployment all logs disappear from argocd.

So I talked to operations to set up the Graylog cluster. I previously had a positive experience with Graylog. Even with all the java-based-memory-greedy arguments from ops I managed to get that cluster.

Next step was to make reading logs as convenient as possible. I didn't like the idea to send log records separately to Graylog. I was looking for some simple implementation where I can somehow aggregate logs so one http request to microservices would be only a log record.

It is too early to pull Jaeger with Open Telemetry just for logs. Tracing is a future problem for us.

Solution was to write a simple logger that supports standard python logging interface and at the same time aggregates log before sending it to Graylog. For argocd I wanted to keep logging line by line.

What I came up with as follows


```python
import logging
import sys
import graypy
from src.config import CONFIG
from colorlog import ColoredFormatter

class MemoryHandler(logging.Handler):
    def __init__(self):
        super().__init__()
        self.log_messages = []

    def emit(self, record):
        log_entry = self.format(record)
        self.log_messages.append(log_entry)

    def dump(self) -> str:
        log_messages = "\n".join(self.log_messages)
        self.log_messages.clear()
        return log_messages


def custom_formatter(level):
    if level >= logging.ERROR:
        fmt = '%(levelname)s: %(message)s'
    else:
        fmt = '%(message)s'
    return logging.Formatter(fmt)

  
class Logger(logging.Logger):
    def __init__(self, name):
        super().__init__(name)

        if name == "graylog":
            graylog_handler = graypy.GELFUDPHandler(CONFIG.GRAYLOG_HOST, CONFIG.GRAYLOG_PORT)
            graylog_handler.setLevel(logging.DEBUG)
            self.addHandler(graylog_handler)
        else:
            stream_handler = logging.StreamHandler(sys.stdout)
            stream_handler.setLevel(logging.DEBUG)

            colored_formatter = ColoredFormatter('%(asctime)s %(log_color)s%(levelname)s: %(message)s')
            stream_handler.setFormatter(colored_formatter)
            self.addHandler(stream_handler)

            self.memory_handler = MemoryHandler()
            self.memory_handler.setLevel(logging.DEBUG)
            self.addHandler(self.memory_handler)

    def handle(self, record):
        custom_fmt = custom_formatter(record.levelno)
        if hasattr(self, 'memory_handler'):
            self.memory_handler.setFormatter(custom_fmt)
        super().handle(record)

logger = Logger("stdout+memory")
graylog = Logger("graylog")
```

This solution works well so far but I kinda have a feeling that sometimes logs mixed up like we are having thread issues. Looking into it

You can download the whole solution [here](/assets/labs_files/lab_148.zip)



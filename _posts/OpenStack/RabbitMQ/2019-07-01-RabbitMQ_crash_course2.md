---
layout: post
title: RabbitMQ crash course (part 2) - Round-robin dispatching
tags:
    - RabbitMQ
author: Radosław Śmigielski
---

Assume we have one queue and multiple clients connected.

![producer-consumers](https://www.rabbitmq.com/img/tutorials/python-two.png){: width=150 height=100 style="margin-left:200px"}

This configuration is also called **task queue** or **round-robin dispatching**.

In order to send multiple messages to the same queue, I modified scrip used
in part one.
```python
#!/usr/bin/env python3
""" If script executed with arguments, all of them will be send over AMQP
to reciver:

    $ python producer.py -n <number of messages>

"""
import argparse
import pika
import os

QNAME='queue0'
RABBIT_PORT = 5672
RABBIT_IP = os.environ.get("RABBIT_IP", None)

def message(seq_number=0):
    _msg = "AMQP MESSAGE {:d}".format(seq_number)
    return _msg

def main():
    if RABBIT_IP is None:
        print("Missing RABBIT_IP env variable")

    parser = argparse.ArgumentParser(description='Send some AMQP messages')
    parser.add_argument('-n', type=int,
                        help="Number of messages to send")
    args = parser.parse_args()

    msg = message()
    print('AMQP producer connecting to {}:{} with message: \"{}\"'.\
          format(RABBIT_IP, RABBIT_PORT, msg))
    connection = pika.BlockingConnection(pika.ConnectionParameters(RABBIT_IP, RABBIT_PORT))
    channel = connection.channel()
    channel.queue_declare(queue=QNAME)

    for x in range(0,args.n):
        channel.basic_publish(exchange='',
                              routing_key=QNAME,
                              body=message(x))
    connection.close()

if __name__ == '__main__':
    main()
```

Messages from queue will be distubuted to connected clients on round-robin
schedule. Below screenshot presents three consummers connected to the same
queue, all reaciving messages.
![producer--many-consumers](https://raw.githubusercontent.com/radeksm/r-blobs/master/radeksm.github.io/RabbitMQ/amqp_receivers.png)

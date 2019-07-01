---
layout: post
title: RabbitMQ crash course (part 1)
tags:
    - RabbitMQ
author: Radosław Śmigielski
---

Let's start from starting single RabbitMQ instance im Docker container.

### Steps ###
1. Download latest docker image
   ```
   docker pull rabbitmq
   ```
2. Run RabbitMQ as a container
   ```
   docker run --rm -d --hostname rabbit0.amql.local --name rabbit0 --label rabbit0 rabbitmq:latest
   ```
3. At this point you should have running RabbitMQ container which you should
   be able connect over AMQP.
   ```
   radek@ant:~$ docker ps
   CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                NAMES
   c23bda22c2fe        rabbitmq:latest     "docker-entrypoint.s…"   5 minutes ago       Up 5 minutes        4369/tcp, 5671-5672/tcp, 25672/tcp   rabbit0
   ```
4. Check status of your RabbitMQ
   ```
   docker exec -it rabbit0 rabbitmqctl status
   docker exec -it rabbit0 rabbitmqctl -p / list_queues
   docker exec -it rabbit0 rabbitmqctl list_queues
   docker exec -it rabbit0 rabbitmqctl list_vhosts
   docker exec -it rabbit0 rabbitmqctl list_connections
   ```

Congrats you now have now fullty running RabbitMQ to which you can
connect to.

Below Python code needs to be run on the same host like
your RabbitMQ container in order to work, it implements simple
producer-consumer model.

![producer-consumer](https://www.rabbitmq.com/img/tutorials/python-one-overall.png){: width=150 height=100 style="margin-left:70px"}

1. Install Python Pika which is AMQP client library. Running below example
   inside Python virtual environment is recomended.
   ```
   python3 -m venv /var/tmp/venv3-amqp
   source /var/tmp/venv3-amqp/bin/activate
   pip install -U pip
   pip install pika
   ```
2. Start from extracting IP address which is assigned to your RabbitMQ
   container and exporting it as env variable
   ```bash
   {% raw %}export RABBIT_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' rabbit0)
   echo $RABBIT_IP{% endraw %}
   ```
3. Push some messages to the queue on yoyr AMQP broker
   ```python
   #!/usr/bin/env python3
   """ If script executed with arguments, all of them will be send
   over AMQP to reciver:

    $ python producer.py arg0 arg1 arg2 ...

   """

   import pika
   import os
   import sys

   QNAME='queue0'
   RABBIT_PORT = 5672
   RABBIT_IP = os.environ.get("RABBIT_IP", None)

   def message():
       _msg = "AMQP MESSAGE"
       if len(sys.argv) > 1:
           _msg = ' '.join(sys.argv[1:])
       return _msg

   def main():
       if RABBIT_IP is None:
           print("Missing RABBIT_IP env variable")

       msg = message()
       print('AMQP producer connecting to {}:{} with message {}'.\
             format(RABBIT_IP, RABBIT_PORT, msg))
       connection = pika.BlockingConnection(pika.ConnectionParameters(RABBIT_IP, RABBIT_PORT))
       channel = connection.channel()
       channel.queue_declare(queue=QNAME)
       channel.basic_publish(exchange='',
                             routing_key=QNAME,
                             body=msg)
       connection.close()

   if __name__ == '__main__':
       main()
   ```
   Each time you run this script it will push one message to **queue0**
   on your RabbitMQ. You can check your queued messages on rabbit:
   ```bash
   radek@ant:~$ docker exec -it rabbit0  rabbitmqctl list_queues
   Timeout: 60.0 seconds ...
   Listing queues for vhost / ...
   name    messages
   queue0  5
   ```
   As you can see there are 5 messages in queue **queue0**.

4. Let's now consume all the messages from queue.
   ```python
   #!/usr/bin/env python3

   import pika
   import os

   RABBIT_IP = os.environ.get("RABBIT_IP", None)
   RABBIT_PORT = 5672
   QNAME='queue0'

   def callback(ch, method, properties, body):
       print(" [x] Received %r" % body)

   def main():
       if RABBIT_IP is None:
           print("Missing RABBIT_IP env variable")

       print('AMQP consumer connecting to {}:{}'.format(RABBIT_IP, RABBIT_PORT))
       connection = pika.BlockingConnection(pika.ConnectionParameters(RABBIT_IP, RABBIT_PORT))
       channel = connection.channel()
       channel.basic_consume(queue=QNAME,
           auto_ack=True,
           on_message_callback=callback)
       channel.start_consuming()
       connection.close()

   if __name__ == '__main__':
       main()
   ```

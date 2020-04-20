---
layout: post
title: Run RabbitMQ on Kubernetes
tags:
    - RabbitMQ
    - Kubernetes
author: Radosław Śmigielski
---

Why?
====
Simple example how to run RabbitMQ on Kubernetes.

How?
====

The simplest possible configuraton
----------------------------------
```bash
kubectl apply -f https://gist.github.com/radeksm/95398eaef3e569604a8a4c52c992c728
```

```yaml
---
# kubectl apply -f rabbit.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: rabbitmq-deployment
  labels:
    app: rabbit
spec:
  replicas: 1

  selector:
    matchLabels:
      app: rabbit

  template:
    metadata:
      labels:
        app: rabbit
    spec:
      containers:
      - name: rabbit
        image: rabbitmq:management
        ports:
        # RabbitMQ ports: 4369 5671 5672 25672
        # UI mgmt ports:  15671 15672
        - containerPort: 4369
        - containerPort: 5671   # AMQP
        - containerPort: 5672   # AMQP
        - containerPort: 25672
        - containerPort: 15671  # Management UI Access
        - containerPort: 15672  # Management UI Access
        env:
          - name: RABBITMQ_DEFAULT_USER
            value: admin
          - name: RABBITMQ_DEFAULT_PASS
            value: password1

---
apiVersion: v1
kind: Service
metadata:
  name: rabbit-service
spec:
  type: NodePort
  selector:
    app: rabbit
  ports:
    - name: rabbit-amqp
      protocol: TCP
      port: 5672
      targetPort: 5672
      nodePort: 30672
    - name: rabbit-ui
      protocol: TCP
      port: 15672
      targetPort: 15672
      nodePort: 31672
```

Kuberenets resource file on [Gist](https://gist.github.com/radeksm/95398eaef3e569604a8a4c52c992c728)

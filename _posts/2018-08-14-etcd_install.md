---
layout: post
title: Etcd installation
tags:
    - etcd
author: Radosław Śmigielski
---

Why?
====
1. minimal or no requirements to start
1. does not require root privileges
1. run etcd instances on my laptop
2. does not require docker or virtual machnne(s)

Download
========
Go to etcd releases page [etcd/releases](https://github.com/coreos/etcd/releases)
and decide wich version you want. Bare in mind that the order of versions on
that page is kind of random. You can search for **"Latest release"**
tag to find the latest and greatest version.

Modify below two variables to whatever you want them to be.
Both variables will be used in following steps (!).
{% highlight bash %}
export ETCD_VER=v3.2.24
export ETCD_INSTALL_DIR=${HOME}/etcd
{% endhighlight %}

Download desirable version
{% highlight bash %}
GOOGLE_URL=https://storage.googleapis.com/etcd
DOWNLOAD_URL=${GOOGLE_URL}

rm -f  ${ETCD_INSTALL_DIR}/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf ${ETCD_INSTALL_DIR} && mkdir -p ${ETCD_INSTALL_DIR}

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz \
     -o ${ETCD_INSTALL_DIR}/etcd-${ETCD_VER}-linux-amd64.tar.gz
{% endhighlight %}


Installation
============
*Installation* may not be the best word in this place. I basically extract 
pre-compiled etcd version to **$ETCD_INSTALL_DIR** directory.

{% highlight bash %}
tar xzvf ${ETCD_INSTALL_DIR}/etcd-${ETCD_VER}-linux-amd64.tar.gz \
    -C ${ETCD_INSTALL_DIR} --strip-components=1
rm -f ${ETCD_INSTALL_DIR}/etcd-${ETCD_VER}-linux-amd64.tar.gz
{% endhighlight %}


Start etcd
==========
This is standalone instance, no clustering.
{% highlight bash %}
${ETCD_INSTALL_DIR}/etcd \
    --name etcd0 \
    --listen-client-urls http://localhost:2379 \
    --advertise-client-urls  http://localhost:2379
{% endhighlight %}

Test your etcd
==============
This is standalone instance, no clustering.
{% highlight bash %}
alias etcdctl="ETCDCTL_API=3 ${ETCD_INSTALL_DIR}/etcdctl --endpoints http://localhost:2379"

etcdctl version
etcdctl cluster-health
etcdctl member list
etcdctl put foo bar
etcdctl get foo
etcdctl check perf
{% endhighlight %}


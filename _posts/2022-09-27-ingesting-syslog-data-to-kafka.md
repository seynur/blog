---
layout: default
title: "Ingesting Syslog data to Kafka"
summary: "When working with event data analytics, especially for security purposes (i.e. SIEM), syslog becomes an important protocol to ingest data. Most of our clients utilize syslog ..."
author: Selim Seynur
image: /assets/img/blog/2022-09-27-kafka-syslog-1.webp
date: 08-01-2021
tags: kafka s3
categories: Kafka
---

# Ingesting Syslog data to Kafka

When working with event data analytics, especially for security purposes (i.e. SIEM), syslog becomes an important protocol to ingest data. Most of our clients utilize syslog to send data from security and networking devices (e.g. firewalls, IDS/IPS, etc.) and as the data pipeline grows utilizing Kafka has several [benefits](https://www.confluent.io/use-case/siem/). How do we send syslog data from networking and security devices to Kafka topics?

![screenshot](/assets/img/blog/2022-09-27-kafka-syslog-1.webp)

## TL;DR
This post covers 2 possible options:

1. [Confluent Syslog Source Connector](https://www.confluent.io/hub/confluentinc/kafka-connect-syslog)
- Requires Confluent license.
- Guarantees at least once delivery.
- It is recommended to run only one task and be deployed in a separate Connect cluster node.
- Supports [rfc 3164](https://tools.ietf.org/html/rfc3164), [rfc 5424](https://tools.ietf.org/html/rfc5424), and Common Event Format (CEF) and produces structured messages (original syslog message is transformed).
2. [syslog-ng](https://www.syslog-ng.com/products/open-source-log-management/)
- Highly portable, well-known application for processing syslog data.
- Kafka module ( syslog-ng-rdkafka) is needed to send data to a Kafka cluster, where kafka-c implementation is utilized.
- Provides capability to apply parsing and filtering logic before sending data to Kafka, hence enabling conditional forwarding.

---

## Solution:
1. Utilize [Confluent Syslog Source Connector](https://www.confluent.io/hub/confluentinc/kafka-connect-syslog) to send syslog data directly to the Confluent Kafka platform.
2. Utilize an intermediate syslog server, such as [syslog-ng](https://www.syslog-ng.com/products/open-source-log-management/), to collect and ingest data to relevant Kafka topics.

## Prerequisites:
Since this post is about Kafka, a prerequisite is to have a working Kafka (i.e. Confluent Platform). You can follow the steps in the [Quick Start](https://docs.confluent.io/platform/current/platform-quickstart.html#step-1-download-and-start-cp) documentation. For my setup, I downloaded the tarball and extracted it locally on my macOS.

## Confluent Syslog Connector:
It is pretty straightforward to install and configure the connector and you can follow the steps in the [Quick Start](https://docs.confluent.io/kafka-connectors/syslog/current/overview.html#quick-start) documentation. I’ve tested this locally and since it’s the same output as described in the documentation I will omit those details here to avoid redundancy but provide some comments below.

This connector requires a valid Confluent license (subscription license — but comes with a 30-day trial) and includes at least once delivery feature, which guarantees that messages will not be lost as long as they reach the connector’s listening syslog port.

From the documentation, note the following statement.

> Confluent recommends you run only one task and deploy the connector into a Connect cluster with a single, fixed node and hostname.

Since we’re limited to one task, there may be concerns of having the connector as a bottleneck (e.g. too many messages overloading) or an operational overhead (e.g. too many connector instances for distributing the load).

The connector accepts messages as strings and supports [rfc 3164](https://tools.ietf.org/html/rfc3164), [rfc 5424](https://tools.ietf.org/html/rfc5424), and Common Event Format (CEF). A nice feature (or a disadvantage, depending on perspective) of the connector is producing structured messages (e.g. JSON) from the incoming syslog messages as outputs to the configured Kafka topic. The connector appears to be running SMT ([Single Message Transforms](https://docs.confluent.io/cloud/current/connectors/single-message-transforms.html)) on the incoming data. If you want to utilize Kafka just as a queue, without changing anything in the original syslog message, then the next alternative (syslog server) may be better.

## Syslog server (syslog-ng):
Syslog-ng is a highly portable application for processing syslog data and we’ve been utilizing its features on many of the projects (e.g. by itself or as part of [Splunk Connect For Syslog](https://splunk.github.io/splunk-connect-for-syslog/main/) distribution). In my test environment, I installed [balabit/syslog-ng](https://hub.docker.com/r/balabit/syslog-ng/) container from Docker Hub.

The test is to send a sample syslog message to the syslog-ng container which in turn ingests the data to the specified Kafka topic.

_NOTE_: If you want to install syslog-ng from scratch or want to use a different container, make sure `syslog-ng-rdkafka` is installed also.

![screenshot](/assets/img/blog/2022-09-27-kafka-syslog-2.webp)

In a nutshell, here are the steps:
1. Install Confluent Kafka and configure it so that it accepts connections from the container app.
2. Install syslog-ng container and configure it to send incoming data to the Kafka cluster.
3. Send sample syslog data (sample Palo Alto firewall traffic log) to the container app (syslog-ng).

[Robin Moffatt](https://www.confluent.io/blog/author/robin-moffatt/) has a great [blog](https://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/) about the connectivity to/from a Kafka cluster. I updated the `server.properties` file according to this blog to make things work.

`$CONFLUENT_HOME/etc/kafka/server.properties` file entries:

```properties
listeners=PLAINTEXT://0.0.0.0:9092,DOCKER_TO_HOST://0.0.0.0:19092
advertised.listeners=PLAINTEXT://localhost:9092,DOCKER_TO_HOST://host.docker.internal:19092
listener.security.protocol.map=PLAINTEXT:PLAINTEXT,DOCKER_TO_HOST:PLAINTEXT
```

Note that I have created a `DOCKER_TO_HOST` listener so that the container can connect to this cluster. Please refer to the blog mentioned above for details.

Once the configuration is in place, you can simply start your Kafka cluster. Since I had already created the test topic, `panos`, I only started Zookeeper and Kafka locally. Please refer to Confluent quick start on how to create a topic.

```sh
$> confluent local services kafka start
```

Next, I updated `syslog-ng.conf` file so that it knows how to connect to the Kafka cluster.

```
@version: 3.38
@include "scl.conf"
@define kafka-implementation kafka-c
source s_local {
  internal();
};
source s_network {
  udp(port(514));
  tcp(port(601));
};
destination d_local {
  file("/var/log/messages");
  file("/var/log/messages-kv.log" template("$ISODATE $HOST $(format-welf --scope all-nv-pairs)\n") frac-digits(3));
};
destination d_kafka {
  kafka(
    bootstrap-servers("host.docker.internal:19092")
    topic("panos")
  );
};
log {
  source(s_local);
  source(s_network);
  destination(d_local);
  destination(d_kafka);
};
```

It’s now time to run the container with the following command and the custom configuration file. I also used `-edv` at the end to make sure the debug messages are displayed on the console.

```bash
$> sudo docker run -it -v "$PWD/syslog-ng.conf":/etc/syslog-ng/syslog-ng.conf -p 514:514/udp -p 601:601 --name balabit-syslog-ng balabit/syslog-ng:latest --no-caps -edv
```

The sample Palo Alto log is as follows:

```bash
$> cat panfw1.log
<13>1 Sep 20 21:47:11 BD-Panorama 1,2022/09/20 21:47:11,007200001165,TRAFFIC,end,1,2022/09/20 21:42:30,192.168.55.101,2.2.2.2,0.0.0.0,0.0.0.0,Permit All,tng\picard,,dns,vsys1,trust,untrust,tunnel.2,ethernet1/1,Syslog To Panorama,2022/09/20 21:42:30,57753,1,53909,53,0,0,0x64,udp,allow,74,74,0,1,2022/09/20 21:42:00,0,any,0,13457860836,0x8000000000000000,192.168.0.0–192.168.255.255,France,0,1,0,aged-out,15,0,0,0,,PA-VM,from-policy
```

At this point, both the syslog-ng and Kafka cluster are up and running and it’s time to ingest some syslog data.

```bash
$> cat panfw1.log| nc -u -w 0 localhost 514
```

You can verify that we have the sample data ingested in panos topic with the following console command.

```bash
$> kafka-console-consumer — bootstrap-server localhost:9092 — topic panos — from-beginning
2022–09–22T12:21:01+00:00 172.17.0.1 1 Sep 20 21:47:11 BD-Panorama 1,2022/09/20 21:47:11,007200001165,TRAFFIC,end,1,2022/09/20 21:42:30,192.168.55.101,2.2.2.2,0.0.0.0,0.0.0.0,Permit All,tng\picard,,dns,vsys1,trust,untrust,tunnel.2,ethernet1/1,Syslog To Panorama,2022/09/20 21:42:30,57753,1,53909,53,0,0,0x64,udp,allow,74,74,0,1,2022/09/20 21:42:00,0,any,0,13457860836,0x8000000000000000,192.168.0.0–192.168.255.255,France,0,1,0,aged-out,15,0,0,0,,PA-VM,from-policy
```

## Conclusion and other considerations
I covered 2 different methods in this post to ingest syslog data to the Kafka cluster.

1. Confluent Syslog Source Connector — requires Confluent license, supports [rfc 3164](https://tools.ietf.org/html/rfc3164), [rfc 5424](https://tools.ietf.org/html/rfc5424), and Common Event Format (CEF), and produces structured messages to the related topic(s). Task limitation on the connector raises the question of scalability and perhaps the issue of sending data from various syslog sources that need to be separated into different topics over the same port (e.g. firewall, routers, IDS/IPS, all sending over udp/514).
2. syslog-ng — is a well know and widely used product also available as an open-source version. It is possible to scale syslog-ng as needed and configuration allows detailed parsing and filtering capabilities such that different types of messages can be filtered to be sent to relevant Kafka topics. For example, just for Palo Alto syslog data, we can filter SYSTEM and TRAFFIC logs and send them to different Kafka topics.

---

### References
- [Confluent Platform Quick Start](htts://docs.confluent.io/platform/current/platform-quickstart.html#step-1-download-and-start-cp)
- [Syslog Source Connector](htts://www.confluent.io/hub/confluentinc/kafka-connect-syslog)
- [Syslog-ng](htts://www.syslog-ng.com/products/open-source-log-management/)
- [Syslog-ng Docker Container (Balabit)](htts://hub.docker.com/r/balabit/syslog-ng/)
- [My Python/Java/Spring/Go/Whatever Client Won’t Connect to My Apache Kafka Cluster in Docker/AWS/My Brother’s Laptop. Please Help!](htts://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/)
- [Confluent Platform Licenses](htts://docs.confluent.io/platform/7.2.1/installation/license.html)

---

---
layout: default
title: "Streaming syslog-ng Events to Kafka: A Practical Guide You Can Follow"
summary: "This guide walks you through sending logs from syslog-ng to Apache Kafka — step by step, from building components to testing with a sample log. A lightweight, Docker-free setup using Kafka in KRaft mode, perfect for real-time log streaming."
author: Öykü Can Şimşir
image: /assets/img/blog/2025-09-15-streaming-syslog-ng-events-to-kafka.webp
date: 15-09-2025
tags: syslog-ng kafka librdkafka eventlog kraft      
categories: Kafka

---

# Streaming Syslog-ng Events to Kafka: A Practical Guide You Can Follow

In today's world of observability and real-time monitoring, efficiently collecting and transferring logs is more important than ever. One of the most popular methods for achieving this is by using `syslog-ng`, a reliable and flexible log forwarder, to gather logs and send them to `Apache Kafka`, a powerful distributed streaming platform.

In this guide, we will walk you through everything you need to know to set this up—from configuring your environment to testing that logs successfully arrive in Kafka. We’ll be compiling everything from source so you have complete control over the versions and features. And just to clarify, we’re not using Docker here; we’ll be working directly on bare metal or virtual machines.

Whether you're experimenting in a local lab or building the foundation for a robust production pipeline, this tutorial is here to support you. 🙌

---

# What You’ll Need (Prerequisites)

Before we get started, here’s what you will need:

- A Linux machine — either Ubuntu or CentOS/Rocky Linux
- Root or sudo access
- A working internet connection (as we’ll be downloading a few items)
- A bit of patience (compiling from source takes a few minutes) 😊

This guide is suitable for:

- 🛠️ System administrators looking to forward logs to Kafka  
- ⚙️ DevOps engineers designing observability stacks  
- 👩‍💻 Developers who want to build or test real-time data pipelines  


Whether you are deploying in a lab environment or preparing a production-ready log pipeline, this hands-on guide provides full control over the components involved—without the use of Docker or containerization. 

📌 **Note**: We will be using Kafka in **KRaft** mode, which means no ZooKeeper! This is a simpler and more modern approach that works well for test setups or small deployments.


---

## 🧱 Step 1: Installing Required Packages (Let’s Lay the Foundation)

Before we begin compiling anything, it's essential to ensure that your system has all the necessary tools and libraries. Think of this step as making sure your toolbox is fully stocked before you start building a piece of furniture—this will save you a lot of time and frustration later on.

We'll be installing a set of packages that are crucial for:

- Compiling software from source (like syslog-ng, librdkafka, and eventlog)
- Enabling support for networking, encryption, and Kafka communication
- Ensuring that everything links together smoothly during the build process

The exact command for installation will depend on your operating system. Please select the one that matches your setup.


***ubuntu/debian based systems:***
*Open your terminal and run:*
```
sudo apt update
sudo apt install -y git gcc make autoconf automake libtool pkg-config \
  libglib2.0-dev libjson-c-dev libcurl4-openssl-dev libssl-dev \
  liblzma-dev libsystemd-dev libz-dev uuid-dev flex bison
```

This grabs compilers (gcc, make), build tools (autoconf, automake, libtool), and a set of development libraries that `syslog-ng` relies on.

***centos/rocky linux (RHEL-based systems):***
*Run the following:*
```
sudo dnf install -y git gcc make autoconf automake libtool pkgconfig \
  glib2-devel json-c-devel libcurl-devel openssl-devel libuuid-devel \
  flex bison zlib-devel
```

Same idea here — development headers, compilers, and build essentials tailored for RHEL-based environments.

💡 **Why are all these packages necessary?**

Syslog-ng is modular and flexible. Compiling it with Kafka support requires pulling in all the essential dependencies manually, including those for crypto, JSON, UUIDs, and networking. Trust us, installing these now will help you avoid a lot of “missing dependency” errors later on.

Once this step is complete, your system will be ready to start building real-time log magic. ✨

---

## 📦 Step 2: Install librdkafka (Your Bridge to Kafka)

Now that your toolbox is ready, it's time to bring in one of the essential components for this integration: `librdkafka`.

Librdkafka is the official C/C++ client library for Apache Kafka, developed by Confluent. This library allows`syslog-ng`to communicate effectively with Kafka, enabling it to send logs across the network in a partitioned format that can be consumed by any downstream system.

For this setup, we’ll use version **1.9.2**, which is a stable and proven release that works well with`syslog-ng`version **3.35.x**.

🧠 **Important**: The compatibility between`syslog-ng`and librdkafka versions is crucial. They depend on specific APIs and features that must align. If you're using a different version of syslog-ng, it's advisable to verify which librdkafka release is supported before moving forward.

### Install Instructions

Let’s grab the source code and build it from scratch:
```
git clone https://github.com/confluentinc/librdkafka.git
cd librdkafka
git checkout v1.9.2
./configure
make -j$(nproc)
sudo make install
sudo ldconfig
```

Here’s a clearer explanation of each step:

1. **git clone**: This command downloads the source code from Confluent's GitHub repository.
2. **git checkout v1.9.2**: This command ensures that you are using the specific version we want, which is v1.9.2.
3. **./configure**: This command sets up the build environment necessary for compilation.
4. **make -j$(nproc)**: This command compiles the code while utilizing all available CPU cores to speed up the process.
5. **sudo make install**: This command installs the compiled libraries onto your system.
6. **sudo ldconfig**: This command refreshes the shared library cache, allowing your system to locate the new libraries.

The compiled libraries usually end up in the directory `/usr/local/lib`. Thanks to `ldconfig`, these libraries will be recognized system-wide. This means that when`syslog-ng`needs to link to Kafka, it will know exactly where to find it.

Once this step is complete, you’ll officially have Kafka enabled! Next up is installing another important component: the eventlog library. ✨

---

## 📚 Step 3: Install the Eventlog Library (A Must-Have for Syslog-ng)

Before we dive into compiling `syslog-ng`, there’s one important piece we need to include: the `eventlog` library.

This library is the unsung hero behind the scenes. It’s a core dependency of `syslog-ng`, handling how logs are structured and written internally. Without it, `syslog-ng` won’t even compile.

Fortunately, building it is quite straightforward! 😊

🛠️ **Let’s Build It!**

Here’s how to download and install it from the source:
```
git clone https://github.com/balabit/eventlog.git
cd eventlog
./autogen.sh
./configure
make -j$(nproc)
sudo make install
sudo ldconfig
```

**What’s Happening Here?**

1. **git clone** pulls the source code from the official Balabit repository.
2. **autogen.sh** sets up all the necessary build scripts (think of it as prepping the kitchen before cooking 🍳).
3. **./configure** detects your system and prepares the Makefile.
4. **make -j$(nproc)** compiles the source code using all your CPU cores (speed matters!).
5. **sudo make install** installs the library onto your system.
6. **sudo ldconfig** updates your system’s shared library cache so everything can find Eventlog when needed.

💡 **Pro Tip:** If `autogen.sh` complains about missing tools like `libtool`, `m4`, or `autoconf`, simply install them using your package manager. These tools help generate the configure scripts from scratch.

Once you’ve installed `eventlog`, you’re officially ready to build `syslog-ng` itself — the heart of this whole operation.

---

## ⚙️ Step 4: Set Up Kafka in KRaft Mode (No ZooKeeper Needed!)

Traditionally, running Kafka required also setting up ZooKeeper, which handled coordination and metadata. However, starting from Kafka 3.x, there is a much simpler way to run things: **KRaft** mode (Kafka Raft Metadata mode). This allows Kafka to manage itself without needing ZooKeeper at all.

In this step, we'll guide you through setting up Kafka in KRaft mode manually—no containers, no fluff. Just pure, hands-on Kafka goodness.

### 4.1 Install Java

Kafka is built on Java, so you'll need a working Java environment before starting it up. Here’s how to install OpenJDK 11 based on your operating system:

***ubuntu:***
```
sudo apt install -y openjdk-11-jdk
```

***centos/rocky:***
```
sudo dnf install -y java-11-openjdk
```

✅ **Note**: You can verify that Java is installed by running `java -version`.

### 4.2 Download and Extract Kafka

We will be using Confluent’s Kafka 7.5.0 Community Edition, which includes everything needed to get started quickly.

```
wget https://packages.confluent.io/archive/7.5/confluent-community-7.5.0.tar.gz
tar -xzf confluent-community-7.5.0.tar.gz
cd confluent-7.5.0
```

🎯 This will create a `confluent-7.5.0/` directory containing all the Kafka tools and scripts we’ll need.

### 4.3 Create a KRaft Configuration File

Kafka requires a configuration file to know how to run. Create a file named `custom-kraft-conf.properties` and paste the following content:

```
# Kraft mode broker config
process.roles=broker,controller
node.id=1
controller.quorum.voters=1@localhost:9093
log.dirs=/tmp/kraft-combined-logs

listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
inter.broker.listener.name=PLAINTEXT
controller.listener.names=CONTROLLER

offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
```

💡 This configuration is ideal for development or testing on a single node. You can adjust it later for production setups with multiple brokers.

### 4.5 Format KRaft Storage and Start Kafka

Before Kafka can start, it needs to initialize the storage directory using the configuration file you just created. 

Once formatted, start the Kafka server using the following command:

```
bin/kafka-storage format -t $(bin/kafka-storage random-uuid) -c custom-kraft-conf.properties
bin/kafka-server-start custom-kraft-conf.properties
```

🎉 That's it! Kafka is now running in KRaft mode with both the broker and controller roles handled by a single process. No ZooKeeper, no external dependencies.

🧪 Up next, we’ll create a topic and prepare Kafka to receive messages from syslog-ng.

---

## Step 5: Configure syslog-ng to Send Logs to Kafka

Now that Kafka is up and running, it’s time to set up syslog-ng so it can forward log messages directly to our Kafka topic. This step connects log collection with streaming.

Let’s begin by creating a custom configuration file. You can place it wherever you prefer, but a common convention is to use a specific directory for configuration files.

```
/etc/syslog-ng/conf.d/syslog-ng-custom.conf
```

Here’s what the contents of the file should look like:
```
@version: 3.35
@include "scl.conf"

# 1. Source: Accept syslog over TCP on port 5140
source s_netcat {
    tcp(ip("0.0.0.0") port(5140));
};

# 2. Destination: Kafka sink
destination d_kafka {
    kafka(
        bootstrap-servers("localhost:9092")
        topic("syslog-ng-topic")
        message("$(format-json --scope all-nv-pairs)")
    );
};

# 3. Log path: Connect source to destination
log {
    source(s_netcat);
    destination(d_kafka);
};

```

Let’s break down the configuration:

- **source s_netcat:** This line tells syslog-ng to listen for incoming syslog messages over TCP on port 5140. You can think of this as a custom “receiver” for logs.
- **destination d_kafka:** This is where the magic happens. Each incoming message is formatted as JSON using the `format-json` template and sent to the Kafka topic named `syslog-ng-topic`.
- **log { ... }:** Finally, we define the pipeline that connects our source (s_netcat) with our Kafka destination (d_kafka).

**Additional Configurations**

If you wish to listen on a different port or use UDP instead of TCP, you can easily adjust the source section to accommodate those changes.

Once this configuration is ready, all that’s left is to start syslog-ng with it and begin streaming logs into Kafka — which we’ll cover in the next step! 🚀

---

## Step 6: Start syslog-ng

Now that our configuration file is ready, it’s time to bring syslog-ng to life and let it start forwarding logs to Kafka. Run `syslog-ng` using the compiled binary and custom config:

```
sudo systemctl start syslog-ng
sudo systemctl status syslog-ng
```

Use `-Fvde` to run in foreground, with verbose and debug output.

*🛠️ Bonus Tip: Test Your Config*

Before starting the service, it’s a good idea to check if your config is valid:

```
syslog-ng -s -f /etc/syslog-ng/conf.d/syslog-ng-custom.conf
```

If everything is okay, you won’t see any errors.

Once `syslog-ng` is running and your Kafka broker is already up, you’re all set to send and receive logs in real time. In the next step, we’ll send a test log message using netcat and watch it flow into Kafka like magic! ✨

---

## Step 7: Send a Test Message with Netcat
Now that everything is set up, it’s time to test the pipeline and see if your logs make it from syslog-ng to Kafka. We’ll use a simple yet powerful tool called `netcat` (nc) to simulate a syslog message sent over TCP.

```
echo "<13>Sep  15 16:25:00 myhost myapp: This is a test from nc" | nc 127.0.0.1 5140
```

**💡 Why This Works**

Your `syslog-ng` config listens for incoming **TCP** messages on port **5140**, parses them, and forwards them to your Kafka topic (`syslog-ng-topic`). So when you send this message using nc, it’s picked up by syslog-ng and sent to Kafka—no extra work needed!

✅ *Pro Tip**: If you have a Kafka consumer running in the background, you should see the message arrive almost instantly, neatly wrapped in JSON format! If you're unsure how to check the messages, feel free to jump to the next section. 😊 


---

## Step 8: Verify in Kafka
You've sent your log—now let's ensure that it actually made it to Kafka. This is the final (and most satisfying) step: watching your log message appear in real-time like magic! 🪄

To do this, use the built-in Kafka consumer tool to read messages from the topic. 

```
bin/kafka-console-consumer --bootstrap-server localhost:9092 \
  --topic syslog-ng-topic --from-beginning
```

If everything is working correctly, you should see output like this:
```
{"TRANSPORT":"rfc3164+tcp","SOURCE":"s_netcat","PROGRAM":"myapp","MSGFORMAT":"rfc3164","MESSAGE":"This is a test from nc","LEGACY_MSGHDR":"myapp: ","HOST_FROM":"localhost","HOST":"localhost"}
```

---

# Summary

Let’s quickly recap what we’ve accomplished in this hands-on journey:

- ✅ Installed all necessary system dependencies
- 🛠️ Built and configured librdkafka and eventlog from source
- ⚙️ Set up Apache Kafka in KRaft mode — no ZooKeeper needed!
- 🧩 Created a Kafka topic and connected it with syslog-ng
- 📤 Sent a test log using netcat and verified it landed in Kafka successfully


By the end of this guide, you now have a real-time logging pipeline using syslog-ng and Kafka — from scratch and without Docker or containers. 🎉 This setup is solid, flexible, and perfect for both experimentation and production use.

🔜 In future posts, we’ll explore more advanced features — like filtering, enriching messages, and partitioning strategies to fine-tune performance.

👀 Looking for more?

You might also enjoy Selim’s excellent article on a similar topic:
➡️ ["Ingesting Syslog data to Kafka"](https://blog.seynur.com/kafka/2021/01/08/ingesting-syslog-data-to-kafka.html) (2021) — a great reference if you’re working with older versions of syslog-ng or Kafka.

Have ideas, suggestions, or want to share how you’ve extended this setup? Contributions are always welcome!
Feel free to fork the setup, build on it, and tag me when you do.

If you run into issues or just want to chat about logs, Kafka, or infrastructure in general — don’t hesitate to reach out to me on [LinkedIn](https://www.linkedin.com/in/%C3%B6yk%C3%BC-can/). I’d love to hear what you’re building.

📦 **Bonus Coming Soon**: If you’re more of an rsyslog fan — don’t worry. A Kafka integration guide for rsyslog is on the way! Stay tuned. 🚀


## References:
- [[1]](https://github.com/confluentinc/librdkafka) Confluent. (2025). ***confluentinc/librdkafka***. GitHub. https://github.com/confluentinc/librdkafka
- [[2]](https://github.com/balabit/eventlog) Balabit. (2025). ***balabit/eventlog***. GitHub. https://github.com/balabit/eventlog
- [[3]](https://github.com/syslog-ng/syslog-ng) syslog-ng. (2025). ***syslog-ng/syslog-ng***. GitHub. https://github.com/syslog-ng/syslog-ng
- [[4]](https://packages.confluent.io/archive/7.5/confluent-community-7.5.0.tar.gz) Confluent. (2025). Confluent Platform 7.5.0 Archive. Confluent Downloads. https://packages.confluent.io/archive/7.5/confluent-community-7.5.0.tar.gz
- [[5]](https://blog.seynur.com/kafka/2021/01/08/ingesting-syslog-data-to-kafka.html) Seynur, S. (2021). ***Ingesting Syslog data to Kafka***. Seynur Blog. https://blog.seynur.com/kafka/2021/01/08/ingesting-syslog-data-to-kafka.html

---
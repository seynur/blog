---
layout: default
title: "Dynamic Log Routing to Kafka with rsyslog: A Practical Guide You Can Follow"
summary: "This guide walks through setting up rsyslog to send log data directly to Kafka topics using a dynamic IP-to-topic mapping. We used a simple JSON lookup file to route incoming logs based on their source IP and confirmed delivery using netcat and a Kafka consumer. This lightweight yet powerful approach is ideal for building scalable log pipelines with minimal overhead."
author: Öykü Can Şimşir
image: /assets/img/blog/2025-09-16-streaming-rsyslog-events-to-kafka.webp
date: 17-09-2025
tags: rsyslog kafka kraft     
categories: Kafka

---

# Dynamic Log Routing to Kafka with rsyslog: A Practical Guide You Can Follow

If you enjoyed setting up `syslog-ng` with Kafka, you’re in for another treat. `rsyslog`—another popular and well-established syslog daemon—offers powerful features that make it an excellent choice for real-time log forwarding. In this guide, we will show you how to stream logs directly from `rsyslog` into Apache Kafka dynamically, with topics determined on the fly based on the source IP address. Yes, it’s that flexible!

We’ll use a convenient JSON-based lookup table to match incoming IPs with their corresponding Kafka topics. This approach allows you to route logs from multiple clients to different topics without hardcoding each one. It’s clean, efficient, and scalable.

This walkthrough builds on the same Kafka KRaft-mode setup we covered in our [syslog-ng + Kafka](https://blog.seynur.com/kafka/2025/09/15/streaming-syslog-ng-events-to-kafka.html) post. If you’re just joining us and haven’t set up Kafka yet, we recommend reading that section first.

As before, we won’t cover container setups or system-wide installations. There are no Kafka installation steps here. We’re keeping it simple and focused—so let’s dive into the rsyslog side of things!

---

## Why rsyslog?

rsyslog is the default syslog daemon on many Linux distributions, and it's known for its speed and flexibility. One of its strengths is the ability to dynamically route logs using lookups, which we’ll take advantage of in this setup.

*What we’re aiming for in this guide:*

- Receive logs over TCP and UDP
- Use a JSON lookup to match incoming IP addresses to Kafka topics
- Send raw log messages to dynamically chosen Kafka topics
- Optionally store the logs in structured directories

Whether you are running a large-scale log pipeline or just testing locally, this setup will provide you with flexibility and insight, allowing you to avoid manually hardcoding every source-to-topic rule.

---

# 📝 Sample rsyslog Configuration

Let’s dive into setting up rsyslog for dynamic log forwarding to Kafka. 

Below is a functional configuration that listens for incoming syslog messages (both UDP and TCP on port 514). It utilizes a JSON lookup file to match IP addresses to Kafka topics, forwarding the raw messages to Kafka while also storing them in structured log files on disk.

```
# Load necessary modules
module(load="imudp")
module(load="imtcp")
module(load="omkafka")

# Listen on UDP and TCP port 514
input(type="imudp" port="514")
input(type="imtcp" port="514")

# Load the lookup table that maps IPs to topic names
lookup_table(name="allhostnames" file="/home/oyku_home/rsyslog/lookup.json" reloadOnHUP="on")
set $.lookupresult = lookup("allhostnames", $fromhost-ip);

# Define templates for file paths and Kafka topics
$template logfile_path,"/opt/syslog/%$.lookupresult%/%FROMHOST-IP%/%$YEAR%-%$MONTH%-%$DAY%-%$HOUR%.log"
$template kafka_topic,"%$.lookupresult%"
$template rawmsg,"%msg%\n"

# Save logs to local disk using dynamic folders
action(
  type="omfile"
  dynaFile="logfile_path"
  template="rawmsg"
  createDirs="on"
)

# Only send to Kafka if the IP matched something in the lookup
if ($.lookupresult != "catchall") then {
  action(
    type="omkafka"
    dynatopic="on"
    broker=["localhost:9092"]
    topic="kafka_topic"
    template="rawmsg"
  )
}
```

🧠 **What’s Happening Here?**

- **Modular Loading:** We load only the necessary modules — imudp, imtcp, and omkafka.
- **Lookup Functionality:** The sender's IP address ($fromhost-ip) is matched against a JSON lookup file. The resulting value determines both:
  - The name of the Kafka topic
  - The folder name for local log storage

- **Kafka Integration:** If a match is found (i.e., it is not a "catchall"), we send the raw log message to the appropriate Kafka topic by enabling `dynatopic="on"`.
- **Disk Storage:** Independent of Kafka forwarding, logs are also stored on disk using a time-based folder structure.

This configuration is clean, efficient, and powerful. It creates a flexible log pipeline where logs are stored and routed intelligently based on their origin. This approach is ideal for scaling across multiple sources without the need to manually define every topic or folder.

---

# 📁 Lookup File Format

To enable all the dynamic features, you'll need a JSON lookup file that maps incoming IP addresses to Kafka topic names.

Here's what a simple `lookup.json` might look like:

```
{
  "version": 1,
  "nomatch": "catchall",
  "type" : "string",
  "table" : [
    {"index": "127.0.0.1", "value":"rsyslog-topic"}
  ]
}
```

**🔍 How It Works:**

- The key represents the IP address of the sender.
- The value indicates the Kafka topic to which you want to send logs from that IP.
- If an IP address doesn't match anything in the table, the fallback value "catchall" will be used (and you can choose not to send those logs to Kafka).


**✨ Pro Tip:** You can add as many IP-topic pairs as needed. There's no need to restart `rsyslog` when you update the file—simply send a HUP signal (using `kill -HUP <pid>`), and your changes will be reloaded instantly.

**💡 Why This Setup Is Beneficial:**

- ✅ **Smart Routing:** You only need one configuration file, but it can handle any number of source IPs and Kafka topics.
- ⚡ **High Performance:** `rsyslog` is extremely efficient—it’s the default logging system for a reason!
- 💾 **Safe Fallback:** Even if Kafka experiences issues, your logs are still written to disk as a backup.

This setup is perfect for hybrid environments where you want the flexibility of Kafka pipelines without sacrificing reliability.

---

# 🚀 Let's Test It!

## Step 1: Send Messages

Everything is set up—Kafka is running, rsyslog is configured, and your lookup.json file knows where each IP's logs should go. Now, it's time to send a test log and watch it arrive in the correct Kafka topic. 🎯

### Step 1: Send a Test Message Using `nc`

Run the following command on your machine to simulate a syslog message:

```
echo "<13>Sep 15 16:25:00 myhost myapp: This is a test from nc" | nc 127.0.0.1 514
```

This command sends a simple syslog-formatted message over **UDP** to port **514**. Ensure that the IP **127.0.0.1** is correctly mapped to `rsyslog-topic` in your lookup file.

## Step 2: Listen to the Kafka Topic

In a separate terminal, listen to the Kafka topic where you expect the logs to be routed:

```
bin/kafka-console-consumer --bootstrap-server localhost:9092 \
 --topic rsyslog-topic --from-beginning
```

### ✅ What You Should See

If everything is working correctly, you should see the message in the Kafka topic like this:

```
2025-09-16T16:25:00+00:00 myhost myapp: This is a test from nc
```

And there you have it—a live log flowing from `rsyslog`, matched by source IP, and delivered directly into Kafka!

💬 **Tip**: Try sending test logs from different IPs (or simulate them using `iptables` and `socat`) to observe how they get dynamically routed to different topics.

---

# Wrapping Up

And that’s a wrap! 🎉 You’ve just created a flexible and dynamic log shipping setup using rsyslog and Kafka. Instead of static topics and rigid configurations, you now have:

- 🔄 Dynamic Kafka topic routing based on source IPs
- ⚡ High-performance logging powered by rsyslog’s efficient engine
- 💾 Disk-based backups for times when Kafka may be temporarily unavailable
- 📦 A lightweight, production-ready solution that doesn’t rely on external dependencies like ZooKeeper

This architecture is ideal for scaling across environments where logs from different sources need to be grouped, processed, and analyzed separately. Whether you’re developing a centralized logging system, a SIEM pipeline, or a robust observability stack, this pattern provides considerable power with minimal overhead.

💬 If you encounter any issues or simply want to discuss logs, Kafka, or infrastructure in general, feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/%C3%B6yk%C3%BC-can/). I’d love to hear about your projects or offer assistance if you're facing challenges.

Happy logging! 🚀📊



## References:
- [[1]](https://blog.seynur.com/kafka/2025/09/15/streaming-syslog-ng-events-to-kafka.html) Simsir Can, Ö. (2025). ***Streaming syslog-ng Events to Kafka: A Practical Guide You Can Follow***. Seynur Blog. https://blog.seynur.com/kafka/2025/09/15/streaming-syslog-ng-events-to-kafka.html
- [[2]](https://blog.seynur.com/kafka/2021/01/08/ingesting-syslog-data-to-kafka.html) Seynur, S. (2021). ***Ingesting Syslog data to Kafka***. Seynur Blog. https://blog.seynur.com/kafka/2021/01/08/ingesting-syslog-data-to-kafka.html
- [[3]](https://www.rsyslog.com/doc/master/index.html) Adiscon. (2025). ***rsyslog Documentation (Master)***. https://www.rsyslog.com/doc/master/index.html
- [[4]](https://www.rsyslog.com/doc/master/configuration/lookuptables.html) Adiscon. (2025). ***Lookup Tables in rsyslog***. https://www.rsyslog.com/doc/master/configuration/lookuptables.html
- [[5]](https://www.rsyslog.com/doc/master/configuration/modules/omkafka.html) Adiscon. (2025). ***omkafka Output Module***. https://www.rsyslog.com/doc/master/configuration/modules/omkafka.html

---
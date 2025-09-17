---
layout: default
title:  "Send Windows Event Logs to Kafka — The Easy Way with Splunk UF & Splunk S2S Kafka Connector"
summary: "In this blog, I show how to collect Windows Event Logs with Splunk Universal Forwarder and stream them directly into Kafka using the Splunk S2S Connector — no HEC, no scripts, just clean, scalable log shipping."
author: Öykü Can Şimşir
image: /assets/img/blog/2025-09-18-streaming-windows-events-to-kafka-with-s2s-connector.webp
date: 18-09-2025
tags: splunk kafka s2sconnector 
categories: Splunk

---

# Send Windows Event Logs to Kafka — The Easy Way with Splunk UF & Splunk S2S Kafka Connector

“I just want to get my **Windows** logs into **Kafka**-*without having to write yet another script*.”

If this sounds like you, then you’re in the right place.

In this post, we will guide you on how to collect Windows Event Logs from a machine and send them to **Kafka** using the **Splunk Universal Forwarder (UF)** and the **Splunk S2S Kafka Connector**.

Although this example focuses on *Security* and *Application* logs, the setup is applicable to any *Windows Event Log channel*, including `Operational`, `System`, or even custom logs.

Let’s dive in.

---

## Why Send Windows Event Logs to Kafka?

Let’s be real — *the goal isn’t just collecting logs*.
It’s building a smart, flexible, and scalable pipeline.

And that’s where **Kafka** shines.
Instead of sending logs directly to your SIEM (like Splunk) and overwhelming it with raw noise, you can:

- 🌀 Ingest everything into Kafka first
- 🧹 Clean, enrich, filter, or transform the data
- 🎯 Forward only what matters to *Splunk* (or anywhere else)

```
[Windows - Splunk UF] --> [Kafka (raw)] --> [Stream Processor] --> [Kafka (clean)] --> [Splunk, Elastic, etc.]

```

This approach keeps your Splunk indexers efficient, helps control your license usage, and maintains a modular architecture. You can even send the enriched logs back to Splunk later—once they are clean, tagged, and relevant.

- Need to keep raw logs for auditing? Kafka can handle that.  
- Need to send alerts to Elastic or S3? No problem.  
- Need to buffer bursts of event traffic? Kafka can manage that too.

In this architecture, Kafka becomes the **central nervous system** of your logging pipeline — and Splunk is just one of many possible destinations.

---

# The Architecture

Let’s break down what’s really happening under the hood.

We’re collecting Windows Event Logs from a remote machine using Splunk Universal Forwarder (UF). UF sends those logs over TCP port 9997, which is a commonly used port in the Splunk ecosystem for forwarder-to-indexer communication.

But instead of sending them to Splunk’s indexers, we’re routing them to Kafka Connect, where the Splunk S2S Connector is actively listening on that port.

The connector reads the raw incoming logs and publishes them directly into a Kafka topic — in our case, windows_event_logs.

From there, you can do whatever you want:
- Process the logs with Kafka Streams or ksqlDB
- Enrich or redact sensitive data
- Route them to multiple destinations (Splunk, Elastic, cloud storage…)

Here’s a simplified view of the flow:

```
Windows Machine
  ⬇ (Event Logs via Splunk UF)
TCP 9997
  ⬇ (Port can be customized)
Kafka Connect Worker + Splunk S2S Connector
  ⬇
Kafka Topic: windows_event_logs
```

This architecture has several advantages:
- 🔌 *Agentless Kafka ingestion* — no need for a custom TCP server or log parser
- ⚖️ *Decoupled processing* — Splunk doesn’t get overloaded with noisy raw logs
- 🔄 *Reusable data* — one log stream can feed multiple tools
- 📏 *Scalable* — add more topics, sources, or sinks as you grow

And yes — while we’re focusing on *Security* & *Application* logs in this example, the same architecture works for any *Windows Event Log channel* you want to collect.

---

# Step-by-Step Setup
## Step 1: Install Splunk Universal Forwarder (UF) on Windows

*Splunk Universal Forwarder (UF)* is a lightweight and reliable way to collect *Windows* Event Logs.

In our case, we configured it to collect logs from the *Security* and *Application* channels.
But you can use this method for any Windows Event Log source you like.

*📚 Need more options or ready-made configurations?*
Check out the official [Splunk Add-on for Microsoft Windows](https://splunk.github.io/splunk-add-on-for-microsoft-windows/) — and download it from [Splunkbase](https://splunkbase.splunk.com/app/742).
It’s built by Splunk and comes with predefined inputs, props, and transforms.

We’ll use that add-on too — but let’s keep it simple to start.

Let’s assume that your Kafka server IP is `X.X.X.X`, and the connector listens on port `9997`.
**Warning**: Be sure to update the IP and port according to your setup.

💡 If you don’t define an index in your `inputs.conf`, the logs will default to `index=default` when they appear in Kafka. 

*Sample `inputs.conf`*:

```
# inputs.conf

[WinEventLog://Security]
disabled = 0
renderXml = true
index = oswinsec

[WinEventLog://Application]
disabled = 0
renderXml = true
index = oswinsec
```

*Sample `outputs.conf`*:

```
# outputs.conf

[tcpout]
defaultGroup = kafka-group

[tcpout:kafka-group]
server = X.X.X.X:9997
```

This configuration sends all collected logs to the **Kafka Connect worker**, via TCP, on port **9997**.

🧠 **Reminder**: This is not Splunk HEC — we’re sending raw UF data directly to Kafka using a plugin.

---

*But First: Kafka Should Be Ready!*

In the next step, we’ll install the *Splunk S2S plugin* for *Kafka Connect* and configure it. 

If your *Kafka* isn’t set up yet, I’ve got you covered. 👉 Head over to my guide on [syslog-ng + Kafka](https://blog.seynur.com/kafka/2025/09/16/streaming-syslog-ng-events-to-kafka.html) and jump to Step 4 — it includes everything you need to set up *Kafka* and verify it’s working.

Once *Kafka* is up and running, we’re ready for *Step 2: Kafka Connect + Splunk Plugin*. 😊

---

## Step 2: Set Up Kafka Connect & Install the Splunk Plugin
*Next up*: let’s prepare **Kafka Connect** and install the **Splunk S2S Source Connector**.

We’ll be using *Kafka Connect* in *distributed mode* — which is the recommended approach for production and scaling.

**🧩 Installing the Connector:**

The easiest way to install the plugin is via the **Confluent Hub CLI**. The connector JAR should end up in a path like `/home/oyku/confluent/plugins/splunk/kafka-connect-v2.2.2.jar`

- Alternatively, you can manually download and unzip the connector, then place it under your *Kafka Connect* plugin directory (e.g., `$KAFKA_HOME/plugins`).

**⚙️ Kafka Connect Worker Configuration:**

Kafka Connect uses a properties file to define how it runs. The default one is usually located at:

```
$KAFKA_HOME/etc/kafka/connect-distributed.properties
```
You can rename or relocate this file if needed — just make sure your startup script points to the right one.

Here’s a minimal but working example:
```
# connect-distributed.properties
############################# Server Basics #############################

bootstrap.servers=X.X.X.X:9092
group.id=connect-cluster


############################# Kafka Converters #############################
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter

key.converter.schemas.enable=true
value.converter.schemas.enable=true

############################# Internal Topic Settings #############################

offset.storage.topic=connect-offsets
offset.storage.replication.factor=1


############################# Internal Connector Settings #############################

config.storage.topic=connect-configs
config.storage.replication.factor=1

status.storage.topic=connect-status
status.storage.replication.factor=1

offset.flush.interval.ms=10000

############################# PLUGINS #############################

plugin.path=/home/oyku/confluent/plugins/splunk,/usr/share/java

```

⚠️ *Don’t forget to update*:
- `bootstrap.servers` with your *Kafka broker address*
- `plugin.path` with the actual path where the connector JAR is located

Once this is set up, you’re ready to launch *Kafka Connect* and start accepting incoming logs via the connector.

---

## 🧩 Step 3: Define the Splunk S2S Connector
Now that *Kafka Connect* is up and running, it’s time to configure the **Splunk S2S Source Connector**.

Here’s a minimal `splunk-s2s-connector.json` file that instructs the connector to:
- 🛠️ Listen for incoming logs on TCP port *9997*
- 📝 Write those logs into a Kafka topic named `windows_event_logs`

```
{
  "name": "splunk-s2s-connector",
  "config": {
    "connector.class": "io.confluent.connect.splunk.s2s.SplunkS2SSourceConnector",
    "kafka.topic": "windows_event_logs",
    "splunk.s2s.port": "9997",
    "splunk.host": "X.X.X.X",
    "splunk.listener": "tcp",
    "splunk.s2s.enable.ack": "false"
  }
}
```

**🚀 Deploy the Connector**
To create the *connector*, use a simple curl POST command:

```
curl -X POST -H "Content-Type: application/json" \
  --data @<full-path>/splunk-s2s-connector.json \
  http://X.X.X.X:8083/connectors
```

*Replace*:
- `<full-path>` with the actual path to your JSON file
- **X.X.X.X** with the IP address of your Kafka Connect worker

**✅ Verify the Connector Is Running**
Once submitted, check the connector’s status with:

```
curl http://X.X.X.X:8083/connectors/splunk-s2s-connector/status | jq
```

If everything is set up correctly, you should see something like:

```
{
  "name": "splunk-s2s-connector",
  "connector": {
    "state": "RUNNING",
    "worker_id": "X.X.X.X:8083"
  },
  "tasks": [
    {
      "id": 0,
      "state": "RUNNING",
      "worker_id": "X.X.X.X:8083"
    }
  ],
  "type": "source"
}
```

⚠️ If the connector isn’t running or the task has failed, go back and check your previous steps — particularly the plugin installation and plugin.path configuration.

---

# 🔎 What Do the Logs Look Like in Kafka?

Once everything is configured, and Splunk UF is restarted with your valid `inputs.conf` and `outputs.conf`, logs should start flowing into *Kafka*.

To verify, you can consume the topic directly using the following command:

```
$KAFKA_HOME/bin/kafka-console-consumer \
  --bootstrap-server X.X.X.X:9092 \
  --topic windows_event_logs
```

You’ll start seeing log events in structured JSON format — with each record including schema metadata and the raw Windows Event Log XML payload.

🛡️ **Example**: `WinEventLog://Security`
```
{
  "schema": {
    "type": "struct",
    "fields": [
      {
        "type": "string",
        "optional": true,
        "doc": "The Splunk S2S event data received",
        "field": "event"
      },
      {
        "type": "string",
        "optional": true,
        "doc": "The host from which the Splunk S2S event was received from",
        "field": "host"
      },
      {
        "type": "string",
        "optional": true,
        "doc": "The source of the Splunk S2S event",
        "field": "source"
      },
      {
        "type": "string",
        "optional": true,
        "doc": "The index for the Splunk S2S event",
        "field": "index"
      },
      {
        "type": "string",
        "optional": true,
        "doc": "The type of the Splunk S2S event source",
        "field": "sourcetype"
      },
      {
        "type": "int64",
        "optional": true,
        "doc": "The unix time parsed from Splunk S2S event",
        "field": "time"
      }
    ],
    "optional": false,
    "name": "io.confluent.connect.splunk.s2s.Event"
  },
  "payload": {
    "event": "<Event xmlns='http://schemas.microsoft.com/win/2004/08/events/event'><System><Provider Name='Microsoft-Windows-Security-Auditing' Guid='{54849625-5478-4994-a5ba-3e3b0328c30d}'/><EventID>4672</EventID><Version>0</Version><Level>0</Level><Task>12548</Task><Opcode>0</Opcode><Keywords>0x8020000000000000</Keywords><TimeCreated SystemTime='2025-09-16T06:07:32.8683389Z'/><EventRecordID>182981</EventRecordID><Correlation ActivityID='{fab1af82-fc1d-0000-50b0-b1fa1dfcdb01}'/><Execution ProcessID='908' ThreadID='756'/><Channel>Security</Channel><Computer>DESKTOP-K3DLSL8</Computer><Security/></System><EventData><Data Name='SubjectUserSid'>S-1-5-18</Data><Data Name='SubjectUserName'>SYSTEM</Data><Data Name='SubjectDomainName'>NT AUTHORITY</Data><Data Name='SubjectLogonId'>0x3e7</Data><Data Name='PrivilegeList'>SeAssignPrimaryTokenPrivilege",
    "host": "DESKTOP-K3DLSL8",
    "source": "WinEventLog:Security",
    "index": "oswinsec",
    "sourcetype": "XmlWinEventLog:Security",
    "time": 1758002852
  }
}
```

🧩 **Example**: `WinEventLog://Application`
```
{
  "schema": {
    "type": "struct",
    "fields": [
      {
        "type": "string",
        "optional": true,
        "doc": "The Splunk S2S event data received",
        "field": "event"
      },
      {
        "type": "string",
        "optional": true,
        "doc": "The host from which the Splunk S2S event was received from",
        "field": "host"
      },
      {
        "type": "string",
        "optional": true,
        "doc": "The source of the Splunk S2S event",
        "field": "source"
      },
      {
        "type": "string",
        "optional": true,
        "doc": "The index for the Splunk S2S event",
        "field": "index"
      },
      {
        "type": "string",
        "optional": true,
        "doc": "The type of the Splunk S2S event source",
        "field": "sourcetype"
      },
      {
        "type": "int64",
        "optional": true,
        "doc": "The unix time parsed from Splunk S2S event",
        "field": "time"
      }
    ],
    "optional": false,
    "name": "io.confluent.connect.splunk.s2s.Event"
  },
  "payload": {
    "event": "<Event xmlns='http://schemas.microsoft.com/win/2004/08/events/event'><System><Provider Name='Microsoft-Windows-Security-SPP' Guid='{E23B33B0-C8C9-472C-A5F9-F2BDFEA0F156}' EventSourceName='Software Protection Platform Service'/><EventID Qualifiers='16384'>16384</EventID><Version>0</Version><Level>4</Level><Task>0</Task><Opcode>0</Opcode><Keywords>0x80000000000000</Keywords><TimeCreated SystemTime='2025-09-16T15:13:01.3560582Z'/><EventRecordID>35900</EventRecordID><Correlation/><Execution ProcessID='0' ThreadID='0'/><Channel>Application</Channel><Computer>DESKTOP-K3DLSL8</Computer><Security/></System><EventData><Data>2125-08-23T15:13:01Z</Data><Data>RulesEngine</Data></EventData></Event>",
    "host": "DESKTOP-K3DLSL8",
    "source": "WinEventLog:Application",
    "index": "oswinsec",
    "sourcetype": "XmlWinEventLog:Application",
    "time": 1758035581
  }
}
```

These logs are delivered in **raw XML format**, which is typical for Windows Event Logs. You can parse or transform them downstream using tools such as:
- 🔧 Logstash or Grok patterns
- 🧪 XML processors (e.g., XPath parsers)
- 📊 Kafka Streams or custom enrichment logic
- 🛡️ SIEM field extraction rules (e.g., *Splunk* props & transforms)

💡 Since the payload includes the full XML structure, you can extract anything — from EventID to User SID, IP addresses, process names, and more.

---

# 🎉 Why This Approach Rocks
- 🧩 *No extra agents* – You’re using *Splunk Universal Forwarder*, which is already an industry standard.
- 🚀 *Streamlined pipeline* – No need to write or maintain custom TCP listeners or log collectors.
- 💡 *Flexible log selection* – Just update your `WinEventLog://...` stanzas to collect different log channels.
- 🔁 *Scalable by design* – Kafka Connect can be clustered and extended as your needs grow.

*🛠️ Bonus Tool: [PADAS](padas.io)*
If you’re looking for a way to transform, enrich, and filter events in real-time, check out our streaming product:
👉 [**PADAS**](padas.io)

I’ll be sharing a detailed walkthrough on how to use [**PADAS**](padas.io) in this kind of pipeline in an upcoming post — stay tuned! 😊

---

# 📚 More Collection Methods?
This blog is part of a series! Check out the others for alternative log collection pipelines:
- 📥 [Streaming rsyslog Events to Kafka](https://blog.seynur.com/kafka/2025/09/17/streaming-rsyslog-events-to-kafka.html)
- 📥 [Streaming syslog-ng Events to Kafka](https://blog.seynur.com/kafka/2025/09/16/streaming-syslog-ng-events-to-kafka.html)

---

# 🙌 Final Thoughts
Getting Windows Event Logs into Kafka doesn’t have to involve messy scripts or fragile file watchers.

This approach is **simple**, **scalable**, and **fully supported** — especially if you’re already using *Splunk UF* in your environment.

Let me know how it works for you, or how you’ve extended it in your own setup.
I’m always excited to hear about real-world pipelines and creative solutions!

---

# Wrapping Up

And that’s a wrap! 🎉 You’ve just built a flexible and dynamic log shipping pipeline using **Splunk UF** and **Kafka**.

Instead of rigid, hardcoded configurations, you now have:
- ✅ A modular setup
- 🔄 Kafka-based streaming
- 📦 Support for any *Windows Event Log* channel
- ✨ Room to grow and adapt as your infrastructure evolves

*💬 Got questions, ideas, or challenges?*
Let’s connect and chat! I’m always happy to talk logs, Kafka, or infrastructure in general.
You can find me on [*LinkedIn*](https://www.linkedin.com/in/%C3%B6yk%C3%BC-can/).

Until next time —
Happy logging! 🚀📊


## References:
- [[1]](https://blog.seynur.com/kafka/2025/09/15/streaming-syslog-ng-events-to-kafka.html) Simsir Can, Ö. (2025). ***Streaming syslog-ng Events to Kafka: A Practical Guide You Can Follow***. Seynur Blog. https://blog.seynur.com/kafka/2025/09/15/streaming-syslog-ng-events-to-kafka.html
- [[2]](https://blog.seynur.com/kafka/2025/09/17/streaming-rsyslog-events-to-kafka.html) Simsir Can, Ö. (2025). ***Streaming rsyslog Events to Kafka***. Seynur Blog. https://blog.seynur.com/kafka/2025/09/17/streaming-rsyslog-events-to-kafka.html
- [[3]](https://splunk.github.io/splunk-add-on-for-microsoft-windows/) Splunk Inc. (2025). ***Splunk Add-on for Microsoft Windows Documentation***. https://splunk.github.io/splunk-add-on-for-microsoft-windows/
- [[4]](https://splunkbase.splunk.com/app/742) Splunk Inc. (2025). ***Splunk Add-on for Microsoft Windows***. Splunkbase. https://splunkbase.splunk.com/app/742
- [[5]](https://docs.confluent.io/kafka-connect-splunk-s2s/current/) Confluent. (2025). ***Splunk S2S Source Connector for Confluent Platform***. https://docs.confluent.io/kafka-connect-splunk-s2s/current/

---
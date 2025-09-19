---
layout: default
title: "What is OpenTelemetry and How to Use It? – A Beginner's Guide"
summary: "OpenTelemetry is an open-source observability framework that collects, processes, and sends telemetry data (traces, metrics, logs) from your applications. This comprehensive guide covers core concepts, architecture, and hands-on setup with Node.js examples."
author: Beste Yiğit
image: /assets/img/blog/2025-09-19-opentelemetry.png
date: 19-09-2025
tags: opentelemetry observability telemetry tracing metrics logs monitoring
categories: OpenTelemetry
---

# What is OpenTelemetry and How to Use It? – A Beginner's Guide

> **Welcome!** In this comprehensive guide, I'll walk you through **OpenTelemetry** (OTel) in a friendly and accessible way—explaining what it is, why it matters, and how you can implement it in your projects.

I'll break down technical concepts like **telemetry** (data about your app's performance) and **observability** (understanding what's happening inside your system) to make everything crystal clear. We'll build a practical JavaScript example together, and I'll explain OpenTelemetry's architecture step by step.

This guide is fully up-to-date as of September 2025, including the latest developments like OpenTelemetry Collector v1.0.

**Let's dive in!** 🚀
## 1. What is OpenTelemetry?

**OpenTelemetry** is an open-source observability framework developed by the **Cloud Native Computing Foundation (CNCF)**. It collects, processes, and sends **telemetry data**—like **traces**, **metrics**, and **logs**—from your applications to analysis tools (e.g., Prometheus, Jaeger, Datadog). 

Born in 2019 from the merger of OpenTracing and OpenCensus, it's now the **go-to standard for observability**.

### The Three Pillars of Observability

OpenTelemetry focuses on three key types of telemetry data:

| **Type** | **Description** | **Example** |
|----------|----------------|-------------|
| **🔍 Traces** | Track a request's journey through your system | What happens when a user clicks a button |
| **📊 Metrics** | Numeric data about system performance | CPU usage, request count, response time |
| **📝 Logs** | Error messages or event records | "User not found", "Database connection failed" |

### Key Benefits

- **Standardized Data Collection**: Gathers telemetry data in a consistent way
- **Vendor Neutrality**: Prevents vendor lock-in and works with multiple analysis tools
- **Open Source**: Free and community-driven development

> **💡 2025 Update**: At KubeCon EU 2025, OpenTelemetry was declared the **"de facto standard."** A new tool called **Weaver** automates telemetry. In JavaScript, traces and metrics are stable, while logs are still in development. Over 12 platforms (Splunk, AWS, Dynatrace) are fully compatible.

## 2. Why Use OpenTelemetry?

Modern systems (think microservices or Kubernetes) are complex, and pinpointing issues can be challenging. OpenTelemetry simplifies this process significantly. Here's why it's become the industry standard:

### 🎯 Core Advantages

| **Feature** | **Benefit** | **Impact** |
|-------------|-------------|------------|
| **🔓 Open Source** | Free and community-driven | 1,106 companies contribute (Splunk 27%, Microsoft 17%) |
| **🌐 Vendor-Neutral** | Works with AWS, Google Cloud, Azure, and more | No vendor lock-in |
| **⚡ Automatic Instrumentation** | Adds telemetry without code changes | Zero-effort monitoring |
| **📡 Standard Protocol (OTLP)** | Ensures consistent data format | Universal compatibility |
| **🚀 Production-Ready** | Stable in Java, Python, Go, JavaScript, and more | Enterprise-grade reliability |
| **💰 Cost Savings** | Avoid hefty monitoring bills | Significant budget reduction (e.g., OpenAI's Datadog bill was $100M+) |

### 📈 2025 Industry Trends

- **Open-source tools are booming**: Prometheus (89% adoption) and OpenTelemetry (85% adoption)
- **40% of companies** use both Prometheus and OpenTelemetry together
- **50% are adopting** proactive observability and AI integration (e.g., anomaly detection)
- **Growing ecosystem**: Over 90+ compatible tools and platforms

## 3. OpenTelemetry Architecture

OpenTelemetry collects, processes, and sends telemetry data (traces, metrics, logs) in a **modular, scalable way**. Here's how the system works:

### 🏗️ Core Components

| **Component** | **Purpose** | **Description** |
|---------------|-------------|-----------------|
| **🔧 Instrumentation** | Generate telemetry data | Adds code to your app (manual or automatic) |
| **📚 APIs and SDKs** | Standard interfaces | Language-specific tools (Tracer, Meter) |
| **🔄 OpenTelemetry Collector** | Central processing hub | Receives, processes, and routes data |
| **📤 Exporting** | Send to analysis tools | Jaeger, Prometheus, Datadog, New Relic |
| **🔗 Context Propagation** | Connect traces | W3C Trace Context across microservices |

### Key Features

- **Manual Instrumentation**: You write the code to generate telemetry data
- **Automatic Instrumentation**: Libraries handle it automatically (e.g., auto-tracing for Express)
- **Standard APIs**: Consistent interfaces across all languages
- **Language SDKs**: Collect and export data efficiently
- **W3C Trace Context**: Seamlessly track requests across distributed systems

> **🚀 2025 Update**: OpenTelemetry Collector v1.0 now supports large-scale enterprise systems with improved performance and reliability.

### 🔄 OpenTelemetry Collector Components

The **OpenTelemetry Collector** is the heart of telemetry data collection, processing, and routing. It consists of three main components that work together:

#### 1. 📥 Receivers

**Purpose**: Collect telemetry data (traces, metrics, logs) from applications or other sources.

| **Type** | **Protocol** | **Use Case** | **Example Endpoint** |
|----------|--------------|--------------|---------------------|
| **OTLP** | HTTP/gRPC | Most common receiver | `http://localhost:4318/v1/traces` |
| **Jaeger** | HTTP | Jaeger tracing data | `http://localhost:14268/api/traces` |
| **Prometheus** | HTTP | Metrics scraping | `http://localhost:9090/metrics` |
| **Zipkin** | HTTP | Zipkin tracing | `http://localhost:9411/api/v2/spans` |
| **Kafka** | Kafka | Message queues | `kafka://localhost:9092` |
| **AWS X-Ray** | HTTP | AWS services | `https://xray.amazonaws.com` |

> **💡 Example**: If your app sends data via OTLP, the Collector's OTLP receiver catches it at `http://localhost:4318/v1/traces`.

#### 2. ⚙️ Processors

**Purpose**: Filter, transform, or optimize data to reduce noise and improve efficiency.

| **Processor** | **Function** | **Configuration** | **Benefit** |
|---------------|--------------|-------------------|-------------|
| **Batch** | Groups data for efficient sending | `send_batch_size: 1024` | Reduces network load |
| **Attributes** | Adds/edits metadata | `region: "us-west-2"` | Enriches data context |
| **Filter** | Keeps/discards specific data | `only error traces` | Reduces noise |
| **Sampling** | Retains percentage of traces | `10% of traces` | Controls data volume |
| **Resource** | Adds source information | `hostname, version` | Improves traceability |

> **💡 Example**: The batch processor groups 1024 items together before sending, significantly reducing network overhead.

#### 3. 📤 Exporters

**Purpose**: Send processed data to analysis tools or storage systems.

| **Exporter** | **Target** | **Protocol** | **Example Endpoint** |
|--------------|------------|--------------|---------------------|
| **OTLP** | Another Collector | HTTP/gRPC | `http://collector:4317` |
| **Jaeger** | Jaeger UI | HTTP | `http://localhost:14268/api/traces` |
| **Prometheus** | Prometheus | HTTP | `http://localhost:8889/metrics` |
| **Logging** | Console/Files | Text | Debug output |
| **Datadog** | Datadog APM | HTTP | `https://api.datadoghq.com` |
| **Kafka** | Message Queue | Kafka | `kafka://localhost:9092` |

> **💡 Example**: The Prometheus exporter makes metrics available at `http://localhost:8889/metrics` for Prometheus to scrape.

<div align="center">
<img src="/assets/img/blog/2025-09-19-opentelemetry.png" alt="OpenTelemetry Architecture" width="600" style="max-width: 100%; height: auto;" />
<p><em>Figure 1. OpenTelemetry Collector architecture showing receivers, processors, and exporters. Source: Created by the author.</em></p>
</div>

### 📋 Summary

The OpenTelemetry Collector follows a simple but powerful workflow:

1. **📥 Receives** data from various sources
2. **⚙️ Processes** data to optimize and enrich it  
3. **📤 Exports** data to analysis tools

Its **modular design** adapts to various systems and needs, making it the perfect solution for modern observability requirements.

## 4. 🚀 OpenTelemetry in Action: Node.js Example

Let's build a **dice-rolling application** to demonstrate OpenTelemetry's tracing and metrics capabilities in a real-world scenario. This app will handle user requests and return random dice roll results while generating comprehensive telemetry data.

### 📋 Project Overview

- **Application**: Simple dice rolling API
- **Features**: Request tracing, metrics collection, error handling
- **Technologies**: Node.js, Express, OpenTelemetry
- **Output**: Traces in Jaeger, Metrics in Prometheus

---

### Step 1: 🛠️ Project Setup

First, let's create our project directory and install the necessary dependencies:

```bash
# Create project directory
mkdir otel-dice-app
cd otel-dice-app

# Initialize npm project
npm init -y

# Install dependencies
npm install express @opentelemetry/api @opentelemetry/sdk-node \
  @opentelemetry/exporter-jaeger @opentelemetry/exporter-trace-otlp-http \
  @opentelemetry/exporter-metrics-otlp-http @opentelemetry/resources \
  @opentelemetry/semantic-conventions @opentelemetry/sdk-trace-node \
  @opentelemetry/sdk-metrics @opentelemetry/instrumentation-express
```

> **💡 Note**: We're installing both Jaeger and OTLP exporters to demonstrate different integration options.

### Step 2: ⚙️ OpenTelemetry Configuration

Create `instrumentation.js` to configure OpenTelemetry for our application:

```javascript
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { ConsoleSpanExporter } = require('@opentelemetry/sdk-trace-node');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-http');
const { PeriodicExportingMetricReader, ConsoleMetricExporter } = require('@opentelemetry/sdk-metrics');
const { OTLPMetricExporter } = require('@opentelemetry/exporter-metrics-otlp-http');
const { resourceFromAttributes } = require('@opentelemetry/resources');
const { ATTR_SERVICE_NAME, ATTR_SERVICE_VERSION } = require('@opentelemetry/semantic-conventions');

// Environment configuration
const isProduction = process.env.NODE_ENV === 'production';
const useCollector = process.env.USE_COLLECTOR === 'true';

// Configure trace exporter based on environment
const traceExporter = isProduction 
  ? (useCollector 
    ? new OTLPTraceExporter({ url: 'http://localhost:4318/v1/traces' })
    : new JaegerExporter({ endpoint: 'http://localhost:14268/api/traces' }))
  : new ConsoleSpanExporter();

// Configure metric exporter based on environment
const metricExporter = isProduction 
  ? (useCollector 
    ? new OTLPMetricExporter({ url: 'http://localhost:4318/v1/metrics' })
    : new ConsoleMetricExporter())
  : new ConsoleMetricExporter();

// Initialize OpenTelemetry SDK
const sdk = new NodeSDK({
  resource: resourceFromAttributes({ 
    [ATTR_SERVICE_NAME]: 'dice-server', 
    [ATTR_SERVICE_VERSION]: '0.1.0' 
  }),
  traceExporter,
  metricReader: new PeriodicExportingMetricReader({ 
    exporter: metricExporter 
  })
});

// Start the SDK
sdk.start();
```

> **🔧 Configuration Details**:
> - **Development**: Uses console exporters for easy debugging
> - **Production**: Supports both Jaeger and OTLP collectors
> - **Service Info**: Identifies our service as "dice-server" v0.1.0

### Step 3: 🎲 Express Application

Create `app.js` with our dice-rolling API:

```javascript
const { trace, metrics } = require('@opentelemetry/api');
const express = require('express');

// Initialize OpenTelemetry instruments
const tracer = trace.getTracer('dice-server', '0.1.0');
const meter = metrics.getMeter('dice-server', '0.1.0');
const counter = meter.createCounter('requests.count', { 
  description: 'Total number of requests' 
});

const app = express();

// Dice rolling endpoint with tracing and metrics
app.get('/rolldice', (req, res) => {
  // Start a new span for this request
  const span = tracer.startSpan('rolldice-request');
  
  // Increment request counter
  counter.add(1);

  try {
    // Parse and validate input
    const rolls = parseInt(req.query.rolls) || NaN;
    
    if (isNaN(rolls)) {
      span.setAttribute('error', 'Invalid rolls parameter');
      span.setStatus({ code: 2, message: 'Invalid input' }); // ERROR status
      span.end();
      return res.status(400).json({ 
        error: 'Rolls parameter is missing or not a number.' 
      });
    }

    // Generate dice rolls
    const result = Array.from({ length: rolls }, () => 
      Math.floor(Math.random() * 6) + 1
    );
    
    // Add attributes to span
    span.setAttribute('rolls.count', rolls);
    span.setAttribute('rolls.result', result.join(','));
    span.setStatus({ code: 1 }); // OK status
    span.end();
    
    // Return result
    res.json({ 
      rolls: result, 
      total: result.reduce((sum, roll) => sum + roll, 0) 
    });
    
  } catch (error) {
    span.setAttribute('error', error.message);
    span.setStatus({ code: 2, message: error.message });
    span.end();
    res.status(500).json({ error: 'Internal server error' });
  }
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

// Start server
const PORT = process.env.PORT || 8080;
app.listen(PORT, () => {
  console.log(`🎲 Dice server running at http://localhost:${PORT}`);
  console.log(`📊 Health check: http://localhost:${PORT}/health`);
  console.log(`🎯 Roll dice: http://localhost:${PORT}/rolldice?rolls=3`);
});
```

> **🎯 Key Features**:
> - **Tracing**: Each request creates a span with detailed attributes
> - **Metrics**: Request counter tracks total API calls
> - **Error Handling**: Proper error status and attributes
> - **Validation**: Input validation with meaningful error messages

### Step 4: Docker and Collector Setup

Create docker-compose.yml:

```yaml
version: '3.8'
services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"
      - "14268:14268"
      - "14250:14250"
    environment:
      - COLLECTOR_OTLP_ENABLED=true
    networks:
      - otel-network
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    command: ["--config=/etc/otel-collector-config.yaml"]
    volumes:
      - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml
    ports:
      - "4317:4317"
      - "4318:4318"
      - "8889:8889"
    depends_on:
      - jaeger
    networks:
      - otel-network
networks:
  otel-network:
    driver: bridge
```

Create otel-collector-config.yaml:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
processors:
  batch:
    timeout: 1s
    send_batch_size: 1024
exporters:
  debug:
    verbosity: detailed
  prometheus:
    endpoint: "0.0.0.0:8889"
  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true
service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp/jaeger, debug]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus, debug]
```

### Step 5: Run the Application

1. **Start Docker services**:
 ```bash
docker-compose up -d
```
2. **Run the app (with Collector)**:
 ```bash
NODE_ENV=production USE_COLLECTOR=true node --require ./instrumentation.js app.js
 ```
3. **Test requests**:
```bash
curl "http://localhost:8080/rolldice?rolls=3"
curl "http://localhost:8080/rolldice?rolls=abc"
 ```

### Step 6: View the Data

- **Jaeger UI**: http://localhost:16686 (select dice-server).
- **Prometheus Metrics**: http://localhost:8889/metrics.
- **Sample Metric Output**:
```text
# HELP requests_count_total Total number of requests
# TYPE requests_count_total counter
requests_count_total{job="dice-server"} 5
```

## 5. OpenTelemetry’s Key Features

- **Multi-Language Support**: Stable for JavaScript, Python, Java, Go (traces and metrics).
- **Automatic Instrumentation**: Works with frameworks like Express or Flask without code changes.
- **Flexibility**: Compatible with 90+ tools (Datadog, Splunk).
- **Use Cases**: Ideal for microservices, cloud, and distributed systems.

## Wrapping Up

OpenTelemetry makes it easy to monitor your systems and troubleshoot issues by collecting telemetry data. It’s open-source, flexible, and a breeze to set up. In 2025, it’s the standard for observability, with cost savings and AI integration making it a game-changer. Get started at opentelemetry.io/docs!

**Resources**:

- [OpenTelemetry Official Documentation](https://opentelemetry.io/docs/) - Comprehensive guides, APIs, SDKs, and best practices
- [IMESH OpenTelemetry Blog](https://imesh.ai/blog/opentelemetry-otel/) - Simplified introduction with microservices focus
- [DataGravity OpenTelemetry Guide](https://www.datagravity.dev/p/what-is-opentelemetry) - Detailed technical explanations and tutorials
- [OpenTelemetry.io](https://opentelemetry.io/) - Official website and community resources
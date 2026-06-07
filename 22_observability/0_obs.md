These are **very important observability topics** for Senior Backend, SDE-2, SDE-3, Staff Engineer, and System Design (HLD) interviews.

A common interview mistake is designing a scalable system but forgetting **how to monitor, debug, and operate it in production**.

---

# What is Observability?

Observability means:

> "How easily can we understand what's happening inside a distributed system?"

When a production issue occurs, observability helps answer:

* Why is the system slow?
* Which service failed?
* Which request caused the failure?
* What changed?
* Which server is overloaded?

The 3 pillars of observability are:

1. Logs
2. Metrics
3. Traces

---

# 1. Three Pillars of Observability

## A. Logs

Logs are event records.

Example:

```text
User 123 logged in
Payment started
Payment failed
Database connection timeout
```

Think of logs as:

> A diary of everything happening in the system.

Example:

```json
{
  "timestamp":"2026-06-07T10:00:00Z",
  "service":"payment-service",
  "userId":"123",
  "status":"FAILED",
  "error":"Card Declined"
}
```

Useful for:

* Debugging
* Auditing
* Error investigation

---

## B. Metrics

Metrics are numerical measurements.

Examples:

```text
CPU Usage = 75%
Memory = 4GB
Requests/sec = 500
Error Rate = 2%
```

Think of metrics as:

> Health vitals of the system.

Useful for:

* Monitoring
* Alerting
* Capacity planning

---

## C. Traces

Trace follows a request across services.

Example:

```text
User Request
   ↓
API Service
   ↓
Order Service
   ↓
Payment Service
   ↓
Database
```

Trace shows:

```text
API Service = 20ms
Order Service = 15ms
Payment Service = 500ms
DB = 480ms
```

Now you know exactly where latency occurred.

Useful for:

* Microservices debugging
* Performance analysis

---

# Interview Answer

> Logs tell me WHAT happened, metrics tell me HOW MUCH happened, and traces tell me WHERE it happened.

---

# 2. Logging Best Practices

Bad logging:

```text
Error occurred
```

Useless.

---

Good logging:

```json
{
  "timestamp":"2026-06-07T10:00:00Z",
  "service":"payment-service",
  "requestId":"abc123",
  "userId":"100",
  "error":"Card Declined"
}
```

---

## Best Practices

### Structured Logs

Use JSON.

Bad:

```text
Payment failed for user 100
```

Good:

```json
{
  "userId":"100",
  "paymentStatus":"FAILED"
}
```

---

### Include Context

Always log:

```text
Request ID
User ID
Service Name
Timestamp
Error Code
```

---

### Use Log Levels

```text
DEBUG
INFO
WARN
ERROR
FATAL
```

Example:

```text
INFO -> User Login
WARN -> Retry Happening
ERROR -> Payment Failed
```

---

### Never Log Sensitive Data

Avoid:

```text
Password
Credit Card
OTP
Token
```

---

# 3. Log Aggregation

In microservices:

```text
API Service
Order Service
Payment Service
Inventory Service
```

Each generates logs.

Finding logs manually is impossible.

---

Log Aggregation means:

> Collect all logs into one central place.

Architecture:

```text
Service Logs
     ↓
Log Agent
     ↓
Kafka
     ↓
Central Log Store
     ↓
Search Dashboard
```

Popular tools:

* Elasticsearch
* Logstash
* Kibana
* Splunk

---

Interview Answer:

> Log aggregation centralizes logs from all services, making debugging and searching production issues easier.

---

# 4. Correlation IDs

One of the most frequently asked observability topics.

---

Imagine:

```text
User places order
```

Request flows through:

```text
API
 ↓
Order Service
 ↓
Payment Service
 ↓
Inventory Service
 ↓
Notification Service
```

Each service creates logs.

How do we connect them?

---

Use a Correlation ID.

Example:

```text
Correlation-ID = XYZ123
```

Passed everywhere:

```text
API Log
[XYZ123]

Order Log
[XYZ123]

Payment Log
[XYZ123]

Inventory Log
[XYZ123]
```

Now searching:

```text
XYZ123
```

shows the entire request journey.

---

Interview Statement:

> Correlation IDs are propagated across services to trace a request end-to-end through logs.

---

# 5. Metrics and Instrumentation

Instrumentation means:

> Adding code to measure important system behavior.

Example:

```java
requestCounter.increment()
```

or

```java
paymentFailureCounter.increment()
```

---

Common Metrics

### Traffic Metrics

```text
Requests/sec
Queries/sec
Messages/sec
```

---

### Latency Metrics

```text
P50
P95
P99
```

Example:

```text
P95 = 300ms
```

Means:

95% requests finish under 300ms.

---

### Error Metrics

```text
5xx Errors
Timeouts
Failures
```

---

### Resource Metrics

```text
CPU
Memory
Disk
Network
```

---

Golden Signals (Google SRE)

Remember:

```text
Latency
Traffic
Errors
Saturation
```

Often asked in interviews.

---

# 6. Alerting and Monitoring

Monitoring:

> Continuously checking system health.

Example:

```text
CPU = 60%
Latency = 100ms
Error Rate = 0.1%
```

---

Alerting:

> Notify engineers when metrics cross thresholds.

Example:

```text
Error Rate > 5%
```

Trigger:

```text
PagerDuty Alert
Slack Alert
Email
SMS
```

---

Good Alert

```text
API Latency > 1s
for 5 mins
```

Bad Alert

```text
Single request > 1s
```

Too noisy.

---

Interview Tip:

Alert on symptoms.

Not infrastructure alone.

Example:

Better:

```text
Checkout Failure Rate > 5%
```

Instead of:

```text
CPU > 80%
```

---

# 7. Dashboards and Runbooks

## Dashboards

Visual view of system health.

Example:

```text
Requests/sec
Error Rate
Latency
CPU
Memory
Queue Size
```

Displayed as graphs.

Popular tools:

* Grafana
* Datadog
* Kibana

---

Dashboard Example

```text
API Gateway

Traffic: 20K RPS
Latency P95: 250ms
Error Rate: 0.5%
CPU: 45%
```

---

## Runbooks

Runbook = Troubleshooting guide.

Example:

```text
If DB CPU > 90%

Step 1:
Check slow queries

Step 2:
Check active connections

Step 3:
Scale read replicas

Step 4:
Contact DBA
```

---

Interview Statement:

> Dashboards help detect issues while runbooks help engineers resolve them quickly.

---

# 8. Distributed Tracing

Critical in microservices.

Example:

```text
User Request
    ↓
API Gateway
    ↓
Order Service
    ↓
Payment Service
    ↓
Inventory Service
```

Trace:

```text
Request ID: ABC123

API Gateway = 20ms
Order Service = 30ms
Payment Service = 500ms
Inventory = 40ms
```

Immediately reveals:

```text
Payment Service is slow
```

---

Concepts

### Trace

Entire request journey.

### Span

One operation.

Example:

```text
Trace
 ├ Span(API)
 ├ Span(Order)
 ├ Span(Payment)
 └ Span(DB)
```

---

Popular Tools

* Jaeger
* Zipkin
* OpenTelemetry

---

# 9. SLOs, SLIs, SLAs

Very common in senior interviews.

---

## SLI (Service Level Indicator)

Measurement.

Example:

```text
Availability = 99.95%
```

or

```text
P95 Latency = 200ms
```

Metric being measured.

---

## SLO (Service Level Objective)

Target.

Example:

```text
99.9% Availability
```

or

```text
95% requests < 200ms
```

Desired goal.

---

## SLA (Service Level Agreement)

Contract with customers.

Example:

```text
99.9% uptime guaranteed
```

If violated:

```text
Refund
Penalty
Service Credits
```

---

Relationship

```text
SLI → Measure

SLO → Internal Target

SLA → Customer Contract
```

Example:

```text
SLI:
Availability = 99.95%

SLO:
Availability >= 99.9%

SLA:
Availability >= 99.5%
```

---

# How to Mention Observability in an HLD Interview

After designing the system, always add:

```text
Observability Layer

Logs
  ↓
Centralized Logging

Metrics
  ↓
Prometheus/Grafana

Tracing
  ↓
Jaeger/OpenTelemetry

Alerts
  ↓
PagerDuty

Dashboards
  ↓
Grafana

Runbooks
  ↓
Incident Response
```

A strong closing statement:

> "To operate the system reliably, I would implement structured logging with correlation IDs, metrics instrumentation for latency/error tracking, distributed tracing across services, dashboards for visibility, alerts on key SLO violations, and runbooks for incident handling."

That answer usually elevates an HLD discussion from a functional design to a production-ready design.



## Prometheus and Grafana (Most Important Monitoring Stack for HLD)

In interviews, you'll often hear:

> "We monitor the system using Prometheus and Grafana."

Many candidates say this without understanding what each tool actually does.

---

# Simple Analogy

Imagine a hospital.

### Prometheus = Nurse

The nurse periodically checks:

* Heart rate
* Blood pressure
* Oxygen level

and records them.

### Grafana = Monitor Screen

The monitor displays:

* Heart rate graph
* Blood pressure graph
* Alerts

---

In distributed systems:

### Prometheus

Collects metrics from services.

### Grafana

Visualizes those metrics.

---

# What is Prometheus?

Prometheus is an open-source monitoring system.

Its main job is:

> Collect, store, and query metrics.

Example metrics:

```text
CPU Usage
Memory Usage
Request Count
Error Count
Latency
Queue Length
```

---

## How Prometheus Works

Suppose you have:

```text
API Service
Payment Service
Order Service
```

Each service exposes metrics.

Example:

```text
/api/metrics

requests_total 1000
errors_total 20
cpu_usage 75
```

Prometheus periodically "scrapes" them.

```text
Every 15 seconds

Prometheus
     ↓
API Service

Prometheus
     ↓
Order Service

Prometheus
     ↓
Payment Service
```

Prometheus pulls the data and stores it.

---

# Prometheus Architecture

```text
Services
   ↓
Metrics Endpoint
   ↓
Prometheus
   ↓
Time Series Database
```

Prometheus stores data as:

```text
Timestamp → Value
```

Example:

```text
10:00 → CPU = 70%
10:01 → CPU = 75%
10:02 → CPU = 80%
```

This is called a **time-series database**.

---

# Instrumentation

Your application publishes metrics.

Example:

```java
requestCounter.increment();
```

or

```java
paymentFailureCounter.increment();
```

Prometheus collects these values.

Common libraries:

* Micrometer (Java)
* Prometheus Client SDKs
* OpenTelemetry

---

# Common Prometheus Metrics

### Counter

Only increases.

```text
Total Requests
Total Errors
```

Example:

```text
requests_total = 1000
```

---

### Gauge

Can go up or down.

```text
CPU Usage
Memory Usage
Queue Size
```

Example:

```text
cpu_usage = 72%
```

---

### Histogram

Measures latency distributions.

```text
Request Latency
```

Useful for:

```text
P50
P95
P99
```

---

# Example HLD Metrics

For BookMyShow:

```text
tickets_booked_total

payment_failures_total

seat_lock_count

booking_latency_ms
```

For WhatsApp:

```text
messages_sent_total

message_delivery_latency

active_users
```

---

# Alerting with Prometheus

Prometheus can trigger alerts.

Example:

```text
Error Rate > 5%
for 5 minutes
```

Action:

```text
Send Slack Alert
Send Email
Trigger PagerDuty
```

---

# What is Grafana?

Grafana is a visualization and dashboard tool.

It doesn't collect metrics itself.

It reads metrics from Prometheus and displays them.

Think:

```text
Prometheus = Database

Grafana = UI
```

---

# Grafana Dashboard

Example dashboard:

```text
API Gateway

Requests/sec : 10,000
Error Rate   : 0.2%
P95 Latency  : 200ms
CPU Usage    : 65%
```

Displayed as graphs.

```text
CPU
│
│      /\
│     /  \
│____/____\____
```

---

# Typical Production Architecture

```text
Application Services
        ↓
     Metrics
        ↓
    Prometheus
        ↓
     Grafana
        ↓
 Engineers
```

---

# Prometheus + Grafana in Microservices

```text
API Service
Order Service
Payment Service
Inventory Service
```

Each exposes:

```text
/metrics
```

Prometheus collects:

```text
Request Count
Error Rate
Latency
CPU
Memory
```

Grafana shows:

```text
Overall System Health
```

on a single dashboard.

---

# Prometheus vs Grafana

| Feature          | Prometheus | Grafana              |
| ---------------- | ---------- | -------------------- |
| Collect metrics  | ✅          | ❌                    |
| Store metrics    | ✅          | ❌                    |
| Query metrics    | ✅          | ❌                    |
| Visualize graphs | ❌          | ✅                    |
| Dashboards       | ❌          | ✅                    |
| Alerts           | ✅          | ✅ (via integrations) |

---

# Typical Interview Answer

If asked:

**"How would you monitor this system?"**

A strong answer is:

> "I would instrument services to expose metrics such as request count, latency, error rate, CPU, memory, and queue depth. Prometheus would scrape and store these metrics in a time-series database. Grafana dashboards would provide visibility into system health, and alerts would be configured on critical SLOs such as error rate, latency, and availability."

---

# One-Line Memory Trick

```text
Logs      → What happened?
Metrics   → How much happened?
Traces    → Where did it happen?

Prometheus → Collects Metrics
Grafana    → Visualizes Metrics
```

This is usually enough observability knowledge for most SDE-2/SDE-3 HLD interviews, and interviewers often appreciate when you mention **Prometheus + Grafana + OpenTelemetry + Correlation IDs + SLOs** together as a complete monitoring strategy.

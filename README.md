# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

----------------

# The service being monitored is:

The application being monitored is a `payment-service`, representing a small payment processing component in a web service or e-commerce platform. It accepts payment requests and returns a result to callers, while also emitting operational telemetry such as request latency, CPU usage, memory usage, and log messages.

## Operational problem being addressed

The service experiences multiple failures in which payment requests slow dramatically or time out. This is the kind of operational issue an AIOps workflow is meant to identify early before it impacts more users or downstream systems.

## Purpose of AIOps in this assessment

AIOps is used here to detect abnormal behaviour from operational data, correlate metric spikes with log events, and surface likely service incidents for investigation. The goal is to show how automated monitoring can convert noisy operational signals into actionable alerts.

## Major component purposes

- `src/aiops_pipeline.py`: It loads the operational dataset, runs each record through the anomaly detector, and publishes/consumes events through the in-memory event pipeline.
- `src/anomaly_detector.py`: It applies rule-based thresholds to detect abnormal service behaviour based on response time, CPU, memory, and log severity.
- `src/event_topic.py`: This stores the in-memory topic used for event streaming.
- `src/event_producer.py`: It publishes detected anomaly events into the topic.
- `src/event_consumer.py`: It retrieves the published anomaly events for downstream processing.

## Part 2: Prepare and Inspect Operational Data

### 1. Fields that represent metrics

The metric fields are:

- `response_time_ms`: time taken to process a payment request, measured in milliseconds
- `cpu_percent`: percentage of CPU utilization
- `memory_percent`: percentage of memory utilization

These values are numeric measurements that describe system health and performance.

### 2. Fields that represent log information

The log-related fields are:

- `log_level`: severity such as `INFO` or `ERROR`
- `message`: human-readable log message describing the event

These records provide context on what happened operationally and help explain what caused the performance problem.

### 3. How timestamps are used

The `timestamp` field uses ISO 8601 format, for example `2026-09-20T10:05:00`. Each record is associated with a single point in time, and the sequence of timestamps shows the service state over a short one-minute interval. This enables detection of when normal behaviour transitions into degradation and recovery.

# 4. Normal behaviour

The following observations appear normal:

- `2026-09-20T10:00:00` through `2026-09-20T10:04:00`
- `2026-09-20T10:07:00` through `2026-09-20T10:09:00`

These records show:

- `response_time_ms` between about 120 and 145 ms
- `cpu_percent` between roughly 42 and 50%
- `memory_percent` between roughly 51 and 57%
- messages such as `Payment request processed successfully`

This pattern indicates steady and healthy request processing without major resource strain.

# 5. Unusual behaviour

The unusual observations occur at:

- `2026-09-20T10:05:00`
- `2026-09-20T10:06:00`

These records show clear signs of abnormal service behaviour:

- `response_time_ms` jumps to 610 ms and then 640 ms
- `cpu_percent` rises to 75% and then 94%
- `memory_percent` rises to 70% and then 91%

This is consistent with a service degradation event caused by high resource consumption and database connectivity issues, which is exactly the sort of problem that AIOps aims to detect and flag.
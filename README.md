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

The service experiences multiple failures in which payment requests slow dramatically or time out. The synthetic data shows that during a short period, latency spikes from roughly 120-150 ms to over 600 ms, while CPU and memory rise sharply, and the service emits error logs indicating timeout conditions. This is the kind of operational issue an AIOps workflow is meant to identify early before it impacts more users or downstream systems.

## Purpose of AIOps in this assessment

AIOps is used here to detect abnormal behaviour from operational data, correlate metric spikes with log events, and surface likely service incidents for investigation. In this assessment, the pipeline ingests service telemetry, checks it against anomaly thresholds, and produces event records that capture the time, service, and reasons for each incident. The goal is to show how automated monitoring can convert noisy operational signals into actionable alerts.

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


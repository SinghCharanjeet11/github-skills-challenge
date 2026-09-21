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

## AIOps scenario summary

This repository simulates a lightweight AIOps workflow for a payment-processing service. The service emits operational telemetry while it handles requests. The goal is to observe, classify, and alert on abnormal behaviour using a simple event-stream pipeline before the issue escalates to customers or upstream systems.

## Description of the operational data

The operational data in `data/service_data.json` contains a sequence of records. Each record includes:

- `timestamp`: ISO 8601 timestamp for the observation time
- `service`: the service name (`payment-service`)
- `response_time_ms`: request latency in milliseconds
- `cpu_percent`: CPU utilization percentage
- `memory_percent`: memory utilization percentage
- `log_level`: severity such as `INFO` or `ERROR`
- `message`: log text describing the outcome of the request

The data represents a short operational window in which the system behaves normally before a degradation event begins.

## Observations from the logs and metrics

Normal behaviour is characterized by:

- response times between roughly 120 and 145 ms
- CPU use around 42-50%
- memory use around 51-57%
- `INFO` log messages stating `Payment request processed successfully`

Unusual behaviour is characterized by:

- response times of 610 ms and 640 ms
- CPU use of 75% and 94%
- memory use of 70% and 91%
- `ERROR` level messages including `Payment service timeout` and `Database connection timeout`

This pattern indicates a short but serious degradation event caused by a resource-intensive failure and database timeout conditions.

## Anomaly-detection findings

Using the provided `AnomalyDetector` in `src/anomaly_detector.py`, the service is flagged as anomalous when metrics exceed their thresholds. The detector identifies:

1. `2026-09-20T10:05:00` as anomalous because the response time exceeds the latency threshold.
2. `2026-09-20T10:06:00` as anomalous because response time, CPU, and memory each exceed their thresholds.

The detector produces a readable structured anomaly event that includes the timestamp, service name, type, and reasons. This exposes why the observation was flagged.


## Final workflow execution result

The final workflow execution was run with:

```bash
cd /workspaces/github-skills-challenge && python src/aiops_pipeline.py
```

The resulting output was:

```text
==================================================
AIOps Pipeline Result
==================================================
Records processed: 10
Anomalies detected: 2
Events consumed: 2

Detected Events:

Service: payment-service
Timestamp: 2026-09-20T10:05:00
Type: ANOMALY
Reasons: High response time

Service: payment-service
Timestamp: 2026-09-20T10:06:00
Type: ANOMALY
Reasons: High response time, High CPU utilization, High memory utilization
```

This confirms that the operational data was processed, the anomaly detection identified the failure window, the event reached the topic and was consumed, and the final downstream output described the detected operational issue.

## Issues identified and corrected

Two issues were found and corrected while preserving the original architecture:

1. Topic mismatch in the pipeline
   - Affected component: `src/aiops_pipeline.py`
   - Cause: the producer was publishing to `service-events` while the consumer was configured to read from `anomaly-events`.
   - Correction: both sides were connected to the same in-memory topic.
   - Result: anomaly events were successfully consumed.

2. Import compatibility issue
   - Affected components: `src/aiops_pipeline.py`, `src/event_producer.py`, and `src/event_consumer.py`
   - Cause: module imports were incompatible with package-style execution and direct script execution.
   - Correction: fallback imports were added so the workflow works in both contexts.
   - Result: the pipeline runs successfully without replacing the existing implementation.

## Limitation and possible improvement

A limitation of the current approach is that the detector is rule-based and uses fixed thresholds. This works well for obvious spikes, but it may miss gradual degradation or unusual incidents that do not exceed the thresholds immediately. A possible improvement would be to add historical baselines or adaptive thresholds so the system can detect both sharp anomalies and slow-moving performance drift.

## Reproduction steps

To reproduce the demonstration on a fresh clone or working copy:

1. Open a terminal in the project root.
2. Run the workflow:

```bash
cd /workspaces/github-skills-challenge && python src/aiops_pipeline.py
```

3. Review the output for the number of records processed, the detected anomalies, and the final reasons logged for each anomaly.
4. Optionally run the test suite:

```bash
cd /workspaces/github-skills-challenge && python -m pytest -q
```

Expected result: the workflow completes successfully and the repository tests pass.

## Final note

This repository demonstrates a minimal but realistic AIOps pattern: collect operational telemetry, detect abnormal conditions, convert them into event records, and surface the issue in a readable downstream output. The corrected workflow is working within the original assessment architecture and does not replace the provided components with a different implementation.
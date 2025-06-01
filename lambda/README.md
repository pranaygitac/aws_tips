
# AWS Lambda Optimization & Behavior Guide

This document summarizes key concepts, optimization strategies, and features of AWS Lambda based on our discussion.

---

## ✅ Lambda Optimization Techniques

### 1. **Optimize External Calls**
- Use HTTP connection pooling (e.g., `requests.Session()` in Python)
- Minimize latency to APIs (use regional endpoints)
- Cache results when possible

### 2. **Use AWS SDK Efficiently**
- Use latest AWS SDK (`boto3`) with resource reuse
- Load clients outside the handler to benefit from execution context reuse

### 3. **Lambda PowerTools (Python)**
- Logging, Tracing, and Metrics utility library
- Compatible with AWS X-Ray and CloudWatch
- Great for structured logging and observability

### 4. **Optimize with ElastiCache**
- Cache DB results in Redis (ElastiCache) to reduce RDS/MySQL load
- TTLs for freshness; use as read-through or write-through cache

### 5. **Execution Context Reuse**
- Reuse DB connections, Redis clients, boto3 clients across invocations
- Minimizes cold start overhead

---

## 📊 Lambda Metrics for Tuning

Use CloudWatch metrics:
- `Duration`: Average execution time
- `Throttles`: Reached concurrency limit
- `Errors`: Failed executions
- `ConcurrentExecutions`: Active concurrent instances
- `Invocations`: Number of executions
- `IteratorAge`: For streams like DynamoDB/Kinesis

Use these to:
- Identify bottlenecks
- Justify Provisioned Concurrency
- Trigger alarms on failure spikes

---

## 🔁 Asynchronous Invocation & DLQs

- Lambda retries async events 3 times (total)
- After retries fail, event is sent to **DLQ** (SQS/SNS)
- Lambda does **not** retry DLQ events — you must reprocess them manually

---

## 🚀 Scaling Behavior

### Scaling by Trigger Type

| Trigger Type           | Scaling Method                             |
|------------------------|--------------------------------------------|
| API Gateway / ALB      | Scales instantly with requests             |
| S3 / SNS / EventBridge | Queued events, Lambda pulls and scales     |
| SQS / DynamoDB / Kinesis | Scales with batch size and partitions    |

### Concurrency

- Initial burst: 1000 concurrent executions (default)
- After that: +500/min until account limit
- Reserved Concurrency: Guaranteed for a function
- Provisioned Concurrency: Pre-warmed instances, no cold starts

---

## 🕵️ X-Ray vs Lambda PowerTools

| Feature      | X-Ray                        | PowerTools                        |
|--------------|------------------------------|-----------------------------------|
| Tracing      | ✅ Distributed tracing        | ✅ Structured tracing              |
| Logging      | ❌ Basic                      | ✅ Structured + context-aware     |
| Metrics      | ❌ Basic                      | ✅ Embedded custom metrics         |
| Use Case     | Visual trace of request flow | Structured logs + app diagnostics |

---

## 📦 lambda_handler() Explained

In Python:
```python
def lambda_handler(event, context):
    # Your function logic here
```
- `event`: Input data to your function (JSON/dict)
- `context`: Metadata (function name, memory, timeout, etc.)

---

## 📚 Useful AWS CLI Commands

**Set Reserved Concurrency:**
```bash
aws lambda put-function-concurrency   --function-name myFunction   --reserved-concurrent-executions 50
```

**Enable Provisioned Concurrency:**
```bash
aws lambda put-provisioned-concurrency-config   --function-name myFunction   --qualifier 1   --provisioned-concurrent-executions 10
```

---

## 📈 Monitor with CloudWatch

Set alarms for:
- Throttles > 0
- Errors > threshold
- DLQ message count > 0

---

## 📬 DLQ Processing Strategy

1. Attach SQS queue as DLQ
2. Set up another Lambda to poll the queue
3. Retry original invocation or alert human operator

---

## 🖼️ Lambda Scaling Diagram
![Lambda Scaling](./lambda_scaling_diagram.png)

---

## 🧩 Additional Notes

- Always set timeouts < dependent system timeouts (e.g., API, DB)
- Avoid blocking calls; prefer async where possible
- Avoid large dependencies (keep deployment package light)
- Keep functions single-purpose

---

_Last updated: 2025-06-01_

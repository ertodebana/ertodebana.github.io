---
title: "LLM Operations and MLOps Pipeline Design"
date: 2026-03-27 10:00:00 +0300
lang: en
locale: en-US
page_id: llmops-pipeline
permalink: /posts/llm-operations-and-mlops-pipeline-design/
categories: [AI, MLOps]
tags: [llm, mlops, ai, machine-learning, llmops, kubernetes]
description: "A practical view of what actually gets hard when you move LLMs from demo mode into production."
---

## Moving LLMs Into Production

Building an impressive LLM demo is easy. Running one without blowing up latency, cost, or product expectations is the part that humbles teams very quickly. Most of the pain shows up in operations long before it shows up in model quality.

Building an LLM demo and operating an LLM service in production are very different jobs. In production you need reliability, observability, cost control, model lifecycle management, and prompt governance all at once.

## A Practical LLMOps Architecture

```
+------------------+     +------------------+     +------------------+
|   Data Pipeline  | --> | Model Training/  | --> |   Model Serving  |
|                  |     | Fine-tuning      |     |                  |
| - Data Collection|     | - Training       |     | - API Gateway    |
| - Preprocessing  |     | - Evaluation     |     | - Load Balancer  |
| - Versioning     |     | - Registry       |     | - Auto-scaling   |
+------------------+     +------------------+     +------------------+
         |                        |                        |
         v                        v                        v
+------------------------------------------------------------------+
|                    Observability & Monitoring                     |
|  Latency | Token Usage | Cost | Quality | Drift Detection         |
+------------------------------------------------------------------+
```

## High-Performance Serving with vLLM

[vLLM](https://github.com/vllm-project/vllm) has become one of the most common ways to serve open-weight LLMs efficiently:

```python
from vllm import LLM, SamplingParams

# Load the model
llm = LLM(
    model="meta-llama/Llama-3-8B-Instruct",
    tensor_parallel_size=2,  # 2 GPUs
    gpu_memory_utilization=0.9,
    max_model_len=4096,
)

# Inference
sampling_params = SamplingParams(
    temperature=0.7,
    top_p=0.9,
    max_tokens=512,
)

outputs = llm.generate(
    ["What is Kubernetes and why is it used?"],
    sampling_params,
)
```

## Deploying LLM Serving on Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: llm-serving
spec:
  replicas: 2
  selector:
    matchLabels:
      app: llm-serving
  template:
    metadata:
      labels:
        app: llm-serving
    spec:
      containers:
        - name: vllm
          image: vllm/vllm-openai:latest
          args:
            - "--model"
            - "meta-llama/Llama-3-8B-Instruct"
            - "--tensor-parallel-size"
            - "1"
          resources:
            limits:
              nvidia.com/gpu: 1
              memory: "32Gi"
            requests:
              nvidia.com/gpu: 1
              memory: "24Gi"
          ports:
            - containerPort: 8000
      nodeSelector:
        gpu-type: a100
      tolerations:
        - key: nvidia.com/gpu
          operator: Exists
          effect: NoSchedule
---
apiVersion: v1
kind: Service
metadata:
  name: llm-service
spec:
  selector:
    app: llm-serving
  ports:
    - port: 80
      targetPort: 8000
  type: ClusterIP
```

## Monitoring and Observability

These are some of the most useful production metrics for LLM serving:

| Metric | Meaning | Target |
|--------|---------|--------|
| **TTFT** | Time to First Token | < 500ms |
| **TPS** | Tokens Per Second | > 30 |
| **P99 Latency** | 99th percentile latency | < 5s |
| **GPU Utilization** | GPU usage | 70%-90% |
| **Cost per Token** | Cost efficiency | Minimize |

### Collecting metrics with Prometheus

```yaml
# prometheus-rules.yaml
groups:
  - name: llm-serving
    rules:
      - alert: HighLatency
        expr: histogram_quantile(0.99, rate(llm_request_duration_seconds_bucket[5m])) > 5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "LLM serving latency is high"

      - alert: LowThroughput
        expr: rate(llm_tokens_generated_total[5m]) < 10
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "LLM throughput is critically low"
```

## Prompt Management Matters Too

Prompt changes should be versioned just like code and models:

```python
# prompt_registry.py
from dataclasses import dataclass
from typing import Optional
import hashlib

@dataclass
class PromptTemplate:
    name: str
    version: str
    template: str
    model: str
    max_tokens: int
    temperature: float

    @property
    def hash(self) -> str:
        return hashlib.sha256(self.template.encode()).hexdigest()[:8]

# Prompt versioning
PROMPTS = {
    "summarize_v2": PromptTemplate(
        name="summarize",
        version="2.0",
        template="""You are a summarization assistant.
Summarize the following text in 3 sentences:

{text}

Summary:""",
        model="llama-3-8b-instruct",
        max_tokens=256,
        temperature=0.3,
    ),
}
```

## A Few Rules That Actually Help

1. **Version models explicitly** and track every release.
2. **Roll out gradually** with A/B tests or canaries.
3. **Enforce rate limits** and quota controls.
4. **Cache repeated requests** where it is safe.
5. **Define fallback behavior** for degraded service states.
6. **Track spend per token** so cost becomes operationally visible.

## Conclusion

In most teams, LLM operations become difficult because of cost, latency, and expectation management before model quality is even the real bottleneck. That is why "we have a good prompt" stops being a serious strategy almost immediately.

With the right pipeline, observability, and cost controls, the complexity is manageable. Without them, even a strong model quickly turns into an expensive experiment.

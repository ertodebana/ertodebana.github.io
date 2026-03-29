---
title: "LLM Operasyonlari ve MLOps Pipeline Tasarimi"
date: 2026-03-27 10:00:00 +0300
categories: [AI, MLOps]
tags: [llm, mlops, ai, machine-learning, llmops, kubernetes]
description: "Buyuk dil modellerini production'a tasimak icin MLOps pipeline tasarimi ve best practice'ler."
---

## LLM'leri Production'a Tasimak

Large Language Model'leri (LLM) gelistirmek bir sey, onlari **production ortaminda guvenilir ve olceklenebilir** sekilde calistirmak tamamen baska bir seydir. Bu yazida, LLM operasyonlari icin gereken MLOps pipeline'ini inceliyoruz.

## LLMOps Pipeline Mimarisi

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
|                    Observability & Monitoring                      |
|  Latency | Token Usage | Cost | Quality | Drift Detection        |
+------------------------------------------------------------------+
```

## Model Serving: vLLM ile Yuksek Performans

[vLLM](https://github.com/vllm-project/vllm), LLM inference icin optimize edilmis bir serving framework'udur:

```python
from vllm import LLM, SamplingParams

# Model yukleme
llm = LLM(
    model="meta-llama/Llama-3-8B-Instruct",
    tensor_parallel_size=2,  # 2 GPU
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
    ["Kubernetes nedir ve neden kullanilir?"],
    sampling_params,
)
```

## Kubernetes Uzerinde LLM Deployment

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

## Monitoring ve Observability

LLM serving icin izlenmesi gereken temel metrikler:

| Metrik | Aciklama | Hedef |
|--------|----------|-------|
| **TTFT** | Time to First Token | < 500ms |
| **TPS** | Tokens Per Second | > 30 |
| **P99 Latency** | 99th percentile latency | < 5s |
| **GPU Utilization** | GPU kullanim orani | 70-90% |
| **Cost per Token** | Token basina maliyet | Minimize |

### Prometheus ile Metrik Toplama

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

## Prompt Management

Production'da prompt'lari yonetmek icin bir sistem gerekir:

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

# Prompt versiyonlama
PROMPTS = {
    "summarize_v2": PromptTemplate(
        name="summarize",
        version="2.0",
        template="""Sen bir ozetleme asistanisin.
Asagidaki metni 3 cumle ile ozetle:

{text}

Ozet:""",
        model="llama-3-8b-instruct",
        max_tokens=256,
        temperature=0.3,
    ),
}
```

## Best Practices

1. **Model versiyonlama**: Her model degisikligini izleyin
2. **A/B testing**: Yeni model versiyonlarini kademeli olarak deploy edin
3. **Rate limiting**: Token kullanim limitleri koyun
4. **Caching**: Tekrar eden sorgulari cache'leyin
5. **Fallback**: Model unavailable oldugunda fallback stratejisi
6. **Cost tracking**: Token bazli maliyet takibi yapin

## Sonuc

LLM'leri production'a tasimak, geleneksel ML model'lerinden cok daha karmasik bir surectir. Dogru MLOps pipeline'i, monitoring ve cost management stratejileri ile bu sureci yonetilebilir hale getirebilirsiniz.

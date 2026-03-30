---
title: "LLM Operasyonları ve MLOps Pipeline Tasarımı"
date: 2026-03-27 10:00:00 +0300
lang: tr
locale: tr-TR
page_id: llmops-pipeline
permalink: /posts/llm-operasyonlari-ve-mlops-pipeline/
categories: [AI, MLOps]
tags: [llm, mlops, ai, machine-learning, llmops, kubernetes]
description: "LLM'leri demo seviyesinden production seviyesine taşırken nerede gerçekten zorlanıldığını ve neyin işe yaradığını özetliyorum."
---

## LLM'leri Production'a Taşımak

Büyük dil modelleriyle etkileyici bir demo yapmak kolay; onu bütçeyi yakmadan, gecikmeyi patlatmadan ve kaliteyi kaybetmeden çalıştırmak zor kısım. Bu yüzden LLMOps tarafında asıl mücadele modelden çok operasyon yüzeyinde yaşanır.

Büyük dil modelleriyle prototip çıkarmak ile onları **güvenilir, gözlemlenebilir ve maliyet kontrollü** biçimde üretime almak aynı şey değildir. LLMOps tarafında model yaşam döngüsü, serving altyapısı, prompt yönetimi ve maliyet optimizasyonu birlikte ele alınmalıdır.

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
|                    Observability & Monitoring                     |
|  Latency | Token Usage | Cost | Quality | Drift Detection         |
+------------------------------------------------------------------+
```

## Model Serving: vLLM ile Yüksek Performans

[vLLM](https://github.com/vllm-project/vllm), LLM inference için optimize edilmiş popüler bir serving framework'üdür:

```python
from vllm import LLM, SamplingParams

# Model yükleme
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
    ["Kubernetes nedir ve neden kullanılır?"],
    sampling_params,
)
```

## Kubernetes Üzerinde LLM Deployment

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

LLM serving için izlenmesi gereken temel metrikler:

| Metrik | Açıklama | Hedef |
|--------|----------|-------|
| **TTFT** | Time to First Token | < 500ms |
| **TPS** | Tokens Per Second | > 30 |
| **P99 Latency** | 99. yüzdelik gecikme | < 5s |
| **GPU Utilization** | GPU kullanım oranı | %70-%90 |
| **Cost per Token** | Token başına maliyet | Minimize |

### Prometheus ile metrik toplama

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

## Prompt Yönetimi

Production ortamında prompt'ları da versiyonlamak gerekir:

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
        template="""Sen bir özetleme asistanısın.
Aşağıdaki metni 3 cümle ile özetle:

{text}

Özet:""",
        model="llama-3-8b-instruct",
        max_tokens=256,
        temperature=0.3,
    ),
}
```

## Sahada işe yarayan birkaç kural

1. **Model versiyonlama** yapın ve her değişikliği izleyin.
2. **A/B testing** ile yeni modelleri kademeli devreye alın.
3. **Rate limiting** ve kota yönetimi ekleyin.
4. **Caching** ile tekrar eden sorguların maliyetini düşürün.
5. **Fallback stratejileri** tanımlayın.
6. **Maliyet takibini** token bazında görünür kılın.

## Sonuç

LLM operasyonları çoğu ekipte model kalitesinden önce maliyet, gecikme ve ürün beklentileri yüzünden zorlaşır. Bu yüzden "iyi prompt yazdık, bitti" çizgisi production gerçekliğinde çok kısa sürer.

Doğru pipeline, iyi gözlemleme ve sert maliyet disiplini olmadan LLM servisi hızla pahalı bir deneye dönüşür. Bunlar yerindeyse, karmaşıklık yönetilebilir olur; değilse en iyi model bile sizi kurtarmaz.

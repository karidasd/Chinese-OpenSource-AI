<div align="center">

# 🐉 The Chinese Open-Source AI Renaissance
**An Architectural Guide to the Models Beating Llama-3**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://GitHub.com/karidasd/Chinese-OpenSource-AI/graphs/commit-activity)

*A definitive technical breakdown of the Eastern AI ecosystem. Why models from Alibaba, DeepSeek, and 01.AI are dominating global benchmarks, how their architectures fundamentally reduce inference costs, and how to deploy them locally.*

</div>

---

## 🛑 The Paradigm Shift

For years, the Open-Source AI narrative was dominated by Silicon Valley (Meta's Llama, Mistral, Databricks). In 2024, the paradigm violently shifted East. Chinese open-weights models are currently achieving State-of-the-Art (SOTA) performance across global benchmarks, frequently surpassing proprietary models like GPT-4o and Claude 3.5 Sonnet in coding, mathematics, and long-context retrieval, while operating at a fraction of the parameter cost.

This repository serves as a Senior Architect's guide to understanding, deploying, and leveraging these models.

---

## 🏛️ The "Four Dragons" (Leading Ecosystems)

### 1. Alibaba Cloud (Qwen)
The **Qwen-2** series (Qwen2-72B-Instruct) is widely considered the undisputed king of Open-Source LLMs as of mid-2024.
- **Strengths:** Peerless multilingual capabilities (29 languages natively supported), exceptional coding proficiency (surpassing Llama-3 70B), and robust vision-language models (Qwen-VL).
- **Architecture:** Dense transformer architecture with SwiGLU activation, RoPE, and Grouped-Query Attention (GQA).

### 2. DeepSeek (DeepSeek-V2 & DeepSeek-Coder)
DeepSeek shocked the industry by training a 236B parameter MoE model that costs incredibly little to train and serve.
- **Strengths:** Coding, Math, and unbelievable cost-efficiency.
- **Architecture (The Secret Sauce):** 
  - **MLA (Multi-Head Latent Attention):** They compressed the KV Cache into a latent vector, reducing memory overhead during inference by 90% compared to standard MHA. This makes serving massive models incredibly cheap.
  - **DeepSeekMoE:** Advanced sparse routing. Out of 236B parameters, only 21B are active during a forward pass.

### 3. 01.AI (Yi Series)
Founded by Kai-Fu Lee, the **Yi-1.5** series focuses on massive context windows and pristine data quality.
- **Strengths:** 200K+ context windows. Extremely high quality pre-training data resulting in powerful reasoning in smaller 34B form factors.

### 4. Zhipu AI (GLM-4)
The General Language Model (GLM) series is a powerhouse originating from Tsinghua University.
- **Strengths:** GLM-4 is highly optimized for complex instruction following and tool-use (Function Calling/Agentic Workflows).

---

## 📊 Benchmarks vs The West

*(Benchmarks comparing Top Chinese Open-Weights against Western Counterparts)*

| Model | Parameters (Active) | Context Length | MMLU | HumanEval (Code) | GSM8K (Math) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Qwen2-72B-Instruct** | 72B | 128K | **84.2** | **86.0** | **91.1** |
| **Llama-3-70B-Instruct** | 70B | 8K | 82.0 | 81.7 | 93.0 |
| **DeepSeek-V2** | 21B | 128K | 78.5 | 81.1 | 88.2 |
| **Mistral Large** | Dense | 32K | 81.2 | 81.0 | 91.2 |

> *Observation:* Qwen2-72B matches or beats Llama-3 70B across almost all reasoning and coding metrics, while DeepSeek-V2 achieves comparable performance while activating only 21B parameters.

---

## 🚀 Deployment & Serving Guide

How to deploy these models locally or in your private cloud, bypassing OpenAI APIs.

### Option 1: High-Throughput Serving (vLLM)
For production environments, **vLLM** is required to utilize PagedAttention. It fully supports Qwen2 and DeepSeek.

```bash
# Install vLLM
pip install vllm

# Spin up a fast OpenAI-compatible server for Qwen2
python -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen2-72B-Instruct \
    --tensor-parallel-size 4 \
    --trust-remote-code
```

### Option 2: Local Development (Ollama)
For local prototyping on MacBooks or consumer GPUs (automatically applies 4-bit quantization).

```bash
# Pull and run the 7B Qwen2 model
ollama run qwen2:7b

# Or DeepSeek-Coder for your local IDE
ollama run deepseek-coder-v2
```

### Option 3: Python (Transformers)
Native integration for custom pipelines.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
device = "cuda"

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2-7B-Instruct",
    torch_dtype="auto",
    device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2-7B-Instruct")

prompt = "Write a highly optimized quicksort algorithm in Python."
messages = [
    {"role": "system", "content": "You are a Principal AI Engineer."},
    {"role": "user", "content": prompt}
]
text = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
model_inputs = tokenizer([text], return_tensors="pt").to(device)

generated_ids = model.generate(model_inputs.input_ids, max_new_tokens=512)
response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]
print(response)
```

---

## 🧠 Architectural Deep-Dive: Why are they so efficient?

To understand why these models are disruptive, we must look at the **KV Cache bottleneck**.

In standard autoregressive LLMs (like Llama-2), caching Key/Value pairs during generation consumes massive amounts of VRAM. A 100K context window can easily consume 40GB of VRAM *just for the cache*, entirely independent of the model weights. 

**DeepSeek's Solution (MLA):** DeepSeek-V2 uses Multi-Head Latent Attention. Instead of caching large Key and Value matrices separately for every head, it compresses them into a single, low-dimensional latent vector `c_t`. During inference, it "decompresses" this vector on the fly using learned projection matrices. 
> *Result:* A 90% reduction in KV Cache footprint. You can serve massive batch sizes of DeepSeek-V2 on hardware that would OOM instantly with Llama-3.

---

<p align="center">
  <i>Curated by <a href="https://github.com/karidasd">Dimitris Karydas</a></i>
</p>

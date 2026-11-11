# Why and How You Create Small Fine-Tuned Models for Your Community

The goal of this repo is to help you understand why you might want to create small fine-tuned models for your community. And if that appeals to you, this should help walk you through creating and serving the model.

This guide is for anyone who is AI knowledgeable but not an AI expert. We will stick with Python and PyTorch on Linux throughout.

For now, we will drive this through examples that are of interest to the author. Over time we hope to bring in a broader array of community focused examples.

Our current possible examples are models tuned for:
1. Blog and abstract writing that actually uses words, style, and tone
2. OCR-ing my particular handwriting
3. ADHD assistance with Reachy-mini robot
4. Expert in restoration ecology for the southern Santa Lucia Mountains of California.
5. House building expert for newbies

These are used as illustrative examples to give projects different ideas about use cases, model types, and data types.

## Before you start
Fine-tuning is not always the answer. These pages cover the decisions worth making before you touch a GPU:

- Do you even need to fine-tune? - the tradeoffs between prompt engineering, RAG, and fine-tuning, plus the other levers that improve a model's accuracy (tool use, constrained decoding, inference-time tricks)
- What does "small" mean? - the tradeoffs you can make between size, accuracy, speed, and money spent on hardware
- Ethics, licensing, and data privacy - what open weights model licenses allow, what your training data allows, and how to handle personal or sensitive data

## Steps to go through
1. Why you're tuning a model - what do you want to do better, and how to turn your need or question into the data and model you need
2. Tools and hardware requirements to do the work
   - Linux first: install guides for NVIDIA (CUDA), AMD (ROCm), and x86_64/ARM (aarch64) CPUs, with code shown in tabs per processor type
   - Apple Silicon (MPS) may come later or be left to community documentation
   - Python environments managed with uv
3. Can you get data to help train for this task - ways to get training data
4. How to pick a starting model
5. Design your evaluation
   - we start with simple human evaluations to get a feel for what the models are actually doing
6. Evaluate the base model
7. What type of fine-tuning - there is no single best way, it depends on your use case and data
   - unfreezing the last 1-3 layers of the base model
   - Some type of LoRA - our starting point for the VLM examples
   - Reinforcement Learning
   - Other
8. How to run experiments with fine-tuning
   - tracking experiments, training runs, tracing, and evaluations with MLflow
   - pinning library versions and dataset snapshots, and setting random seeds, so runs are reproducible
9. How to serve your model - llama.cpp to run it locally on your own machine, vLLM to serve it to others
10. User interfaces to your model - web application, chat interface, agentic application, custom agentic application
    - how to put a bot interface on top of it
    - how to use an agentic harness with your model

## Shortlists (decisions pending)

These are candidates, not choices. The author will pick from each list later.

### Candidate models for the handwriting example

Open weights, 25B or less, LoRA fine-tuning to start.

| Model | Size | Type | Fine-tuning notes |
|---|---|---|---|
| [Qwen3-VL-4B](https://huggingface.co/Qwen/Qwen3-VL-4B-Instruct) / [Qwen3-VL-8B](https://huggingface.co/Qwen/Qwen3-VL-8B-Instruct) | 4-8B | general VLM, strong at OCR | best-documented VLM LoRA path (TRL/PEFT) |
| [Qwen2.5-VL-3B](https://huggingface.co/Qwen/Qwen2.5-VL-3B-Instruct) / [Qwen2.5-VL-7B](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct) | 3-7B | general VLM | the default target in most VLM fine-tuning tutorials |
| [SmolVLM2-2.2B](https://huggingface.co/HuggingFaceTB/SmolVLM2-2.2B-Instruct) (also 256M/500M) | 0.2-2.2B | general VLM | the model the smol-course VLM unit uses; fits modest hardware |
| [DeepSeek-OCR](https://huggingface.co/deepseek-ai/DeepSeek-OCR) | 3B | purpose-built document OCR | young fine-tuning ecosystem |
| [GOT-OCR2.0](https://huggingface.co/stepfun-ai/GOT-OCR2_0) | 580M | purpose-built OCR | tiny, nonstandard stack |
| [Moondream2](https://huggingface.co/vikhyatk/moondream2) | ~1.9B | general VLM | laptop-friendly, official fine-tuning docs |

### Candidate training frameworks

Open-source only. Cross-reference: the "Training frameworks" section in references.md.

| Framework | What it is | Accelerators | Notes |
|---|---|---|---|
| [TRL](https://huggingface.co/docs/trl) + [PEFT](https://huggingface.co/docs/peft) | HF's SFT/DPO/GRPO trainers on PyTorch, with LoRA/QLoRA/DoRA via PEFT | CUDA, ROCm, CPU; MPS works but slow | most portable; you see all the knobs |
| [Unsloth](https://docs.unsloth.ai/) | optimized single-GPU fine-tuning with GGUF export for llama.cpp | CUDA only in practice | fastest on consumer GPUs; custom kernels, not stock PyTorch |
| [Axolotl](https://github.com/axolotl-ai-cloud/axolotl) | YAML-driven recipes | CUDA-focused | config over code |
| [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) | config-driven fine-tuning, CLI + web UI | CUDA-focused | broad model coverage |

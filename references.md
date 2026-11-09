# References

This document contains links to documents that are helpful for either explaining concepts, giving advice, or recipes for doing small model fine-tuning.

## Core concepts

- https://sebastianraschka.com/faq/docs/lora-vs-full-finetuning.html - When LoRA beats full fine-tuning (and when it doesn't)
- https://magazine.sebastianraschka.com/p/lora-and-dora-from-scratch - Implements LoRA and DoRA in plain PyTorch so you can see what PEFT libraries actually do

## Do you even need to fine-tune?

- https://www.ibm.com/think/topics/rag-vs-fine-tuning-vs-prompt-engineering - IBM's breakdown of the three levers - what each one changes, and how to choose between them
- https://magazine.sebastianraschka.com/p/practical-tips-for-finetuning-llms - Raschka's practical fine-tuning tips, including the cases where fine-tuning isn't worth the trouble
- https://arxiv.org/abs/2409.15790 - Survey of sub-7B language models: how size trades off against data quality, architecture, and speed; useful grounding for defining "small"

## Courses

- https://huggingface.co/learn/smol-course/unit0/1 - Hugging Face's free hands-on course on fine-tuning *small* models: SFT, chat templates, evaluation, DPO, and VLMs, all runnable locally (https://github.com/huggingface/smol-course)

## End-to-end fine-tuning guides

- https://www.distillabs.ai/learn/how-to-fine-tune-a-small-language-model/ - Step-by-step guide covering data preparation, LoRA adapters, training configuration, and deployment

## Getting training data

- https://argilla.io/ - Open-source platform for collaborative data curation and annotation, well suited to community labeling efforts
- https://distilabel.argilla.io/latest/ - Framework for building pipelines that generate synthetic training data with LLMs, useful when community data is scarce
- https://huggingface.co/learn/cookbook/enterprise_cookbook_argilla - Walkthrough of using Argilla to evaluate base models and build training data

## Fine-tuning on consumer hardware

- https://docs.unsloth.ai/ - Tooling for fine-tuning on a single consumer GPU (8-24GB) or free Colab, with GGUF export for llama.cpp; CUDA-first, so not the path for Macs or most AMD GPUs

## Fine-tuning vision-language models

- https://github.com/merveenoyan/smol-vision - Recipes for fine-tuning small VLMs (and OCR-style vision models) with TRL/PEFT; pairs with the VLM unit of the smol-course

## Training frameworks (to compare before choosing)

- https://huggingface.co/docs/trl - TRL: SFT, DPO, and GRPO trainers on top of PyTorch; the most portable option across accelerators
- https://huggingface.co/docs/peft - PEFT: the LoRA/QLoRA/DoRA library underneath most of the other tools
- https://github.com/axolotl-ai-cloud/axolotl - Axolotl: YAML-driven fine-tuning, CUDA-focused
- https://github.com/hiyouga/LLaMA-Factory - LLaMA-Factory: config-driven fine-tuning (CLI + web UI) across many model types

## Reinforcement learning and preference tuning

- https://huggingface.co/docs/trl/en/dpo_trainer - TRL's DPO trainer: learn from preference pairs without training a separate reward model
- https://huggingface.co/docs/trl/en/grpo_trainer - TRL's GRPO trainer: learn from group-relative rewards, the R1-style recipe
- https://rlhfbook.com/ - Nathan Lambert's open RLHF book, for the full map of where DPO and GRPO-style methods sit

## Evaluation

- https://huggingface.co/spaces/OpenEvals/evaluation-guidebook - Practical guide to designing evals, with common pitfalls and how to think critically about results
- https://kindatechnical.com/generative-ai-llm/evaluating-fine-tuned-models-benchmarks-evals.html - Task-specific evals, LLM-as-judge, and detecting catastrophic forgetting in fine-tuned models

## Experiment tracking and reproducibility

- https://mlflow.org/docs/latest/ - MLflow docs: experiment tracking, tracing, and LLM evaluation in one tool (the one this repo will use)
- https://pytorch.org/get-started/locally/ - The official install matrix for picking the optimized PyTorch build for your accelerator (CUDA, ROCm, MPS, CPU)

## Serving

- https://docs.vllm.ai/en/latest/ - Reference for vLLM inference and serving, including serving LoRA adapters on top of a base model
- https://github.com/ggml-org/llama.cpp - LLM inference on CPUs and every GPU family including Apple Silicon; the "runs on my machine" side of serving, with GGUF as its model format

## Task-specific examples

- https://huggingface.co/blog/fine-tune-whisper - Canonical walkthrough of fine-tuning a model to understand your own voice; the same pattern applies to personalizing recognition tasks (and is relevant if a project needs speech input)
- https://huggingface.co/docs/transformers/model_doc/trocr - TrOCR, a vision model family for OCR; a likely starting point for the handwriting example, since handwriting recognition is a vision task rather than a text-LLM task

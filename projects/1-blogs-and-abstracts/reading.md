# Reading list: tuning a small text model for blog and abstract writing style

Compiled 2026-11-11. All URLs verified with curl (HTTP 200) on this date, except the one in the last section, which blocks automated checks.

Aimed at someone who already understands LoRA and SFT basics from the root
references.md. That file already covers the generic toolchain (TRL/PEFT, the
smol-course root, Unsloth, MLflow, the eval guidebook, DPO/GRPO, Argilla), so
this list only adds what is specific to this project: the honest
prompting-versus-fine-tuning framing for voice, turning past posts and
abstracts into SFT data, the text-only fine-tuning recipes, and evaluating
something as subjective as "does this sound like me?".

## 1. The honest framing - why tune, and why LoRA fits voice

plan.md says it straight: prompting a strong general model with a few writing
samples gets most of the way, and the fine-tune is justified by running
locally on small hardware and by consistent style without prompt bloat. These
sources back that framing up:

- https://zenvanriel.com/ai-engineer-blog/lora-vs-full-fine-tuning-personal-writing-style/ - LoRA vs full fine-tuning for personal voice, from someone who did it: LoRA wins for style, roughly 1-5M tokens of cleaned paired data is enough, and the stated goal is "sounds like me without a 4,000-word system prompt"
- https://arxiv.org/abs/2305.11206 - LIMA: 1,000 curated examples beat far larger noisy sets; why a few hundred well-chosen posts and abstracts can carry a style tune
- https://arxiv.org/abs/2405.09673 - "LoRA learns less and forgets less": LoRA captures surface behavior like style while preserving knowledge - exactly the tradeoff this project wants (Raschka's LoRA pieces in references.md are the mechanism side)
- https://arxiv.org/abs/2507.21509 - Persona vectors (Anthropic): steering character traits through activations instead of weights; the non-fine-tuning lever worth knowing for the "Do you even need to fine-tune?" page

## 2. Data - turning past posts and abstracts into training pairs

The data exists; the work is structure and curation:

- https://huggingface.co/docs/trl/en/dataset_formats - the dataset formats SFTTrainer accepts (standard vs conversational); this is the schema every example has to land in
- https://huggingface.co/docs/transformers/en/chat_templating - chat templates: the training pairs must be rendered in the target model's template or the tune teaches the wrong distribution
- https://minimaxir.com/2019/09/howto-gpt2/ - the original fine-tune-your-own-writing walkthrough (GPT-2 era); the tooling is dated but the corpus-curation and data-cleaning lessons still apply
- https://arxiv.org/abs/2308.06259 - Instruction backtranslation: generate instructions for documents you already have - how to get the prompt half of the pair for existing posts and abstracts

Practical notes that fall out of these: hold out the most recent posts as an
eval set (a temporal split, not random); dedup near-identical drafts; abstracts
are naturally (paper → abstract) pairs, blogs are (topic or outline → post).

## 3. Recipes - the text SFT + LoRA walkthroughs

- https://huggingface.co/learn/smol-course/en/unit1/1 - the smol-course SFT unit: instruction tuning SmolLM3, chat templates, loss masking; the course the repo already adopts
- https://huggingface.co/docs/trl/en/sft_trainer - TRL's SFTTrainer reference: the exact knobs (assistant-only loss, packing, completion-only training) the walkthroughs abstract away
- https://www.philschmid.de/fine-tune-llms-in-2024-with-trl - the canonical text walkthrough: QLoRA + chat template + SFTTrainer on an 8B model (notebook-based, but the repo's no-notebooks rule applies to delivery, not reading)
- https://github.com/huggingface/alignment-handbook - HF's full-script SFT/DPO recipes for small instruct models (Zephyr lineage); the closest thing to this repo's "plain Python scripts" delivery
- https://unsloth.ai/docs/get-started/fine-tuning-llms-guide - the consumer-GPU fast path; CUDA-only in practice, the tradeoff the README framework table already flags
- https://unsloth.ai/docs/basics/running-and-saving-models/saving-to-gguf - how a fine-tune becomes a GGUF llama.cpp can run; connects training to the repo's serving decision
- https://github.com/hiyouga/LLaMA-Factory/blob/main/examples/train_lora/qwen3_lora_sft.yaml - one concrete LoRA SFT YAML for Qwen3; config-over-code, for comparing against the TRL path
- https://mlflow.org/docs/latest/llms/transformers/index.html - the MLflow flavor for HF models; the repo logs runs to MLflow from the first experiment onward

## 4. Evaluation - "does this sound like me?"

There is no CER/WER equivalent for voice. The roadmap starts with human
evaluations; these make the rest defensible:

- https://huggingface.co/docs/transformers/en/perplexity - computing perplexity with transformers; a cheap style-capture check - perplexity on held-out personal writing should drop versus the base model
- https://arxiv.org/abs/2306.05685 - the MT-Bench paper: the canonical LLM-as-judge methodology, including the position and verbosity biases that matter when a judge compares writing styles
- https://arxiv.org/abs/2404.04475 - Length-Controlled AlpacaEval: how to debias automatic judges for length and style; the guardrail if a judge is used as a second opinion alongside human A/B tests

For the human side, the OpenEvals evaluation-guidebook in references.md covers
the blind A/B design. DPO on liked/disliked drafts is the natural follow-on
once human preferences exist (TRL DPO trainer in references.md).

## 5. Model landscape - candidate small text models

Candidates, not choices: README.md has no text-model shortlist yet, so nothing
here resolves that. All are open weights and LoRA-tunable on modest hardware.

- https://huggingface.co/HuggingFaceTB/SmolLM3-3B - Apache-2.0, 3B; the model the smol-course SFT unit tunes, so the gentlest on-ramp
- https://huggingface.co/Qwen/Qwen3-4B-Instruct-2507 - the non-thinking Qwen3 instruct variant, generally considered the stronger prose writer of the family
- https://huggingface.co/google/gemma-4-12B-it - current-generation small Gemma; E2B/E4B variants exist for tighter hardware
- https://huggingface.co/meta-llama/Llama-3.1-8B-Instruct - the most LoRA-documented 8B; custom Meta license, worth reading before redistribution
- https://huggingface.co/openai/gpt-oss-20b - Apache-2.0 MoE, ~3.6B active parameters, runs within 16GB
- https://huggingface.co/allenai/OLMo-2-1124-7B-Instruct - fully open weights, data, and training code; the cleanest fit for the repo's community framing

For choosing among them:

- https://eqbench.com/creative_writing.html - EQ-Bench's creative writing leaderboard; how small models actually rank at long-form prose
- https://www.distillabs.ai/learn/best-small-language-model-for-fine-tuning-2025/ - a 12-model small-LM fine-tuning benchmark (Qwen 3, Llama 3.2, Gemma 3) from the same shop as the end-to-end guide in references.md

## Not verified to the curl-200 standard

- https://medium.com/autocomplete-real-world-ai/i-fine-tuned-gemma-4-to-sound-like-me-7f45a8a576f3 - blocks curl (403); read in a browser. Useful for the honest framing: its core finding is that if a base model already sounds like you, the fine-tune is almost invisible - which is exactly the prompting baseline to run first.

## Suggested reading order

1. The zenvanriel piece and LIMA, to set expectations for what a style tune can and cannot do
2. smol-course unit 1 and the TRL SFTTrainer docs
3. Chat templating plus the TRL dataset formats doc, then draft the (instruction → post/abstract) schema
4. Pick the candidate model with EQ-Bench and the Distillabs comparison
5. Train via the alignment-handbook path (or Unsloth if CUDA-only is acceptable), logging to MLflow
6. Evaluate: prompting baseline first, then perplexity on held-out posts, blind human A/B, and an MT-Bench-style judge with length control as a second opinion

The prompting baseline is deliberately last-but-first: run it before training
to quantify the gap, and again after, as the thing the fine-tune has to beat.

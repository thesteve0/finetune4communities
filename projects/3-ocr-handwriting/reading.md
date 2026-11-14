# Reading list: tuning VLMs for handwriting OCR

Compiled 2026-11-11. All URLs verified with curl (HTTP 200) on this date, except the two in the last section, which block automated checks.

Aimed at someone who already understands VLMs and has tuned them for tool calling. The gaps this list fills are the OCR-specific parts: image-to-text data, resolution-driven quality, objective CER/WER evaluation, and the fine-tuning recipes.

## 1. What's different about OCR as a tuning target

- https://qwenlm.github.io/blog/qwen2.5-vl/ - explains dynamic resolution and how Qwen2.5-VL treats text in images; OCR quality tracks vision-encoding choices more than anything tuned in the decoder
- https://arxiv.org/abs/2502.13923 - Qwen2.5-VL Technical Report; the OCR-centric training data and text-recognition benchmarks show what a production OCR curriculum looks like
- https://www.microsoft.com/en-us/research/publication/trocr-transformer-based-optical-character-recognition-with-pre-trained-models/ - the TrOCR paper: the classic image-encoder to text-decoder formulation fine-tuned on line images, the foundation later work builds on

## 2. Data - building the personal handwriting set

- https://huggingface.co/docs/datasets/en/image_dataset - how to structure a custom image + text dataset (ImageFolder / metadata CSV) for VLM training
- https://raw.githubusercontent.com/hiyouga/LLaMA-Factory/main/data/README.md - LLaMA-Factory's multimodal data spec; the messages format with image references is what most VLM SFT pipelines consume, including TRL
- https://fki.tic.heia-fr.ch/databases/iam-handwriting-database - the IAM handwriting database; line images + transcripts, ideal for validating the whole pipeline before investing time in personal data
- https://github.com/NielsRogge/Transformers-Tutorials/blob/master/TrOCR/Fine_tune_TrOCR_on_IAM_Handwriting_Database_using_Seq2SeqTrainer.ipynb - the canonical notebook: fine-tune TrOCR on IAM line crops with Seq2SeqTrainer and compute WER end-to-end; teaches line-cropping, image preprocessing, and the eval loop that transfer directly to a VLM

## 3. Recipes - the actual fine-tuning walkthroughs

The repo already links smol-vision and the smol-course root in references.md; these are the VLM-specific units and Qwen recipes:

- https://huggingface.co/learn/smol-course/en/unit3/3 - the smol-course's Fine-Tuning VLMs unit; adapts the SFT/TRL techniques from unit 1 to VLMs, using SmolVLM2 (already on the README shortlist)
- https://huggingface.co/learn/cookbook/en/fine_tuning_vlm_trl - HF Cookbook: fine-tuning a VLM with TRL's SFTTrainer, including the chat-template and multimodal-collator plumbing
- https://huggingface.co/docs/trl/en/training_vlm_sft - TRL's official VLM SFT guide; the reference for the exact data format SFTTrainer expects from vision datasets
- https://www.f22labs.com/blogs/complete-guide-to-fine-tuning-qwen2-5-vl-model/ - complete Qwen2.5-VL LoRA walkthrough: setup, data prep, training, inference
- https://roboflow.com/blog/fine-tune-qwen-2-5 - fine-tune Qwen2.5-VL on a custom dataset to read documents into structured output; closest to the transcribe-my-handwriting shape
- https://unsloth.ai/docs/models/tutorials/qwen3-how-to-run-and-fine-tune/qwen3-vl-how-to-run-and-fine-tune - Unsloth's Qwen3-VL page; relevant if the CUDA-only tradeoff is acceptable for the consumer-GPU path
- https://github.com/zhangfaen/finetune-Qwen2.5-VL - deliberately minimal training-loop implementation; good for seeing the knobs TRL hides
- https://github.com/2U1/Qwen-VL-Series-Finetune - one script covering Qwen2-VL / Qwen2.5-VL / Qwen3-VL with HF + Liger-Kernel, useful when swapping shortlisted models

## 4. Evaluation - CER/WER instead of human judgment

- https://towardsdatascience.com/evaluating-ocr-output-quality-with-character-error-rate-cer-and-word-error-rate-wer-853175297510/ - the standard CER/WER explainer with Python implementation
- https://github.com/jitsi/jiwer - the library used to compute both metrics
- https://www.docsumo.com/blog/ocr-accuracy - what CER thresholds mean practically (when 5% CER is usable)

## 5. Model landscape - the purpose-built OCR models

- https://arxiv.org/abs/2409.01704 - GOT-OCR 2.0 paper; explains why it is tiny and why its stack is nonstandard (the one plan.md says to avoid as the starter)
- https://arxiv.org/abs/2510.18234 - DeepSeek-OCR paper (contexts optical compression); 3B purpose-built document model with the DeepEncoder design
- https://docs.moondream.ai/finetuning - Moondream's official fine-tuning docs; the most laptop-friendly path on the shortlist

## Not verified to the curl-200 standard

Both have useful content but block automated checks; read them in a browser.

- https://deepwiki.com/QwenLM/Qwen3-VL/7-fine-tuning - overview of which components to tune (vision encoder vs MLP vs LLM) on Qwen3-VL; rate-limits curl (429)
- https://www.datacamp.com/tutorial/fine-tuning-qwen3-vl-8b - January 2026 Qwen3-VL LoRA tutorial; Cloudflare-blocks curl (403)

## Suggested reading order

1. smol-course unit3/3
2. HF Cookbook VLM recipe
3. TrOCR/IAM notebook
4. Swap IAM for personal line crops
5. CER/WER + jiwer for the eval step

The Qwen blog and tech report can be skimmed first to anchor the "why resolution matters" intuition.

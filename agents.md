# agents.md

Guidance for AI agents working in this repository.

## What this project is

A guide for creating small fine-tuned models for communities, aimed at people who are AI knowledgeable but not AI experts. It is built example-first: each example (LLM judges, blog/abstract writing, handwriting OCR, ADHD assistance, restoration ecology, house building) will eventually get working datasets and code. Python and PyTorch on Linux only. Open-source frameworks only.

## Current state

- `README.md` - the roadmap: "Before you start" pages plus 10 steps. Also holds two shortlist tables for pending decisions.
- `references.md` - annotated links that map onto the roadmap.
- `plan.md` - staging order and dependencies for the six examples.
- `projects/1-llm-judges/README.md` - the judge-training outline; projects 2-6 have reading lists. Directory numbers match the learning sequence.
- `LICENSE`
- No code yet.

## Fixed author decisions (treat these as constraints, do not relitigate)

- **Serving: llama.cpp and vLLM only. NEVER Ollama or SGLang.** llama.cpp is for running locally on your own machine; vLLM is for serving to others.
- **No full fine-tuning in this guide.** The options are unfreezing the last 1-3 layers of the base model, LoRA-family adapters (the starting point for VLM work), and reinforcement learning. The framing is "no single best way - it depends on your use case and data".
- **Linux first.** First-class targets: NVIDIA (CUDA), AMD (ROCm), x86_64 and aarch64 CPUs. Apple Silicon (MPS) comes later or is left to community documentation.
- **Python + PyTorch only.** Show people how to install the optimized PyTorch build for their accelerator.
- **uv** for Python environment management.
- **MLflow** for experiment tracking, tracing, and evaluations.
- **Reproducibility:** pinned library versions, pinned dataset snapshots, fixed random seeds.
- **No notebooks.** The author dislikes them strongly; delivery is plain Python scripts. Final code layout is still undecided.
- **Judges first, writer second.** Train three small LLM judges to evaluate content created by the blog/abstract writing model. Begin with abstracts, then extend to blog posts. Initial judge data can come from existing drafts and untuned generators; do not make judge training depend on the fine-tuned writer.
- **Judge supervision:** the author's narrated critiques plus great (1), fair (0), or bad (-1) ratings. Train for an explanation grounded in the candidate text plus a rating, using supervised fine-tuning. Reasoning/explanation capability is part of the exercise; no reinforcement learning is required for this starting point.
- **Three-judge voting** is part of the first example. Reuse the curated dataset for three small-model training runs; aim for quick, straightforward training once the data exists, but measure cost and runtime rather than promising them. Compare individual judges and the ensemble against held-out human judgments. Shared errors can limit voting gains; human review remains the reference.
- **Evaluation** starts with simple human evaluations; a proper eval design will be written when that roadmap step is reached.

## How to work here

- **Propose, then apply.** For anything beyond typos, describe the change and get the author's approval first. The author reviews wording closely.
- **Preserve the author's voice** in README.md. The examples list is verbatim; do not polish it.
- **Verify every URL** (curl, expect HTTP 200) before adding it to references.md. Annotations are one line, bullet-list format, no tables in references.md. Never remove a source without asking; if a link dies, flag it.
- **The shortlist tables in README.md are open decisions.** Do not resolve them silently.
- **Do not add status markers** (checkboxes, "done") to the README roadmap.
- Keep README.md a roadmap, not the guide content itself. Full explanations belong in future pages and examples.

## Open decisions

- Which VLM/OCR model(s) to feature for the handwriting example (shortlist in README).
- The happy-path training framework: TRL/PEFT vs Unsloth vs Axolotl vs LLaMA-Factory (shortlist in README, links also in references.md).
- Reinforcement learning: DPO vs GRPO-style training - "Reinforcement Learning" stays on the roadmap until this is understood. Background reading: TRL DPO/GRPO docs and rlhfbook.com in references.md.
- Judge model selection and how to vary the three judges (base models, seeds, or other training choices); voting details, including the all-different case. Do not assume the judges are independent or generalize beyond the writing rubric.
- Code delivery layout within the numbered example folders (uv.lock? configs?).
- Pages to be written: "Do you even need to fine-tune?" (prompt engineering vs RAG vs fine-tuning, plus other accuracy levers: few-shot examples, tool use, constrained decoding, inference-time scaling, validators and retry loops, model routing, distillation), "What does small mean?" (size/accuracy/speed/hardware-cost tradeoffs), "Ethics, licensing, and data privacy".

## Conventions for code when it lands

- uv-managed environments, `python`, pinned versions via uv.lock or pinned requirements.
- Fixed seeds; log runs to MLflow from the first experiment onward.
- Processor-specific code (CUDA / ROCm / x86_64 CPU / aarch64 CPU) presented in tabs per processor type.
- Permissively-licensed or self-collected datasets only; no personal data in the repo unless it is the authors or there is a github documentation with approval of the user (this matters for the ADHD example).
- Every example ships with its dataset and runnable code.

# Reading list: tuning a small model for ADHD assistance on Reachy Mini

Compiled 2026-11-11. All URLs verified with curl (HTTP 200) on this date.

Aimed at someone who already understands LoRA and SFT basics from the root
references.md. That file already covers the generic toolchain (TRL/PEFT, the
smol-course root, Unsloth, MLflow, the eval guidebook, the DPO/GRPO trainer
docs, rlhfbook.com, Argilla, llama.cpp, vLLM), so this list only adds what is
specific to this project: RLHF as the mechanism for adapting the assistant to
one user's feedback, the ADHD-assistant task definition and its cautions, the
voice-first local serving stack, and the Reachy Mini integration layer.

plan.md's caveat is the spine of this list: the fine-tuned model is one
component in a larger robot system, and most of the project is integration
work. Within that system, the tune earns its keep in two places. First, a
consistent assistant personality from a model small enough to run locally -
which is also the privacy argument, since personal health-adjacent data never
leaves the author's machine. Second, adapting the assistant's behavior to one
user over time through their own feedback, instead of an ever-growing system
prompt. That second mechanism is RLHF, which is why it anchors this list.

## 1. The honest framing - what the model does in a robot system

The "Do you even need to fine-tune?" logic (the IBM piece in references.md)
applies doubly here: a strong general model with a good system prompt and
tools does most of the assistant work. The tune is for personality
consistency on small local models and adaptation over time - and the
evaluation has to prove the adaptation is real. These sources frame it:

- https://pmc.ncbi.nlm.nih.gov/articles/PMC13505058/ - peer-reviewed evaluation of how widely used LLMs answer common ADHD-related questions (ChatGPT, Gemini, DeepSeek): broadly accurate but uneven in depth and clinical usefulness; grounding for what a general model already does and where an assistant needs care
- https://arxiv.org/abs/2310.13548 - Anthropic's sycophancy paper: models trained on human feedback drift toward agreeing with the person giving it; the central failure mode when the feedback signal is one person's own preferences, and the thing the human evaluations must probe
- https://arxiv.org/abs/2407.00693 - BAPO: personalizing through preference optimization degrades general knowledge, and how much depends on how heterogeneous the preferences are; the forgetting tension this project's evals have to catch when tuning a general assistant into a personal one

## 2. RLHF - learning the assistant's behavior from your feedback

The repo's open DPO-vs-GRPO decision (README roadmap, agents.md) lands here
for real. The methods map, from simple to ambitious:

- https://huggingface.co/blog/rlhf - the "Illustrating RLHF" explainer: preference data, reward model, PPO, and why the whole pipeline exists; the conceptual baseline before the DPO-family simplifications
- https://huggingface.co/blog/pref-tuning - Labonne's practical tour of the DPO family with runnable training examples; the fastest bridge from the rlhfbook.com map (references.md) to an actual recipe
- https://arxiv.org/abs/2305.18290 - the DPO paper ("your language model is secretly a reward model"): pairwise preference learning without training a separate reward model; the workhorse for logged feedback pairs
- https://arxiv.org/abs/2402.01306 - KTO: learns from unary feedback - a single thumbs-up or thumbs-down per response, no pairs required; the realistic shape of feedback from daily use of an assistant, where nobody writes contrastive pairs
- https://arxiv.org/abs/2402.03300 - DeepSeekMath, the paper that introduced GRPO: group-relative advantage without a value model, the R1-style recipe behind the repo's other open RL option
- https://huggingface.co/docs/trl/en/online_dpo_trainer - TRL's online DPO: generate, collect preferences, update, repeat - the loop a personal assistant actually runs as feedback accumulates

Practical shape for this project: the feedback loop is thumbs-up/down on the
robot's suggestions plus occasional side-by-side choices, which maps directly
onto KTO and online DPO. GRPO-style RL only fits where a reward is
verifiable - in a robot, tool feedback ("did the user actually start the
timer?") is the verifiable part, and open-r1 in section 4 is the recipe path.

## 3. Data - feedback logs, preference pairs, and the privacy constraint

The primary dataset is the author's own usage: conversations, thumbs on
suggestions, chosen-versus-rejected responses. Hold out recent weeks as the
eval set - a temporal split, not random, for the same reason as the
blog-abstract list. Argilla (references.md) is the labeling tool for building
preference pairs; MLflow logs the runs from the first experiment.

- https://arxiv.org/abs/2106.01144 - "Towards Emotional Support Dialog Systems" and its ESConv corpus: the closest public dataset to supportive-conversation style, for bootstrapping tone before personal data exists; coaching support is not therapy, and that boundary belongs in the data design, not just the docs
- https://owasp.org/www-project-top-10-for-large-language-model-applications/ - OWASP's LLM Top 10; "sensitive information disclosure" is the entry that matters for a mic-and-camera robot in a home, and the checklist to run before any log or transcript goes anywhere near the repo

The privacy constraint from agents.md and plan.md is also the data
constraint: ADHD-adjacent logs are personal health data, so they never enter
the repo, and the point of a small local model is that they never leave the
machine either. That is why model size in section 7 is a privacy decision,
not just a hardware one.

## 4. Recipes - running the preference tune on small models

- https://huggingface.co/learn/smol-course/en/unit2/1 - the smol-course preference-alignment unit: hands-on DPO with SmolLM3, chat template and all; the on-ramp the repo already adopts
- https://huggingface.co/docs/trl/en/kto_trainer - TRL's KTO trainer; the direct implementation of the unary-feedback loop in section 2
- https://huggingface.co/docs/trl/en/online_dpo_trainer - TRL's online DPO trainer; the iterative generate-annotate-update loop
- https://unsloth.ai/docs/get-started/reinforcement-learning-rl-guide - Unsloth's RL guide for DPO and GRPO on consumer GPUs; CUDA-only in practice, the tradeoff the README framework table already flags
- https://github.com/hiyouga/LLaMA-Factory/blob/main/examples/train_lora/qwen3_lora_dpo.yaml - one concrete LoRA DPO config for Qwen3-4B-Instruct-2507; the config-over-code path, for comparing against TRL
- https://github.com/huggingface/open-r1 - HF's fully open reproduction of the R1 pipeline; the GRPO-style recipes if the project goes the verifiable-reward route

## 5. Serving and the voice-first stack

- https://maartengrootendorst.substack.com/p/a-visual-guide-to-quantization - what Q4/Q8 quantization actually does to weights and quality; the mental model for shrinking a tuned model until it fits the hardware it must run on
- https://github.com/ggml-org/whisper.cpp - CPU-first Whisper speech-to-text from the llama.cpp family; keeps voice transcripts on the same local path as the model itself
- https://github.com/rhasspy/piper - fast local text-to-speech from the Rhasspy project; the usual voice for small self-hosted assistants

Hardware reality from the Reachy Mini specs: the Lite version is
USB-connected and the model runs on the host computer; the wireless version
runs on an onboard Raspberry Pi CM4, where only the smallest quantized models
live. Either way, llama.cpp (references.md) plus Unsloth's GGUF export closes
the loop from tuned adapter to on-device runtime - and this is the path the
privacy argument in plan.md depends on. vLLM (references.md) enters only if
the assistant is ever served beyond one machine.

## 6. Integration - Reachy Mini and the agentic layer

Roadmap steps 9-10 (serving, user interfaces, agentic harnesses) live here
more than anywhere else in the repo:

- https://huggingface.co/docs/reachy_mini/en/index - official Reachy Mini documentation (note the underscore)
- https://huggingface.co/docs/reachy_mini/platforms/reachy_mini/get_started - wireless-unit setup, with the onboard CM4 that sets the model-size ceiling
- https://huggingface.co/docs/reachy_mini/SDK/quickstart - first behaviors from the Python SDK
- https://huggingface.co/docs/reachy_mini/SDK/integration - the AI-integrations guide: connecting LLMs, building apps, and publishing them
- https://github.com/pollen-robotics/reachy_mini - the SDK source, including the MuJoCo simulation for developing before hardware arrives
- https://huggingface.co/spaces/pollen-robotics/reachy_mini_conversation_app - the reference architecture for this project: realtime voice, personality-aware tools, MCP tool spaces, choreographed motion - with a local-endpoint option for the LLM backend
- https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md - the Reachy Mini repo's own guide for building its apps with AI coding agents
- https://huggingface.co/docs/reachy_mini/SDK/javascript-sdk - browser apps that drive the robot over WebRTC; the easiest way to put a user interface in front of the model
- https://huggingface.co/blog/pollen-robotics/make-and-publish-your-reachy-mini-apps - publishing an app to the robot's HF-Spaces app store

## 7. Model landscape - candidates for robot-class hardware

Candidates, not choices. Tool-calling matters as much as prose here: the
assistant's useful actions (timers, reminders, calendar, motion) arrive as
tool calls, and a robot that calls tools badly is furniture.

- https://huggingface.co/Qwen/Qwen3-1.7B - the smallest Qwen3; the tier that runs on CM4-class onboard compute
- https://huggingface.co/Qwen/Qwen3-4B-Instruct-2507 - the non-thinking 4B instruct with solid tool-calling for its size; the realistic upper end for a host-driven robot
- https://huggingface.co/HuggingFaceTB/SmolLM3-3B - the model the smol-course tunes in both its SFT and preference units; the gentlest path from course to project
- https://huggingface.co/LiquidAI/LFM2-1.2B - Liquid AI's LFM2 family, designed for edge AI and on-device deployment, with tool-use support
- https://huggingface.co/LiquidAI/LFM2-2.6B - the larger LFM2 sibling for the host-computer path
- https://gorilla.cs.berkeley.edu/leaderboard.html - the Berkeley function-calling leaderboard; how small models actually rank at the tool-calling this project leans on

## Suggested reading order

1. Reachy Mini docs and the conversation-app architecture - know the system the model plugs into before tuning anything
2. The "Illustrating RLHF" blog and Labonne's pref-tuning tour for the map, then smol-course unit 2 hands-on
3. The KTO paper and TRL's KTO trainer to design the thumbs-up/down feedback loop
4. The sycophancy paper and BAPO - the two failure modes the evaluation must catch
5. The quantization guide, then llama.cpp + whisper.cpp + piper for the local voice stack
6. Pick the candidate with the Berkeley leaderboard, keeping tool-calling ahead of prose

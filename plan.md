# Example staging plan

Recorded 2026-11-11.

This documents the staging order for the five possible examples in README.md
("Our current possible examples" list, numbered 1-5 there), the reasoning behind
the order, and the dependencies that could change it. It stages examples only;
it does not resolve the open decisions in README.md (candidate VLM/OCR models,
training framework choice).

## Staging order

1. Blog and abstract writing - first
2. Handwriting OCR - second
3. ADHD assistance with Reachy-mini - third
4. Restoration ecology, southern Santa Lucia Mountains - fourth
5. House building expert for newbies - last

## Why this order

### 1. Blog and abstract writing first

- Gentlest on-ramp: the canonical text-SFT LoRA path (TRL/PEFT, chat
  templates), the most documented recipe in the ecosystem.
- Data already exists: the author's past posts and abstracts.
- Subjective evaluation ("does this sound like me?") fits the roadmap plan to
  start with simple human evaluations.
- Honest framing needed: this is the example where prompting a strong general
  model with a few writing samples gets most of the way. The fine-tune is
  justified for running locally on small hardware and for consistent style
  without prompt bloat. The guide should say so.

### 2. Handwriting OCR second

- Strongest justification for fine-tuning of the five: generic models read
  neat print; idiosyncratic personal handwriting is where tuning earns its
  keep.
- Objective, scriptable evaluation (CER/WER against ground truth the author
  already knows). Good teaching vehicle for the evaluation roadmap step.
- Data is self-generated: write, scan, crop lines; the labels are free.
- Introduces the VLM branch of the roadmap. Avoid GOT-OCR2.0 as the starter
  (nonstandard stack, per the README shortlist notes).

### 3. ADHD assistance with Reachy-mini third

- After one text example and one vision example, this is the most novel
  remaining territory: serving to others (vLLM), user interfaces, and agentic
  harnesses (roadmap steps 9-10). No other example covers these as well.
- The privacy constraint in agents.md is also the argument for the model: a
  small model served locally with llama.cpp keeps personal health data on the
  author's own machine.
- Caveat: the fine-tuned model is one component in a larger robot system; most
  of the project is integration work.

### 4. Restoration ecology fourth

- Shares the text toolchain with example 1, so the fine-tuning mechanics are
  familiar by the time it starts.
- Its genuinely new work - curating scarce hyper-local sources, the
  RAG-versus-fine-tuning decision, evaluation that needs domain expertise -
  is the same work example 5 needs, so the two belong adjacent at the end.
- Strongest eventual fit with the repo's community framing; keep it in mind
  as the showcase example.

### 5. House building last

- The textbook answer is RAG over curated sources, not fine-tuning;
  fine-tuning is poor at injecting factual knowledge.
- Safety-critical advice means hallucination and liability risk, and
  evaluation is open-ended and fuzzy.

## On staging example 3 ahead of example 4

The author's rationale: example 4 is very similar to example 1, which is done
first, so it adds less novelty. Agreed, with a refinement. Example 4 overlaps
example 1 only in the fine-tuning mechanics; its hard parts overlap example 5
instead. So example 4 lands next to its real sibling at the end, and example 3
takes the slot where it adds the most new territory.

## Dependencies that could reorder 3 and 4 in practice

- Example 3 needs Reachy-mini hardware and personal-data handling approvals
  before any data work starts. If either stalls, example 4 can move up at no
  toolchain cost, since it reuses example 1's stack.
- Example 4 needs a data-availability survey before committing (local
  literature, agency reports, theses, community-science data).
- Examples 4 and 5 both need an explicit pass through the "Do you even need to
  fine-tune?" page; RAG is likely the primary pattern, with fine-tuning as a
  complement for tone and format.

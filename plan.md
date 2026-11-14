# Example staging plan

Recorded 2026-11-11.

This documents the staging order for the six possible examples in README.md
("Our current possible examples" list, numbered 1-6 there), the reasoning behind
the order, and the dependencies that could change it. It stages examples only;
it does not resolve the open decisions in README.md (candidate VLM/OCR models,
training framework choice).

## Staging order

1. LLM judges for the blog and abstract writing model - first
2. Blog and abstract writing - second
3. Handwriting OCR - third
4. ADHD assistance with Reachy-mini - fourth
5. Restoration ecology, southern Santa Lucia Mountains - fifth
6. House building expert for newbies - last

## Why this order

### 1. LLM judges first

- A bounded text-SFT task: learn to explain a judgment and output great (1),
  fair (0), or bad (-1), rather than generate an entire piece of writing.
- Supervision comes from the author's ratings and narrated critiques of
  candidate abstracts and blog posts. Start with abstracts, then extend to posts.
- Train three small judges on the curated dataset and implement voting; the goal
  is quick, straightforward training once the data exists. Measure runtime and
  compare the panel with individual judges rather than assuming voting helps.
- The purpose is to evaluate content created by the writing model in example 2.
  Existing drafts and untuned generators bootstrap judge data, so there is no
  dependency on completing the writer first.
- Human ratings remain the reference. Hold out complete draft families and test
  explanations and ratings on unseen writing. Model choices, diversity between
  judges, and the all-different voting fallback remain open decisions.
- The outline is in projects/1-llm-judges/README.md. The numbered project
  directories follow this staging order; code layout within them is undecided.

### 2. Blog and abstract writing second

- Reuses the text-SFT LoRA mechanics from the judge exercise for generation.
- Source material already exists: the author's past posts and abstracts.
- Uses the judges from example 1 alongside human evaluation; freeze judge
  versions when comparing writers and keep test material out of training.
- Subjective evaluation ("does this sound like me?") fits the roadmap plan to
  start with simple human evaluations.
- Honest framing needed: this is the example where prompting a strong general
  model with a few writing samples gets most of the way. The fine-tune is
  justified for running locally on small hardware and for consistent style
  without prompt bloat. The guide should say so.

### 3. Handwriting OCR third

- Strongest justification for fine-tuning of the original five use cases: generic models read
  neat print; idiosyncratic personal handwriting is where tuning earns its
  keep.
- Objective, scriptable evaluation (CER/WER against ground truth the author
  already knows). Good teaching vehicle for the evaluation roadmap step.
- Data is self-generated: write, scan, crop lines; the labels are free.
- Introduces the VLM branch of the roadmap. Avoid GOT-OCR2.0 as the starter
  (nonstandard stack, per the README shortlist notes).

### 4. ADHD assistance with Reachy-mini fourth

- After the text examples and one vision example, this is the most novel
  remaining territory: serving to others (vLLM), user interfaces, and agentic
  harnesses (roadmap steps 9-10). No other example covers these as well.
- The privacy constraint in agents.md is also the argument for the model: a
  small model served locally with llama.cpp keeps personal health data on the
  author's own machine.
- Caveat: the fine-tuned model is one component in a larger robot system; most
  of the project is integration work.

### 5. Restoration ecology fifth

- Shares the text toolchain with examples 1 and 2, so the fine-tuning mechanics are
  familiar by the time it starts.
- Its genuinely new work - curating scarce hyper-local sources, the
  RAG-versus-fine-tuning decision, evaluation that needs domain expertise -
  is the same work example 6 needs, so the two belong adjacent at the end.
- Strongest eventual fit with the repo's community framing; keep it in mind
  as the showcase example.

### 6. House building last

- The textbook answer is RAG over curated sources, not fine-tuning;
  fine-tuning is poor at injecting factual knowledge.
- Safety-critical advice means hallucination and liability risk, and
  evaluation is open-ended and fuzzy.

## On staging the robot ahead of restoration ecology

The robot adds serving, user interfaces, and agentic integration after the text
and vision examples. Ecology reuses the text fine-tuning mechanics, but its hard
parts overlap construction: source curation, retrieval, and domain-expert
evaluation. Those two examples therefore remain adjacent at the end. The writing
judges do not automatically become expert judges for either domain.

## Dependencies that could reorder examples 4 and 5 in practice

- Example 4 needs Reachy-mini hardware and personal-data handling approvals
  before any data work starts. If either stalls, example 5 can move up at no
  toolchain cost, since it reuses the text stack.
- Example 5 needs a data-availability survey before committing (local
  literature, agency reports, theses, community-science data).
- Examples 5 and 6 both need an explicit pass through the "Do you even need to
  fine-tune?" page; RAG is likely the primary pattern, with fine-tuning as a
  complement for tone and format.

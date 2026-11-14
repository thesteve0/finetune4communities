# Training small LLM judges for the blog and abstract writing model

## Purpose

Train three small LLM judges to evaluate content created by the writing model in
[example 2](../2-blogs-and-abstracts/reading.md). The judges learn the author's
writing preferences from ratings and narrated critiques, then vote on candidate
abstracts and blog posts. This is a specialist writing evaluator, not a general
factuality or safety judge.

This is the first fine-tuning exercise: a bounded text task with a short
explanation and a three-point rating as its output. Start with abstracts, then
extend to blog posts. Once the dataset exists, the goal is quick, straightforward
training runs for three small models; actual training and inference costs are
measurements, not assumptions.

## Dataset and supervision

Each example contains:

- The writing brief, including intended audience and relevant constraints.
- A candidate abstract or blog post.
- A consistent rubric describing the author's voice and writing preferences.
- The author's narrated explanation of what works, what does not, and why.
- One rating: great (1), fair (0), or bad (-1).

Existing drafts, edited variants, and untuned model outputs supply candidates
before the fine-tuned writer exists. Published writing supplies reference
material, but is not enough by itself: the judges need rated examples across all
three categories, including realistic borderline cases.

Define the rating boundaries during dataset design. Fair means partially meeting
the rubric, not uncertainty or missing evidence. Do not claim factual accuracy
from voice judgments; any factual checks need the relevant source material.

Keep all versions of an underlying abstract, post, or brief in the same data
split. Reserve human-rated test examples that neither the writer nor the judges
train on. The author's explanations are supervision for useful critiques, not
proof that generated explanations expose a model's internal reasoning.

## Training sequence

1. Establish the human rubric and collect ratings with narrated critiques.
2. Split the data and evaluate prompted base judges before training.
3. Supervised-fine-tune a small judge to produce a concise, text-grounded
   explanation and a rating, using the repo's parameter-efficient training options.
4. Reuse the dataset to train three small judges. Model selection and the source
   of variation between judges remain open decisions.
5. Evaluate each judge and implement voting over their ratings. Two or more
   matching ratings form a majority; explicitly handle the all-different case
   rather than treating it as agreement. The fallback policy remains to be chosen.
6. Use the validated judges to help evaluate outputs from example 2, alongside
   blind human review. Freeze and record judge versions for writer comparisons.

## What to measure

- Agreement with held-out human ratings, per-class errors, and especially human
  bad / judge great disagreements.
- Whether explanations identify real features of the candidate and follow the rubric.
- Each judge versus the voting result; disagreement frequency and shared errors.
- Training time, inference time, and hardware use for individuals and the panel.
- Sensitivity to superficial cues such as length, and instructions embedded in
  candidate content that should be evaluated rather than obeyed.

Three judges can share the same mistakes, particularly when trained on the same
data. Voting is an experiment to validate, not a substitute for human judgment.
Log datasets, rubric versions, models, seeds, and results in MLflow.

## Scope and open decisions

No model, training framework, or final code layout is selected here. Reasoning
capability and learning to explain judgments are part of the example; long
reasoning traces are not required. Judge-driven optimization of the writer is
not part of this first exercise: using a judge for evaluation is different from
training a writer to maximize its score.

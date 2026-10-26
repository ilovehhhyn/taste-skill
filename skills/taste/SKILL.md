---
name: taste
description: Design, instrument, and interpret ML research experiments with hypothesis-driven diagnostics, training plots, and checkpoint output samples every 10,000 optimizer steps. Use when developing training/evaluation code or analyzing learning dynamics and experimental comparisons.
---

# ML Research Taste

Make the next experiment resolve an uncertainty. Inspect what the model learns, give simple explanations a fair chance, and preserve evidence that changes the story.

## Before changing an experiment

Write a short research note: question, predicted observation, competing explanation, smallest distinguishing experiment, and continue/revise/stop criterion. Preserve the user's objective, scientific geometry, baselines, and resource limits. Tune competing methods appropriately and compare at the intended final budget; an early checkpoint lead can reverse.

When asked to instrument a run, read [diagnostics.md](references/diagnostics.md), inspect the existing training/evaluation loop, and add the smallest compatible logging hook. When asked only to analyze existing artifacts, use those artifacts and report missing evidence; do not launch new training or evaluation implicitly. [sources.md](references/sources.md) explains the research basis; read it only when attribution is needed.

## Required cadence

Produce plots **and actual model-output samples at every 10,000 completed optimizer updates**: 10,000, 20,000, 30,000, and so on. Use a persistent global counter across resumes, not microbatches, gradient-accumulation substeps, epochs, or per-rank counts. Map the framework counter to actual updates, including skipped optimizer steps. Also capture a baseline before new training and a final report, even for runs shorter than 10,000 steps. A user-specified cadence overrides this default.

Integrate this into the training job; a skill is not a background scheduler. Retain cheap scalars at the project's normal logging cadence, then refresh cumulative plots and generate samples at each boundary. Do not wait until step 10,000 to check for an obvious bug or divergence. On resume, preserve the schedule and existing artifacts without duplicate or overwritten reports. If an exact boundary is missed, record the actual model step and the gap; never relabel later weights as an earlier checkpoint.

## What to plot

Use global optimizer step on the main x-axis, with token-count and elapsed-time comparisons where available. Annotate LR, context-length, data-mixture, reward, or code changes. Save the underlying numbers, units, counts, and raw curves alongside any smoothing.

| Diagnostic | Plot and inspect |
|---|---|
| Learning | Train and held-out loss, task success/accuracy, and train–validation gap. Keep token-weighted loss aggregation consistent. Separate optimized reward from independently checked task success. |
| Generation behavior | Answer/output token-length distribution and mean, median, p90, p99; EOS versus length-cap termination, repetition, and malformed/empty outputs. Compare lengths for successes and failures. |
| Optimization and mechanism | LR per parameter group and gradient norm; learned gates/mixing coefficients by layer over training, including signed values and heatmaps. Add activation/update norms only when they test the hypothesis. |
| Efficiency | Tokens/sec or examples/sec, step time, evaluation/generation latency tails, and measured device memory. Distinguish loss per step from quality per token or wall-clock second. |
| Objective-specific evidence | RL: reward components, KL, entropy, clipping, critic diagnostics if present. Retrieval/sequence models: performance by length, difficulty, or retrieval distance. Select only relevant signals. |

Plot all applicable core rows; mark unavailable signals explicitly and explain what instrumentation would supply them. Do not invent coefficients or add a critic to measure one. Avoid expensive tensor dumps or full-vocabulary buffers just for diagnostics.

## What to sample at every boundary

Keep **8 fixed diagnostic inputs plus 8 fresh seeded inputs** by default, using a separate diagnostic split and stable decoding settings. Preserve the fixed input IDs across resumes; keep an untouched test set for final claims. For non-generative models, save input, target, prediction, and error instead of generated text.

Save readable samples and machine-readable records containing actual step/checkpoint identity, input ID and text, target if available, generated output, output-token count, stop reason, and task result. Record model/tokenizer revision, decoding settings, seed, and sample-selection policy once per report. Save full bounded generations; label any display truncation and link the complete record. Print a few representative successes/failures and the artifact path in the job log. Never fabricate outputs or dump the weight tensors as a substitute.

## Interpret and act

At each report, inspect changed samples, behavioral tails, regressions, and suspicious internal transitions. Distinguish observations from explanations. Give one plausible alternative and the smallest useful follow-up; do not start it without authorization. A readable reasoning trace is not proof of the model's internal mechanism.

Verify decoded inputs, labels/loss masks, padding/packing, special tokens, and evaluation configuration before attributing an effect to an algorithm. Restore training mode and isolate diagnostic RNG/data loading; respect distributed collectives. Budget diagnostic generation inside the authorized allocation and preserve OOM evidence. Follow the project's retry limits; do not silently retry, change GPUs, or shrink scientific batch geometry.

Use the project's artifact conventions, or `diagnostics/<run-id>/step-000010000/` with plots, scalar data, `samples.jsonl`, `samples.md`, and a short observation/decision note. Report what is instrumented, what was actually verified, and which signals remain unavailable. Test hooks with cheap synthetic data and simulated boundary/resume cases; that does not establish GPU fit or real model behavior.

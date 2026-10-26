# Diagnostic implementation notes

Read when instrumenting a training/evaluation loop. Adapt to its framework; do not introduce a new tracking service just for this skill.

## Boundary and artifact contract

Run the reporting hook after the completed optimizer update at each positive multiple of 10,000. Track completed reports by run ID, actual global step, and model/checkpoint identity. Resume from the persisted training counter and sample-panel state. Do not replay old boundaries against new weights. Record an instrumentation-start gap for a run first instrumented after step zero.

Test boundary logic without training: 9,999 produces no periodic report; 10,000 produces one; repeated observation of the same completed boundary produces none; resume at 15,000 next reports at 20,000; a final step at 23,000 produces a final report. A skipped optimizer update and accumulation microstep do not advance the update counter. Reuse a report when final and periodic events refer to the same model state. Incomplete reports must remain distinguishable from completed ones; preserve partial evidence on failure.

Reuse existing checkpoint/evaluation machinery. Bind samples and metrics to the same immutable state. A lightweight diagnostic report need not save another full optimizer checkpoint. If disk checkpoints and reporting have different cadences, record the actual evaluated step and weight/code identity; do not imply a restartable checkpoint exists when it does not.

Only one writer should publish artifacts. All ranks needed for sharded forward passes, checkpointing, or metric reductions must participate; rank-zero-only inference can deadlock a sharded model. Reuse the framework's distributed evaluation path. Complete or explicitly fail diagnostics before advancing the model state; asynchronous reporting needs an immutable snapshot and a separately budgeted allocation.

## Metric selection and meaning

| Plot family | Useful detail | Interpretive limit |
|---|---|---|
| Loss and task performance | Raw and smoothed train loss; held-out loss at report boundaries; aligned validation examples; task scores by difficulty/length | Count valid target tokens for loss normalization; do not compare masked/unmasked or differently tokenized losses as identical quantities. |
| Output lengths | Histogram/ECDF plus mean/median/p90/p99 over time; token cap, EOS rate, correct/incorrect slices | Define generated tokens versus prompt tokens and reasoning versus final-answer lengths if separately observable. Cap-hit lengths are censored; 16 samples give noisy tail estimates. Use a larger existing evaluation cohort for quantiles when available and report n. |
| Failure behavior | Repetition, invalid format, empty output, tool errors, truncation | Define the detector and denominator. State the number of examples; do not present a heuristic flag as independently verified task failure. |
| Coefficients | Signed gates, residual/embedding mixing weights, router statistics by layer and update; absolute values alongside normalization | Separate learned coefficients from configured hyperparameters. A scalar's magnitude is not its causal contribution; activation scale can compensate. Test important interpretations with interventions. |
| Optimization | LR groups, pre-clipping gradient norm, clipping frequency; selected activation or update-to-weight norms | Label pre/post-clipping norms; obtain correct global/sharded reductions. Detach and reduce measurements rather than retaining autograd graphs. |
| Cost and memory | Training-only throughput/step time, diagnostic overhead, latency quantiles, device-wide and process-local memory | Separate training, generation, and diagnostic costs. Tiny-sample latency tails are noisy. CPU success and a first optimizer step do not establish GPU fit. |

For RL/preference optimization, add available reward components versus independent success, policy KL against its named reference, entropy, clip fraction, and critic loss/explained variance when a critic exists. Examine success by output length to expose length reward shortcuts. For autoregressive pretraining, a small held-out continuation panel is often sufficient; do not add RL-only metrics. For retrieval or architecture studies, choose controlled slices that challenge the proposed mechanism, such as distractor count, retrieval distance, context length, or composition depth.

## Samples that support comparison

Use fixed diagnostic IDs and separate fresh samples; store panel-selection seed, data revision, and decoding configuration. Freeze the fixed panel once. A dedicated generator or saved/restored RNG state prevents diagnostics from changing training randomness. Separate diagnostic loading from the training iterator. Preserve and restore module training flags and gradient mode even on errors. Do not silently change decoding temperature, token cap, prompt template, or tokenizer mid-comparison.

Example record shape (illustrative, not measured data):

```json
{"global_step":10000,"state_id":"run-a:10000:weights-id","panel":"fixed","input_id":"diag-003","input":"...","target":null,"output":"...","output_tokens":42,"stop_reason":"eos","task_success":null}
```

Persist the full bounded output, not hidden reasoning that the model API does not expose. Store metadata and missing-value reasons separately if convenient. Redact secrets/private data according to project policy; outputs are local artifacts unless the user authorizes publishing. A raw response and reward remain separate from an independently scored outcome.

## GPU fit and measurement

Before authorized diagnostic generation/evaluation, document per-device parameters/precision, gradients and optimizer states (including lazy state creation), activations, KV cache at the diagnostic batch and maximum length, temporary logits/full-vocabulary buffers, and co-resident processes. Mark unmeasured workspace/fragmentation as unknown. Reuse the training model when possible instead of loading a second copy. Do not silently lower the scientific training geometry to fit diagnostics.

Probe device-wide NVML memory and process-local allocated/reserved/peak memory around training, evaluation, generation, and restoration. Preserve the first exact OOM and an allocator snapshot when supported. Use the project's resource recorder (including Cado when available), retaining typed failures and peaks; route failures through its configured alert mechanism and distinguish recorded evidence from confirmed alert delivery. Do not claim GPU fit based on synthetic/CPU checks. Respect project retry limits; after three failed fixes stop and identify the doubtful assumption unless the project's limit is stricter.

## Reading a report

Record observation → competing explanation → next discriminating experiment. Inspect examples responsible for an aggregate change. Check schedule/data/code boundaries before declaring an internal phase transition. Give simple baselines a comparable tuning effort; compare final budgets as well as trajectories. A fixed diagnostic panel supports debugging, not an untouched test-set claim. Preserve negative findings, corrections, and missing evidence.

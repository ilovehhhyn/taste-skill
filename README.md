# taste

A lightweight ML research skill for **Codex and Claude Code**. It asks the coding agent to instrument experiments, inspect model behavior, challenge explanations, and choose informative follow-ups.

**Every 10,000 completed optimizer updates**, produce training plots and actual model-output samples. Also capture a baseline and the final state. The skill is instructions, not an autonomous monitor: the agent must wire the reporting hook into your training code before a run. No model, training framework, tracking service, or API dependency is bundled.

## Install

1. Clone this repository to a location you will keep:

   ```bash
   git clone https://github.com/ilovehhhyn/taste-skill.git
   cd taste-skill
   ```

2. Install for either or both agents, **from that checkout**:

   **Codex — personal installation**

   ```bash
   mkdir -p "$HOME/.agents/skills"
   test ! -e "$HOME/.agents/skills/taste" && test ! -L "$HOME/.agents/skills/taste" &&
     ln -s "$PWD/skills/taste" "$HOME/.agents/skills/taste"
   ```

   **Claude Code — personal installation**

   ```bash
   mkdir -p "$HOME/.claude/skills"
   test ! -e "$HOME/.claude/skills/taste" && test ! -L "$HOME/.claude/skills/taste" &&
     ln -s "$PWD/skills/taste" "$HOME/.claude/skills/taste"
   ```

   These macOS/Linux commands share one checkout and return a nonzero status if the destination already exists, leaving it unchanged. Keep the checkout in place. On systems without symlink support, copy `skills/taste/` into the appropriate `taste/` destination instead, including its references. For a project-only installation, copy it to `<project>/.agents/skills/taste/` for Codex or `<project>/.claude/skills/taste/` for Claude Code; avoid duplicate personal and project installations.

3. Start a new agent session in your **ML project**, then invoke the skill below. To update symlink installations later, run `git pull --ff-only` in this checkout. Copy installations need their files updated separately.

Paths and invocation follow the official [Codex skills documentation](https://learn.chatgpt.com/docs/build-skills) and [Claude Code skills documentation](https://code.claude.com/docs/en/skills). These instructions target local Codex and Claude Code sessions.

## Use

**Codex**

```text
Use $taste to instrument this training loop. Plot the relevant diagnostics and
save actual model outputs every 10,000 optimizer steps. Show the changes and
test the hooks without launching GPU training.
```

**Claude Code**

```text
/taste Instrument this training loop with plots and model-output samples every
10,000 optimizer steps. Test the hooks without launching GPU training.
```

For existing results:

```text
Use the taste skill to analyze these run artifacts. Identify the strongest
observation, competing explanations, and the smallest distinguishing experiment.
Do not launch new runs.
```

The skill can also be selected automatically for relevant ML experiment work. To change the cadence or panel size, say so explicitly, for example: “Use taste with a 2,000-update interval and 4 fixed + 4 fresh samples.” Installing a skill does not grant compute allocation or change a running job.

## What it produces

| Area | Default diagnostics |
|---|---|
| Learning | Train/validation loss, task scores, and generalization gap versus optimizer step; tokens and elapsed time when available |
| Behavior | Output-token length distribution and quantiles, EOS/cap rates, repetition, malformed outputs, success/failure slices |
| Mechanism | LR, gradient norms, and applicable learned gates/mixing coefficients by layer and step |
| Cost and objective | Throughput, latency, memory; relevant RL rewards/KL/entropy/critic or retrieval/length/difficulty slices |
| Samples | 8 fixed + 8 fresh seeded inputs, actual bounded outputs or predictions, targets, lengths, stop reasons, and task outcomes |

Artifacts use the project's conventions or `diagnostics/<run-id>/step-000010000/`: plots, underlying scalar data, `samples.jsonl`, `samples.md`, and a short observation/decision note. Unavailable signals are labeled, not invented. Plot tails from a larger existing evaluation cohort when possible; 16 examples are diagnostic, not a reliable tail estimate.

The hook must preserve training state/RNG, handle distributed evaluation and resumes, and stay within the resource budget. Scalar logs cannot supply missing model outputs; generating fresh outputs from saved checkpoints requires an authorized evaluation and does not recover originally generated samples. CPU/synthetic checks verify wiring, not GPU fit or model quality.

See [SKILL.md](skills/taste/SKILL.md), [implementation notes](skills/taste/references/diagnostics.md), and [research sources](skills/taste/references/sources.md). The cadence is a user-chosen convention; the sources motivate **what to inspect**, not a universal 10,000-step rule.

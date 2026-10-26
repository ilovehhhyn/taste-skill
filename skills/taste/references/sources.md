# Research basis

These sources informed metric selection and experimental judgment. The **10,000-update cadence is the skill author's requested convention**, not a frequency established by these papers. The 8 fixed + 8 fresh sample default is a bounded starting point, not a statistically sufficient evaluation set.

| Source | Practice incorporated |
|---|---|
| [Karpathy: A Recipe for Training Neural Networks](https://karpathy.github.io/2019/04/25/recipe/) and [sample evolution](https://karpathy.github.io/2015/05/21/rnn-effectiveness/) | Inspect real model inputs and generated behavior across training. |
| [Müller: learned mixing coefficients](https://snimu.github.io/2025/08/11/modded-nanogpt-lambdas.html) | Plot coefficients by layer and step, align with schedules, and distinguish speculation from measurement. |
| [Müller: follow-up experiments](https://snimu.github.io/2025/10/30/trying-other-peoples-ideas.html) | Separate per-step loss gains from time-to-target gains. |
| [Cai: generation-cap analysis](https://x.com/jackcai1206/status/2110842873532895612) | Inspect output-length tails and truncation when comparing runtime and accuracy. |
| [Wen, Hall, Ma, Liang: optimizer comparisons](https://arxiv.org/abs/2509.02046) | Fair tuning and final-budget comparison; intermediate rankings can reverse. |
| [SimPO implementation notes](https://github.com/princeton-nlp/SimPO) | Check special tokens, evaluation versions, baseline hyperparameters, and capability tradeoffs. |
| [Xia on simpler data-selection baselines](https://x.com/xiamengzhou/status/1897319460257603809) | Revise a method's scope when simpler alternatives work. |
| [Olah: Research Taste Exercises](https://colah.github.io/notes/taste/) and [Nanda: research taste](https://forum.effectivealtruism.org/posts/puEjjCAHkbrLz8nbc/my-research-process-understanding-and-cultivating-research-taste) | Explicit predictions, competing explanations, feedback, and belief updates. |
| [Measuring Faithfulness in Chain-of-Thought Reasoning](https://www.anthropic.com/research/measuring-faithfulness-in-chain-of-thought-reasoning) | Treat generated reasoning as evidence requiring causal tests, not a guaranteed internal explanation. |

The instructions synthesize these practices; they do not assert that every researcher uses this exact checklist or that logging alone establishes a mechanism.

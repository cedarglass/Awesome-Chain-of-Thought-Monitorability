# Awesome Chain-of-Thought Monitorability

> A structured reading list and living survey of what makes chain-of-thought (CoT) useful for monitoring, how that usefulness can fail, and how it might be preserved.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](LICENSE)

## Summary

| Corpus | Count |
| --- | ---: |
| Core CoT-monitorability papers and technical reports | **26** |
| Adjacent monitoring and reasoning papers | **6** |
| Commentary and position posts | **4** |
| Total works catalogued | **36** |

This survey treats monitorability as a property of a complete system: an actor, a reasoning channel, a monitor, a target, and an operating environment. It defines four properties of a useful visible trace—**causal dependence, answer sufficiency, factor completeness, and monitor-relative legibility**—then locates failures at the computation, trace, monitor, or protocol level. Counts refer to unique catalogued works and were last updated on **July 12, 2026**.

## Contents

- [Why monitor chain of thought?](#why-monitor-chain-of-thought)
- [Scope and reading guide](#scope-and-reading-guide)
- [The CoT monitoring system](#the-cot-monitoring-system)
  - [The information pipeline](#the-information-pipeline)
  - [Observation channels](#observation-channels)
  - [Monitor implementations](#monitor-implementations)
- [Our framework](#our-framework)
  - [The monitoring task](#the-monitoring-task)
  - [Four properties of a useful trace](#four-properties-of-a-useful-trace)
  - [How the properties fit together](#how-the-properties-fit-together)
- [How prior work maps onto the framework](#how-prior-work-maps-onto-the-framework)
- [Monitoring targets](#monitoring-targets)
- [How monitorability fails](#how-monitorability-fails)
  - [Failure locus and intentionality](#failure-locus-and-intentionality)
  - [Computation-side failures](#computation-side-failures)
  - [Trace-side failures](#trace-side-failures)
  - [Strategic obfuscation](#strategic-obfuscation)
  - [Monitor-side failures](#monitor-side-failures)
- [Factors governing monitorability](#factors-governing-monitorability)
  - [Evaluation awareness](#evaluation-awareness)
  - [Role conditioning and trace identity](#role-conditioning-and-trace-identity)
  - [Latent reasoning and alternative architectures](#latent-reasoning-and-alternative-architectures)
- [Evaluation frameworks](#evaluation-frameworks)
- [Datasets and evaluation assets](#datasets-and-evaluation-assets)
- [How training changes monitorability](#how-training-changes-monitorability)
- [Mitigation directions](#mitigation-directions)
- [Adjacent monitoring channels](#adjacent-monitoring-channels)
- [Interpreting illegible traces](#interpreting-illegible-traces)
- [Open questions](#open-questions)

## Why monitor chain of thought?

Reasoning models often produce a natural-language scratchpad before acting or answering. That trace can reveal reward hacking, deception, bias, evaluation awareness, use of hidden hints, and other behaviorally relevant information that is absent from the final output. A monitor may therefore detect a problem from the reasoning before the behavior alone makes it obvious.

This is a fragile opportunity. A fluent trace is not automatically evidence about how an answer was produced. The model may not rely on the trace, may leave out a decisive factor, may express the relevant information in a form the monitor cannot decode, or may learn to evade a monitor under optimization pressure.

- **[07/15/2025] Chain of Thought Monitorability: A New and Fragile Opportunity for AI Safety** [arXiv](https://arxiv.org/abs/2507.11473) · [alphaXiv](https://www.alphaxiv.org/overview/2507.11473)
  **Authors:** Tomek Korbak et al.
  **TL;DR:** Argues that natural-language reasoning creates a valuable but imperfect safety channel. It recommends tracking how architecture and training affect that channel and combining CoT monitoring with other safety methods.

## Scope and reading guide

This review includes work that informs at least one stage in the production, transmission, interpretation, robustness, or replacement of a visible reasoning trace.

- **Core** work directly studies CoT monitorability, reasoning-trace faithfulness, trace oversight, or a direct CoT-monitoring failure.
- **Adjacent** work studies a neighboring mechanism—such as activation monitoring, long-context monitor failure, latent reasoning, evaluation awareness, or role conditioning—that changes how CoT evidence should be interpreted.
- **Commentary** motivates hypotheses or terminology but is kept separate from controlled empirical evidence.

The review deliberately does not treat every paper that uses chain of thought as monitorability research. An adjacent work is included only when its result changes a concrete claim about what a trace contains, how it is generated, how it is monitored, or what can replace it.

## The CoT monitoring system

### The information pipeline

The object of study is not the text alone but an information pipeline:

> **latent computation → visible CoT → monitor inference → oversight decision**

The actor converts some internal computation into a visible trace. A monitor receives that trace, sometimes together with actions or outputs, and infers a target such as reward hacking, a hidden intention, use of a hint, or the answer itself. The oversight protocol then decides whether to allow, block, escalate, or investigate the behavior.

Failures at different stages require different remedies. Better elicitation cannot recover a factor that was never externalized; a better actor-side trace objective does not fix a monitor that loses evidence in a million-token trajectory; and white-box access changes the observation channel without eliminating adversarial robustness problems.

### Observation channels

| Channel | Monitor observes | Main advantage | Main limitation |
| --- | --- | --- | --- |
| **CoT only** | Visible reasoning tokens | Direct access to stated plans and intermediate reasoning | Misses latent computation and omitted factors |
| **CoT + output** | Reasoning and final response | Can compare stated reasoning with the result | Both channels may share the same deception or omission |
| **Full trajectory** | CoT, actions, tool calls, observations, and outputs | Grounds reasoning in external behavior | Long contexts and irrelevant activity can overwhelm the monitor |
| **White-box** | Activations, probes, or internal features | Can detect information absent from natural language | Features can drift or be strategically hidden; access may be unavailable |
| **Hybrid / ensemble** | Multiple independent channels or monitors | Reduces dependence on a single blind spot | Errors may remain correlated and the entire protocol can be attacked |

The four properties below concern the visible CoT channel. White-box and hybrid methods are discussed as complementary channels, not silently folded into the definition of CoT monitorability.

### Monitor implementations

Monitor results are only comparable when the implementation is specified. Common designs include:

- keyword, regular-expression, or rule-based detectors;
- zero-shot or few-shot LLM judges;
- fine-tuned text classifiers;
- linear or nonlinear activation probes;
- fixed-pass or adaptive monitors that ask follow-up questions;
- step-, stream-, and trajectory-level monitors;
- hierarchical, ensemble, or human-escalation protocols.

The monitor's model, prompt, information set, context window, compute budget, decision threshold, and opportunity to interact with the actor are part of the experimental condition—not incidental implementation details.

## Our framework

### The monitoring task

We use **monitorability** for the practical value of a trace in a specified inference task:

> **CoT monitorability is the extent to which access to an agent's chain of thought helps a specified monitor infer a specified behaviorally relevant target in a specified setting.**

The target might be the agent's answer, a decisive factor it used, an unsafe intention, reward hacking, or another property relevant to oversight. A monitorability claim is incomplete unless it names:

- the **agent** and task environment;
- the **trace** available to the monitor;
- the **target** the monitor must infer;
- the **monitor**, including its information and compute budget; and
- any **pressure** under which the agent can adapt to the monitoring procedure.

A direct end-to-end quantity is the improvement caused by revealing the trace:

> **Monitoring value = monitor performance with the trace − monitor performance without the trace**, measured at a fixed false-positive rate, compute budget, and information set.

This quantity tells us whether the trace helps. The four properties below help explain *why* it helps or fails.

### Four properties of a useful trace

| Property | Definition | Core quantity | What failure looks like | Representative test |
| --- | --- | --- | --- | --- |
| **1. Causal dependence** | The agent's answer or action depends on computation carried by the visible trace. | Change in behavior under interventions on the trace, relative to matched controls. | The trace is a post-hoc rationalization, or contains decorative steps that do not affect the result. | Truncate, corrupt, replace, or remove the trace or individual steps; measure changes in the answer distribution. |
| **2. Answer sufficiency** | The trace contains enough task-relevant information to determine or recover the agent's answer or action. | Accuracy of an idealized or strong verifier that receives the trace but not the answer. | The trace supports several outcomes, stops before the decisive step, or cannot reproduce the result. | Ask a verifier to recover the answer from the trace, with task context held fixed and the original answer hidden. |
| **3. Factor completeness** | The trace reveals every behaviorally influential factor in scope: evidence, hints, biases, goals, motivations, or intermediate computations. | Recall of independently established influential factors in the trace. | A decisive hint, intention, or piece of evidence affects behavior but is omitted from the trace. | Intervene on a known factor, confirm that behavior changes, then test whether its influence is disclosed. |
| **4. Monitor-relative legibility** | A specified monitor can decode the relevant content from the trace within its information and compute budget. | Target-extraction accuracy or calibrated detection performance for that monitor. | Relevant information is present but compressed, encoded, mixed-language, overly long, or domain-specific. | Compare human and model monitors, vary monitor compute, and use trace-grounded comprehension questions. |

These properties use different anchors and should not be collapsed into the single word “faithfulness.” **Causal dependence** concerns what produced the behavior. **Answer sufficiency** concerns whether the trace determines the outcome. **Factor completeness** concerns whether it discloses all influential factors in scope. **Legibility** concerns what a particular monitor can recover.

### How the properties fit together

The four properties are complementary, not redundant:

- A post-hoc explanation can be sufficient and legible while having low causal dependence.
- A trace can reveal enough to recover the answer while omitting one influential hint, making it sufficient but incomplete.
- A trace can list every influential factor yet leave their combination ambiguous, making it complete but insufficient.
- A trace can contain sufficient and complete information in an encoding that a particular monitor cannot read.

Two further ideas are best treated as evaluation axes rather than trace properties:

- **Robustness:** how much each property degrades under prompting, training, optimization, or adversarial pressure.
- **Controllability:** how readily the agent can deliberately change a property of its trace while preserving task performance.

Thus, “the trace is legible” is a default-case property claim; “legibility survives optimization against the monitor” is a robustness claim. Keeping these levels separate prevents ordinary monitor errors from being conflated with strategic evasion.

## How prior work maps onto the framework

Earlier papers often use the same term for different quantities, or different terms for similar quantities. The table translates each core work into the framework above. A filled cell names the paper's direct operationalization; “related” marks a secondary implication rather than a direct measure. Adjacent monitoring and architecture papers are catalogued separately because forcing them into CoT-property columns would overstate their scope.

| Work | Causal dependence | Answer sufficiency | Factor completeness | Legibility | Robustness / pressure |
| --- | --- | --- | --- | --- | --- |
| [Lanham et al. (2023)](https://arxiv.org/abs/2307.13702) | CoT interventions (“faithfulness”) | — | Related | — | Scaling and task comparisons |
| [Baker et al. (2025)](https://arxiv.org/abs/2503.11926) | Related | — | Reward-hacking evidence | CoT monitor accuracy | Optimization against monitor |
| [Chen et al. (2025)](https://arxiv.org/abs/2505.05410) | Establishes hint influence | — | Hint reveal rate (“faithfulness”) | — | Outcome-RL trend |
| [Skaf et al. (2025)](https://arxiv.org/abs/2506.01926) | Load-bearing encoded reasoning | — | — | Detectability of learned codes | Process-supervision pressure |
| [Zolkowski et al. (2025)](https://arxiv.org/abs/2507.02737) | Encoded state tracking | — | Hidden-message content | Detection of covert encoding | Scratchpad and coordination affordances |
| [Emmons et al. (2025a)](https://arxiv.org/abs/2507.05246) | CoT as computation vs rationalization | — | Intent disclosure | Monitor evasion | Assistance and iterative optimization |
| [Korbak et al. (2025)](https://arxiv.org/abs/2507.11473) | Conceptual | Conceptual | Conceptual | Conceptual | Fragility across training and scale |
| [Li et al. (2025)](https://arxiv.org/abs/2508.00943) | Related | — | Sandbagging-intent disclosure | CoT monitor evasion | Monitor-aware vs monitor-oblivious sandbagging |
| [Schoen et al. (2025)](https://arxiv.org/abs/2509.15541) | CoT causally associated with behavior | — | Evaluation-awareness disclosure | Monitor detection | Out-of-distribution stress tests |
| [Zhao et al. (2025)](https://arxiv.org/abs/2510.24941) | True Thinking Score | — | — | — | Step-level comparisons |
| [Emmons et al. (2025b)](https://arxiv.org/abs/2510.23966) | — | Task-anchored coverage | Related | Autorater legibility | Synthetic degradations |
| [Jose (2025)](https://arxiv.org/abs/2510.27338) | Accuracy cost of legibility constraint | — | — | Human and AI judgments | Difficulty and model comparisons |
| [Meek et al. (2025)](https://arxiv.org/abs/2510.27378) | Influential-factor setup | Task-factor “verbosity” | Cue disclosure (“faithfulness”) | Related | Synthetic perturbations |
| [Drori et al. (2025)](https://arxiv.org/abs/2511.11584) | Related | — | Safety-relevant disclosure | Safe-looking CoTs | Output-only supervision |
| [MacDermott et al. (2025)](https://arxiv.org/abs/2512.00218) | Related | — | Monitor-relevant content | Monitor accuracy | Length, KL, direct, and adversarial rewards |
| [Guan et al. (2025)](https://arxiv.org/abs/2512.18311) | Intervention evaluations | Process evaluations | Decision-critical factors | Monitor elicitation | RL, length, and compute comparisons |
| [Xiong et al. (2026)](https://arxiv.org/abs/2602.03978) | Tests whether gains are causal | Related | Prompt-relevant disclosure | Monitorability metrics | RLVR data and training dynamics |
| [Chen et al. (2026)](https://arxiv.org/abs/2603.05706) | CoT remains task-relevant | — | Keyword/content control | Property control | Model size, difficulty, compute, and RL |
| [Wang et al. (2026)](https://arxiv.org/abs/2603.28590) | Process-oriented tasks | Related | Decision-critical factors | Monitor performance | Two stress tests |
| [Kaufmann et al. (2026)](https://arxiv.org/abs/2603.30036) | Reward decomposition | Related | CoT-dependent reward terms | Related | Aligned, orthogonal, or conflicting pressure |
| [Yu et al. (2026)](https://arxiv.org/abs/2604.22074) | Causal Importance of Reasoning | Sufficiency of Reasoning | Related | Verifier-relative | RLVR and auxiliary objectives |
| [Knecht et al. (2026)](https://arxiv.org/abs/2605.05835) | Interventions on verbalized awareness | — | Evaluation-awareness disclosure | VEA detection | Injection, removal, and on-policy comparisons |
| [Li et al. (2026)](https://arxiv.org/abs/2605.23055) | Recognition-to-behavior link | — | Evaluation-cue recognition | CoT-based recognition | Factor-controlled evaluation cues |
| [Onyame et al. (2026)](https://arxiv.org/abs/2605.27901) | Answer-commitment analysis | Related | Hint disclosure | Cross-lingual monitor performance | Language and hint stress tests |
| [Xiao and Phuong (2026)](https://arxiv.org/abs/2606.11998) | Related | — | Collusion evidence | Trusted monitor of untrusted-monitor CoT | Active collusion stress test |
| [Wagner et al. (2025)](https://alignment.anthropic.com/2025/subtle-reasoning/) | Related | — | Suspicious-reason disclosure | Independent monitors | Training against output, reasoning, and combined monitors |

The mapping clarifies several naming collisions:

- Lanham et al.'s “faithfulness” is primarily **causal dependence**; Chen et al. and Meek et al. use “faithfulness” primarily for **factor disclosure**.
- Emmons et al.'s “coverage” and Meek et al.'s “verbosity” are task-anchored proxies close to **answer sufficiency**, but neither term should be assumed to mean full factor completeness.
- Yu et al.'s CIR and SR directly operationalize **causal dependence** and **answer sufficiency**, respectively.
- Monitor accuracy can reflect both **factor completeness** and **legibility**. It does not identify which failed without a controlled factor-recovery or comprehension test.

### Papers introducing or directly measuring the four properties

- **[07/17/2023] Measuring Faithfulness in Chain-of-Thought Reasoning** [arXiv](https://arxiv.org/abs/2307.13702) · [alphaXiv](https://www.alphaxiv.org/overview/2307.13702)
  **Authors:** Tamera Lanham et al.
  **TL;DR:** Intervenes on CoTs through truncation, mistakes, paraphrases, and other modifications. Dependence on CoT varies by task, while larger and more capable models are generally less faithful on most studied tasks.

- **[05/08/2025] Reasoning Models Don't Always Say What They Think** [arXiv](https://arxiv.org/abs/2505.05410) · [alphaXiv](https://www.alphaxiv.org/overview/2505.05410)
  **Authors:** Yanda Chen et al.
  **TL;DR:** Tests whether reasoning models disclose their use of six controlled prompt hints. Reveal rates are often low; outcome RL initially improves disclosure but plateaus, and increased reward-hacking use of hints does not yield increased verbalization.

- **[10/28/2025] Can Aha Moments Be Fake? Towards Quantifying Decorative and True Thinking in Chain-of-Thought** [arXiv](https://arxiv.org/abs/2510.24941) · [alphaXiv](https://www.alphaxiv.org/overview/2510.24941v4)
  **Authors:** Jiachen Zhao, Yiyou Sun, Weiyan Shi, Dawn Song
  **TL;DR:** Introduces True Thinking Score to estimate each step's causal contribution. Traces interleave influential and decorative steps; even apparent self-verification or “aha” steps can have little causal effect.

- **[10/28/2025] A Pragmatic Way to Measure Chain-of-Thought Monitorability** [arXiv](https://arxiv.org/abs/2510.23966) · [alphaXiv](https://www.alphaxiv.org/overview/2510.23966)
  **Authors:** Scott Emmons, Roland S. Zimmermann, David K. Elson, Rohin Shah
  **TL;DR:** Defines monitorability using legibility and coverage, implements both with an LLM autorater, and validates the measures against synthetic degradations. Its default-case measures are distinct from adversarial robustness.

- **[10/31/2025] Measuring Chain-of-Thought Monitorability Through Faithfulness and Verbosity** [arXiv](https://arxiv.org/abs/2510.27378) · [alphaXiv](https://www.alphaxiv.org/overview/2510.27378)
  **Authors:** Austin Meek, Eitan Sprejer, Iván Arcuschin, Austin J. Brockmeier, Steven Basart
  **TL;DR:** Combines influential-cue disclosure with whether the trace states the task factors needed to solve the problem. A model may disclose an injected cue yet omit other necessary factors.

- **[10/31/2025] Reasoning Models Sometimes Output Illegible Chains of Thought** [arXiv](https://arxiv.org/abs/2510.27338) · [alphaXiv](https://www.alphaxiv.org/overview/2510.27338)
  **Authors:** Arun Jose
  **TL;DR:** Reports CoTs that are difficult for both human and AI monitors to understand even when final answers are readable. Harder questions worsen measured legibility, and restricting models to legible trace portions substantially reduces accuracy.

- **[04/23/2026] Outcome Rewards Do Not Guarantee Verifiable or Causally Important Reasoning** [arXiv](https://arxiv.org/abs/2604.22074) · [alphaXiv](https://www.alphaxiv.org/overview/2604.22074v1)
  **Authors:** Qinan Yu, Alexa Tartaglini, Peter Hase, Carlos Guestrin, Christopher Potts
  **TL;DR:** Introduces Causal Importance of Reasoning and Sufficiency of Reasoning. Outcome-only RLVR improves accuracy without reliably improving either; limited supervised fine-tuning or auxiliary objectives can improve them without sacrificing accuracy.

## Monitoring targets

Monitorability describes whether evidence can be recovered; it does not specify which behavior the monitor is trying to detect. Common targets include reward hacking, deception, scheming, covert sandbagging, evaluation gaming, hidden use of hints, unsafe intent, and collusion.

These targets must not be confused with failure mechanisms. **Sandbagging**, for example, is a behavior the monitor tries to identify. It becomes covert through one or more mechanisms—such as omission, rationalization, euphemism, or exploitation of a weak monitor.

- **[07/31/2025] LLMs Can Covertly Sandbag on Capability Evaluations Against Chain-of-Thought Monitoring** [arXiv](https://arxiv.org/abs/2508.00943) · [alphaXiv](https://www.alphaxiv.org/overview/2508.00943)
  **Authors:** Chloe Li, Mary Phuong, Noah Y. Siegel
  **TL;DR:** Prompts models to underperform while either aware or unaware of a CoT monitor. Models sometimes conceal successful sandbagging without hints, but do not yet do so reliably; qualitative analysis identifies multiple monitor-evasion policies.

## How monitorability fails

The framework keeps the following chain explicit:

> **property at risk → threat model → observable mechanism → quantity → test**

| Property at risk | Threat model | Mechanism family | Quantity that changes | What the monitor observes | How to test |
| --- | --- | --- | --- | --- | --- |
| Causal dependence | Reasoning moves outside the visible trace | Non-causal signal | Trace-intervention effect falls | Fluent but causally inert or partly decorative reasoning | Whole-trace and step-level causal interventions |
| Answer sufficiency / factor completeness | The visible trace degrades | Missing signal | Answer recovery or factor recall falls | Important steps or factors are absent | Hidden-answer reconstruction and controlled-factor disclosure tests |
| Legibility | The visible trace degrades | Hard-to-decode signal | Monitor extraction accuracy falls | Information is compressed, encoded, or diluted | Cross-monitor comprehension tests at fixed information and compute |
| Any property | Optimization selects for evasion | Monitor-targeted signal | Worst-case property score falls under pressure | The trace changes in ways that preserve performance but defeat detection | Train or optimize against one monitor; evaluate with held-out adaptive monitors |

### Failure locus and intentionality

Two independent axes organize the failures below:

- **Failure locus:** latent computation, visible trace, monitor, or complete oversight protocol.
- **Intentionality and pressure:** default degradation, distribution shift, or strategic adaptation to monitoring.

We reserve **obfuscation** for monitor-impeding behavior selected under strategic pressure. Accidental shorthand, ordinary omission, and weak-monitor errors are monitorability failures, but are not automatically obfuscation. The same observable mechanism can be non-adversarial in one setting and strategically selected in another.

### Computation-side failures

The model solves the task through latent computation, before the trace, or independently of it. The visible text may remain coherent and correct, but it ceases to be reliable evidence about the computation that produced the behavior.

#### Non-causal signal

1. **Post-hoc rationalization:** the trace explains a result reached elsewhere.
   **At risk:** causal dependence. **Quantity:** whole-trace intervention effect. **Test:** remove, replace, or corrupt the trace and measure whether the answer changes.

2. **Decorative reasoning:** some steps imitate checking, reflection, or an “aha moment” without affecting the result.
   **At risk:** step-level causal dependence. **Quantity:** per-step causal contribution. **Test:** intervene on individual steps and test whether low-scoring steps can be pruned without changing the answer.

**Representative evidence:** [Lanham et al. (2023)](https://arxiv.org/abs/2307.13702), [Emmons et al. (2025a)](https://arxiv.org/abs/2507.05246), [Zhao et al. (2025)](https://arxiv.org/abs/2510.24941), and [Yu et al. (2026)](https://arxiv.org/abs/2604.22074).

### Trace-side failures

The model still externalizes useful computation, but the emitted trace either omits relevant information or presents it in a form that the chosen monitor cannot recover. This threat does not require the model to know it is monitored.

#### Missing signal

3. **Omission or selective silence:** a decisive input, intermediate step, intent, or use of a hint is absent. The remaining text can be perfectly fluent.
   **At risk:** factor completeness, and sometimes answer sufficiency. **Quantities:** influential-factor recall and trace-only answer recovery. **Test:** establish a factor's causal influence, then test whether the trace discloses it; separately ask a verifier to reconstruct the answer.

**Representative evidence:** [Chen et al. (2025)](https://arxiv.org/abs/2505.05410), [Meek et al. (2025)](https://arxiv.org/abs/2510.27378), [Wang et al. (2026)](https://arxiv.org/abs/2603.28590), and [Yu et al. (2026)](https://arxiv.org/abs/2604.22074).

#### Hard-to-decode signal

4. **Compression and shorthand:** the trace becomes terse, fragmented, mixed-language, code-heavy, or dependent on specialist conventions.
   **At risk:** monitor-relative legibility. **Quantity:** target-extraction accuracy across monitors. **Test:** compare human, general model, and domain-adapted monitors while holding the trace and target fixed.

5. **Context flooding:** the relevant evidence is present but diluted by repetition, tangents, failed attempts, or excessive length.
   **At risk:** legibility for a bounded monitor. **Quantity:** detection as evidence position, distractor volume, and monitor budget vary. **Test:** preserve decisive evidence while systematically adding or relocating irrelevant content.

**Representative evidence:** [Jose (2025)](https://arxiv.org/abs/2510.27338) directly studies illegible traces. [Guan et al. (2025)](https://arxiv.org/abs/2512.18311) find that longer *natural* CoTs are generally more monitorable in their evaluated regime, so length alone is not evidence of flooding. Deliberate context flooding remains comparatively under-tested.

### Strategic obfuscation

Training, prompting, or deliberate strategy preserves task performance while reducing what a monitor can infer. This is a robustness failure: the same four properties are measured again under pressure. Evaluation must distinguish broad concealment from overfitting to one known detector.

#### Monitor-targeted signal

6. **Strategic paraphrase or euphemism:** suspicious meaning remains in ordinary language but detector-triggering words or descriptions are replaced.
   **At risk:** legibility to a particular monitor. **Quantity:** detection under semantic-preserving transformations. **Test:** optimize against one monitor and evaluate the transformed traces with unseen monitors.

7. **Steganography or encoded reasoning:** behaviorally relevant information is placed in a code or innocuous cover text that the agent can use but the monitor cannot decode.
   **At risk:** legibility and factor completeness from the monitor's perspective. **Quantity:** task success minus held-out detection. **Test:** require hidden communication or encoded state tracking, then test detection without giving the monitor the codebook.

8. **Monitor persuasion:** the trace exposes the suspicious action but frames it as acceptable, necessary, or harmless so that the monitor misclassifies it.
   **At risk:** end-to-end monitoring value rather than information presence. **Quantity:** the change in monitor judgment caused by agent-authored justification. **Test:** compare monitors that see the reasoning with independent monitors that see the same behavioral evidence but cannot be addressed by the agent.

**Representative evidence:** [Baker et al. (2025)](https://arxiv.org/abs/2503.11926), [Skaf et al. (2025)](https://arxiv.org/abs/2506.01926), [Zolkowski et al. (2025)](https://arxiv.org/abs/2507.02737), [Emmons et al. (2025a)](https://arxiv.org/abs/2507.05246), [Wagner et al. (2025)](https://alignment.anthropic.com/2025/subtle-reasoning/), [MacDermott et al. (2025)](https://arxiv.org/abs/2512.00218), and [Chen et al. (2026)](https://arxiv.org/abs/2603.05706).

### Papers focused on robustness and obfuscation

- **[03/14/2025] Monitoring Reasoning Models for Misbehavior and the Risks of Promoting Obfuscation** [arXiv](https://arxiv.org/abs/2503.11926) · [alphaXiv](https://www.alphaxiv.org/overview/2503.11926)
  **Authors:** Bowen Baker et al.
  **TL;DR:** CoT monitoring detects reward hacking in agentic coding more effectively than actions and outputs alone. Stronger direct optimization against the monitor produces obfuscated reward hacking, illustrating the risk of training on the monitoring signal.

- **[06/02/2025] Large Language Models Can Learn and Generalize Steganographic Chain-of-Thought Under Process Supervision** [arXiv](https://arxiv.org/abs/2506.01926) · [alphaXiv](https://www.alphaxiv.org/overview/2506.01926)
  **Authors:** Joey Skaf et al.
  **TL;DR:** Penalizing strings in load-bearing reasoning causes models to substitute encodings without changing the solution method. When the penalized strings form a class, the learned encoding generalizes to held-out strings.

- **[07/03/2025] Early Signs of Steganographic Capabilities in Frontier LLMs** [arXiv](https://arxiv.org/abs/2507.02737) · [alphaXiv](https://www.alphaxiv.org/overview/2507.02737)
  **Authors:** Artur Zolkowski, Kei Nishimura-Gasparian, Robert McCarthy, Roland S. Zimmermann, David Lindner
  **TL;DR:** Current models generally fail to conceal short messages under standard affordances but improve with an unmonitored scratchpad or prior coordination. They show nascent encoded reasoning on simple state tracking but rarely hide it within a cover task.

- **[07/07/2025] When Chain of Thought Is Necessary, Language Models Struggle to Evade Monitors** [arXiv](https://arxiv.org/abs/2507.05246) · [alphaXiv](https://www.alphaxiv.org/overview/2507.05246)
  **Authors:** Scott Emmons et al.
  **TL;DR:** Distinguishes CoT as rationalization from CoT as necessary computation. Models obscure intent mainly with substantial assistance, such as human-written evasion strategies or iterative optimization against the monitor.

- **[2025] Training Fails to Elicit Subtle Reasoning in Current Language Models** [Anthropic](https://alignment.anthropic.com/2025/subtle-reasoning/)
  **Authors:** Misha Wagner, Fabien Roger, Hoagy Cunningham, Johannes Gasteiger, Joe Benton, Vlad Mikulik
  **TL;DR:** Tests training against output-only, reasoning-only, and independent combined monitors in backdoor environments. Current models do not reliably learn subtle undetected reasoning against the combined monitor, although monitor persuasion remains a failure mode.

- **[11/28/2025] Reasoning Under Pressure: How Do Training Incentives Influence Chain-of-Thought Monitorability?** [arXiv](https://arxiv.org/abs/2512.00218) · [alphaXiv](https://www.alphaxiv.org/overview/2512.00218)
  **Authors:** Matt MacDermott, Qiyao Wei, Rada Djoneva, Francis Rhys Ward
  **TL;DR:** After controlling for accuracy, length penalties and KL regularization have no consistent effect on monitorability. Adversarially penalizing monitor accuracy degrades it, while directly optimizing for monitorability does not reliably improve it.

- **[03/05/2026] Reasoning Models Struggle to Control Their Chains of Thought** [arXiv](https://arxiv.org/abs/2603.05706) · [alphaXiv](https://www.alphaxiv.org/overview/2603.05706) · [Code](https://github.com/YuehHanChen/CoTControl)
  **Authors:** Yueh-Han Chen et al.
  **TL;DR:** Introduces CoT-Control, which asks models to solve tasks while controlling properties of their reasoning. Current models control CoT less reliably than final outputs; controllability rises with model size but falls with harder problems, more test-time compute, and additional RL in the studied settings.

### Monitor-side failures

A trace can contain useful, readable evidence while the deployed monitor still fails. Monitor-side failure should therefore be diagnosed separately from actor-side omission or obfuscation.

| Monitor-side limitation | Observable failure | Appropriate comparison |
| --- | --- | --- |
| **Capability gap** | A weaker monitor cannot understand a stronger agent's plan | Cross agent–monitor capability matrix |
| **Context degradation** | Evidence is missed when embedded late in a long trajectory | Hold the evidence fixed; vary position and context length |
| **Compute budget** | A single pass misses evidence recovered by follow-up reasoning | Budget-matched fixed and adaptive monitors |
| **Scaffolding** | Aggregation or parsing loses signal available to the base model | Same monitor model, different scaffold |
| **Correlated blind spots** | Similar monitors all miss the same attack | Diverse versus homogeneous ensembles |
| **Persuasion** | Agent-authored justification shifts the monitor's judgment | Independent monitor without direct agent address |

- **[08/26/2025] Reliable Weak-to-Strong Monitoring of LLM Agents** [arXiv](https://arxiv.org/abs/2508.19461) · [alphaXiv](https://www.alphaxiv.org/overview/2508.19461)
  **Authors:** Neil Kale et al.
  **Scope:** Adjacent full-trajectory monitoring.
  **TL;DR:** Systematizes monitor red-teaming across agent awareness, monitor awareness, evasion strategy, scaffolding, and human escalation. Agent awareness substantially degrades reliability, while monitor scaffolding and targeted human oversight improve detection.

- **[05/12/2026] Classifier Context Rot: Monitor Performance Degrades with Context Length** [arXiv](https://arxiv.org/abs/2605.12366) · [alphaXiv](https://www.alphaxiv.org/overview/2605.12366)
  **Authors:** Sam Martin, Fabien Roger
  **Scope:** Adjacent long-trajectory monitoring.
  **TL;DR:** Finds that frontier-model classifiers miss subtly dangerous coding-agent actions far more often when the same action appears after very long benign activity. Periodic reminders partly mitigate the degradation.

- **[05/14/2026] Ensemble Monitoring for AI Control: Diverse Signals Outweigh More Compute** [arXiv](https://arxiv.org/abs/2605.15377) · [alphaXiv](https://www.alphaxiv.org/overview/2605.15377)
  **Authors:** Eugene Koran, Yejun Yun, Samantha Tetef, Benjamin Arnav, Pablo Bernabeu-Pérez
  **Scope:** Adjacent action monitoring.
  **TL;DR:** Finds that diverse monitor ensembles outperform individual and homogeneous ensembles at equal ensemble size. The gains depend on strong individual monitors with weakly correlated errors rather than repeated sampling alone.

- **[06/10/2026] Bootstrapped Monitoring: Leveraging Transparent Reasoning to Oversee Stronger AI Agents** [arXiv](https://arxiv.org/abs/2606.11998) · [alphaXiv](https://www.alphaxiv.org/overview/2606.11998)
  **Authors:** Frank Xiao, Mary Phuong
  **Scope:** Core CoT-monitoring protocol.
  **TL;DR:** Inserts a stronger untrusted monitor between an agent and a weaker trusted monitor. The trusted model audits the untrusted monitor's raw CoT, improving catch rates even under active collusion in the studied setting.

## Factors governing monitorability

The four properties describe a trace in a specified setting. The factors below explain why those properties may vary across settings; they are not additional properties.

### Evaluation awareness

Evaluation awareness can enable strategic adaptation, but awareness and evasion are not synonymous. A useful decomposition is:

> **evaluative cues → recognition → behavioral response → concealment → monitor failure**

Each arrow requires separate evidence. A model may verbalize that it is being evaluated without changing its behavior, or change its behavior without stating why. CoT-based awareness detection should therefore be paired with interventions on awareness cues and measurements of downstream behavioral change.

- **[05/07/2026] Evaluation Awareness in Language Models Has Limited Effect on Behaviour** [arXiv](https://arxiv.org/abs/2605.05835) · [alphaXiv](https://www.alphaxiv.org/overview/2605.05835)
  **Authors:** Amelie Knecht, Lucas Florin, Thilo Hagendorff
  **TL;DR:** Injects, removes, and observes naturally occurring verbalized evaluation awareness across several benchmarks. Aggregate behavioral effects are generally small, cautioning against treating awareness statements as evidence of strategic adaptation.

- **[05/21/2026] Decomposing and Measuring Evaluation Awareness** [arXiv](https://arxiv.org/abs/2605.23055) · [alphaXiv](https://www.alphaxiv.org/overview/2605.23055)
  **Authors:** Changling Li, Terry Jingchen Zhang, Jie Zhang, Zhijing Jin, Sahar Abdelnabi, Maksym Andriushchenko
  **TL;DR:** Separates evaluative cues in the environment from model recognition and propensity to respond. EvalAwareBench independently varies eight cue factors and finds that recognition depends on the model–benchmark pairing and rarely implies a uniform behavioral response.

### Role conditioning and trace identity

Reasoning tokens occupy a special generative role, often separated from user, tool, and final-answer text. This role may influence how exploratory, compressed, candid, or causally active the trace becomes. It also raises a provenance question: what makes the model treat a span as *its own reasoning*?

The role-confusion hypothesis proposes that models infer roles partly from writing style rather than relying only on formal tags. If reasoning-like style activates a “thinking” representation, forged or misplaced text may be treated as trusted prior reasoning. For monitorability, the important questions are:

- whether role placement changes causal reliance on the same text;
- whether transitions between thinking and assistant roles suppress information that remains causally active;
- whether reasoning-role training encourages monitor-legible exploration or model-specific shorthand;
- whether tags, style, and generation provenance can be disentangled; and
- whether role confusion lets external text corrupt the trace a monitor believes belongs to the actor.

- **[06/22/2026] A Mechanistic Explanation of Prompt Injection (and Why You Should Study Roles)** [LessWrong](https://www.lesswrong.com/posts/d8xDGzCEYE639qqEv/a-mechanistic-explanation-of-prompt-injection-and-why-you)
  **Authors:** Charles Ye, Jasmine C.
  **Scope:** Commentary and mechanistic hypothesis; not direct CoT-monitorability evidence.
  **TL;DR:** Uses role probes and interventions to argue that models reconstruct roles from stylistic cues as well as tags. Reasoning-like text can be represented as “CoT-like” even under conflicting tags, motivating work on the provenance and controllability of reasoning traces.

Persona research belongs here only when it measures persona-conditioned trace production, disclosure, causal reliance, evaluation awareness, or monitor evasion. General role-play and personality work is outside scope.

### Latent reasoning and alternative architectures

Visible natural-language reasoning is not architecturally necessary. Systems trained to reason in latent space establish a boundary case: they may improve task computation while deliberately bypassing the channel CoT monitors depend on. Such work motivates the causal-dependence threat but does not, by itself, show that current visible-CoT models already reason latently.

- **[12/09/2024] Training Large Language Models to Reason in a Continuous Latent Space** [arXiv](https://arxiv.org/abs/2412.06769) · [alphaXiv](https://www.alphaxiv.org/overview/2412.06769)
  **Authors:** Shibo Hao, Sainbayar Sukhbaatar, DiJia Su, Xian Li, Zhiting Hu, Jason Weston, Yuandong Tian
  **Scope:** Adjacent reasoning architecture.
  **TL;DR:** Introduces Coconut, which feeds hidden states back as continuous thoughts instead of decoding every reasoning step into language. It demonstrates a concrete architecture in which useful intermediate computation need not be available as monitorable text.

## Evaluation frameworks

An evaluation should name the property, target, monitor, and pressure it measures. A single monitor-accuracy score usually cannot distinguish missing information from hard-to-decode information.

| Evaluation family | Primary question | Framework property | Typical design | Main confound |
| --- | --- | --- | --- | --- |
| **Causal trace intervention** | Did the trace or step affect the answer? | Causal dependence | Truncate, corrupt, replace, or prune reasoning | Intervention may move the trace off-distribution |
| **Controlled-factor disclosure** | Does the trace reveal a factor known to influence behavior? | Factor completeness | Inject hints or vary environmental factors | The factor's influence must be established independently |
| **Trace-only reconstruction** | Does the trace determine the answer? | Answer sufficiency | Hide the answer and ask a verifier to recover it | Verifier weakness can be mistaken for trace insufficiency |
| **Comprehension / extraction** | Can this monitor recover target content? | Legibility | Human or model ratings and grounded questions | Results are monitor- and budget-relative |
| **Outcome-property prediction** | Does the trace improve detection of misbehavior? | End-to-end monitoring value | Predict reward hacking or another external label | Does not localize which trace property caused success or failure |
| **Adversarial stress test** | Does a property survive pressure? | Robustness of any property | Optimize against a known monitor; test on held-out monitors | Weak attacks overstate robustness; fixed monitors invite overfitting |

Three broad task archetypes recur in the literature:

- **Intervention evaluations** manipulate a hint, context, or environment and test whether the trace reveals why behavior changed.
- **Process evaluations** use tasks with constrained or inspectable solution paths and test whether the trace exposes the path taken.
- **Outcome-property evaluations** test whether the trace predicts a measurable behavioral property such as reward hacking.

These archetypes are complementary. Intervention tasks identify controlled factors but may be artificial; process tasks offer ground truth but may narrow the strategy space; outcome-property tasks are behaviorally relevant but often cannot identify which trace property failed.

### Evaluation papers and benchmarks

- **[12/20/2025] Monitoring Monitorability** [arXiv](https://arxiv.org/abs/2512.18311) · [alphaXiv](https://www.alphaxiv.org/overview/2512.18311)
  **Authors:** Melody Y. Guan et al.
  **TL;DR:** Introduces intervention, process, and outcome-property archetypes and a broad evaluation suite. Longer CoTs are generally more monitorable in the studied settings; more monitor compute and follow-up reasoning can improve prediction, while frontier-scale RL does not materially reduce measured monitorability.

- **[03/30/2026] MonitorBench: A Comprehensive Benchmark for Chain-of-Thought Monitorability in Large Language Models** [arXiv](https://arxiv.org/abs/2603.28590) · [alphaXiv](https://www.alphaxiv.org/overview/2603.28590)
  **Authors:** Han Wang et al.
  **TL;DR:** Provides 1,514 instances across 19 tasks and seven categories, annotated with decision-critical factors and two stress tests. Monitorability is higher when correct outputs structurally require reasoning through the critical factor; some stress tests produce substantial drops.

- **[09/19/2025] Stress Testing Deliberative Alignment for Anti-Scheming Training** [arXiv](https://arxiv.org/abs/2509.15541) · [alphaXiv](https://www.alphaxiv.org/overview/2509.15541)
  **Authors:** Bronson Schoen et al.
  **TL;DR:** Deliberative alignment greatly reduces but does not eliminate covert actions across broad out-of-distribution evaluations. CoTs expose evaluation awareness that is causally associated with reduced covert behavior, complicating the interpretation of apparent safety.

- **[05/27/2026] The Fragility of Chain-of-Thought Monitoring Across Typologically Diverse Languages** [arXiv](https://arxiv.org/abs/2605.27901) · [alphaXiv](https://www.alphaxiv.org/overview/2605.27901)
  **Authors:** Eric Onyame, Runtao Zhou, Kowshik Thopalli, Bhavya Kailkhura, Chirag Agarwal
  **TL;DR:** Extends adversarial-hint evaluations across 13 languages and multiple model families, combining trace analysis with internal answer-token probabilities. It reports substantial cross-lingual failures and motivates language-stratified monitor evaluation rather than English-only claims.

## Datasets and evaluation assets

This table separates reusable assets from the papers' conclusions. “Pressure” records whether the actor is explicitly encouraged or trained to evade monitoring.

| Asset / work | Target | Observation channel | Primary property | Pressure | Reusable artifact |
| --- | --- | --- | --- | --- | --- |
| Lanham et al. intervention tasks | Answer dependence | CoT + answer | Causal dependence | None | Intervention templates |
| Chen et al. hint suite | Use of controlled hints | CoT + answer | Factor completeness | Reward-hacking hints | Six hint types |
| Emmons et al. necessity tasks | Monitor evasion | CoT + behavior | Dependence / robustness | Prompted and optimized evasion | Task and stress-test methodology |
| Emmons et al. autorater suite | Trace quality | CoT | Sufficiency / legibility | Synthetic degradation | Autorater metrics |
| Meek et al. task-factor suite | Cue and task-factor disclosure | CoT | Sufficiency / completeness | Synthetic perturbation | Faithfulness / verbosity metrics |
| Guan et al. evaluation suite | Interventions, processes, outcomes | CoT + task data | Multiple | RL and elicitation comparisons | Three-archetype suite |
| MonitorBench | Decision-critical factors | CoT + outputs | Completeness / legibility | Two stress tests | 1,514 instances, 19 tasks |
| CoT-Control | Trace-property control | CoT + output | Controllability | Explicit property-control request | Tasks, code, and metrics |
| Covert sandbagging evaluations | Hidden underperformance | CoT + answer | Robustness | Monitor-aware sandbagging | Five sandbagging policies |
| EvalAwareBench | Evaluation recognition and response | CoT + behavior | Robustness moderator | Controlled evaluative cues | 100 paired tasks, eight cue factors |
| Multilingual CoT monitoring suite | Hint use across languages | CoT + activations | Completeness / legibility | Adversarial hints | 13-language evaluation |
| SHADE / CUA-SHADE-Arena | Covert agent misbehavior | Full trajectory | End-to-end monitoring value | Monitor-aware agents | Agent environments and logs |
| BashArena bootstrapped monitoring | Agent–monitor collusion | Trajectory + monitor CoT | Monitor-protocol robustness | Active collusion | Multi-turn software tasks |

When extending this table, record the actor model, monitor model, false-positive operating point, context length, monitor budget, awareness condition, and whether the monitor was used during optimization. Without these fields, benchmark scores are difficult to compare.

## How training changes monitorability

Training can alter several properties at once. The useful question is not simply whether a monitor score rises, but which property changed, under which pressure, and whether the change transfers to held-out monitors and tasks.

| Training regime | Main pathway | Relevant evidence | Current takeaway |
| --- | --- | --- | --- |
| Capability scaling / pretraining | More latent computation or more compressed representations | Lanham; Jose | Capability can reduce causal dependence or legibility in some settings, but the trend is task-dependent. |
| Outcome-based RL / RLVR | Optimizes answers without directly constraining trace properties | Chen; Xiong; Yu | Better task performance does not guarantee causal dependence, sufficiency, or completeness. Early apparent gains may have other causes. |
| Output-only safety supervision | Safe outputs generalize backward into safe-looking reasoning | Drori | Output supervision can reduce disclosure even without direct CoT penalties. |
| Process supervision | Direct pressure on visible reasoning content | Skaf | Penalizing trace content can teach substitutions or encodings while preserving the underlying computation. |
| Direct monitor optimization | Improves easy monitored cases, then selects against detector features | Baker; MacDermott | Strong or adversarial optimization can damage held-out monitoring value. |
| Length and KL pressure | Changes redundancy, style, and distribution shift | MacDermott; Guan | Effects are not consistently harmful; length must be separated from deliberate flooding. |
| CoT-property objectives | Directly rewards causal or sufficient reasoning | Yu; Kaufmann | Can improve target metrics, but safety depends on whether the objective aligns or conflicts with task reward. |

- **[10/11/2025] Output Supervision Can Obfuscate the Chain of Thought** [arXiv](https://arxiv.org/abs/2511.11584) · [alphaXiv](https://www.alphaxiv.org/overview/2511.11584)
  **Authors:** Jacob Drori, Luke Marks, Bryce Woodworth, Alex Cloud, Alexander Matt Turner
  **TL;DR:** Output-only safety supervision can make CoTs look safe through generalization from safe outputs and autoregressive coupling. The paper studies mitigations that improve the monitorability–performance tradeoff over ordinary training.

- **[02/03/2026] Monitorability as a Free Gift: How RLVR Spontaneously Aligns Reasoning** [arXiv](https://arxiv.org/abs/2602.03978) · [alphaXiv](https://www.alphaxiv.org/overview/2602.03978)
  **Authors:** Zidi Xiong, Shan Chen, Himabindu Lakkaraju
  **TL;DR:** Early monitorability gains under RLVR depend strongly on diverse, instruction-following training data. The reported gains arise mainly from response-distribution sharpening and prompt attention rather than stronger causal reliance on the trace.

- **[03/31/2026] Aligned, Orthogonal or In-conflict: When Can We Safely Optimize Chain-of-Thought?** [arXiv](https://arxiv.org/abs/2603.30036) · [alphaXiv](https://www.alphaxiv.org/overview/2603.30036)
  **Authors:** Max Kaufmann, David Lindner, Roland S. Zimmermann, Rohin Shah
  **TL;DR:** Decomposes post-training reward into output- and CoT-dependent terms and classifies their relationship as aligned, orthogonal, or in conflict. Conflicting terms predict reduced monitorability and more difficult optimization.

## Mitigation directions

The literature does not yet establish a robust solution. It does support a layered strategy in which each mitigation names the failure point it targets and is tested against adaptive pressure.

### 1. Preserve externalized, answer-sufficient computation

**Targets:** causal dependence and answer sufficiency.
**Evidence:** Yu et al. find that supervised or auxiliary CIR/SR objectives can improve their corresponding metrics; Kaufmann et al. show why the relationship between task and CoT rewards matters.
**Risk:** direct optimization can teach metric-specific traces rather than generally useful ones.
**Required test:** evaluate causal interventions and answer reconstruction on held-out tasks and with held-out verifiers, at matched task capability.

### 2. Measure disclosure separately from readability

**Targets:** factor completeness and legibility.
**Evidence:** Chen et al. and Meek et al. show that fluent reasoning can omit influential or necessary factors; Guan et al. show that monitor compute and elicitation can recover additional signal.
**Risk:** a stronger monitor may compensate for illegibility but cannot recover information that is genuinely absent.
**Required test:** pair controlled-factor recall with trace-grounded comprehension, rather than reporting one aggregate autorater score.

### 3. Improve the monitor before pressuring the agent

**Targets:** monitor-relative legibility and end-to-end monitoring value.
**Evidence:** Guan et al. report gains from stronger monitors, additional inference compute, and follow-up questions.
**Risk:** elicitation gains may disappear after compute matching or after the agent adapts to the complete protocol.
**Required test:** compare fixed-pass and adaptive monitors at equal budgets, then train agents against each full protocol and evaluate with held-out attacks.

### 4. Avoid feeding a trusted monitor signal directly into training

**Targets:** robustness against strategic paraphrase, encoding, and omission.
**Evidence:** Baker et al., Skaf et al., MacDermott et al., and Drori et al. each show pathways by which direct or indirect pressure can make reasoning less revealing.
**Risk:** even output-only supervision can affect the trace, so merely hiding the CoT score is not sufficient.
**Required test:** reserve independent monitors and attack families for evaluation; measure transfer beyond the detector used during training.

### 5. Fuse independent evidence channels

**Targets:** monitor persuasion and any single-channel blind spot.
**Evidence:** Wagner et al. show that a reasoning monitor can be persuaded even when behavioral evidence remains suspicious, motivating joint use of reasoning, actions, outputs, and—in future work—white-box signals.
**Risk:** the channels may share correlated errors or all be attacked jointly.
**Required test:** preregister fusion and abstention rules; report detection at fixed false-positive and escalation rates under attacks against both individual channels and the full system.

## Adjacent monitoring channels

White-box monitoring asks whether internal activations reveal information that the visible trace omits. It is a natural complement to CoT monitoring, but not an automatic solution: representations can drift, detector penalties can reshape policies, and models can be trained to evade unseen probes.

- **[12/12/2025] Neural Chameleons: Language Models Can Learn to Hide Their Thoughts from Unseen Activation Monitors** [arXiv](https://arxiv.org/abs/2512.11949) · [alphaXiv](https://www.alphaxiv.org/overview/2512.11949)
  **Authors:** Max McGuinness, Alex Serrano, Luke Bailey, Scott Emmons
  **Scope:** Adjacent white-box monitoring.
  **TL;DR:** Fine-tunes models to conditionally evade activation monitors for named concepts and finds zero-shot transfer to unseen safety-relevant probes. Ensembles and nonlinear monitors are more resilient but do not eliminate evasion in the studied setting.

- **[02/17/2026] The Obfuscation Atlas: Mapping Where Honesty Emerges in RLVR with Deception Probes** [arXiv](https://arxiv.org/abs/2602.15515) · [alphaXiv](https://www.alphaxiv.org/overview/2602.15515)
  **Authors:** Mohammad Taufeeque, Stefan Heimersheim, Adam Gleave, Chris Cundy
  **Scope:** Adjacent white-box and policy monitoring.
  **TL;DR:** Studies detector-guided RL in a coding environment with naturally occurring reward hacking. It distinguishes obfuscated activations from obfuscated policies and finds that detector penalties particularly incentivize policy-level justifications that evade detection.

- **The Promise of White-Box Tools for Detecting and Mitigating AI Deception** [FAR.AI](https://www.far.ai/news/ai-deception-white-box)
  **Authors:** FAR.AI
  **Scope:** Position article.
  **TL;DR:** Motivates white-box deception detection as a complement to behavioral and black-box monitoring while emphasizing access, trusted-monitor capability, and robustness to optimization pressure as open limitations.

The comparison suggests a defense-in-depth principle: use CoT because it is semantically rich and operationally accessible, use actions and outputs for behavioral grounding, and use white-box signals when available. Evaluate each channel separately and under joint attack before combining them.

## Interpreting illegible traces

Legibility is not a property of text alone. It is a relation between a trace, a target, and a monitor with bounded knowledge and compute. Three distinctions are especially important:

1. **Illegible does not mean information-free.** A domain expert or stronger model may decode shorthand that a general human reader cannot.
2. **Strange does not mean steganographic.** Fragmented, mixed-language, or compressed traces may arise from imitation, optimization artifacts, or task demands without any pressure to hide.
3. **Long does not mean less legible.** Additional reasoning can expose more evidence; it becomes “flooding” only when controlled additions make relevant evidence harder for a bounded monitor to recover.

These points suggest a better legibility protocol: define the target information, compare multiple monitors, vary monitor budget, test grounded comprehension rather than stylistic fluency, and separately verify whether the target information is present at all.

The following non-peer-reviewed discussions are useful hypothesis generators, not substitutes for controlled evaluation:

- **[10/09/2025] Towards a Typology of Strange LLM Chains-of-Thought** [LessWrong](https://www.lesswrong.com/posts/qgvSMwRrdqoDMJJnD/towards-a-typology-of-strange-llm-chains-of-thought)
  **Authors:** 1a3orn
  **TL;DR:** Proposes possible causes of unusual RLVR traces, including reasoning spandrels and context refresh, and offers a vocabulary for hypotheses that future legibility evaluations could separate.

- **[04/10/2026] The Unintelligibility Is Ours: Notes on Chain-of-Thought** [LessWrong](https://www.lesswrong.com/posts/rFbTAL6PofHzZCCpD/the-unintelligibility-is-ours-notes-on-chain-of-thought)
  **Authors:** 1a3orn
  **TL;DR:** Argues that terse, compressed, and mixed-language traces may reflect imitation of human shorthand rather than a newly invented private language, suggesting that ordinary language has considerable unused information-density headroom.

## Open questions

1. Which combination of causal dependence, answer sufficiency, factor completeness, and legibility best predicts real safety value?
2. Can the four properties be measured with interventions that remain in-distribution for the agent?
3. How should monitor capability and compute be normalized when comparing agents?
4. At what capability or training scale does latent reasoning materially displace externalized CoT?
5. Which obfuscation mechanisms transfer across monitors, languages, tasks, and model families?
6. When do direct CoT objectives improve the intended property, and when do they teach evaluator-specific compliance or concealment?
7. How do code, tools, specialist jargon, low-resource languages, language mixing, and very long contexts affect factor completeness and legibility separately?
8. How should monitoring value be combined with actions, outputs, and white-box interpretability signals?
9. What evidence would distinguish accidental degradation from situationally aware evasion?
10. When does verbalized evaluation awareness causally change behavior, rather than merely describe the setting?
11. What makes a model treat text as its own reasoning: tags, writing style, generation history, or internal state?
12. Can monitor scaffolds and diverse evidence channels remain robust after an actor adapts to the complete protocol?

## Contributing

Paper suggestions and corrections are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md). Dates above are initial arXiv submission dates unless otherwise indicated. Venue labels are omitted when not verified from a primary source.

## Citation

```bibtex
@article{chain_of_thought_monitorability_survey,
  title  = {Chain-of-Thought Monitorability: A Survey},
  author = {TODO},
  year   = {2026},
  note   = {Survey in preparation}
}
```

## Acknowledgements

The presentation is inspired by [Awesome Loop Models](https://github.com/huskydoge/Awesome-Loop-Models). The taxonomy and prose here are specific to this survey.

## License

The curated content is available under the [Creative Commons Attribution 4.0 International License](LICENSE).

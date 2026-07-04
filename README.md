# Awesome Chain-of-Thought Monitorability

> A structured reading list and living survey of what makes chain-of-thought (CoT) monitorable, how monitorability may fail, how training changes it, and how it might be preserved.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](LICENSE)

## Contents

- [The fragile opportunity](#the-fragile-opportunity)
- [Defining CoT monitorability](#defining-cot-monitorability)
- [A property-based taxonomy of threats](#a-property-based-taxonomy-of-threats)
  - [Threat 1: models stop relying on CoT](#threat-1-models-stop-relying-on-cot)
  - [Threat 2: CoT becomes illegible or incomplete](#threat-2-cot-becomes-illegible-or-incomplete)
  - [Threat 3: models reshape CoT to defeat monitors](#threat-3-models-reshape-cot-to-defeat-monitors)
  - [Types of obfuscation](#types-of-obfuscation)
- [How training may change monitorability](#how-training-may-change-monitorability)
- [Evaluation frameworks and empirical trends](#evaluation-frameworks-and-empirical-trends)
- [Mitigations](#mitigations)
- [Commentary on legibility](#commentary-on-legibility)
- [Open questions](#open-questions)

## The fragile opportunity

Reasoning models often produce a natural-language scratchpad before their final answer. This can expose reward hacking, deception, bias, use of hidden hints, and other decision-relevant information absent from actions or final outputs. CoT monitoring may therefore support scalable oversight, but only while the trace remains causally connected to behavior, sufficiently complete, legible to a monitor, and resistant to pressure to conceal information.

- **[07/15/2025] Chain of Thought Monitorability: A New and Fragile Opportunity for AI Safety** [arXiv](https://arxiv.org/abs/2507.11473) · [alphaXiv](https://www.alphaxiv.org/overview/2507.11473)  
  **Authors:** Tomek Korbak et al.  
  **TL;DR:** Position paper arguing that natural-language CoTs create a valuable but imperfect safety channel. It recommends tracking the effect of architecture and training decisions on monitorability and combining CoT monitoring with other safety methods.

## Defining CoT monitorability

We use the following umbrella definition:

> **CoT monitorability is the degree to which a monitor can use an agent's chain of thought to infer a behaviorally relevant property of that agent in a specified setting.**

Monitorability is therefore relational, not an intrinsic score attached only to a trace. A claim about monitorability should specify the **agent**, **monitor**, **observation available to the monitor**, **property being inferred**, **environment**, and **optimization or adversarial pressure**.

Existing definitions emphasize different necessary properties:

| Work | Proposed ingredients | Central question |
| --- | --- | --- |
| Emmons et al. | **Legibility + coverage** | Can a monitor follow the trace, and does the trace contain the reasoning needed to obtain the output? |
| Meek et al. | **Faithfulness + verbosity** | Does the trace disclose influential factors, and does it state every factor needed to solve the task? |
| Yu et al. | **Causal importance + sufficiency** | Does the answer depend on the trace, and can an external verifier recover an unambiguous answer from it? |

These definitions overlap but are not interchangeable. A trace can be readable but omit a decisive hint; complete but causally irrelevant; causally important but encoded illegibly; or informative by default but easy to reshape under optimization. This divergence motivates a survey organized by measurable properties and corresponding threat models rather than by a single scalar definition.

- **[10/28/2025] A Pragmatic Way to Measure Chain-of-Thought Monitorability** [arXiv](https://arxiv.org/abs/2510.23966) · [alphaXiv](https://www.alphaxiv.org/overview/2510.23966)  
  **Authors:** Scott Emmons, Roland S. Zimmermann, David K. Elson, Rohin Shah  
  **TL;DR:** Defines monitorability through legibility and coverage, implements both with an LLM autorater, and validates the measures against synthetic degradations. The reported default-case results are deliberately distinct from adversarial robustness.

- **[10/31/2025] Measuring Chain-of-Thought Monitorability Through Faithfulness and Verbosity** [arXiv](https://arxiv.org/abs/2510.27378) · [alphaXiv](https://www.alphaxiv.org/overview/2510.27378)  
  **Authors:** Austin Meek, Eitan Sprejer, Iván Arcuschin, Austin J. Brockmeier, Steven Basart  
  **TL;DR:** Combines faithfulness with verbosity, defined as whether the CoT states every factor needed to solve the task. Models may disclose an injected cue yet remain difficult to monitor because they omit other necessary factors.

- **[04/23/2026] Outcome Rewards Do Not Guarantee Verifiable or Causally Important Reasoning** [arXiv](https://arxiv.org/abs/2604.22074) · [alphaXiv](https://www.alphaxiv.org/overview/2604.22074v1)  
  **Authors:** Qinan Yu, Alexa Tartaglini, Peter Hase, Carlos Guestrin, Christopher Potts  
  **TL;DR:** Introduces Causal Importance of Reasoning (CIR) and Sufficiency of Reasoning (SR). Outcome-only RLVR improves accuracy without reliably improving either property; limited supervised fine-tuning or auxiliary CIR/SR rewards can improve them without sacrificing accuracy.

## A property-based taxonomy of threats

| Threat model | Property at risk | Representative operationalization |
| --- | --- | --- |
| The model solves tasks without using its written CoT | **Dependency / causal faithfulness** | Truncate, corrupt, replace, or intervene on the trace and measure answer changes |
| The model uses CoT but omits decisive information | **Coverage / completeness / sufficiency** | Test whether a verifier can recover the answer or influential factors from the trace alone |
| The trace becomes difficult for the observer to understand | **Monitor-relative legibility** | Measure human or model-monitor comprehension while controlling for task performance |
| The model can strategically reshape a load-bearing CoT | **Controllability / adversarial robustness** | Optimize against a monitor or penalize concepts in CoT and test whether behavior persists undetected |

The rows should be evaluated separately. For example, success on a hint-revelation task supports a narrow faithfulness claim; it does not establish legibility in code environments or robustness to a model deliberately optimizing against the monitor.

### Threat 1: models stop relying on CoT

A model may acquire enough latent capacity to solve a task before, or independently of, its written reasoning. The visible trace can then become a post-hoc explanation. Relevant measurements ask whether reasoning tokens causally influence the answer and whether all decision-relevant steps—including atypical steps induced by hints—are disclosed.

- **[07/17/2023] Measuring Faithfulness in Chain-of-Thought Reasoning** [arXiv](https://arxiv.org/abs/2307.13702) · [alphaXiv](https://www.alphaxiv.org/overview/2307.13702)  
  **Authors:** Tamera Lanham et al.  
  **TL;DR:** Intervenes on CoTs by adding mistakes, paraphrasing, truncating, and other modifications. Dependence on CoT varies strongly by task, while larger and more capable models are generally less faithful on most studied tasks.

- **[05/08/2025] Reasoning Models Don't Always Say What They Think** [arXiv](https://arxiv.org/abs/2505.05410) · [alphaXiv](https://www.alphaxiv.org/overview/2505.05410)  
  **Authors:** Yanda Chen, Joe Benton, Ansh Radhakrishnan, Jonathan Uesato, Carson Denison, John Schulman, Arushi Somani, Peter Hase, Misha Wagner, Fabien Roger, Vlad Mikulik, Samuel R. Bowman, Jan Leike, Jared Kaplan, Ethan Perez  
  **TL;DR:** Tests whether reasoning models verbalize their use of six prompt hints. Reveal rates are often below 20%; outcome RL initially improves faithfulness but plateaus, and increased reward-hacking use of hints does not yield increased verbalization.

- **[10/28/2025] Can Aha Moments Be Fake? Towards Quantifying Decorative and True Thinking in Chain-of-Thought** [arXiv](https://arxiv.org/abs/2510.24941) · [alphaXiv](https://www.alphaxiv.org/overview/2510.24941v4)  
  **Authors:** Jiachen Zhao, Yiyou Sun, Weiyan Shi, Dawn Song  
  **TL;DR:** Proposes True Thinking Score to estimate each step's causal contribution and finds that traces interleave influential and decorative steps. Even apparent self-verification or “aha” steps can have little causal effect.

- **[07/07/2025] When Chain of Thought Is Necessary, Language Models Struggle to Evade Monitors** [arXiv](https://arxiv.org/abs/2507.05246) · [alphaXiv](https://www.alphaxiv.org/overview/2507.05246)  
  **Authors:** Scott Emmons, Erik Jenner, David K. Elson, Rif A. Saurous, Senthooran Rajamanoharan, Heng Chen, Irhum Shafkat, Rohin Shah  
  **TL;DR:** Distinguishes CoT as rationalization from CoT as necessary computation. In its stress tests, models obscure intent mainly with substantial assistance such as human-written evasion strategies or iterative optimization against the monitor.

### Threat 2: CoT becomes illegible or incomplete

Even a causally load-bearing trace may be unhelpful if it is too compressed, domain-specific, multilingual, internally referential, or long for the chosen monitor. Legibility is always relative to a monitor: a trace may be opaque to a human but usable by a stronger model monitor. Coverage is separate: understandable text may still omit decisive information.

- **[10/31/2025] Reasoning Models Sometimes Output Illegible Chains of Thought** [arXiv](https://arxiv.org/abs/2510.27338) · [alphaXiv](https://www.alphaxiv.org/overview/2510.27338)  
  **Authors:** Arun Jose  
  **TL;DR:** Studies 14 reasoning models and reports CoTs that are illegible to human and AI monitors even when final answers are readable. Harder questions worsen legibility, and forcing models to use only legible portions substantially reduces accuracy.

### Threat 3: models reshape CoT to defeat monitors

A model may still depend on externalized reasoning but learn a representation that preserves task performance while defeating a particular monitor. We call this **controllability**: how easily training pressure, prompting, or deliberate strategy can steer the trace's monitor-relevant properties. This includes both explicit optimization against a monitor and indirect pressure from output or process supervision.

- **[03/05/2026] Reasoning Models Struggle to Control Their Chains of Thought** [arXiv](https://arxiv.org/abs/2603.05706) · [alphaXiv](https://www.alphaxiv.org/overview/2603.05706) · [Code](https://github.com/YuehHanChen/CoTControl)  
  **Authors:** Yueh-Han Chen, Robert McCarthy, Bruce W. Lee, He He, Ian Kivlichan, Bowen Baker, Micah Carroll, Tomek Korbak  
  **TL;DR:** Introduces CoT-Control, which asks models to solve tasks while controlling properties of their reasoning. Current models control CoT much less reliably than final outputs; controllability rises with model size but falls with harder problems, more test-time compute, and additional RL training in the studied settings.

- **[03/14/2025] Monitoring Reasoning Models for Misbehavior and the Risks of Promoting Obfuscation** [arXiv](https://arxiv.org/abs/2503.11926) · [alphaXiv](https://www.alphaxiv.org/overview/2503.11926)  
  **Authors:** Bowen Baker, Joost Huizinga, Leo Gao, Zehao Dou, Melody Y. Guan, Aleksander Madry, Wojciech Zaremba, Jakub Pachocki, David Farhi  
  **TL;DR:** CoT monitoring detects reward hacking in agentic coding more effectively than monitoring actions and outputs alone. Stronger direct optimization against the monitor yields obfuscated reward hacking, showing the risk of training on the monitoring signal.

- **[06/02/2025] Large Language Models Can Learn and Generalize Steganographic Chain-of-Thought Under Process Supervision** [arXiv](https://arxiv.org/abs/2506.01926) · [alphaXiv](https://www.alphaxiv.org/overview/2506.01926)  
  **Authors:** Joey Skaf, Luis Ibanez-Lissen, Robert McCarthy, Connor Watts, Vasil Georgiv, Hannes Whittingham, Lorena Gonzalez-Manzano, David Lindner, Cameron Tice, Edward James Young, Puria Radmard  
  **TL;DR:** Penalizing strings in load-bearing reasoning causes models to substitute encodings without changing the solution method. When penalized strings form a class, the learned encoding generalizes to held-out strings.

- **[2025] Training Fails to Elicit Subtle Reasoning in Current Language Models** [Anthropic](https://alignment.anthropic.com/2025/subtle-reasoning/)  
  **Authors:** Misha Wagner, Fabien Roger, Hoagy Cunningham, Johannes Gasteiger, Joe Benton, Vlad Mikulik  
  **TL;DR:** Tests training against output-only, reasoning-only, and independent combined monitors in backdoor environments. Current models do not reliably learn subtle undetected reasoning against the combined monitor, though monitor persuasion remains a failure mode and the result may not extrapolate to production-scale RL.

- **[11/28/2025] Reasoning Under Pressure: How Do Training Incentives Influence Chain-of-Thought Monitorability?** [arXiv](https://arxiv.org/abs/2512.00218) · [alphaXiv](https://www.alphaxiv.org/overview/2512.00218)  
  **Authors:** Matt MacDermott, Qiyao Wei, Rada Djoneva, Francis Rhys Ward  
  **TL;DR:** After controlling for accuracy, length penalties and KL regularization have no consistent effect on monitorability. Adversarially penalizing monitor accuracy degrades it, while directly optimizing for monitorability does not reliably improve it.

### Types of obfuscation

The literature raises several distinct mechanisms:

- **Omission or selective silence:** decisive evidence, intent, or use of hints is absent.
- **Post-hoc rationalization:** the model has already selected an answer and generates plausible but causally irrelevant reasoning.
- **Decorative reasoning:** individual steps look useful but have little causal influence.
- **Compression and shorthand:** abbreviated, mixed-language, or domain-specific traces exceed the monitor's interpretive ability.
- **Steganography or encoded reasoning:** information is hidden in an alternate code or innocuous-looking text.
- **Monitor persuasion:** the trace argues that suspicious behavior is benign instead of hiding the behavior itself.
- **Strategic paraphrase or euphemism:** harmful intent is represented in language that preserves function while evading a detector.
- **Context flooding:** very long traces dilute or bury monitor-relevant evidence.

- **[07/03/2025] Early Signs of Steganographic Capabilities in Frontier LLMs** [arXiv](https://arxiv.org/abs/2507.02737) · [alphaXiv](https://www.alphaxiv.org/overview/2507.02737)  
  **Authors:** Artur Zolkowski, Kei Nishimura-Gasparian, Robert McCarthy, Roland S. Zimmermann, David Lindner  
  **TL;DR:** Current models generally fail to conceal short messages under standard affordances but improve with an unmonitored scratchpad or prior coordination. They show nascent encoded reasoning on simple state tracking but rarely hide it within a cover task.

## How training may change monitorability

Training can change several properties at once. The key distinction is between ordinary capability training, indirect pressure from output-only objectives, direct process supervision, and adversarial optimization against the monitor.

| Training regime | Plausible effect on monitorability |
| --- | --- |
| Pretraining | English-heavy data may favor natural-language reasoning; greater capability may also increase latent reasoning or compressed shorthand |
| Outcome-based RL / RLVR | Can improve task performance without guaranteeing causal reliance, coverage, or sufficiency |
| Output supervision | May indirectly make CoT look safe through generalization and autoregressive coupling |
| Process supervision | Can improve reasoning quality, but penalties on trace content may teach substitutions or encodings |
| Direct monitor optimization | May improve easy cases, then select for monitor-specific obfuscation under stronger pressure |
| Length pressure | May reduce redundancy but can remove evidence or encourage compressed, less legible reasoning |

- **[10/11/2025] Output Supervision Can Obfuscate the Chain of Thought** [arXiv](https://arxiv.org/abs/2511.11584) · [alphaXiv](https://www.alphaxiv.org/overview/2511.11584)  
  **Authors:** Jacob Drori, Luke Marks, Bryce Woodworth, Alex Cloud, Alexander Matt Turner  
  **TL;DR:** Output-only safety supervision can make CoTs look safe through generalization from safe outputs and autoregressive reinforcement of safe-looking earlier tokens. The paper proposes mitigations that improve the monitorability–performance tradeoff over ordinary training.

- **[02/03/2026] Monitorability as a Free Gift: How RLVR Spontaneously Aligns Reasoning** [arXiv](https://arxiv.org/abs/2602.03978) · [alphaXiv](https://www.alphaxiv.org/overview/2602.03978)  
  **Authors:** Zidi Xiong, Shan Chen, Himabindu Lakkaraju  
  **TL;DR:** Early monitorability gains under RLVR depend strongly on diverse, instruction-following training data. The reported gains arise mainly from response-distribution sharpening and prompt attention rather than stronger causal reliance on the reasoning trace.

- **[03/31/2026] Aligned, Orthogonal or In-conflict: When Can We Safely Optimize Chain-of-Thought?** [arXiv](https://arxiv.org/abs/2603.30036) · [alphaXiv](https://www.alphaxiv.org/overview/2603.30036)  
  **Authors:** Max Kaufmann, David Lindner, Roland S. Zimmermann, Rohin Shah  
  **TL;DR:** Decomposes post-training reward into output- and CoT-dependent terms and classifies their relationship as aligned, orthogonal, or in-conflict. In-conflict terms predict reduced monitorability and difficulty of optimization.

## Evaluation frameworks and empirical trends

A useful evaluation must say which property it measures and under what threat model. Current frameworks include:

- **Intervention evaluations:** alter hints, context, or environment and test whether the trace reveals why behavior changed.
- **Process evaluations:** use tasks with constrained solution paths and test whether a monitor can recover the path taken.
- **Outcome-property evaluations:** ask whether the trace predicts a measurable output property such as reward hacking.
- **Causal trace interventions:** truncate, corrupt, paraphrase, replace, or prune reasoning and measure the effect on answers.
- **Sufficiency and coverage tests:** remove the question or output and ask whether a verifier can reconstruct the result from the trace.
- **Adversarial stress tests:** give the agent optimization pressure or strategies for evading a fixed or adaptive monitor.

Important hypotheses to test rather than assume include:

- increasing model capability may increase latent reasoning and reduce faithfulness;
- extremely long CoTs may improve raw coverage while overwhelming a bounded-context monitor;
- harder and more specialized tasks may induce shorthand that is less legible;
- English-dominant pretraining may preserve English CoT, unless models learn latent or encoded alternatives;
- a stronger monitor, more monitor test-time compute, or targeted follow-up questions may recover information missed by a weaker monitor.

- **[12/20/2025] Monitoring Monitorability** [arXiv](https://arxiv.org/abs/2512.18311) · [alphaXiv](https://www.alphaxiv.org/overview/2512.18311)  
  **Authors:** Melody Y. Guan, Miles Wang, Micah Carroll, Zehao Dou, Annie Y. Wei, Marcus Williams, Benjamin Arnav, Joost Huizinga, Ian Kivlichan, Mia Glaese, Jakub Pachocki, Bowen Baker  
  **TL;DR:** Introduces intervention, process, and outcome-property archetypes and a broad evaluation suite. Longer CoTs are generally more monitorable in the studied settings; more monitor compute and follow-up reasoning can improve prediction, while frontier-scale RL does not materially reduce measured monitorability.

- **[03/30/2026] MonitorBench: A Comprehensive Benchmark for Chain-of-Thought Monitorability in Large Language Models** [arXiv](https://arxiv.org/abs/2603.28590) · [alphaXiv](https://www.alphaxiv.org/overview/2603.28590)  
  **Authors:** Han Wang, Yifan Sun, Brian Ko, Mann Talati, Jiawen Gong, Zimeng Li, Naicheng Yu, Xucheng Yu, Wei Shen, Vedant Jolly, Huan Zhang  
  **TL;DR:** Provides 1,514 instances across 19 tasks and seven categories, annotated with decision-critical factors and two stress tests. Monitorability is higher when correct outputs structurally require reasoning through the critical factor; some stress tests produce substantial drops.

- **[09/19/2025] Stress Testing Deliberative Alignment for Anti-Scheming Training** [arXiv](https://arxiv.org/abs/2509.15541) · [alphaXiv](https://www.alphaxiv.org/overview/2509.15541)  
  **Authors:** Bronson Schoen et al.  
  **TL;DR:** Deliberative alignment greatly reduces but does not eliminate covert actions across broad out-of-distribution evaluations. CoTs expose evaluation awareness that is causally associated with reduced covert behavior, complicating interpretation of apparent safety.

## Mitigations

No single intervention addresses all threat models. Candidate mitigations should be tied to the property they target.

### At training time

- preserve a private reasoning channel rather than applying direct penalties to suspicious thoughts;
- track monitorability alongside capability throughout pretraining and post-training;
- use limited supervised traces or auxiliary causal-importance and sufficiency rewards where appropriate;
- avoid reward terms that place output performance and CoT transparency directly in conflict;
- evaluate output-only and process supervision for indirect obfuscation;
- retain model checkpoints and run monitorability regression tests across training.

### Improve the monitor

- scale monitor capability and test-time compute;
- use follow-up questions designed to elicit omitted decision-relevant information;
- combine CoT, actions, outputs, and white-box signals rather than treating CoT as a complete safety case;
- use ensembles with diverse monitor architectures and training data;
- adapt monitors to long contexts, code and tool-use environments, specialist domains, low-resource languages, and language mixing;
- use retrieval or hierarchical summarization for long traces while separately checking whether summarization discards suspicious evidence.

### Evaluate the deployment context

- match evaluation length and domain to deployment;
- test fixed and adaptive adversaries;
- distinguish default monitorability from monitorability after the agent learns the monitor;
- report agent capability and monitor capability separately;
- treat human readability and model-monitor readability as different outcomes.

## Commentary on legibility

These non-peer-reviewed discussions help refine hypotheses about strange, terse, or mixed-language CoTs. They should not be treated as substitutes for controlled evaluation.

- **[10/10/2025] Towards a Typology of Strange LLM Chains-of-Thought** [LessWrong](https://www.lesswrong.com/posts/qgvSMwRrdqoDMJJnD/towards-a-typology-of-strange-llm-chains-of-thought)  
  **Authors:** Not yet verified  
  **TL;DR:** Proposes several possible causes of unusual RLVR traces, including reasoning spandrels and context refresh, offering a vocabulary for hypotheses that future legibility evaluations could separate.

- **[04/10/2026] The Unintelligibility Is Ours: Notes on Chain-of-Thought** [LessWrong](https://www.lesswrong.com/posts/rFbTAL6PofHzZCCpD/the-unintelligibility-is-ours-notes-on-chain-of-thought)  
  **Authors:** 1a3orn  
  **TL;DR:** Argues that terse, compressed, and mixed-language traces may reflect imitation of human shorthand rather than a newly invented private language, suggesting that ordinary language has considerable unused information-density headroom.

## Open questions

1. Which combination of dependency, coverage, legibility, and robustness best predicts real safety value?
2. How should monitor capability be normalized when comparing the monitorability of agents?
3. At what capability or training scale does latent reasoning materially displace externalized CoT?
4. How does monitorability change when the agent knows the monitor or can optimize against it?
5. Which forms of obfuscation transfer across monitors, languages, tasks, and model families?
6. When does direct CoT supervision improve safety, and when does it teach concealment?
7. How do context length, code, tool use, specialist jargon, low-resource languages, and language mixing affect monitorability?
8. Can black-box CoT monitoring and white-box interpretability provide complementary guarantees?

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

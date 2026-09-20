---
title: "The Next Eval Idea I Want to Test: Discriminative Prompt Discovery"
slug: "discriminative-prompt-discovery"
date: 2026-09-20T20:19:31+10:00
draft: false
description: "A proposed workflow for finding realistic prompts that reveal stable, useful differences between language models."
tags: ["llm-evaluation", "synthetic-data", "model-selection", "prompt-engineering"]
---

*I have not run this experiment yet. I am writing it down because I want to try it when I have the bandwidth, and because writing the method out is a good way to find the holes in it.*

I keep coming back to the same problem when I think about model selection for synthetic data. Public benchmarks can tell me whether a model is broadly capable. Price and latency tests can tell me whether it is practical. Neither tells me where two plausible models will behave differently on the work I actually care about.

Approved production trajectories are usually the strongest starting point for workload realism and, when representatively sampled and reliably scored, for performance. Synthetic evaluation data is useful when compliance rules restrict access to raw trajectories, important failures are too sparse to appear in a practical sample, or a new feature has no production history. It fills a defined coverage gap and can accelerate model selection, but it does not remove privacy or compliance obligations. When enough production evidence becomes available, I would use it to check and recalibrate the synthetic benchmark.

Imagine choosing between a hosted Gemini model and a locally deployed Gemma model for text generation. I can run both against a sample of domain prompts and compare their average scores. That gives me a baseline, but the average hides the interesting part. I want to know which kinds of prompt expose a stable difference, and whether that difference survives beyond the examples used to find it.

The working name I have for this is **Discriminative Prompt Discovery**.

I am not claiming a new evaluation discipline. The idea combines behavioral testing, contrast sets, item discrimination and adaptive benchmark construction into a workflow for model selection within one domain. Here, discriminative simply means a repeatable difference between the candidate models that matters to the decision. The part I want to test is whether that difference holds up on fresh cases.

## What I want to test

The process starts by mapping the coverage of the available evidence. Synthetic seeds are added only for known gaps, such as restricted data, sparse failures or a feature that has not launched. Both models receive the same cases under comparable settings and budgets. When a repeatable gap appears, I create nearby examples that change one meaningful feature at a time.

Suppose the task is to generate a customer support response from a return policy. Standard products can be returned within 30 days, while engraved products can be returned only when defective. One model applies the exception correctly and the other misses it.

That single result is interesting, but it does not establish much. I would create related cases by changing the engraved product to a standard product, changing a working item to a defective one, moving the exception within the policy, paraphrasing it, adding irrelevant details, and combining it with a formatting constraint.

If the same pattern appears across several independent scenario families, I have a useful behavioral hypothesis. For example, one model may be less reliable when it must preserve an exception while satisfying several output constraints. The contrast cases help test that hypothesis. They do not reveal the internal cause of the behavior.

A useful differentiator should answer three questions. Does it produce a repeatable gap between the models? Does it represent work the system is likely to encounter? Does the pattern survive on new scenario families? A strange riddle might separate two models dramatically and still have no value for the decision.

[CheckList](https://aclanthology.org/2020.acl-main.442/) provides a useful foundation for this kind of behavioral testing. The work on [contrast sets](https://aclanthology.org/2020.findings-emnlp.117/) shows how small, meaningful changes can expose local decision boundaries. [IDGen](https://arxiv.org/abs/2409.18892) focuses directly on prompts with high item discrimination, while [AutoBencher](https://arxiv.org/abs/2407.08351) treats benchmark creation as an adaptive search problem.

The process has two phases: an exploratory discovery loop followed by a frozen confirmation stage.

![The process starts with domain requirements and production evidence. Approved synthetic cases fill restricted access or missing coverage. Candidate models are compared in a discovery loop where domain realism is checked before model separation is scored, then contrast sets are created. The capability, scoring rule, model settings and primary comparisons are then locked. Fresh scenario families enter a one way confirmation stage where the benchmark is frozen, model identity is hidden, response order is randomized, and representative and challenge results are reported separately.](discriminative-prompt-discovery-process.png)

The loop in the first half is deliberately exploratory. Prompts can be changed, rerun and discarded while I am learning what might separate the models. Once a capability and scoring rule are chosen, the process crosses into confirmation. Nothing from the confirmation stage should feed back into the reported result.

## Keeping the search honest

The obvious failure mode is selection bias. If I search long enough, I will find prompts where either model looks unusually strong or unusually weak. Reporting only those prompts would prove that I can search, not that I have found a dependable model difference.

The generator can add another source of bias. A test set produced by one of the candidate model families may favor its vocabulary, assumptions or style. Automated judges can prefer longer answers, familiar phrasing or outputs from their own family. Prompt templates and inference settings can also suit one model better than another.

The synthetic cases also need their own quality gate. A case should be accepted only if it is answerable, consistent with the domain rules, plausible within the intended workflow, and representative of either normal use or a clearly defined failure mode. A domain expert or an independent reviewer should be able to explain why the case could matter. Cases that are confusing only because they are malformed, unrealistic or based on an ambiguous expected answer should be removed.

Differentiating power comes after that realism check. Among cases that pass the quality gate, I would look for a stable paired performance gap across repeated generations and independent scenario families. Optimizing for separation alone would invite strange trick questions that distinguish the models without telling me anything useful. I would also retain cases where either model wins and where both fail, rather than building a suite that supports only one preferred conclusion.

My main safeguard would be a hard boundary between discovery and confirmation.

Before confirmation, I would lock the capability definitions, scoring rules, model configurations and primary comparisons. I would also decide what size of paired difference matters, how many scenario families and repeated generations to use, and how to handle ties or failed calls.

The confirmation benchmark would then be generated from fresh source material and different scenario families. Close paraphrases of discovery examples would stay out. Its items could be written by people, produced by an independent generator, or drawn from a balanced pool of generators. In each case, the items should be reviewed and accepted before anyone sees the candidate model outputs.

I would run confirmation once and report every primary capability comparison, rather than highlighting only the largest gap. If I inspect the results and decide to change the benchmark, that becomes discovery for a new experiment. It is no longer the original confirmation run.

The frozen confirmation benchmark would contain two suites. The representative suite would be sampled from production trajectories whenever possible, so its score could estimate normal performance. If the product has not launched, that suite would reflect the best available design assumptions and would need recalibration once real trajectories exist. The challenge suite would contain more rare and difficult cases, so it could expose robustness problems. Those scores should remain separate because they answer different questions.

Objective checks should handle format validity, required fields, length limits and known factual rules. Subjective qualities need blind review with model identity hidden and response order randomized. If an LLM judge is involved, I would first compare its decisions with a sample reviewed by humans and test whether swapping response order changes the result.

## The first version I would build

I would turn that coverage map into five or six capabilities that matter to the task and identify the gaps that justify synthetic generation. Each capability would have several independent scenario families, and each family would contain ordinary cases plus controlled contrasts. Repeated generations would measure variation within a case.

The final comparison should be paired because both models answer the same prompts. Any uncertainty estimate should operate at the scenario family level, since paraphrases from the same family are related observations. Treating every variation as independent would make the result look more precise than it is.

I would record the exact model version, system prompt, sampling settings, source material, generator, judge, rubric and human corrections. Tools such as [Distilabel](https://distilabel.argilla.io/dev/) could support generation and filtering, while [Inspect AI](https://inspect.aisi.org.uk/) or [Promptfoo](https://www.promptfoo.dev/docs/intro/) could run the comparison. The choice of tool matters less than preserving the boundary between exploration and confirmation.

[DSPy](https://github.com/stanfordnlp/dspy) looks like a plausible way to prototype the discovery loop. A DSPy program could generate structured cases, run a fixed program under [Gemini and Gemma](https://github.com/stanfordnlp/dspy/blob/main/docs/docs/learn/programming/language_models.md), and use a [custom metric](https://github.com/stanfordnlp/dspy/blob/main/docs/docs/diving-deeper/metrics-and-evaluation.md) to score validity, relevance, the quality gap and repeatability. An optimizer such as [GEPA](https://github.com/stanfordnlp/dspy/blob/main/docs/docs/getting-started/gepa-optimization.md) could improve the generator during discovery. The paired comparison, review, compliance controls and uncertainty estimates would still sit outside DSPy, and confirmation families should never enter optimization. Since DSPy caches model calls by default, repeated stochastic trials need distinct rollout identifiers with a nonzero temperature, or caching must be disabled. Otherwise, cached responses could look like stability.

There is one more design choice to make explicit. Running the same fixed program against both models compares model behavior under one shared setup. Optimizing a separate program for each model compares the best model and prompt systems I can produce under equal optimization budgets. Both questions are useful, but they should be reported separately.

The output is useful beyond naming a single winner. For each scenario family, I would record success rate, input and output tokens, latency, retries and accepted output rate. That would let me compare the cost per accepted case rather than relying on the advertised token price alone. The same evidence could support a model router: routine cases could go to the cheaper model, while cases that resemble a known failure family could be sent to the stronger model. The routing policy would need its own validation on the frozen benchmark, including a fallback for cases where the route is uncertain.

## What would count as a useful result

I do not expect this process to produce a universal ranking. It might show that one model is more reliable on policy exceptions, while another is accurate enough and much cheaper for routine generation. It might justify routing difficult cases to one model and sending the rest to another. It might also show no dependable difference at all, leaving cost, latency, privacy and operational simplicity to decide the choice.

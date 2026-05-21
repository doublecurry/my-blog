---
title: 'The Evolution of Large Model Chain of Thought'
description: 'A structured overview of Chain of Thought, self-consistency, Tree of Thoughts, and agentic reasoning patterns.'
created: 2026-03-17
updated: 2026-03-20
tags:
  - AI
  - Reasoning
  - Prompt Engineering
---

# The Evolution of Large Model Chain of Thought (CoT)

As large language models scaled up in parameters and compute, the expectation was that high-level reasoning would simply emerge once the model was big enough. In practice, scale alone did not deliver complex logical reasoning. The technique that produced a real step-change in logic, math, and decision-making is a prompting pattern called **Chain of Thought** (CoT).

## What is CoT?

A traditional LLM maps input directly to output. CoT requires the model to write out the intermediate solution steps before giving a final answer.

The analogy is a math exam: writing only the answer, even if correct, will not get full marks; the graders want the known conditions, the derivations, and the calculations laid out step by step. By forcing the model to display the intermediate reasoning chain (Input $\rightarrow$ Reasoning Chain $\rightarrow$ Output), CoT lowers the error rate on complex tasks and gives the output some degree of interpretability.

---

## Innovation Driven by Demand: The Evolutionary Path of CoT

Since its introduction in 2022, CoT has moved from a simple prompting trick into a broader family of reasoning architectures. The direction of travel is from **single-thread passive deduction** to **structured active search**.

### I - The starting point: Few-shot-CoT and Zero-shot-CoT

* **Few-shot-CoT**: the original form. Providing a few manually written examples with detailed reasoning steps in the prompt leads the model to produce a similar reasoning process for new questions.
* **Zero-shot-CoT**: removes the dependence on hand-written examples. Adding **"Let's think step by step"** at the end of the prompt is enough to trigger the model into producing intermediate steps on its own.

### II - Improving robustness: CoT-Self-Consistency (CoT-SC)

A single chain of thought has very low error tolerance: one wrong step in the middle propagates to the final answer. **CoT-SC** addresses this by sampling multiple reasoning paths for the same problem and selecting the most frequent final answer through a majority vote, which noticeably improves stability and accuracy.

### III - Structured Search: Tree of Thoughts (ToT)

Where CoT-SC runs several independent chains in parallel and votes at the end, **ToT** models reasoning as a search tree:

* **Nodes and Expansion**: each reasoning step is a node, and multiple branches can be expanded at a branching point.
* **Evaluation and Backtracking**: the model can score the current path and, if it reaches a dead end, use Depth-First Search (DFS) or Breadth-First Search (BFS) to backtrack and try alternative branches from an earlier step.

### IV - Wider network structures: GoT, AoT, and PoT

* **Graph of Thoughts (GoT)**: generalises the tree to a directed acyclic graph. Branches can merge (aggregation) or be iteratively refined at a single node, which fits non-linear thinking better.
* **Algorithm of Thoughts (AoT)**: ToT and GoT trigger many LLM calls, which is expensive. AoT internalises the logic of classical algorithms such as depth-first search into a single generation, doing self-correction inside one continuous context.
* **[Program of Thoughts (PoT)](https://arxiv.org/abs/2211.12588)**: numerical errors are a known weakness of LLM reasoning. PoT has the model write the calculation logic as code (typically Python), executed by an external interpreter. This cleanly separates "reasoning" from "computation".

### V - Efficiency Breakthrough: Skeleton-of-Thought (SoT)

Step-by-step generation is slow. **SoT** mirrors how a person outlines an article: have the model first generate a skeleton of the answer, then expand each skeleton node in parallel. Latency drops sharply with limited cost to coherence.

---

## 2025-2026 Frontier: Chain-of-Action-Thought (COAT)

ToT and related methods achieve structured reasoning, but they often rely on external heuristics or human-set rules to decide when to stop or backtrack. The next step is for the model to manage that decision itself.

**[Chain-of-Action-Thought (COAT)](https://satori-reasoning.github.io/blog/satori/)** frames LLM reasoning as a sequential decision-making problem. A representative example is the [Satori Model](https://icml.cc/virtual/2025/poster/44324) presented at ICML 2025. The model emits not only text but also specific **meta-action tokens**:

* **`<|continue|>`**: keep building on the current reasoning trajectory.
* **`<|reflect|>`**: pause and check the prior steps for logical errors.
* **`<|explore|>`**: abandon the current path and look for an alternative when a dead end is detected.

Combined with a Restart-and-Explore (RAE) mechanism trained via reinforcement learning, billion-parameter models (7B scale) using COAT have matched or beaten much larger traditional models on complex math and cross-domain reasoning tasks — a case of "small model + refined autonomous search" closing the gap.

---

## Limitations and Future Outlook

CoT still has real problems:

1. **Compute and cost**: producing many intermediate steps means heavy token consumption and high inference latency, which makes CoT hard to use in latency-sensitive scenarios.
2. **Scale sensitivity**: traditional CoT depends on the base model's parameter count (usually tens of billions). Forcing small models to use it often degrades into incoherent reasoning, although COAT pushes back on this.
3. **Pseudo-logic**: "because… therefore…" output is sometimes just probabilistic fitting rather than real causal deduction. A reader can be misled by reasoning that looks clean but does not actually hold.

CoT is both a way to improve LLM reasoning and a way to look inside what is otherwise a black box. The model has moved from passively accepting prompts to generating, reflecting on, and correcting its own steps — the reasoning loop is becoming more complete, though it is not yet closed.

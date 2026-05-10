---
title: "Can LLMs tell if a Lean 4 statement is faithful to a natural-language statement?"
date: 2026-05-09
permalink: /posts/2026/post2/
tags:
  - Lean
  - formalization
---

I tested this question with two local LLMs: Qwen3.6-27B and DeepSeek-Prover-V2-7B.

The setup is quite simple yet a crucial bottleneck in the Lean 4 data flywheel: a natural-language statement and a candidate Lean 4 statement. We need to know if the candidate Lean 4 statement is faithful to the natural-language statement, or if it is a false positive.

## Dataset

I used the [ProofNetVerif](https://huggingface.co/datasets/PAug/ProofNetVerif) dataset, which contains 3,752 examples of natural-language statements and their corresponding Lean 4 statements. The problems here are from the ProofNet benchmark. It has 2,300 validation rows and 1,452 test rows. The problems are primarily drawn from popular undergraduate pure mathematics textbooks and cover topics such as real and complex analysis, linear algebra, abstract algebra, and topology.

ProofNetVerif has multiple candidate Lean predictions per original problem id. I do not want one problem with many predictions to dominate the benchmark, so for each original `id`, I sample `min(5, number of rows with this id)` rows with a fixed seed. This gives 1,653 sampled ProofNetVerif rows before Lean filtering. Then I only benchmark Lean predictions that compile. After this compile filtering, there are 1,381 reference-free rows. For reference-based modes, I also need the reference Lean statement to compile, so there are 1,339 reference-based rows.

There are not many datasets of this kind, and ProofNetVerif is one of the bigger ones. I also have a local dataset that me and another PhD student labeled ourselves, the problems there are mostly graduate-level math problems from textbooks. For this run I used the unified JSON files averaged over the two reviewers. This gives 161 local rows before live Lean filtering, and 158 rows after filtering out the 3 predictions that do not compile. Local rows do not have reference Lean statements, so I only run reference-free modes on them.

For the local labels, I map the averaged 0-10 human score to a binary label:

```text
TRUE  iff score is 8, 9, or 10
FALSE otherwise
```

## Methodology

I used the following generation configs and chat formats for the two LLMs:

```python
KMAX = 31
k_values = [1, 3, 5, 7, 11, 15, 21, 31]

qwen = {
    "model": "Qwen/Qwen3.6-27B",
    "served_model_name": "qwen3.6-27b",
    "temperature": 1.0,
    "top_p": 0.95,
    "top_k": 20,
    "max_tokens": 8192,
    "n": 31,
    "vllm_args": [
        "--max-model-len", "32768",
        "--reasoning-parser", "qwen3",
        "--language-model-only",
    ],
    "extra_body": {
        "chat_template_kwargs": {
            "enable_thinking": True,
            "preserve_thinking": True,
        },
        "top_k": 20,
    },
    "serving_profile": {
        "tensor_parallel_size": 2,
        "data_parallel_size": 4,
        "request_concurrency": 8,
        "max_num_seqs": 256,
        "max_num_batched_tokens": 65536,
        "enable_prefix_caching": True,
        "enable_chunked_prefill": True,
    },
}

deepseek = {
    "model": "deepseek-ai/DeepSeek-Prover-V2-7B",
    "served_model_name": "deepseek-prover-v2-7b",
    "temperature": 0.7,
    "top_p": 0.95,
    "top_k": 20,
    "max_tokens": 2048,
    "n": 31,
    "vllm_args": [
        "--dtype", "bfloat16",
        "--max-model-len", "32768",
        "--trust-remote-code",
    ],
    "extra_body": {"top_k": 20},
    "serving_profile": {
        "tensor_parallel_size": 1,
        "data_parallel_size": 8,
        "request_concurrency": 16,
        "max_num_seqs": 256,
        "max_num_batched_tokens": 32768,
        "enable_prefix_caching": True,
        "enable_chunked_prefill": True,
    },
}
```

Qwen thinking is turned on. I use `preserve_thinking=True` and the qwen3 reasoning parser. DeepSeek-Prover-V2-7B does not have the same Qwen-style thinking switch, so whatever reasoning it produces is just prompted reasoning in the normal output.

I use vLLM chat completions and let the tokenizer chat template format the messages. I do not use structured outputs or force the answer through constrained decoding, because that changes the generation behavior. The model can reason, but the parser only grades the final `TRUE` or `FALSE` answer.

The prompt is always the same binary judging task: decide whether the candidate Lean statement formalizes the same mathematical claim as the natural-language statement. The model is told to think through the semantic correspondence and end with:

```text
Final Answer: TRUE
```

or

```text
Final Answer: FALSE
```

We use several modes:

```text
RF:
  natural-language statement + candidate Lean statement

RF+Loogle:
  natural-language statement + candidate Lean statement + cached Loogle context from the candidate Lean

RB:
  natural-language statement + reference Lean statement + candidate Lean statement

RB+Loogle:
  natural-language statement + reference Lean statement + candidate Lean statement + cached Loogle context from candidate/reference Lean
```

ProofNetVerif runs all four modes. The local dataset runs only RF and RF+Loogle, because the local Lean statements are candidate predictions, not reference statements.

For majority@k, I sample 31 completions once for every prompt, then compute prefix majority for `k = 1, 3, 5, 7, 11, 15, 21, 31`. If any completion in the prefix does not have a parseable final answer, that majority@k row is marked invalid. This is strict, but it is intentional: I do not want to silently reinterpret unparsable outputs.

## Results

Before we look at the results, we need to decide on which metric is the most important for a specific downstream application.

So what are the downstream applications of such a checker? Most likely such a checker is used when an autoformalizer is used, it is fair to assume that the formal statement candidate passes through a compilation checker first. In synthetic data generation:
- the synthesized Lean statements are used for RL training, whole proof generation. In this case, we care mostly about **false postive rate**, because a false positive training task can either contain a mathematically false statement which the model would never be able to prove, hence wasting compute, or an easier/harder version of the original statement which is arguably harmless.
- the synthesized Lean statements are used for SFT training, meaning a prover generates a proof, and the whole thing is bundled to be trained on. This can be common in distilling a larger model. Again, the false positive examples can be mathematically wrong, yet they would not make into the SFT training data because the prover cannot prove them, yet this wastes compute.
- the synthesized Lean statements are used for graph search expert iteration training, meaning a graph search algorithm with prover tactics-level prior runs on the synthesized Lean statement to find **successful/proved acyclic subgraphs**. In this algorithm, a false positive with wrong math content can still provide signals, since a sub-proof can still be true and found by the search. For example, $x^2 \geq x$ for real $x$ is a wrong statement, yet if the prover decides to split at $x = 1$, a sub-proof of $x^2 \geq x, /forall x \geq 1$ can still be found and used.

However in any case, a false positive wastes compute, and since the prover input almost always contains the NL statement, any false positive Lean statement, specifically the relaxation/extension of the NL statement, should not be trained on, because the prover gets reward for proving a misaligned NL-Lean pair.

But my personally opinion is that this is not prover's job, and false positives are not that scary in training a prover, since the prover at deployment is used on *faithful Lean statement* with other parts of the system making sure this is the case.

So I would focus on *positive precision* and *false positive*.

Positive precision here means: among all Lean statements that the checker says are faithful, what fraction are actually faithful. This is the thing we always want to be high, because this is directly measuring the purity of the synthetic data that gets through the checker. If the checker accepts 100K candidate statements, this metric is basically asking how much poison we let into those 100K.

False positive rate is also important, but it has a subtle base-rate issue. If the autoformalizer is already good, meaning most of its Lean candidates are faithful, then there are only a small fraction of unfaithful Lean statements in the first place. In that setting, even a checker with a not-too-high false positive rate will only leak a small absolute amount of unfaithful statements into the accepted set.

Put differently:

```text
bad fraction among accepted =
  false_positive_rate * fraction_unfaithful
  /
  (true_positive_rate * fraction_faithful + false_positive_rate * fraction_unfaithful)
```

So the same false positive rate means different things depending on how good the upstream autoformalizer is.

In other words, positive precision is the deployment metric, because it directly answers "when I keep this example, how likely is it to be good?" False positive rate is more like a robustness diagnostic: if the autoformalizer gets worse, or the domain gets harder, or the candidate distribution has more bad statements, then the same false positive rate can suddenly leak many more bad statements.

So when reading the results, I care most about:

- positive precision, since this is accepted-data purity
- false positive rate, since this tells how much the checker leaks bad candidates under a harder or noisier candidate distribution
- recall/yield only after precision is acceptable, because a checker that rejects everything is safe but not very useful

Here are the RF+Loogle results at majority@31, which is the closest setting to the realistic use case where we only have the natural-language problem and the candidate Lean statement:

| dataset | model | positive precision | false positive rate | faithful recall | invalid@31 |
| --- | --- | ---: | ---: | ---: | ---: |
| ProofNetVerif | Qwen3.6-27B | 76.0% | 13.7% | 90.4% | 0.4% |
| ProofNetVerif | DeepSeek-Prover-V2-7B | 47.4% | 32.9% | 60.9% | 60.4% |
| local | Qwen3.6-27B | 80.9% | 37.3% | 100.0% | 3.8% |
| local | DeepSeek-Prover-V2-7B | 53.8% | 41.4% | 73.7% | 69.6% |

The first obvious result is that Qwen is much better as a semantic judge here. On ProofNetVerif RF+Loogle, Qwen has 76.0% positive precision and 13.7% false positive rate. DeepSeek has 47.4% positive precision and 32.9% false positive rate, and the invalid@31 rate is huge.


Loogle also does not obviously help. For Qwen on ProofNetVerif, RF+Loogle is slightly better than RF on positive precision, 76.0% vs 75.0%, and slightly better on false positive rate, 13.7% vs 14.5%. This is a small effect. On local, Qwen gets slightly worse with Loogle: positive precision goes from 82.4% to 80.9%, and false positive rate goes from 33.3% to 37.3%.

Reference-based mode is a diagnostic ceiling rather than the real deployment setting. On ProofNetVerif RB+Loogle, Qwen reaches 84.1% positive precision and 8.3% false positive rate. DeepSeek is still poor there, with 42.6% positive precision and 48.8% false positive rate. So the reference helps Qwen, but does not rescue DeepSeek.


The flatness in majority voting is mostly because Qwen is very stable. On ProofNetVerif, 88.1% of Qwen vote groups are unanimous across all 31 completions. On local, 69.3% are unanimous. So increasing k often does not change anything.

DeepSeek has the opposite problem. Only around 0.2-0.3% of DeepSeek vote groups are unanimous, and around 62% of its vote groups have at least one invalid completion. Since the majority rule is strict and marks a prefix invalid if any vote in the prefix is invalid, the invalid@31 number becomes very large even though the per-completion invalid rate is only around 4%.

My initial not-quite-scientific takeaways:

- Qwen3.6-27B is a usable first-pass judge, especially on ProofNetVerif, but the positive precision is still not high enough to blindly trust for a high-value synthetic data pipeline.
- Loogle context needs more investigation. Needs to investigate incorporating it as a tool rather than using retrieved statement-specific cache.
- Majority@k mostly confirms stable behavior for Qwen and exposes invalid-output brittleness for DeepSeek.
- positive precision may not drop as problems get harder, but with harder NL statement, both the false positive rate and the unfaithfullness of the autoformalizer likely rises, resulting in more pollution.


Next questions would be:
1. if a naive autoformalizer + Qwen3.6-27B as checker pipeline can produce useful training data for prover, leading to consistent gain over multiple rounds of generating and filtering data.
2. test other checker workflow, especially the proprietary models, local tool-use agent, and agent team for checking faithfulness.
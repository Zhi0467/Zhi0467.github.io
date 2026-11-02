---
layout: archive
title: "Research"
permalink: /research/
author_profile: true
---

## Ongoing projects

**Lean prover training with graph search.** I'm writing a Lean prover training framework that supports graph-search-based expert iteration, built for throughput and flexibility. In parallel, I'm studying how to check Lean autoformalization for semantic faithfulness, so the checker and the trainer can feed each other as a data flywheel. The goal is a strong specialized Lean prover trained with graph-search expert iteration and other post-training methods.

**Controlling degenerate chain-of-thought.** Reasoning models sometimes overthink or loop. I'm building an online gated controller that detects these behaviors and intervenes during generation. This is mostly probing and steering work.

**Self on-policy distillation for SWE agents.** I'm extending self on-policy distillation (SDPO/OPD) to the multi-turn agentic setting, to see whether its denser training signal lets a small model become a better SWE agent.

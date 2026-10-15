# 🏁 Proving Grounds — picking a local model for agentic / multi-agent work

Everyone says "just use Llama 70B." For **agentic, tool-heavy, multi-agent** workloads running **locally
on one box**, that advice is often wrong — and the reasons are measurable, not matters of taste. Proving
Grounds is the evidence-backed shortlist we arrived at after running a 14-agent local system on Apple
Silicon (MLX).

> These are field observations of *public* models on real agent workloads — to save you the same testing,
> not a formal leaderboard. Bring your own numbers; PRs welcome.

---

## TL;DR

> For agentic/tool-calling/multi-agent on a single machine, prefer a **Mixture-of-Experts (MoE)** model
> with **low active parameters** over a dense model of similar quality. On one box, **prompt-processing
> speed and RAM footprint dominate** — and that's exactly where MoE wins.

## What we observed (and why it matters)

| Finding | Why it kills agentic/multi-agent use |
|---|---|
| **Dense 70B/72B** → ~95 tok/s **prompt processing** → ~3 min on a 16k-token prompt | Agents carry big tool/context prompts every turn; multi-minute prompt processing = constant timeouts. |
| **Dense 70B weak at tool-calling** (eval F1 ~0.61 vs ~0.92–0.97 for the Qwen3 MoE family) | It *narrated* which tool it would use instead of emitting a tool call. Agents need calls, not prose. |
| **Full-precision 72B ≈ 254/256 GB RAM** | Zero headroom; you can't run the model *and* the rest of the stack. |
| **235B (MoE, ~22B active) ≈ 171 GB** → choked at ~3 concurrent agents | Too heavy for many concurrent agents on one box. |
| **~35B MoE with ~3B active ≈ 30% RAM** → ran **all 14 agents + the supporting stack** | Tiny footprint + fast per-token = the actual answer for multi-agent on one machine. |

## The rules we now follow

1. **MoE, not dense**, for agentic/tool-heavy/multi-agent on one box. Low *active* params = fast
   prompt-processing + small footprint, which is what actually bites you locally.
2. **Test tool-calling explicitly** before committing — measure whether the model *emits calls* (not just
   describes them). A model that narrates tool use is unusable as an agent, regardless of its chat scores.
3. **Budget RAM for the whole system**, not just the model — leave room for embeddings, a vector DB,
   queues, and N concurrent agent sessions.
4. **Prompt-processing speed > generation speed** for agents — their prompts are big and frequent.
5. **Measure on *your* hardware + *your* prompts.** tok/s and RAM are platform-specific; treat the table
   above as a starting hypothesis, not gospel.

## A quick way to test a candidate

- Run a fixed **16k-token** prompt and time **prompt processing** (not just generation).
- Score **tool-calling**: give it 20–30 tool-use tasks and measure how often it emits a valid call vs
  narrates. Track precision/recall (F1).
- Load it alongside your real stack and ramp **concurrent agents** until latency/RAM breaks.

## Built by

Built by **[KodeKing](https://www.kodeking.net)** · author **[Fazal Shah](https://www.fazalshah.com)**.
We build local, private, multi-agent AI systems. Add your own measurements via PR.

## License

MIT — see [LICENSE](LICENSE).

<!-- fazalshah-personal-site:start -->
## Author and engineering notes

Published by [Fazal Shah](https://fazalshah.com/). Explore my local AI experiments, Mac hardware test notes, agent engineering articles, and public tools on my [personal website](https://fazalshah.com/).

[All public repositories](https://github.com/fazalrshah?tab=repositories) · [YouTube builds and walkthroughs](https://www.youtube.com/@fazalrshah)
<!-- fazalshah-personal-site:end -->

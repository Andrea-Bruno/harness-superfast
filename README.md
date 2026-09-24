# The Superfast Harness

### A Decision Gate that makes AI agents react in milliseconds, not seconds

*A white paper on speeding up AI agent harnesses with a fast "System One" decision model.*

---

> **Read this first — license and author.**
> This work is by **Andrea Bruno**. It is released under **Creative Commons Attribution 4.0 (CC BY 4.0)**.
> You may copy it, share it, change it, build on it, and use it commercially. The only rule: **credit Andrea Bruno** as the author of the idea and this document, link the license, and say if you changed something. See the `LICENSE` file.
>
> **About a patent.** This architecture is new. A new thing that can be used in a real product can often be patented around the world. The author made a clear choice: **not to patent it.** The idea is given to everyone, as a personal contribution to open technology. The one thing we ask in return is **attribution** — say who came up with it. That is the whole condition.

---

## 1. The problem, in one line

In today's AI agents, **every user message wakes up a big, slow, expensive language model — even when the answer needs no big thinking at all.**

Think about what an agent does when you type something. It usually calls a large LLM to decide:
- What does the user want? (intent)
- Do I need a tool, or can I just answer?
- Which tool should I use first?
- Do I need the attachments, or the chat history?
- Is this action safe?

Each of these is a small, simple decision. But right now they are made by a **generative LLM** — the same heavy model that writes code and essays. That costs **1 to 3 seconds** and real money **per message**, even for "hello" or "what did we just change?".

On a normal cloud API this is just costly. On a **home computer or a small GPU (even 4 GB of VRAM)**, it is painfully slow. The agent feels sluggish.

## 2. The idea: put a fast "decider" in front of the big model

Human brains have two ways of thinking. Psychologists call them **System 1** and **System 2**:
- **System 1** is fast and automatic. You see a face and instantly know if it is a friend. No effort.
- **System 2** is slow and careful. You solve 17 × 24 in your head. Effortful.

A big LLM is a System 2. It is great at reasoning, but slow and costly for every little thing.

A new kind of small model — a **System One decision model** — is built only for the fast part. It does **not** write text. It looks at a situation and returns **typed answers with confidence scores**, in one quick pass.

**The invention is where we put it.** We place a System One model at the **front door** of the agent. We call this the **Decision Gate**.

```
   User message
        |
        v
  +------------------+     fast, ~tens of ms, no text generated
  |  DECISION GATE   |     answers typed questions in ONE pass
  | (System One)     |
  +------------------+
        |
        |  a routing decision (with confidence)
        v
   +----------------------------------------------+
   |  direct answer | one tool | enriched LLM | full LLM |
   +----------------------------------------------+
```

The Gate reads the message and answers simple questions at once:
- *Is this just chat?* → answer right away, **no big model at all**.
- *Does it need a tool, and which one?* → run that tool.
- *Can I answer from the chat history or an attachment?* → use it, skip the search.
- *Is it complex or risky?* → hand it to the full LLM as usual.

Only the messages that truly need deep thinking reach the big model. Everything else is handled in milliseconds.

## 3. What the Decision Gate actually returns

A System One model answers three kinds of typed questions, all in parallel, in a single forward pass:

| Type | Question shape | Output |
|------|----------------|--------|
| **Choice** | "Pick one of these options" | a probability for each option + a confidence score |
| **Score**  | "Rate this from low to high" | a value across ordered levels + confidence |
| **Noul**   | "Yes or no?" | the probability that it is true |

Because the answers are **typed and calibrated**, the routing logic is simple and safe:
- High confidence → take the fast path.
- Low confidence → **fail open** to the full LLM. The agent is never worse than before; it is only faster when it is sure.

The Gate **cannot hallucinate a route**. It does not invent sentences. It only picks among options you defined, with a number attached.

## 4. Why this is fast and cheap

| | Old way (LLM decides) | With the Decision Gate |
|---|---|---|
| A simple chat message | 1–3 s (full LLM) | **tens of ms** (System One) |
| Picking a tool | 1–3 s | **tens of ms** |
| Cost per decision | full LLM price | a tiny fraction |
| Hallucination on routing | possible | **none** (no text) |

The big model is still there for the hard work. We just stop paying it for the easy work.

## 5. It runs on home hardware

This is the key for people who run models locally. The open-source System One models are **small** and run on **CPU only, or a small GPU with about 4 GB of VRAM** — the kind of machine many people already own.

Real, open-source options you can use today (Apache-2.0):

| Model | What it is | How you run it |
|-------|-----------|----------------|
| **Von** (`wfzyx`) | Open, non-autoregressive System One model. Has a **Python and a TypeScript SDK** (`von-sdk`). Serves a Jev-compatible `/v1/systemone` endpoint. | `von serve` (HTTP) or in-process via the SDK |
| **OpenJev** (`razorback16`) | Open, Jev-compatible System One decision server | HTTP server |
| **Laya** (`NandhaKishorM` / Convai Innovations) | Open, local decision model | local server |

The commercial reference is **Jev** by **TypeSafe AI** (the model that defined the "System One" idea). The open models above let you get the same effect **on your own machine, for free, with no cloud call.**

> The Decision Gate talks to any of these through one small interface. Swap the backend without changing the harness.

## 6. A concrete example: Qwen Code

> **This is only an example.** The architecture is **not tied to Qwen Code**. It can be added to any agent harness that has a "user message → decide → act" loop. If you use this design in your own harness, please credit Andrea Bruno (see the license).

Qwen Code is an open-source coding agent. Its loop is exactly the pattern this fixes: every message goes to a big LLM to decide intent, tools, and safety.

**How the Decision Gate would fit Qwen Code:**
- A new module sits between the input handler and the agent loop.
- On each message, it calls a local System One model (for example **Von** via `von-sdk`) with a small set of typed questions: intent, needs-a-tool, which-tool-first, use-history, use-attachments, urgency.
- High-confidence chat → answer or short-circuit without the big model.
- Clear tool request → start the tool.
- "Answer from history" → skip the re-investigation.
- Low confidence or risky → the normal full agent loop.

**The user sees it as optional.** In Qwen Code it would be turned on with a single command:

```
/superfast
```

Turn it off and nothing changes. Turn it on and the agent feels much more responsive, especially on a local model.

Qwen Code already has a two-stage safety classifier in its Auto Mode. That proves the harness is **already shaped for a fast decision model** — the Decision Gate is a natural, low-risk extension of a pattern that is already there.

> The full technical plan for the Qwen Code version is in [`qwen-code-implementation.md`](./qwen-code-implementation.md). That file is a working guide, not a fixed contract — it is expected to be improved while building.

## 7. What is claimed (the novel point)

Stated plainly, the contribution is:

1. Placing a **non-generative System One decision model at the entry point** of an AI agent harness, so that the routine decisions of the agent loop (intent, tool selection, context gating, safety triage) are made by a fast, calibrated model instead of a generative LLM.
2. A **priority-ordered routing layer** that reads the typed, probabilistic answers and sends each message to the cheapest correct path — direct answer, single tool, enriched call, or full LLM — and **fails open to the full LLM whenever confidence is low**, so the agent never loses capability.
3. Making this **optional and reversible at runtime** (a single toggle), and **runnable on consumer hardware** by backing the Gate with small open-source System One models.

The combination — *front-door decision gate + typed parallel questions + fail-open routing + consumer-hardware backend + runtime toggle* — is the point of this work.

## 8. Honest notes

- The **decision models themselves are not ours.** Jev (TypeSafe AI) and the open analogs (Von, OpenJev, Laya) are third-party. What is presented here is the **architecture and method** that uses them to speed up an agent harness.
- The Gate is a **bias with a guardrail**, not a guarantee. When it is unsure, it must fall back to the full model. The design goal is "faster when sure, never worse when not."
- This document describes a **design and method**. It is not a claim that any specific product already ships it.

## 9. How to use this work

You are free to build on this. Please:
- **Credit Andrea Bruno** as the author of the Superfast Harness / Decision Gate concept and this white paper.
- Keep the **CC BY 4.0** notice and link.
- Say if you changed anything.

## 10. License

**CC BY 4.0** — Attribution 4.0 International.
Copyright (c) 2026 **Andrea Bruno**.
Full text: [`LICENSE`](./LICENSE) · License deed: <https://creativecommons.org/licenses/by/4.0/>

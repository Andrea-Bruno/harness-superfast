# Adoption and Outreach Tracker

This page keeps a public record of where the Superfast Harness idea has been offered to other open-source agent projects. For each project we open a request in their own repository, we share the white paper and the license, and we ask that any adopted code keeps a credit to Andrea Bruno as required by CC BY 4.0. Later we can look at this table to see who considered the idea and whether the license terms were respected.

The idea itself is described in the [README](./README.md) and the [implementation guide](./qwen-code-implementation.md). The original license is in [LICENSE](./LICENSE).

How to read the table. Stars is the size of the project at the time we looked. Fork is our copy where we build the change. Issue and Discussion are the posts we opened in their repository. Pull request is the change we send them. License credit is whether their project docs carry the required credit after adoption. Pending means not done yet.

## The five we are working on now

We started with the five largest projects. We fork each one, build the Decision Gate inside it the same way we did for Qwen Code, and send a pull request that asks them to take the change and to keep the credit.

| Project | Stars | Language | Fork | Issue | Discussion | Pull request | License credit |
|---|---|---|---|---|---|---|---|
| OpenClaw | 390k | TypeScript | [fork](https://github.com/Andrea-Bruno/openclaw) | [#157418](https://github.com/openclaw/openclaw/issues/157418) | not available (discussions off) | [#157417](https://github.com/openclaw/openclaw/pull/157417) | pending |
| OpenCode | 209k | TypeScript | [fork](https://github.com/Andrea-Bruno/opencode) | [#51156](https://github.com/anomalyco/opencode/issues/51156) | not available (discussions off) | [#51155](https://github.com/anomalyco/opencode/pull/51155) | pending |
| DeepSeek Harness | 234k | TypeScript + Python | [fork](https://github.com/Andrea-Bruno/deepseek-harness) | not open (issues disabled) | [#7768](https://github.com/deepseek-ai/deepseek-harness/discussions/7768) | blocked (external PRs off); [branch ready](https://github.com/Andrea-Bruno/deepseek-harness/tree/feat/superfast-decision-gate) | pending |
| Claude Code | 147k | docs and plugins only, no agent source | [fork](https://github.com/Andrea-Bruno/claude-code) | [#96789](https://github.com/anthropics/claude-code/issues/96789) | not available (discussions off) | not possible (no agent source) | pending |
| Gemini CLI | 107k | TypeScript | [fork](https://github.com/Andrea-Bruno/gemini-cli) | [#29483](https://github.com/google-gemini/gemini-cli/issues/29483) | [#29484](https://github.com/google-gemini/gemini-cli/discussions/29484) | [#29482](https://github.com/google-gemini/gemini-cli/pull/29482) | pending |

### OpenClaw

OpenClaw runs tasks through a large set of skills. The natural place for the gate is just before the main model is asked what to do, so a plain request can be answered or routed without waking the big model. The exact spot in their loop still needs to be read from their current source before we wire it.

### DeepSeek Harness

DeepSeek Harness is built around plugins. The gate fits best as a plugin that runs before the main model call and returns a fast typed decision. Their repository does not allow issues, so we reach them through a discussion instead.

### OpenCode

OpenCode keeps a session loop that sends each user message to the model. The gate sits at the start of that loop, before the model call, and can answer or route simple turns in one fast pass. This is close to how it works in Qwen Code.

### Claude Code

Claude Code is a terminal coding agent with a turn loop that calls the model on every message. The gate sits before that call. We send the change and ask them to keep the credit if they take it.

### Gemini CLI

Gemini CLI is the upstream that Qwen Code was built from, so the port is close to a direct copy. The gate is a small module in the core that reads each user turn and, in shadow mode, records what it would have routed without changing anything. It is off by default and fails open.

## The next five (round 2, 2026-09-26)

We picked the next five projects from the list below, choosing the largest ones that are real agent harnesses with a "user message to decide to act" loop. We left out Daytona (a code sandbox with no decision loop), LiteLLM (an LLM API proxy), and oh-my-openagent (an ambiguous prompt/graph tool), because proposing a Decision Gate there would be off target. Same method as before: fork, build the gate inside as a shadow-mode, off-by-default, fail-open increment, and send an issue and a pull request that ask them to keep the credit.

| Project | Stars | Language | Fork | Issue | Discussion | Pull request | License credit |
|---|---|---|---|---|---|---|---|
| OpenHands | 89k | TypeScript (Agent Canvas frontend; the agent runs on a separate Agent Server) | [fork](https://github.com/Andrea-Bruno/OpenHands) | [#17714](https://github.com/OpenHands/OpenHands/issues/17714) | not available | [#17715](https://github.com/OpenHands/OpenHands/pull/17715) (draft) | pending |
| deer-flow | 82k | Python | [fork](https://github.com/Andrea-Bruno/deer-flow) | [#5877](https://github.com/bytedance/deer-flow/issues/5877) | not available | [#5878](https://github.com/bytedance/deer-flow/pull/5878) | pending |
| Goose | 54k | Rust | [fork](https://github.com/Andrea-Bruno/goose) | [#12529](https://github.com/aaif-goose/goose/issues/12529) | not available | [#12530](https://github.com/aaif-goose/goose/pull/12530) | pending |
| Aider | 49k | Python | [fork](https://github.com/Andrea-Bruno/aider) | [#5769](https://github.com/Aider-AI/aider/issues/5769) | not available | [#5770](https://github.com/Aider-AI/aider/pull/5770) | pending |
| Khoj | 37k | Python | [fork](https://github.com/Andrea-Bruno/khoj) | [#1434](https://github.com/khoj-ai/khoj/issues/1434) | not available | [#1435](https://github.com/khoj-ai/khoj/pull/1435) | pending |

### OpenHands

The current OpenHands repository is no longer the Python controller and CodeActAgent the plan assumed. It is now the Agent Canvas TypeScript and React frontend, and the agent loop and the real model call run on a separate Agent Server reached through a client library. We ported the gate to TypeScript and wired the shadow hook at the real front door of this client, the point where a raw user message is dispatched to the agent backend. The gate module is `src/superfast/decision-gate.ts` and the shadow call sits in the `sendMessage` path in `src/contexts/conversation-websocket-context.tsx`. The pull request is a draft because their process reserves a human-only section in the template and wants a maintainer label on the issue before a PR is reviewed.

### deer-flow

deer-flow is a LangChain and LangGraph harness driven by a middleware chain on a lead agent, not the graph coordinator and planner the plan guessed. We wired the gate as a middleware inserted at the front of the chain, so its before-model hook reads the raw latest user message before any context is injected. The module is `backend/packages/harness/deerflow/superfast/decision_gate.py` and the hook is in `build_middlewares()` in the lead agent file. It only logs the route and the latency and never changes state.

### Goose

Goose is a Rust agent. We added the gate as a new module `crates/goose/src/agents/superfast.rs` and wired a shadow call in `Agent::reply_impl` right after the existing user-prompt hook and before the reply loop. It runs on a detached task and only logs the route and the latency through tracing. The build passes cargo fmt and cargo check with no new dependencies. Their process wants an issue to reach a Ready state on their board before a PR is merged, so the pull request is framed as a reference for discussion.

### Aider

Aider is a Python pair-programming agent. We added `aider/superfast.py` and wired a one-line shadow hook at the top of `Coder.send_message` in `aider/coders/base_coder.py`, the method that appends the user prompt and calls the model. It runs on a background thread and only logs the route and the latency through aider's own output. Lint and formatting pass. Their contributor license agreement is a human step if the maintainers proceed.

### Khoj

Khoj is a Python personal AI assistant. The package lives under `src/khoj/` and the chat entry is the FastAPI `event_generator` in `src/khoj/routers/api_chat.py`. We added the gate as `src/khoj/utils/superfast_gate.py` and wired a one-line shadow hook right after the empty-query guard and before the first model call. It runs as a fire-and-forget background task and only logs the route and the latency through the project logger. It uses aiohttp, which Khoj already depends on. Ruff check and format pass. No CLA is required and the pull request opened without restriction.

## The next six (round 3, 2026-09-26)

We continued down the list and reached the remaining projects that are real agent harnesses with a "user message to decide to act" loop. We did them one at a time. Same method as before: fork, build the gate inside as a shadow-mode, off-by-default, fail-open increment, and send an issue and a pull request that ask them to keep the credit.

| Project | Stars | Language | Fork | Issue | Discussion | Pull request | License credit |
|---|---|---|---|---|---|---|---|
| CodeWhale | 41k | Rust | [fork](https://github.com/Andrea-Bruno/Codewhale) | [#6603](https://github.com/Hmbown/Codewhale/issues/6603) | not available | [#6604](https://github.com/Hmbown/Codewhale/pull/6604) | pending |
| SWE-agent | 20k | Python | [fork](https://github.com/Andrea-Bruno/SWE-agent) | [#1570](https://github.com/SWE-agent/SWE-agent/issues/1570) | not available | [#1571](https://github.com/SWE-agent/SWE-agent/pull/1571) | pending |
| OpenHarness | 16k | Python | [fork](https://github.com/Andrea-Bruno/OpenHarness) | [#361](https://github.com/HKUDS/OpenHarness/issues/361) | not available | [#362](https://github.com/HKUDS/OpenHarness/pull/362) | pending |
| Omnigent | 10k | Python | [fork](https://github.com/Andrea-Bruno/omnigent) | [#8332](https://github.com/omnigent-ai/omnigent/issues/8332) | not available | [#8333](https://github.com/omnigent-ai/omnigent/pull/8333) | pending |
| Mini-SWE-agent | 8k | Python | [fork](https://github.com/Andrea-Bruno/mini-swe-agent) | [#972](https://github.com/SWE-agent/mini-swe-agent/issues/972) | not available | [#973](https://github.com/SWE-agent/mini-swe-agent/pull/973) | pending |
| Mistral Vibe | 5k | Python | [fork](https://github.com/Andrea-Bruno/mistral-vibe) | [#1147](https://github.com/mistralai/mistral-vibe/issues/1147) | not available | [#1148](https://github.com/mistralai/mistral-vibe/pull/1148) | pending |

### CodeWhale

CodeWhale is a Rust terminal coding agent that welcomes issues and pull requests. We added the gate as `crates/tui/src/superfast.rs` and wired a shadow call in `run_turn` in `crates/tui/src/core/engine/turn_loop.rs`, right before the model dispatch and only on the first step of a turn. It runs on a detached task and only logs the route and the latency through tracing. The build passes cargo fmt and cargo check under the workspace deny-warnings policy. The repo is MIT licensed, which is compatible with our CC BY 4.0 attribution ask.

### SWE-agent

SWE-agent solves GitHub issues. The loop had moved to `sweagent/agent/agents.py`. We added `sweagent/superfast/decision_gate.py` and wired a shadow hook in `DefaultAgent.forward` right before the model query. It uses the project's existing requests client and only logs the route and the latency. Ruff check and format pass. No CLA is required.

### OpenHarness

OpenHarness is a Python open agent harness with a built-in personal agent. We added `src/openharness/superfast/decision_gate.py` and wired a shadow hook in `QueryEngine.submit_message` in `src/openharness/engine/query_engine.py`, fired as a background asyncio task before the model loop. It uses the project's existing httpx client. Ruff check passes and a changelog entry was added per their contributing guide. No CLA is required.

### Omnigent

Omnigent is a meta-harness that orchestrates other coding agents. We added `omnigent/superfast/decision_gate.py` and wired a shadow hook in the runner endpoint `post_session_events` in `omnigent/runner/app.py`, the last hop before a turn is dispatched to a harness. It is fire and forget and only logs the route and the latency. Ruff check and format pass. The repo requires only the DCO, which the commit carries.

### Mini-SWE-agent

Mini-SWE-agent is a deliberately tiny agent. We kept the change tiny to respect that. We added `src/minisweagent/utils/superfast.py` and wired a one-line shadow hook in `DefaultAgent.query` in `src/minisweagent/agents/default.py`, right before the model call, running on a daemon thread. It uses the project's existing requests client. Ruff check passes. No CLA is required.

### Mistral Vibe

Mistral Vibe is a minimal CLI coding agent by Mistral. We added `vibe/core/superfast_gate.py` and wired a shadow hook in `AgentLoop._conversation_loop` in `vibe/core/agent_loop/_loop.py`, fired as a background asyncio task before the turn loop. It uses the project's existing httpx client and disables proxy tunnelling for the local decision call. Ruff check and format pass. Their contributing guide says the repo is not currently accepting code contributions but does encourage issues and feedback, so the pull request may be treated as a proposal rather than an accepted contribution. Both the issue and the pull request are open regardless.

## The rest, to be done over time

These are the other projects from our list that have a public repository. We will reach them later, spread out so the work looks like normal open-source contribution rather than a single large push.

| Project | Stars | Language | Repo | Status |
|---|---|---|---|---|
| OpenHands | 89k | Python | [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) | done (round 2) |
| deer-flow | 82k | Python | [bytedance/deer-flow](https://github.com/bytedance/deer-flow) | done (round 2) |
| Daytona | 71k | TypeScript | [daytonaio/daytona](https://github.com/daytonaio/daytona) | todo |
| oh-my-openagent | 69k | TypeScript | [code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) | todo |
| LiteLLM | 59k | Python | [BerriAI/litellm](https://github.com/BerriAI/litellm) | todo |
| Aider | 49k | Python | [Aider-AI/aider](https://github.com/Aider-AI/aider) | done (round 2) |
| CodeWhale | 41k | Rust | [Hmbown/Codewhale](https://github.com/Hmbown/Codewhale) | done (round 3) |
| Khoj | 37k | Python | [khoj-ai/khoj](https://github.com/khoj-ai/khoj) | done (round 2) |
| SWE-agent | 20k | Python | [SWE-agent/SWE-agent](https://github.com/SWE-agent/SWE-agent) | done (round 3) |
| OpenHarness | 16k | Python | [HKUDS/OpenHarness](https://github.com/HKUDS/OpenHarness) | done (round 3) |
| Mini-SWE-agent | 8k | Python | [SWE-agent/mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) | done (round 3) |
| Mistral Vibe | 5k | Python | [mistralai/mistral-vibe](https://github.com/mistralai/mistral-vibe) | done (round 3) |
| Omnigent | 10k | Python | [omnigent-ai/omnigent](https://github.com/omnigent-ai/omnigent) | done (round 3) |
| Goose | 54k | Rust | [aaif-goose/goose](https://github.com/aaif-goose/goose) | done (round 2) |

Two names from the original list, MiniMax Code and Hermes, did not have a clear public repository we could find, so they are left out until we get the right link.

## What we ask of every project

If a project takes the idea or the code, we ask for one thing under the license. Keep a short credit to Andrea Bruno and a link to this repository in their project documentation. That is the whole condition. The idea is free to use, change, and ship, including in commercial products. We chose not to patent it so that everyone can build on it.

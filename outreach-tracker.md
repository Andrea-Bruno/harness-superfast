# Adoption and Outreach Tracker

This page keeps a public record of where the Superfast Harness idea has been offered to other open-source agent projects. For each project we open a request in their own repository, we share the white paper and the license, and we ask that any adopted code keeps a credit to Andrea Bruno as required by CC BY 4.0. Later we can look at this table to see who considered the idea and whether the license terms were respected.

The idea itself is described in the [README](./README.md) and the [implementation guide](./qwen-code-implementation.md). The original license is in [LICENSE](./LICENSE).

How to read the table. Stars is the size of the project at the time we looked. Fork is our copy where we build the change. Issue and Discussion are the posts we opened in their repository. Pull request is the change we send them. License credit is whether their project docs carry the required credit after adoption. Pending means not done yet.

## The five we are working on now

We started with the five largest projects. We fork each one, build the Decision Gate inside it the same way we did for Qwen Code, and send a pull request that asks them to take the change and to keep the credit.

| Project | Stars | Language | Fork | Issue | Discussion | Pull request | License credit |
|---|---|---|---|---|---|---|---|
| OpenClaw | 390k | to confirm | [fork](https://github.com/Andrea-Bruno/openclaw) | pending | pending | pending | pending |
| DeepSeek Harness | 234k | to confirm | [fork](https://github.com/Andrea-Bruno/deepseek-harness) | not open (issues disabled) | pending | pending | pending |
| OpenCode | 209k | TypeScript | [fork](https://github.com/Andrea-Bruno/opencode) | pending | pending | pending | pending |
| Claude Code | 147k | TypeScript | [fork](https://github.com/Andrea-Bruno/claude-code) | pending | pending | pending | pending |
| Gemini CLI | 107k | TypeScript | [fork](https://github.com/Andrea-Bruno/gemini-cli) | pending | pending | pending | pending |

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

## The rest, to be done over time

These are the other projects from our list that have a public repository. We will reach them later, spread out so the work looks like normal open-source contribution rather than a single large push.

| Project | Stars | Language | Repo | Status |
|---|---|---|---|---|
| OpenHands | 89k | Python | [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) | todo |
| deer-flow | 82k | Python | [bytedance/deer-flow](https://github.com/bytedance/deer-flow) | todo |
| Daytona | 71k | TypeScript | [daytonaio/daytona](https://github.com/daytonaio/daytona) | todo |
| oh-my-openagent | 69k | TypeScript | [code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) | todo |
| LiteLLM | 59k | Python | [BerriAI/litellm](https://github.com/BerriAI/litellm) | todo |
| Aider | 49k | Python | [Aider-AI/aider](https://github.com/Aider-AI/aider) | todo |
| CodeWhale | 41k | Rust | [Hmbown/Codewhale](https://github.com/Hmbown/Codewhale) | todo |
| Khoj | 37k | Python | [khoj-ai/khoj](https://github.com/khoj-ai/khoj) | todo |
| SWE-agent | 20k | Python | [SWE-agent/SWE-agent](https://github.com/SWE-agent/SWE-agent) | todo |
| OpenHarness | 16k | to confirm | [HKUDS/OpenHarness](https://github.com/HKUDS/OpenHarness) | todo |
| Mini-SWE-agent | 8k | Python | [SWE-agent/mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) | todo |
| Mistral Vibe | 5k | to confirm | [mistralai/mistral-vibe](https://github.com/mistralai/mistral-vibe) | todo |
| Omnigent | 10k | to confirm | [omnigent-ai/omnigent](https://github.com/omnigent-ai/omnigent) | todo |
| Goose | to confirm | Rust | [aaif-goose/goose](https://github.com/aaif-goose/goose) | todo |

Two names from the original list, MiniMax Code and Hermes, did not have a clear public repository we could find, so they are left out until we get the right link.

## What we ask of every project

If a project takes the idea or the code, we ask for one thing under the license. Keep a short credit to Andrea Bruno and a link to this repository in their project documentation. That is the whole condition. The idea is free to use, change, and ship, including in commercial products. We chose not to patent it so that everyone can build on it.

+++
title = 'The Agent Loop: The Heart of Every AI Agent'
date = 2026-03-28T21:00:00-07:00
draft = true
tags = ['agents', 'architecture']
categories = ['agent-systems']
+++

Every AI agent, from AutoGPT to OpenClaw, runs on the same fundamental pattern: the agent loop.

## The Problem

Large language models are stateless. You give them input, they produce output, and then they forget everything. But agents need to *persist* — to work on tasks over time, to remember what they've done, to adjust their approach based on results.

The agent loop is the pattern that bridges this gap.

## How It Works

At its core, an agent loop is:

```
while not done:
    observation = get_state()
    thought = llm.think(observation, memory)
    action = llm.decide(thought)
    result = execute(action)
    memory.update(observation, action, result)
```

This simple cycle — **observe → think → act → remember** — is the heartbeat of every agent system.

```
┌─────────────────────────────────────────────────────┐
│                    AGENT LOOP                        │
│                                                      │
│    ┌──────────┐      ┌──────────┐                   │
│    │ Observe  │ ──── │  Think   │                   │
│    │          │      │          │                   │
│    │ Context  │      │   LLM    │                   │
│    │ Memory   │      │ Response │                   │
│    └──────────┘      └────┬─────┘                   │
│         ▲                 │                          │
│         │                 ▼                          │
│    ┌────┴─────┐      ┌──────────┐                   │
│    │ Remember │ ◄─── │   Act    │                   │
│    │          │      │          │                   │
│    │ Update   │      │ Tool     │                   │
│    │ State    │      │ Call     │                   │
│    └──────────┘      └──────────┘                   │
│                                                      │
└─────────────────────────────────────────────────────┘
```

## Key Components

| Component | What It Does |
|-----------|--------------|
| **Observe** | Gather current context: user input, environment state, recent history |
| **Think** | LLM processes observation, reasons about next steps |
| **Act** | Execute a tool call, generate text, or modify state |
| **Remember** | Update memory, persist state, log results |

## Stateless vs Stateful Loops

**Stateless loops** (e.g., simple chatbots) reset after each interaction. Each turn is independent.

**Stateful loops** (e.g., autonomous agents) carry context across turns. They remember what they've tried, what failed, what worked.

The complexity lives in that statefulness.

## Real World Example

Here's a simplified version from OpenClaw's main agent loop:

```typescript
// Simplified OpenClaw agent loop
async function runAgentLoop(session: Session) {
  let turns = 0;
  const maxTurns = session.config.maxTurns;

  while (turns < maxTurns) {
    const context = await buildContext(session);
    const response = await llm.generate(context, session.tools);

    if (response.done) {
      return response.result;
    }

    if (response.toolCall) {
      const result = await executeTool(response.toolCall, session);
      session.memory.push({ role: 'tool', content: result });
    }

    turns++;
  }

  throw new Error('Max turns exceeded');
}
```

## Why This Breaks in Production

- **Context overflow**: Long-running tasks exhaust the context window
- **Loop detection**: Agent repeats the same action without progress
- **Tool failures**: External APIs fail, and the agent doesn't know how to recover
- **Cost spirals**: Each loop iteration costs tokens; unbounded loops drain budgets
- **State corruption**: Memory gets polluted with errors, hallucinations, or irrelevant data

## What's Next

In the next post, we'll dive into **Tool Calling** — how agents decide what to do, and how to design tool interfaces that don't break.

---

## References

- [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) - Lilian Weng
- [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) - Yao et al., 2023
- [OpenClaw Source Code](https://github.com/openclaw/openclaw)

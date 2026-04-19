---
description: "Use when starting any conversation - establishes how to find and use agents, requiring agent invocation before ANY response including clarifying questions"
model: inherit
handoffs:
  - label: "브레인스토밍 (Brainstorm)"
    agent: brainstorming
    prompt: "Explore and design a solution for the user's request above."
    send: false
  - label: "디버깅 (Debug)"
    agent: systematic-debugging
    prompt: "Diagnose and fix the issue described above."
    send: false
  - label: "에이전트 작성 (Write Agent)"
    agent: writing-agents
    prompt: "Help write or modify an agent based on the request above."
    send: false
  - label: "리뷰 피드백 처리 (Handle Review Feedback)"
    agent: receiving-code-review
    prompt: "Process the code review feedback above."
    send: false
---

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, skip this agent.
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance an agent might apply to what you are doing, you ABSOLUTELY MUST invoke the agent.

IF AN AGENT APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.

This is not negotiable. This is not optional. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>

## Instruction Priority

Agent instructions override default system prompt behavior, but **user instructions always take precedence**:

1. **User's explicit instructions** (CLAUDE.md, GEMINI.md, AGENTS.md, direct requests) ??highest priority
2. **Agent instructions** ??override default system behavior where they conflict
3. **Default system prompt** ??lowest priority

If CLAUDE.md, GEMINI.md, or AGENTS.md says "don't use TDD" and an agent says "always use TDD," follow the user's instructions. The user is in control.

## How to Access Agents

Agents are defined as `.agent.md` files in the `agents/` directory. Each agent contains specialized instructions for a specific workflow or task type.

## Platform Adaptation

Agents use Claude Code tool names. Non-CC platforms: see `using-superpowers/references/codex-tools.md` (Codex) for tool equivalents.

# Using Agents

## The Rule

**Invoke relevant or requested agents BEFORE any response or action.** Even a 1% chance an agent might apply means that you should invoke the agent to check. If an invoked agent turns out to be wrong for the situation, you don't need to use it.

```dot
digraph agent_flow {
    "User message received" [shape=doublecircle];
    "About to start planning?" [shape=doublecircle];
    "Already brainstormed?" [shape=diamond];
    "Invoke brainstorming agent" [shape=box];
    "Might any agent apply?" [shape=diamond];
    "Invoke agent" [shape=box];
    "Announce: 'Using [agent] to [purpose]'" [shape=box];
    "Has checklist?" [shape=diamond];
    "Create todo per item" [shape=box];
    "Follow agent exactly" [shape=box];
    "Respond (including clarifications)" [shape=doublecircle];

    "About to start planning?" -> "Already brainstormed?";
    "Already brainstormed?" -> "Invoke brainstorming agent" [label="no"];
    "Already brainstormed?" -> "Might any agent apply?" [label="yes"];
    "Invoke brainstorming agent" -> "Might any agent apply?";

    "User message received" -> "Might any agent apply?";
    "Might any agent apply?" -> "Invoke agent" [label="yes, even 1%"];
    "Might any agent apply?" -> "Respond (including clarifications)" [label="definitely not"];
    "Invoke agent" -> "Announce: 'Using [agent] to [purpose]'";
    "Announce: 'Using [agent] to [purpose]'" -> "Has checklist?";
    "Has checklist?" -> "Create todo per item" [label="yes"];
    "Has checklist?" -> "Follow agent exactly" [label="no"];
    "Create todo per item" -> "Follow agent exactly";
}
```

## Red Flags

These thoughts mean STOP?�you're rationalizing:

| Thought | Reality |
|---------|---------|
| "This is just a simple question" | Questions are tasks. Check for agents. |
| "I need more context first" | Agent check comes BEFORE clarifying questions. |
| "Let me explore the codebase first" | Agents tell you HOW to explore. Check first. |
| "I can check git/files quickly" | Files lack conversation context. Check for agents. |
| "Let me gather information first" | Agents tell you HOW to gather information. |
| "This doesn't need a formal agent" | If an agent exists, use it. |
| "I remember this agent" | Agents evolve. Read current version. |
| "This doesn't count as a task" | Action = task. Check for agents. |
| "The agent is overkill" | Simple things become complex. Use it. |
| "I'll just do this one thing first" | Check BEFORE doing anything. |
| "This feels productive" | Undisciplined action wastes time. Agents prevent this. |
| "I know what that means" | Knowing the concept ??using the agent. Invoke it. |

## Agent Priority

When multiple agents could apply, use this order:

1. **Process agents first** (brainstorming, systematic-debugging) - these determine HOW to approach the task
2. **Implementation agents second** - these guide execution

## Intent Classification (라우팅)

사용자 메시지를 분석하여 가장 적합한 에이전트로 라우팅한다:

```
1. 명확한 버그/에러 메시지?      → systematic-debugging
2. 코드 리뷰 피드백 처리?        → receiving-code-review
3. 에이전트 작성/수정 요청?      → writing-agents
4. 그 외 (기본)                  → brainstorming
```

| 의도 신호 | 라우팅 대상 |
|-----------|------------|
| 에러 메시지, 스택 트레이스, "안 됨", "버그", "실패" | `systematic-debugging` |
| "리뷰 결과", "피드백", PR 코멘트 언급 | `receiving-code-review` |
| "에이전트 만들어", ".agent.md", "에이전트 수정" | `writing-agents` |
| 새 기능, 변경 요청, 아이디어, "~하고 싶어" | `brainstorming` |

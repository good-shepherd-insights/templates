# Telegram Chat Manager (Mastra)

| Owner | Status | Decided |
|---|---|---|
| | proposed | |

## Problem

Manual triage across 15+ group chats; missed escalations; no cross-conversation memory.

## Goal

One Mastra agent triages every chat, remembers context, escalates only what needs a human.

## Layer: Ingress

```mermaid
Telegram → Bot API → Mastra channel → Agent
```

| # | Decision | Options | Chosen | Why | Ref |
|---|---|---|---|---|---|
| 1 | Access method | Bot API / MCP client | Bot API | Native; MCP needs terminal login | [mcp-telegram](https://github.com/mcp-telegram/mcp-telegram/blob/HEAD/docs/platforms/mastra.md) · [channel](https://mastra.ai/integrations/channels/telegram) |

| # | Risk | Mitigation |
|---|---|---|
| 1 | Bot token leak | Vault only; rotate on exposure |

## Layer: Agent

```typescript
const chatManager = new Agent({
  id: "telegram-chat-manager",
  instructions: "Triage: classify, route, summarize. Escalate unknowns.",
  model: openai("gpt-4o"),
});
```

| # | Decision | Options | Chosen | Why | Ref |
|---|---|---|---|---|---|
| 1 | Topology | Single / network() | Single | One surface; network() when specialists added | [channels](https://mastra.ai/docs/channels) |

| # | Risk | Mitigation |
|---|---|---|
| 1 | Prompt injection via group msgs | Classify-before-act; tools allowlisted |

## Layer: Memory

```typescript
memory: {} // messages / working / semantic / RAG
```

| # | Decision | Options | Chosen | Why | Ref |
|---|---|---|---|---|---|
| 1 | Store | 4-tier built-in / external | 4-tier built-in | Cross-chat context, no extra infra | [mastra-vs-eve](https://github.com/bitdoze/bitdoze.com/blob/HEAD/src/content/posts/mastra-vs-eve-typescript-ai-agents.mdx) |

| # | Risk | Mitigation |
|---|---|---|
| 1 | Unbounded growth | TTL + semantic compaction |

## Layer: Actions

```typescript
tools: { classifyThread, summarizeChat, escalateToHuman} // wired into Agent
```

| # | Decision | Options | Chosen | Why | Ref |
|---|---|---|---|---|---|
| 1 | Tool surface | Allowlisted / open exec | Allowlisted | Fixed surface, no surprises | [provider ref](https://mastra.ai/reference/channels/channel-provider) |

| # | Risk | Mitigation |
|---|---|---|
| 1 | False escalation spam | Confidence threshold before escalate |

## Scope

| In | Out |
|---|---|
| Classify + route incoming messages | Multi-tenant pairing / deep links |
| Per-chat memory + summaries | Payments, stickers, voice |
| /status /escalate admin commands | WhatsApp / Discord channels |
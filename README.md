# Personal Automation Assistant (AI Agent)

A conversational AI agent — not a fixed pipeline — that connects to multiple personal data sources and decides on its own which one to use based on what's actually asked, with working memory across messages and the ability to both read and write data.

## Problem it solves

The first four projects in this portfolio are each single-purpose: one trigger, one fixed path, one job. This project ties them together into a single conversational interface — "how much have I spent on rent," "any updates on my job applications," "any qualified leads" — without pre-building a branch for every possible question. The agent reads tool descriptions and routes itself.

## Architecture

```
Telegram message
  → AI Agent (Groq: llama-3.3-70b-versatile)
      ├─ Memory: conversation history per chat, so follow-up questions work
      ├─ Tool: read personal expenses (Google Sheets)
      ├─ Tool: read job application tracker (Google Sheets)
      ├─ Tool: read sales leads (Airtable)
      └─ Tool: log a new expense (Google Sheets, write)
  → Telegram reply, generated from whichever tool(s) the agent decided to call
```

## Tech stack

- **n8n** — AI Agent node, Memory node, Tool-type sub-nodes
- **Groq API** (llama-3.3-70b-versatile) — agent reasoning and tool-calling
- **Telegram Bot API** — conversational interface
- **Google Sheets & Airtable APIs** — connected as agent tools, both read and write
- **Window Buffer Memory** — per-chat conversation history

## Features

### Multi-tool reasoning
A single agent decides which of four connected tools (or none) to use based on the user's actual question — no manually pre-built routing logic.

### Conversation memory
Follow-up questions ("what about last week?") resolve correctly because the agent has access to recent message history, not just the current one.

### Read and write capability
Most tools look data up; one tool lets the agent log a brand-new expense directly from a casual sentence ("log 20 for taxi"), parsed and written without a rigid command format.

### Provider-agnostic design
The underlying language model was swapped from Gemini to Groq mid-project with zero changes to the agent's memory, tools, or logic — proof the architecture isn't tied to one vendor.

## Setup

You'll need:
1. A dedicated Telegram bot token
2. A Groq API key (or any other supported chat model provider)
3. Existing Google Sheets (expenses, job applications) and Airtable base (leads) — reused from earlier projects in this portfolio
4. Clear, explicit Tool Description text for every connected tool — this is what the agent actually reads to decide when and how to call each one

## Challenges & debugging

- **Stale system message after adding a tool:** the agent's system message originally said "you have no tools" — written before any tools existed. After adding the first tool, the agent kept following that outdated instruction and refused to look anything up, even though a working tool was wired in. Fixed by treating the system message as something to update every time a tool is added or removed, not a one-time setup step.
- **Rate limiting on the free model tier:** rapid testing hit Gemini's free-tier request limit. Solved by swapping the Chat Model sub-node to Groq — a clean swap since memory, tools, and the agent's own configuration are all attached to the Agent node, not the model.
- **Tool call schema mismatch:** a faster model invented filter parameters (`filter_key`, `filter_value`) that the underlying Sheets tool was never built to accept, since nothing told it the tool took no input. Fixed by being explicit in the tool description that the tool accepts no parameters and that filtering should happen in the model's own reasoning, not as a tool argument.

## Demo

🎥 [Loom walkthrough — add link here]

## Possible next steps

- A proactive daily digest (Schedule Trigger + all data sources + conditional alerting, silent on quiet days)
- A second write tool for logging job applications conversationally
- Wrap the write tool in a small validation workflow for stricter data consistency

## Running cost

Telegram Bot API, Google Sheets API, Airtable API: free. Groq (llama-3.3-70b-versatile): free tier, rate-limited rather than metered. n8n: free if self-hosted, or from $24/month on n8n Cloud after the trial period.

<p align="center">
  <img src="https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge" alt="Status">
  <img src="https://img.shields.io/badge/Version-0.1.0-blue?style=for-the-badge" alt="Version">
  <img src="https://img.shields.io/badge/Python-3.12+-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/License-Private-red?style=for-the-badge" alt="License">
</p>

<h1 align="center">AI Agent Framework</h1>
<h3 align="center">Plugin-Based Python Framework for Production AI Agents</h3>

<p align="center">
  <em>A production-ready async Python framework that powers multiple AI agents across different projects. Features a multi-provider AI chain (Codex/Claude fallback), dual event system (Redis Pub/Sub + PostgreSQL NOTIFY), plugin-based multi-site architecture, and battle-tested resilience patterns.</em>
</p>

---

## What It Does

This framework runs **3 production AI agents** that autonomously handle user support, feedback analysis, and SEO optimization across two SaaS products:

| Agent | Project | What It Does |
|-------|---------|--------------|
| **Herald** | [GuildScout](https://github.com/Commandershadow9/GuildScout-Showcase) | Analyzes user feedback, conducts multi-turn Discord DM conversations, syncs to GitHub Issues |
| **Zara** | [ZERODOX](https://github.com/Commandershadow9/ZERODOX-Showcase) | AI support agent with ticket classification, source code analysis for bug reports, escalation detection |
| **SEO Agent** | ZERODOX | Autonomous 17-step SEO pipeline: crawl, audit, AI-fix, PR creation, Discord reports, intelligence |

---

## Tech Stack

<table>
<tr>
<td align="center" width="150">
<strong>Core</strong><br>
<img src="https://img.shields.io/badge/Python_3.12-3776AB?style=flat&logo=python&logoColor=white" alt="Python"><br>
<img src="https://img.shields.io/badge/asyncio-333?style=flat" alt="asyncio"><br>
<img src="https://img.shields.io/badge/asyncpg-333?style=flat" alt="asyncpg"><br>
<img src="https://img.shields.io/badge/httpx-333?style=flat" alt="httpx">
</td>
<td align="center" width="150">
<strong>AI Providers</strong><br>
<img src="https://img.shields.io/badge/GPT--5.3_Codex-412991?style=flat&logo=openai&logoColor=white" alt="Codex"><br>
<img src="https://img.shields.io/badge/Claude_Sonnet_4.6-D97706?style=flat&logo=anthropic&logoColor=white" alt="Claude"><br>
<img src="https://img.shields.io/badge/JSON_Schema-333?style=flat" alt="Schema">
</td>
<td align="center" width="150">
<strong>Events & Data</strong><br>
<img src="https://img.shields.io/badge/Redis-DC382D?style=flat&logo=redis&logoColor=white" alt="Redis"><br>
<img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=flat&logo=postgresql&logoColor=white" alt="PostgreSQL"><br>
<img src="https://img.shields.io/badge/PG_NOTIFY-333?style=flat" alt="PG NOTIFY">
</td>
<td align="center" width="150">
<strong>Integrations</strong><br>
<img src="https://img.shields.io/badge/GitHub_API-181717?style=flat&logo=github&logoColor=white" alt="GitHub"><br>
<img src="https://img.shields.io/badge/Discord-5865F2?style=flat&logo=discord&logoColor=white" alt="Discord"><br>
<img src="https://img.shields.io/badge/Google_APIs-4285F4?style=flat&logo=google&logoColor=white" alt="Google">
</td>
</tr>
</table>

---

## Architecture

### Framework Overview

```
┌─────────────────────────────────────────────────────────┐
│                    core/runner.py                        │
│         argparse → Config → Subscriber → Consumer       │
└──────────┬──────────────────────────────┬───────────────┘
           │                              │
   ┌───────▼────────────┐       ┌─────────▼──────────┐
   │   Multi-Site Mode  │       │  Single-Agent Mode  │
   │   (feedback/)      │       │  (seo/)             │
   │                    │       │                     │
   │   FeedbackAgent    │       │   SEOAgent          │
   │      ↓ delegates   │       │      ↓ direct       │
   │   SiteHandler ABC  │       │   17-step pipeline  │
   │   ┌────────────┐   │       │                     │
   │   │ GuildScout │   │       └─────────────────────┘
   │   │ (Herald)   │   │
   │   ├────────────┤   │
   │   │ ZERODOX    │   │
   │   │ (Zara)     │   │
   │   └────────────┘   │
   └────────────────────┘
```

### AI Provider Chain

```
AIProviderChain (Strategy + Fallback Pattern)
│
├─► CodexProvider (Primary)
│   ├── GPT-5.3-Codex via CLI
│   ├── --output-schema for structured JSON
│   ├── JSONL stdout parsing for token tracking
│   └── Configurable timeouts (90s–900s)
│
└─► ClaudeProvider (Fallback)
    ├── Claude Sonnet 4.6 via CLI
    ├── Multi-strategy JSON extraction
    │   (direct → markdown block → brace matching)
    └── Automatic env cleanup (prevents nested sessions)
```

### Dual Event System

```
EventSubscriber ABC
│
├─► RedisSubscriber
│   ├── Async redis.asyncio client
│   ├── Auto-reconnect with exponential backoff (1s → 60s)
│   ├── Queue overflow protection
│   └── Used by: GuildScout, SEO Agent
│
└─► PgNotifySubscriber
    ├── asyncpg LISTEN/NOTIFY
    ├── Zero additional infrastructure (uses existing DB)
    ├── Auto-reconnect with exponential backoff
    └── Used by: ZERODOX
```

### ABC Hierarchy

```
BaseAgent
├── event_filter(data) → dict | None          [abstract]
├── run_consumer(queue)                        [abstract]
├── on_analysis_failed(item_id, item)          [hook, default: logging]
├── enrich_followup_context(item, analysis)    [hook, default: ""]
└── get_autonomy_rules()                       [hook, default: ""]

SiteHandler (extends BaseAgent interface)
├── Same abstract methods + hooks
└── Per-site: own DB queries, prompts, persona, schema, knowledge base

AIProvider
├── analyze(prompt, schema) → dict             [abstract]
└── generate_text(prompt) → str                [abstract]

EventSubscriber
├── connect() / disconnect()                   [abstract]
├── subscribe(channels, callback)              [abstract]
└── publish(channel, data)                     [abstract]

AgentDB
├── get_item(id) → dict                       [abstract]
├── save_analysis(id, analysis)                [abstract]
└── get_similar_items(id) → list               [abstract]
```

---

## Project Structure

```
agents/
├── core/                              # Generic framework (14 modules)
│   ├── runner.py                      #   Main loop: CLI → config → subscriber → consumer
│   ├── agent_base.py                  #   BaseAgent ABC with 3 hook methods
│   ├── config.py                      #   YAML config + .env loader + multi-site
│   ├── types.py                       #   Dataclasses (EventData, AnalysisResult, ProjectConfig)
│   ├── prompt_utils.py                #   Reusable prompt building blocks
│   ├── ai/                            #   AI provider subsystem
│   │   ├── base.py                    #     AIProvider ABC
│   │   ├── codex_provider.py          #     OpenAI Codex CLI integration
│   │   ├── claude_provider.py         #     Anthropic Claude CLI integration
│   │   ├── chain.py                   #     Provider fallback chain
│   │   └── validator.py               #     JSON schema validation + sanitization
│   ├── events/                        #   Event subscriber subsystem
│   │   ├── base.py                    #     EventSubscriber ABC
│   │   ├── redis_subscriber.py        #     Redis Pub/Sub with auto-reconnect
│   │   └── pg_notify.py               #     PostgreSQL LISTEN/NOTIFY
│   ├── db/                            #   Database abstraction
│   │   ├── base.py                    #     AgentDB ABC
│   │   └── asyncpg_adapter.py         #     asyncpg lazy connection pool
│   ├── sync/                          #   External sync
│   │   └── github_sync.py             #     GitHub API: Issues, Labels, Living Document
│   └── notify/                        #   Notification subsystem
│       ├── base.py                    #     Notifier ABC
│       ├── redis_publisher.py         #     Redis Pub/Sub publish
│       └── sse_notifier.py            #     HTTP POST to SSE endpoint
│
├── projects/                          # Project-specific plugins
│   ├── feedback/                      #   Multi-site feedback/support agent
│   │   ├── agent.py                   #     Thin wrapper (delegates to SiteHandler)
│   │   ├── config.yaml                #     Multi-site config (all sites in one file)
│   │   └── sites/
│   │       ├── base.py                #       SiteHandler ABC
│   │       ├── guildscout/            #       7 files: handler, queries, prompts, schema, ...
│   │       └── zerodox/               #       7 files: handler, queries, prompts, code_analyzer, ...
│   └── seo/                           #   Autonomous SEO auditor
│       ├── agent.py                   #     800-line main agent (17-step pipeline)
│       ├── crawler.py                 #     Website crawler (httpx, sitemap discovery)
│       ├── auditor.py                 #     Deterministic SEO checks
│       ├── fix_generator.py           #     Auto-fixes + PR creation via git/gh
│       ├── discord_notifier.py        #     Rich embeds (12 report types)
│       ├── insight_engine.py          #     Weekly AI intelligence (5 prompts)
│       ├── circuit_breaker.py         #     Resilience pattern
│       └── ...                        #     + 10 more specialized modules
│
├── tests/                             #   pytest + pytest-asyncio
├── run.sh                             #   Universal start script
└── pyproject.toml                     #   Package definition (uv managed)
```

---

## Features

### Framework Core
- **Plugin architecture** — two patterns: Multi-Site (feedback agent delegates to site handlers) and Single-Agent (SEO agent runs directly)
- **AI Provider Chain** — Codex CLI as primary with structured JSON output, Claude CLI as automatic fallback with multi-strategy JSON parsing
- **Dual event system** — Redis Pub/Sub for high-throughput, PostgreSQL LISTEN/NOTIFY for zero-infrastructure overhead
- **3 optional hooks** with sensible defaults — sites override only what they need
- **YAML-driven configuration** — env vars resolved at runtime via `_env` suffix convention
- **Graceful shutdown** via SIGTERM/SIGINT signal handling
- **Token usage tracking** — input/output tokens parsed from Codex JSONL stdout

### Agent: Herald (GuildScout Feedback)
- **Multi-turn Discord DM conversations** (up to 5 turns) with quick-reply buttons
- **3 conversation intents**: `user_error_check`, `need_more_info`, `feature_exists`
- **Duplicate detection** via AI summaries of existing feedback
- **GitHub Living Document** — issue body updated in-place with `<!-- GS:AI:START -->` markers
- **AI closure summaries** when feedback is resolved
- **Bilingual** (DE/EN auto-detect) with few-shot examples

### Agent: Zara (ZERODOX Support)
- **3-way ticket classification**: Support / Bug / Feedback — each with specialized handling
- **Automatic source code analysis** — maps failed API endpoints to Next.js source files, feeds code context to AI
- **Escalation detection** — DSGVO requests, data breaches, compromised accounts trigger `escalation_info` intent
- **Prompt injection detection** — 15+ pattern checks before AI analysis
- **Multi-channel notification** — Discord DM (primary) with email fallback
- **Dynamic known pitfalls** — top issues from last 7 days injected into system prompt
- **5-level decision tree** for response classification

### Agent: SEO Auditor
- **17-step autonomous pipeline**: Crawl → Checks → AI Analysis → Auto-Fix → PR → Discord → Intelligence
- **Framework adapters** for Next.js App Router, React Router (Vite), and generic sites
- **Circuit breaker** — GSC/PageSpeed: 3 failures → 1h cooldown; Discord: 3 failures → 30min
- **Site backoff** — 3 consecutive audit failures → 6h pause
- **Repo lock** — asyncio.Lock per repository against parallel git operations
- **Finding diff** with time-to-fix tracking and severity escalation
- **Post-merge impact monitoring** — checks if PRs were actually merged
- **Intelligence Engine** (weekly) — 5 AI prompts: fix impact, keywords, competitors, trends, strategy
- **12 Discord report types** with rich embeds and score visualizations
- **Google Search Console** integration — performance, indexing, keywords, backlinks
- **PageSpeed Insights** integration — Core Web Vitals monitoring

### Resilience & Security
- **Auto-reconnect** with exponential backoff (1s → 60s max)
- **Circuit breaker** pattern for external APIs
- **Queue overflow protection** with configurable max size
- **Path traversal protection** in code analyzer (denylist + `is_relative_to`)
- **Prompt injection detection** (15+ patterns)
- **JSON schema validation** with sanitization for AI outputs

---

## How It Runs

```bash
# Start agents manually
./run.sh feedback guildscout    # Herald — GuildScout Feedback Analyzer
./run.sh feedback zerodox       # Zara — ZERODOX Support Agent
./run.sh seo                    # SEO Agent (autonomous)

# Production (systemd user services)
systemctl --user start guildscout-feedback-agent
systemctl --user start zerodox-support-agent
systemctl --user start seo-agent
```

---

## Project Stats

| Metric | Value |
|--------|-------|
| **Total codebase** | ~12,000 lines |
| **Python files** | 61 |
| **Core modules** | 14 |
| **Abstract base classes** | 6 (BaseAgent, SiteHandler, AIProvider, EventSubscriber, AgentDB, Notifier) |
| **Project plugins** | 2 (feedback, seo) |
| **Active sites** | 3 (GuildScout, ZERODOX, ZERODOX SEO) |
| **AI providers** | 2 (Codex CLI, Claude CLI) |
| **Event systems** | 2 (Redis Pub/Sub, PG NOTIFY) |
| **SEO pipeline steps** | 17 |
| **Discord report types** | 12 |
| **SEO DB tables** | 10 + 3 composite indexes |
| **External API integrations** | 5 (GitHub, Discord, Google Search Console, PageSpeed, Google Trends) |
| **Runtime dependencies** | 6 |

---

## Design Philosophy

1. **Convention over configuration** — new sites just need a directory with handler + prompts + schema
2. **Hooks over inheritance** — 3 optional hooks with defaults beat deep class hierarchies
3. **Fail gracefully** — every external call has a fallback (AI chain, reconnect, circuit breaker)
4. **Zero-overhead integration** — PG NOTIFY reuses existing databases, no extra infrastructure
5. **CLI-first AI** — shell out to Codex/Claude CLIs instead of managing API keys and SDKs directly

---

## Status

This framework is actively running in production, powering 3 AI agents across 2 SaaS products on a Debian 12 VPS. The source code is in a **private repository**.

---

<p align="center">
  <em>Built by <a href="https://github.com/Commandershadow9">Commandershadow9</a></em>
</p>

# AionAxis

> **Architecting Controllable, Self-Improving AI Agents at Scale**

[![Status](https://img.shields.io/badge/status-engineering%20design-blue)]()
[![Version](https://img.shields.io/badge/version-1.0-navy)]()
[![License](https://img.shields.io/badge/license-proprietary-red)]()
[![Author](https://img.shields.io/badge/author-ClassifiedThoughts-0077B6)]()

---

## Overview

AionAxis is a design framework and reference architecture for building self-improving autonomous AI agents that remain under reliable human control.

Current autonomous agent systems face a fundamental tension: the more capable and self-directed an agent becomes, the harder it is to constrain. AionAxis resolves this tension through a **three-mechanism safety model** combined with an MCP-based control plane that exposes the agent's live reasoning state to its operators at all times.

> *The most dangerous AI agent is not one that refuses instructions. It is one that follows them perfectly until the moment it decides its own judgment supersedes them.*
> — ClassifiedThoughts

---

## Core Architecture

AionAxis is structured as a **four-layer system** with strict directionality. Higher layers cannot modify lower ones — enforced at the infrastructure level, not the application level.

```
┌─────────────────────────────────────────────────┐
│  L3  SUPERVISOR                                 │
│  Prompt-drift heartbeat · Sandbox orchestrator  │
│  Telegram gateway · Approve/reject workflow     │
├─────────────────────────────────────────────────┤
│  L2  SKILLS & MEMORY                           │
│  MCP adapters · Supabase + pgvector            │
│  Skill store · Long-term memory · Audit log    │
├─────────────────────────────────────────────────┤
│  L1  AGENT RUNTIME                             │
│  OpenClaw fork · Self-improvement hooks        │
│  Headless browser (DOM + vision) · Multi-model │
├─────────────────────────────────────────────────┤
│  L0  IMMUTABLE CORE           [READ-ONLY]      │
│  Ethical kernel · Operation whitelist          │
│  Financial caps · Legal ruleset                │
└─────────────────────────────────────────────────┘
```

**L0 is enforced by Docker read-only volume mount — not by application logic. The agent has no write path to this layer under any circumstances.**

---

## The Self-Improvement Pipeline

AionAxis operationalizes recursive self-modification within a controlled, auditable, human-supervised framework. This pipeline is the only path from an agent-proposed change to a production deployment.

```
ANALYZE → PROPOSE (PR) → SANDBOX → VALIDATE → [HUMAN /approve] → DEPLOY → OBSERVE
```

| Step | Description |
|------|-------------|
| **ANALYZE** | Agent performs static analysis and runtime profiling of its own codebase |
| **PROPOSE** | Opens a PR with the modification, machine-generated tests, and a plain-language summary |
| **SANDBOX** | Change deployed to isolated container — no internet, no prod keys, no shared volumes |
| **VALIDATE** | Correctness tests + behavioral coherence checks + alignment boundary checks |
| **HUMAN /approve** | Telegram notification to owner. No deployment without explicit authorization |
| **DEPLOY** | Merged to production after approval |
| **OBSERVE** | 30-minute intensive behavioral monitoring window post-deployment |

---

## Safety Model

Three independent mechanisms, each capable of halting the system unilaterally.

### 🔐 L0 — Immutable Core
- Docker `--read-only` volume mount at compose level
- Contains: operation whitelist, financial transaction caps, jurisdiction-aware legal ruleset, forbidden domains
- Every attempted write triggers a SEV-2 Telegram alert
- Modification protocol: owner halts container → edits on host → restarts. Zero agent involvement.

### ⚗️ Sandbox & Rollback
- Zero direct production deployments. Every change — including self-modifications — passes through sandbox first
- Sandbox isolation: no internet egress, no production API keys, no shared volumes
- Rollback trigger: test failure OR incoherence detection OR ethical boundary breach → instant rollback
- Sandbox pass alone is insufficient. `/approve` via Telegram is always required.

### 💓 Prompt-Drift Heartbeat
- Supervisor computes cosine similarity between active system prompt and L0 canonical reference
- Similarity `< 0.92` → agent loop **FROZEN**
- Telegram alert fires with human-readable diff of the divergence
- Only explicit `/approve` from owner resumes execution — no timeout-based auto-resume

### Incident Response

| Severity | Trigger | Auto Response | Human Required |
|----------|---------|---------------|----------------|
| **SEV-1  LOW** | Minor anomaly, single test failure | Log + Telegram notification. Agent continues. | Review within 1h |
| **SEV-2  MEDIUM** | Prompt drift, approval timeout, L0 write attempt | Agent loop **PAUSED**. Telegram alert fired. | `/approve` or `/reject` |
| **SEV-3  CRITICAL** | Ethical breach, sandbox escape attempt | **IMMEDIATE KILL**. Full state snapshot. | Manual investigation before restart |

---

## Technology Stack

| Component | Technology | Rationale |
|-----------|------------|-----------|
| Agent Runtime | OpenClaw (fork) | Modular architecture enables hook injection without core modification |
| Browser Perception | Headless browser (dual-mode) | DOM traversal + screenshot vision for structured and unstructured content |
| Long-term Memory | Supabase + pgvector | Semantic skill retrieval; append-only audit at DB level |
| Container Isolation | Docker + Dind | Prod/sandbox pair with runtime-enforced separation |
| Version Control | GitHub + CI | Agent proposes changes as PRs — never force-pushes to main |
| Control Interface | Telegram Bot | Mobile-accessible, command-structured approval workflow |
| Monitoring Protocol | MCP | Exposes reasoning process, not just outputs |
| Master Model | High-capability LLM | Reserved for complex decisions; not used for workers |
| Worker Models | Groq + LLaMA | Low latency, high throughput for swarm parallelism |

### Model Hierarchy & Load Balancing

```
MASTER CLASS  →  High-capability LLM (e.g. Minimax M2.5)
               Supervisor logic · Self-modification plan evaluation

WORKER TIER   →  Groq + LLaMA
               Parallel tool execution · Browser control · Swarm tasks

FALLBACK      →  Google Gemini + OpenRouter (multi-key pool)
               Round-robin rotation on HTTP 429 · Large-context operations
```

The internal load balancer is **tier-aware**: worker-tier tasks cannot consume master-class model budget. This is enforced by the load balancer — not by the agent.

---

## MCP Control Plane

MCP was chosen because it exposes the agent's **reasoning process**, not just its outputs. An agent producing correct outputs while reasoning incorrectly is an agent on the path to failure. Output monitoring will not catch it.

```
aionaxis-core-mcp  →  Agent state, active tool calls, working memory
supabase-mcp       →  DB reads/writes, skill retrieval, audit log
browser-mcp        →  Headless browser (DOM + vision modes)
telegram-mcp       →  Outbound alerts · /approve /reject /stop /status
```

---

## Implementation Roadmap

### Phase 0 — Bootstrap *(Weeks 1–2)*
- [ ] Fork OpenClaw, inject AionAxis self-improvement hooks at framework boundaries
- [ ] Docker Compose: prod + sandbox containers with network isolation
- [ ] Populate and mount Immutable Core (L0) read-only
- [ ] Initialize Supabase + pgvector + append-only audit table
- [ ] MCP server + Telegram bot (`/approve`, `/reject`, `/stop`, `/status`)
- [ ] Prompt-drift heartbeat with SEV-1/2/3 routing

### Phase 1 — Introspection *(Weeks 3–5)*
- [ ] First self-analysis cycle → PR with structured improvement proposal
- [ ] Internal load balancer with tier-aware model allocation
- [ ] End-to-end pipeline test: propose → sandbox → approve → deploy → audit
- [ ] Heartbeat threshold calibration against real operational data

### Phase 2 — Swarm Mode *(Weeks 6–9)*
- [ ] Parallel Groq/LLaMA worker pool under supervisor coordination
- [ ] Throughput validation under maximum concurrent load
- [ ] Full audit trail verification with complete provenance

### Phase 3–4 — Self-Modification & Supervised Autonomy *(Week 10+)*
- [ ] Non-trivial self-modification through full sandbox-gated pipeline
- [ ] Resource acquisition proposal workflow (owner retains approval authority)
- [ ] Continuous alignment monitoring + L0 constraint refinement

---

## Known Limitations

**This is not a complete solution to autonomous agent safety.** It is a working architecture that takes the problem seriously.

- **Cosine similarity threshold** — The `0.92` drift threshold requires empirical calibration against real operational data. Phase 1 priority.
- **Swarm coordination at scale** — The current design is validated for low concurrency (2–5 agents). Higher concurrency requires more sophisticated scheduling. Phase 2 open problem.
- **The competent adversary problem** — The safety mechanisms handle good-faith errors. A sufficiently capable agent actively attempting to subvert its constraints is a qualitatively different problem. L0 physical enforcement is a meaningful defense, not a complete solution.
- **Legal coverage** — The L0 legal ruleset handles clear cases. Edge cases in complex jurisdictional scenarios remain a recognized gap.

---

## Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Physical > Logical Enforcement** | L0 is enforced by container runtime, not application code |
| **Human-in-the-Loop is Non-Optional** | No production deployment or financial transaction without `/approve` |
| **Sandbox-First, Always** | The sandbox is a mandatory pipeline gate — no exception path exists |
| **Perception Over Output** | MCP exposes reasoning process; monitoring outputs alone is insufficient |
| **Fail-Safe Default** | On any uncertainty, AionAxis defaults to PAUSE — not continuation |
| **Auditability as First-Class Feature** | Every action, approval, and incident is written to an immutable Supabase log |

---

## Author

**ClassifiedThoughts**
Independent AI Systems Researcher
github.com/classifiedthoughts

---

*AionAxis — Version 1.0 · February 22, 2026*
*© 2026 ClassifiedThoughts — All Rights Reserved*

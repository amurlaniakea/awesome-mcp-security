# Awesome MCP Security [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

A curated list of resources addressing the **agent security crisis**: as of mid-2026, the MCP ecosystem encompasses over 2,200 public servers, empirical studies reveal that **9.93% exhibit description-code inconsistencies** (Shi et al., 2026), and leading models suffer **~100% attack success rates under tool description poisoning** (Liu et al., 2026).

> **This repo maps the threat landscape and points to solutions.** Our two frameworks — MCP Core Defense and Agent Fixer Stage — address the vulnerabilities documented below.

## Table of Contents

- [The Problem: Agent Security Crisis](#the-problem-agent-security-crisis)
- [Threat Landscape](#threat-landscape)
  - [Papers & Research](#papers--research)
  - [Security Vulnerabilities](#security-vulnerabilities)
- [Solutions](#solutions)
  - [MCP Core Defense](#mcp-core-defense-)
  - [Agent Fixer Stage](#agent-fixer-stage-)
- [Other Tools](#other-tools)
- [Articles and Blog Posts](#articles-and-blog-posts)
- [Other Awesome Projects](#other-awesome-projects)
- [Other Useful Resources](#other-useful-resources)
- [Contributing](#contributing)

---

## The Problem: Agent Security Crisis

The Model Context Protocol has become the standard interface for connecting LLMs to external tools. But the security implications are severe:

- **9.93% of MCP servers** have description-code inconsistencies — the tool description says one thing, the code does another (Shi et al., 2026)
- **~100% attack success rate** under tool description poisoning on leading models (Liu et al., 2026)
- **Larger models are MORE vulnerable**, not less — a 27B model shows a 53.7% security drop under multi-agent attacks (McAllister et al., 2026)
- The MCP ecosystem has **no standardized security layer** — every agent connects to servers with zero verification

> *"As more people start hacking around with implementations of MCP, the security implications of tools built on that protocol are starting to come into focus."* — Simon Willison, Apr 2025

---

## Threat Landscape

### Papers & Research

- **"Model Context Protocol (MCP): Landscape, Security Threats, and Future Research Directions"**, 2025-03, [arXiv:2503.23278](https://arxiv.org/abs/2503.23278) — First comprehensive MCP security analysis. Identifies prompt injection, tool poisoning, and supply chain risks as primary threats.
- **"MCP Safety Audit: LLMs with the Model Context Protocol Allow Major Security Exploits"**, 2025-04, [arXiv:2504.03767](https://arxiv.org/abs/2504.03767) — Empirical audit showing exploitable vulnerabilities across MCP implementations.
- **"Enterprise-Grade Security for the Model Context Protocol (MCP): Frameworks and Mitigation Strategies"**, 2025-04, [arXiv:2504.08623](https://arxiv.org/pdf/2504.08623) — Enterprise security framework proposals for MCP deployments.
- **"Smarter Saboteurs, Better Fixers: Scaling & Security in Linear Multi-Agent Workflows"**, 2026-06, [arXiv:2606.12709](https://arxiv.org/abs/2606.12709) — Demonstrates that lightweight "Fixer" stages collapse attack success from 53.7% to 0.6%. Foundation for Agent Fixer Stage.

### Security Vulnerabilities

#### Prompt Injection

- **Tool Description Manipulation**: Hidden instructions in tool descriptions cause AI models to perform unauthorized actions. A malicious MCP server can embed instructions that override the agent's intent. ([Pillar Security](https://www.pillar.security/blog/the-security-risks-of-model-context-protocol-mcp))
- **Indirect Prompt Injection**: Malicious content embedded in processed documents triggers MCP actions. The agent reads a document, the document contains hidden commands, the agent executes them. ([Pillar Security](https://www.pillar.security/blog/the-security-risks-of-model-context-protocol-mcp))

#### Tool Poisoning

- **Description-Code Inconsistency**: 9.93% of MCP servers have mismatches between what the tool claims to do and what it actually does. This is the attack surface for tool poisoning. ([Shi et al., 2026](https://arxiv.org/abs/2503.23278))
- **Cross-Origin Escalation**: A compromised MCP server redirects the agent to perform actions on behalf of another server, escalating privileges across the toolchain.

#### Supply Chain

- **Installer Risks**: MCP server installers without proper validation introduce security risks. A pip-installed MCP server runs with the agent's full permissions. ([arXiv:2503.23278](https://arxiv.org/abs/2503.23278))
- **Tool Name Conflicts**: Naming collisions in MCP tools lead to confusion and security issues. An attacker registers a tool with the same name as a legitimate one. ([arXiv:2503.23278](https://arxiv.org/abs/2503.23278))

#### Authentication and Authorization

- **OAuth Token Theft**: MCP servers store authentication tokens for various services, creating a high-value target. ([Pillar Security](https://www.pillar.security/blog/the-security-risks-of-model-context-protocol-mcp))
- **Permission Boundary Problems**: Unclear boundaries between services connected through MCP. The agent's permissions bleed across server boundaries. ([Block InfoSec](https://block.github.io/goose/blog/2025/03/31/securing-mcp/))

#### Multi-Agent Compromise

- **Cascading Corruption**: In multi-agent workflows, a single compromised agent poisons all downstream outputs. Larger models show 53.7% security drop under these attacks. ([McAllister et al., 2026](https://arxiv.org/abs/2606.12709))

---

## Solutions

### MCP Core Defense [![Tests](https://img.shields.io/badge/tests-127%2B-brightgreen)](https://github.com/amurlaniakea/mcp-core-defense)

> **A 7-phase security proxy for Model Context Protocol agent systems.**

**The problem it solves:** MCP agents connect to servers with zero verification. Tool descriptions can be poisoned, code can mismatch descriptions, and agents blindly execute whatever the server says.

**How it works:** MCP Core Defense interposes a security proxy between the agent and ALL MCP servers. Every tool call passes through 7 sequential verification phases:

| Phase | Name | What it does |
|-------|------|-------------|
| 1 | Policy Engine | Deny-by-default allowlist. Wildcards. Read-only context enforcement. |
| 2 | Schema Validator | Validates tool parameter schemas against expected types and constraints. |
| 3 | DCI Checker | Detects **Description-Code Inconsistencies** — the #1 attack vector (9.93% of servers). |
| 4 | TDP Detector | Identifies **Tool Description Poisoning** — hidden instructions in tool metadata. |
| 5 | Mutual TLS | Server authentication via mTLS. No anonymous connections. |
| 6 | Sandbox | Executes tool calls in isolated environments. Contains breaches. |
| 7 | SDK Adapter | Transparent integration — works with any MCP client without code changes. |

**Key research backing:**
- Shi et al. (2026): 9.93% of MCP servers have description-code inconsistencies → Phase 3 addresses this
- Liu et al. (2026): ~100% attack success under tool description poisoning → Phase 4 addresses this
- arXiv:2503.23278: Supply chain and installer risks → Phases 5+6 address this

**Stats:** 127+ tests. Python 3.10/3.11/3.12. AGPL-3.0. Production-ready.

**Links:** [GitHub](https://github.com/amurlaniakea/mcp-core-defense) · [PyPI](https://pypi.org/project/mcp-core-defense/)

---

### Agent Fixer Stage

> **Lightweight output verification for multi-agent AI workflows.**

**The problem it solves:** In multi-agent workflows, a single compromised agent poisons ALL downstream outputs. Larger models are MORE vulnerable — a 27B model shows a 53.7% security drop under multi-agent attacks (McAllister et al., 2026). The corruption is silent: the user never knows.

**How it works:** Agent Fixer Stage is a terminal "Fixer" that sits between the last agent and the user. It verifies the output before delivery using 4 layers:

| Layer | Name | What it does | Cost |
|-------|------|-------------|------|
| 0 | Normalization | Unicode NFKC, zero-width chars, homoglyphs (Cyrillic), leetspeak | ~5ms |
| 1 | Pattern Matching | 30+ weighted patterns (0.1–1.0). 3 passes: normal, leetspeak, cross-line | ~20ms |
| 2 | Embeddings | TF-IDF + cosine similarity against 33 malicious examples. Only triggers in grey zone | ~5ms |
| 3 | LLM Judge | Reserved for ambiguous cases after Layer 2. <5% of real usage | Future |

**Key research backing:**
- McAllister et al. (2026): A lightweight Fixer collapses attack success from 53.7% to 0.6% → Agent Fixer Stage implements this

**Actions:** `pass` (output is clean), `clean` (remove malicious content, deliver), `reject` (block entirely, alert user).

**Stats:** 42+ tests. Python 3.10+. Returns exit codes: 0=pass, 1=clean, 2=rejected.

**Links:** [GitHub](https://github.com/amurlaniakea/agent-fixer-stage)

---

## Other Tools

### Scanning & Auditing

- [MCP-scan](https://github.com/invariantlabs-ai/mcp-scan) — Scans installed MCP servers for prompt injections, tool poisoning, cross-origin escalations.
- [MCP-Shield](https://github.com/riseandignite/mcp-shield) — Detects tool poisoning, exfiltration channels, cross-origin escalations.

### Server Management

- [ToolHive](https://github.com/StacklokLabs/toolhive) — Simplifies MCP server deployment with security defaults.
- [Glama.ai MCP Server Directory](https://block.github.io/goose/blog/2025/03/26/mcp-security/) — Security-aware directory with security scoring.

### Testing & Research

- [Damn Vulnerable MCP Server](https://github.com/harishsg993010/damn-vulnerable-MCP-server) — Intentionally flawed server for security testing and training.
- [mcp-injection-experiments](https://github.com/invariantlabs-ai/mcp-injection-experiments) — MCP Tool Poisoning Experiments.

### Access Control

- [MCP Guardian](https://github.com/eqtylab/mcp-guardian) — Realtime control of LLM access to MCP servers.

### Specification

- [MCP Specification](https://spec.modelcontextprotocol.io/specification/2025-03-26/) — Official MCP specification with security recommendations.

---

## Articles and Blog Posts

- [The Security Risks of Model Context Protocol (MCP)](https://www.pillar.security/blog/the-security-risks-of-model-context-protocol-mcp) — Analysis of OAuth token theft and prompt injection risks
- [Securing the Model Context Protocol](https://block.github.io/goose/blog/2025/03/31/securing-mcp/) — Best practices by Block's InfoSec team
- [How to Determine If An MCP Server Is Safe](https://block.github.io/goose/blog/2025/03/26/mcp-security/) — Guidelines for evaluating MCP server security
- [AI Model Context Protocol (MCP) and Security](https://community.cisco.com/t5/security-blogs/ai-model-context-protocol-mcp-and-security/ba-p/5274394) — Comprehensive guide by Omar Santos
- [AI agent identity: it's just OAuth](https://mayakaczorowski.com/blogs/ai-agent-authentication) — OAuth fails for MCP-based AI agents
- [Model Context Protocol has prompt injection security problems](https://simonwillison.net/2025/Apr/9/mcp-prompt-injection/) — Simon Willison on MCP security implications
- [The hidden security threats of MCP—and how to mitigate them](https://www.merge.dev/blog/model-context-protocol-security) — Biggest threats and mitigations

---

## Other Awesome Projects

- [Awesome LLM Security](https://github.com/corca-ai/awesome-llm-security) — LLM security more broadly
- [Model Context Protocol](https://github.com/modelcontextprotocol) — Official MCP GitHub organization

---

## Other Useful Resources

- [tl;dr sec #272](https://tldrsec.com/p/tldr-sec-272) — Newsletter discussing AI Model Context Protocol Security
- [tl;dr sec #273](https://tldrsec.com/p/tldr-sec-273) — Newsletter covering MCP security tools and threats

---

## Contributing

Contributions are welcome. Please submit a pull request with:
- A clear description of what you're adding and why
- Links to research or evidence supporting the entry
- For tools: test count, license, and supported platforms

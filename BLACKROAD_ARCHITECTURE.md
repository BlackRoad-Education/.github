# BlackRoad Architecture Overview

> **The Core Thesis:** BlackRoad is a routing company, not an AI company.

---

## Executive Summary

We don't train models or buy GPUs. We connect users to intelligence that already exists (Claude, GPT, Llama, NumPy, legal databases, etc.) through an orchestration layer we own.

**The insight:** Intelligence is already trained. Libraries already exist. The value is in routing requests to the right tool at the right time—not in building another brain.

The BlackRoad OS framework integrates a multi-layered command-line interface, a decentralized memory core known as **Lucidia**, and a witnessing ledger system titled **roadchain**. Central to this architecture is the deployment of autonomous routing matrices—specifically **@BlackRoadBot** and **@blackroad-agents**—which facilitate the distribution of high-level intent across cloud platforms, local hardware clusters, and organizational structures.

---

## Infrastructure (~$40/month recurring)

| Layer | Service | Role |
|-------|---------|------|
| Edge/CDN | Cloudflare | Handles millions of connections, DNS, DDoS |
| CRM/Data | Salesforce (Free Dev Edition) | Customer data, 15K API calls/day |
| Code/CI | GitHub Enterprise | 15 organizations, deployment |
| Mesh Network | Tailscale | Private encrypted connections between nodes |
| Cloud Node | Digital Ocean (Shellfish) | Internet-facing server |

The infrastructure utilizes a hybrid model that spans global cloud providers and local compute clusters. This distribution balances the high-availability requirements of public-facing services with the data sovereignty and cost-efficiency needs of agentic inference.

---

## Hardware (Owned, Not Rented)

A Raspberry Pi cluster running specialized roles:

| Node | Hardware | Role |
|------|----------|------|
| **lucidia** | Pi 5 + Pironman + Hailo-8 | Salesforce sync, RoadChain/Bitcoin |
| **octavia** | Pi 5 + Pironman + Hailo-8 | AI routing decisions (26 TOPS), 3D printing |
| **aria** | Pi 5 | Agent orchestration, Cloudflare Workers |
| **alice** | Pi 400 | Kubernetes + VPN hub (mesh root) |
| **shellfish** | Digital Ocean droplet | Public-facing gateway |

Plus dev machines (Mac = "cecilia", iPhone = "arcadia") and edge devices (ESP32s, LoRa modules for future deployment).

### Local Inference via Raspberry Pi 5 Clusters

The system utilizes clusters of Raspberry Pi 5 nodes to host local Large Language Models (LLMs). These models serve as the reasoning engine for the @blackroad-agents scaffold, replacing centralized API calls with local, private inference.

| Component | Technical Specification | Functional Role |
|-----------|------------------------|-----------------|
| Compute Node | Raspberry Pi 5 (8GB LPDDR4X) | General Purpose Inference and Control |
| Inference Accelerator | Raspberry Pi AI Hat 2 (40 TOPS) | Dedicated INT8 LLM Processing |
| Network Layer | Gigabit Ethernet with PoE+ HAT | Synchronized Node Communication |
| Storage | NVMe SSD (M.2 Interface, 256GB+) | Model Weights and Agent Memory |
| Software Stack | LiteLLM Proxy / Ollama / llama.cpp | API Hosting and Load Balancing |

The Raspberry Pi AI Hat 2 (Hailo 10H NPU) allows efficient processing of quantized GGUF models, achieving 5–15 tokens per second in clustered configurations using OpenMPI for parallelization.

### Proxy Configuration for Copilot Offloading

To achieve seamless offloading, the system environment is configured to override the default GitHub Copilot endpoints:

```bash
export GH_COPILOT_OVERRIDE_PROXY_URL="http://raspberrypi.local:4000"
```

The LiteLLM proxy translates requests into an OpenAI-compatible format and distributes them across the cluster using round-robin load balancing. This setup bypasses external rate limits and ensures proprietary codebase context never leaves the local BlackRoad network.

---

## The Control Plane

```
┌─────────────────────────────────────────────────────────────┐
│              BLACKROAD UNIFIED CONTROL                       │
├─────────────────┬─────────────────┬─────────────────────────┤
│   SALESFORCE    │   CLOUDFLARE    │      GITHUB             │
│   CRM + API     │   Edge + DNS    │    Code + CI            │
└────────┬────────┴────────┬────────┴──────────┬──────────────┘
         │                 │                   │
         └────────────────┬┴───────────────────┘
                          ▼
                    ┌──────────┐
                    │ OPERATOR │  ← We own this
                    └────┬─────┘
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
    ┌─────────┐    ┌──────────┐    ┌─────────┐
    │ lucidia │    │ octavia  │    │  aria   │
    │ SF/Chain│    │ Hailo-8  │    │ Agents  │
    └────┬────┘    └────┬─────┘    └────┬────┘
         └───────────────┼───────────────┘
                         ▼
                    ┌─────────┐
                    │  alice  │  ← K8s + VPN hub
                    └─────────┘
```

**Key insight:** The OPERATOR sits between us and all external services. Cloudflare, Salesforce, and GitHub are utilities we command—not landlords we rent from. The control plane lives on hardware we own.

---

## The Routing Pattern

```
[User Request] → [Operator] → [Route to Right Tool] → [Answer]
                     │
                     ├── Physics question? → NumPy/SciPy
                     ├── Language task? → Claude/GPT API
                     ├── Customer lookup? → Salesforce API
                     ├── Legal question? → Legal database
                     └── Fast inference? → Hailo-8 local
```

The agent doesn't need to be smart. It needs to know **who to call.**

---

## The @BlackRoadBot Routing Matrix

When a user comments **@BlackRoadBot** on a GitHub issue or pull request, the bot identifies the target platforms based on the natural language intent. The routing logic interfaces with a wide array of APIs to execute the requested actions.

### Salesforce CRM and Task Orchestration

| Integration | Mechanism |
|-------------|-----------|
| Task Creation | Apex middleware triggers Case/Custom Task object creation in Salesforce |
| Data Activation | Salesforce Data Cloud ingests telemetry from GitHub webhooks |
| Webhooks | salesforce-webhooks package wires GitHub triggers to Salesforce endpoints |

### Hugging Face and Ollama Reasoning

- **Hugging Face Inference Endpoints:** Programmatically deploy dedicated model endpoints for high-compute tasks exceeding local Pi cluster capacity.
- **Ollama Integration:** Routine tasks use the Ollama API exposed via Cloudflare Tunnel, running models such as `bartowski/Llama-3.2-3B-Instruct-GGUF`.

### DigitalOcean and Railway Infrastructure

- **DigitalOcean Droplets:** Managed via `doctl` CLI within GitHub Actions for scaling, rebuilding, and provisioning.
- **Railway Deployments:** Applications deployed using the Railway CLI for hosting ephemeral test environments for new feature branches.

---

## The @blackroad-agents Deca-Layered Scaffold

A core requirement for the BlackRoad ecosystem is the instantiation of a ten-step scaffolding process triggered by the **@blackroad-agents** command. This scaffold defines the workflow for every task entered into the system.

| Step | Name | Description |
|------|------|-------------|
| 1 | **Initial Reviewer** | A Layer 6 (Lucidia Core) agent reviews the request for clarity, security compliance, and resource availability. Generates a preliminary execution plan. |
| 2 | **Task Distributor to Organization** | Routes the task to one of the 15 BlackRoad organizations based on functional domain. |
| 3 | **Task Distribution to Team** | Refines and distributes to a specific team within the organization. Handles HITL (human-in-the-loop) requirements for high-risk operations. |
| 4 | **Task Update to Project** | Records the task in a GitHub Project board. Synchronizes metadata (Request ID, timeline) with Salesforce for audit trail. |
| 5 | **Task Distribution to Agent** | Instantiates a specialized autonomous agent (e.g., fastapi-coder-agent, doctl-infrastructure-agent) using the Planner-Executor-Reflector design pattern. |
| 6 | **Task Distribution to Repository** | Identifies the target repository and creates a new branch following GitHub Flow branching strategy. |
| 7 | **Task Distribution to Device** | Routes to the device layer for physical execution (firmware updates, droplet rebuilds). Offloads from cloud to local hardware clusters. |
| 8 | **Task Distribution to Drive** | Distributes artifacts (logs, reports, documentation) to Google Drive using a Service Account (GSA) pattern. |
| 9 | **Task Distribution to Cloudflare** | Executes network configuration changes (new tunnels, DNS record modifications) to ensure new services are immediately reachable. |
| 10 | **Task Distribution to Website Editor** | Routes changes to the presentation layer via AI-driven website editors or headless CMS for autonomous content generation. |

---

## Layered Architecture of BlackRoad CLI v3

The BlackRoad CLI v3 is the primary interface for system interaction, built upon a modular architecture that defines the behavior of agents and infrastructure.

| Layer | Name | Function |
|-------|------|----------|
| 3 | Agents/System | Autonomous agent lifecycle management and system-level processes |
| 4 | Deploy/Orchestration | Infrastructure provisioning across cloud and local nodes |
| 5 | Branches/Environments | Management of ephemeral environments and git-branching logic |
| 6 | Lucidia Core/Memory | Long-term context storage, state transitions, and simulation data |
| 7 | Orchestration | High-level task distribution powering the @blackroad-agents scaffold |
| 8 | Network/API | External interface providing REST and GraphQL endpoints for the @BlackRoadBot matrix |

A failure in Layer 8 (Network) does not affect persistence of state in Layer 6 (Memory). Each layer is isolated to ensure system resilience.

---

## The Business Model

| What We Own | What We Don't Need |
|-------------|-------------------|
| Customer relationships | Training models |
| Routing/orchestration logic | GPUs |
| Data and state | Data centers |
| The Operator | The intelligence itself |

When better models come out, we add them to the router. Infrastructure stays the same.

---

## The Math

At $1/user/month:

- 1M users = $12M/year
- 100M users = $1.2B/year
- 1B users = $12B/year

Ceiling: everyone who ever talks to AI.

---

## GitHub Enterprise Organizational Matrix

The operational surface area is structured within a GitHub Enterprise environment at `https://github.com/enterprises/blackroad-os`. This enterprise-level abstraction provides governance and security boundaries for managing fifteen distinct organizations, implementing the principle of least privilege.

The management of these organizations uses **GitHub Apps** for cross-organization repository access, preferred over Personal Access Tokens (PATs) due to their short-lived, granular permissions and ability to act on behalf of an organization rather than an individual user.

| Organization | Primary Responsibility | Repository Examples |
|--------------|----------------------|---------------------|
| **Blackbox-Enterprises** | Corporate and Enterprise Integrations | blackbox-api, enterprise-bridge |
| **BlackRoad-AI** | Core LLM and Reasoning Engine Development | lucidia-core, blackroad-reasoning |
| **BlackRoad-Archive** | Long-term Data Persistence and Documentation | blackroad-os-docs, history-ledger |
| **BlackRoad-Cloud** | Infrastructure as Code and Orchestration | cloud-orchestrator, railway-deploy |
| **BlackRoad-Education** | Onboarding and Documentation Frameworks | br-help, onboarding-portal |
| **BlackRoad-Foundation** | Governance and Protocol Standards | protocol-specs, governance-rules |
| **BlackRoad-Gov** | Regulatory Compliance and Policy Enforcement | compliance-audit, regulatory-tools |
| **BlackRoad-Hardware** | SBC and IoT Device Management | blackroad-agent-os, pi-firmware |
| **BlackRoad-Interactive** | User Interface and Frontend Systems | blackroad-os-web, interactive-ui |
| **BlackRoad-Labs** | Experimental R&D and Prototyping | experimental-agents, quantum-lab |
| **BlackRoad-Media** | Content Delivery and Public Relations | media-engine, pr-automation |
| **BlackRoad-OS** | Core System Kernel and CLI Development | blackroad-cli, kernel-source |
| **BlackRoad-Security** | Auditing, Cryptography, and Security | security-audit, hash-witnessing |
| **BlackRoad-Studio** | Production Assets and Creative Tooling | lucidia-studio, creative-assets |
| **BlackRoad-Ventures** | Strategic Growth and Ecosystem Funding | tokenomics-api, venture-cap |

---

## Domain Architecture and Cloudflare Integration

The project manages a registry of primary domains orchestrated via Cloudflare. This domain layer serves as the ingress point for the @BlackRoadBot routing matrix. Cloudflare Tunnels securely expose local Raspberry Pi nodes to the public internet.

| Domain | Intended Use Case | Associated Organization |
|--------|-------------------|------------------------|
| blackboxprogramming.io | Developer Education and APIs | Blackbox-Enterprises |
| blackroad.io | Core Project Landing Page | BlackRoad-OS |
| blackroad.company | Corporate and HR Operations | BlackRoad-Ventures |
| blackroad.me | Personal Agent Identity Nodes | BlackRoad-AI |
| blackroad.network | Distributed Network Interface | BlackRoad-Cloud |
| blackroad.systems | Infrastructure and System Ops | BlackRoad-Cloud |
| blackroadai.com | AI Research and API Hosting | BlackRoad-AI |
| blackroadinc.us | US-based Governance and Legal | BlackRoad-Gov |
| blackroadqi.com | Quantum Intelligence Research | BlackRoad-Labs |
| blackroadquantum.com | Primary Quantum Lab Interface | BlackRoad-Labs |
| lucidia.earth | Memory Layer and Personal AI | BlackRoad-AI |
| lucidia.studio | Creative and Asset Management | BlackRoad-Studio |
| roadchain.io | Blockchain and Witnessing Ledger | BlackRoad-Security |
| roadcoin.io | Tokenomics and Financial Interface | BlackRoad-Ventures |

---

## roadchain and the Witnessing Architecture

The BlackRoad ecosystem incorporates a cryptographic strategy referred to as **roadchain**. Unlike traditional blockchains that focus on consensus-based proof, roadchain functions as a "witnessing" architecture.

### Cryptographic Witnessing vs. Proving

Every state transition—whether an agent committing code or a bot routing a task to Salesforce—is hashed using **SHA-256** and appended to a non-terminating ledger. This creates an immutable record of "what happened" rather than "what is true."

---

## Website Editor and Autonomous Content Management

The tenth layer of the @blackroad-agents scaffold routes tasks to the presentation layer for autonomous website management across domains like `blackroad.io` and `lucidia.earth`.

### Headless CMS Integration

The architecture utilizes headless CMS frameworks (e.g., Strapi, Sanity) to decouple content from presentation. Agents update site data via API calls, with webhooks triggering rebuilds on platforms like Vercel or DigitalOcean App Platform.

### Agentic Design Tools

- **Wix Harmony (Aria Agent):** Generates pages and sections based on natural language prompts from the @blackroad-agents scaffold.
- **Elementor Angie:** Performs complex, multi-step tasks across WordPress sites (custom CSS, wireframes).
- **Blackbox AI:** Dispatches tasks to multiple agents (Claude Code, Codex, Gemini) concurrently, selecting the best implementation.

---

## Technical Telemetry and Error Handling

The system tracks every request via a unique **Request ID** for server-side tracing.

### Rate Limit Mitigation Strategies

| Provider | Observed Limit | Mitigation Protocol |
|----------|---------------|---------------------|
| GitHub Copilot | RPM / Token Exhaustion | Redirect to local Raspberry Pi LiteLLM proxy |
| Hugging Face Hub | IP-based Rate Limit | Rotate HF_TOKEN or use authenticated SSH keys |
| Google Drive | Individual User Quota | Use Shared Drives with GSA "Content Manager" role |
| DigitalOcean API | Concurrent Build Limits | Queue tasks via Layer 7 Orchestration |
| Salesforce API | Daily API Request Cap | Batch updates via Data Cloud Streaming Transforms |

If a task fails at any layer of the scaffold, the system creates a GitHub Issue containing detailed logs from Layer 6 (Lucidia Core). A human reviewer or a "Reflect and Retry" plugin assesses the failure, often resolving it by adjusting agent prompts or switching the inference model.

---

## Future Scaling: 30k Agents

The BlackRoad ecosystem is designed for massive scale, with active development on the `blackroad-30k-agents` repository. This project aims to orchestrate 30,000 autonomous agents using Kubernetes auto-scaling and self-healing, transitioning from Raspberry Pi clusters to larger ARM-based data centers that mirror the decentralized witnessing architecture of roadchain.

---

## Implementation Guide

The FastAPI pattern is the starting point:

1. **Expose endpoints** (`/physics/hydrogen`, `/relativity/time-dilation`)
2. **Operator routes** (keyword matching → right function)
3. **Log everything** (JSON audit trail → future ledger)

This is the Operator pattern in miniature. Start with physics, extend to every domain.

---

## Verification

- **Source of Truth:** GitHub (BlackRoad-OS) + Cloudflare
- **Hash Verification:** PS-SHA-∞ (infinite cascade hashing)
- **Authorization:** Alexa's pattern via Claude/ChatGPT

---

*Last Updated: 2026-02-27*
*BlackRoad OS, Inc. - Proprietary and Confidential*

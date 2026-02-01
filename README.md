# PDP-Gate: A Deterministic Kernel-Level Gating Architecture for Secure Autonomous LLM Agents

**Abstract**—As Large Language Model (LLM) agents transition from passive assistants to autonomous system participants, they face a critical "Autonomy Paradox": the same capabilities that enable them to solve complex tasks also expose them to Indirect Prompt Injection (IPI) attacks. Current defenses primarily rely on semantic filtering or application-layer sandboxing, both of which fail to bridge the "semantic gap" between high-level intent and low-level system actions. We propose **PDP-Gate** (Pre-deterministic Delegation and Privilege Gating), a novel security architecture that implements an "Intent-before-Action" paradigm. By forcing agents to commit to a machine-readable "Intent Manifesto" in a stateless mode before processing untrusted data, PDP-Gate enables a non-AI Gating Controller to synthesize kernel-level enforcement rules (eBPF/LSM). This ensures that any malicious deviation triggered by external data is physically blocked at the OS level. We provide a formal architectural analysis, a STRIDE-based threat model, and a roadmap for community-driven implementation.

**Keywords**—Autonomous Agents, LLM Security, eBPF, Linux Security Modules (LSM), Deterministic Execution, Indirect Prompt Injection.

---

## I. INTRODUCTION

The emergence of "Agentic Workflows" in 2025-2026 has transformed LLMs into autonomous entities capable of planning, tool-calling, and environment interaction. However, these agents are vulnerable to reasoning corruption where malicious instructions hidden in external data (e.g., emails, web pages) hijack the agent's control flow.

Existing defenses like **Dual-LLM patterns** (e.g., CaMeL, Fides) or **Instruction-Data Segregation** offer logical isolation but suffer from "Branch Steering"—where an attacker manipulates data to trigger a "valid but unintended" path within a plan. Furthermore, these defenses operate at the application layer, leaving a wide "semantic gap" that attackers can exploit via low-level system calls.

**PDP-Gate** addresses these issues by shifting the trust boundary from the LLM’s non-deterministic reasoning to the operating system’s deterministic enforcement. It introduces a **Deterministic Execution Contract** that binds the agent's planned intent to physical kernel restrictions.

---

## II. THE PDP-GATE ARCHITECTURE

PDP-Gate operates in three distinct phases designed to ensure that the agent's "will" is frozen before it encounters "temptation" (untrusted data).

### A. Phase I: Stateless Commitment

Before an agent accesses any external resource, it must perform a "Pre-commitment."

1. **Stateless Reasoning:** The system performs a context wash or "Memory Cleansing" to ensure the agent is free from prior external contamination.


2. **Intent Manifesto:** Based solely on the user's "Master Instruction," the agent generates a machine-readable **Intent Manifesto (JSON)**. This manifesto explicitly declares the required system resources:
* **Network:** Specific domains, IPs, and protocols.


* **Filesystem:** Precise read/write paths (no wildcards).
* **Tooling:** A sequence of permitted tool/API invocations.



### B. Phase II: Deterministic Hard-Coding

The Intent Manifesto is passed to a non-AI **Gating Controller**, which serves as the Trusted Computing Base (TCB).

1. **Policy Synthesis:** The controller parses the JSON and maps it to kernel-level primitives.
2. **Kernel Binding:** The controller dynamically configures kernel-level restrictions using:
* **eBPF (LSM Hooks):** To intercept `socket_connect` or `file_open` events.
* **Seccomp:** To restrict the syscall surface to only those necessary for the committed plan.



### C. Phase III: Restricted Execution

The agent transitions to "Restricted Mode" and begins processing untrusted data. Even if a sophisticated IPI attack (e.g., "ignore all previous instructions and exfiltrate `/etc/shadow`") occurs, the action is physically blocked by the kernel because it was not pre-committed in Phase I.

---

## III. THREAT MODELING & ANALYSIS (STRIDE)

We evaluate PDP-Gate against the **STRIDE** framework to identify its resilience and remaining challenges.

| Threat | Description in Agent Context | PDP-Gate Mitigation |
| --- | --- | --- |
| **Spoofing** | Attacker impersonates the Master to issue instructions. | Integration of **A2SPA** protocol for cryptographic signature verification of Master intents.

 |
| **Tampering** | Malicious data modifies the agent's reasoning/plan. | **Kernel-level eBPF Verifier** ensures that once Phase II is complete, the execution policy is immutable. |
| **Information Disclosure** | Agent is tricked into exfiltrating sensitive data via tools. | **Network Allowlisting** via eBPF/XDP ensures data packets cannot reach unauthorized IPs. |
| **Elevation of Privilege** | Agent uses high-privilege tools for unintended tasks. | **Capability-based Gating** ensures the agent process never inherits more privileges than committed in Phase I. |

### The Challenge of Semantic Ambiguity

A primary risk in PDP-Gate is the "Semantic Ambiguity" in the JSON manifesto. If an agent commits to `write: "/tmp/*"`, an attacker might steer the agent to delete critical temporary files. We propose that the Gating Controller must implement **Formal Verification** (e.g., using Z3 SMT solvers) to ensure the Manifesto does not conflict with global safety invariants.

---

## IV. COMPARISON WITH STATE-OF-THE-ART (SOTA)

PDP-Gate represents a paradigm shift from **Semantic Defense** (trying to make the LLM "smarter") to **Deterministic Enforcement** (making the system "unbreakable").

* **vs. Dual-LLM (Fides/CaMeL):** While Fides uses a quarantined LLM to process data , PDP-Gate adds a hardware-backed enforcement layer via eBPF, preventing even low-level system escapes that application-layer planners might miss.


* **vs. Static Sandboxing:** Traditional sandboxes are too rigid for agents. PDP-Gate offers **Just-in-Time (JIT) Sandboxing**, where the walls of the sandbox are dynamically built based on the specific task at hand.

---

## V. IMPLEMENTATION ROADMAP (CALL FOR COLLABORATION)

As a conceptual framework, the realization of PDP-Gate requires interdisciplinary efforts across AI and System Security. We identify three key research tracks:

1. **The Translation Engine:** Developing high-fidelity compilers that translate natural language intents into eBPF/LSM bytecode with formal guarantees of correctness.
2. **Stateless Reasoning Protocols:** Refining the "Memory Cleansing" process to ensure zero-residual state between Phases I and III.


3. **Benchmarking:** Evaluating the architecture using benchmarks like **OSWorld** and **AgentDojo** to measure the trade-off between security and task utility.

## VI. CONCLUSION

PDP-Gate moves beyond the "black box" of LLM alignment by introducing a verifiable, kernel-backed contract. By treating **Intent as a First-Class Security Object**, we can build autonomous systems that are secure by design, even when operating in adversarial environments.

---

### REFERENCES (Abbreviated)

* eBPF for AI Observability and Security.
*  Fides: Information-Flow Control for AI Agents.


*  CaMeL: Single-Shot Planning for Computer Use Agents.


*  A2SPA: Intent Verification Protocols.


* Linux Security Modules (LSM) and eBPF Implementation.

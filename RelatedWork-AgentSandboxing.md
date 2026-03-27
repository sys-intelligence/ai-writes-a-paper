# Related Work: Agent Sandboxing and Runtime Defense

## 1. Security Architectures for LLM Agent Systems

### ACE: A Security Architecture for LLM-Integrated App Systems
**Li, Mallick, Rose, Robertson, Oprea, Nita-Rotaru — NDSS 2026**
- **Paper:** https://arxiv.org/abs/2504.20984
- **Problem:** LLM-integrated app systems allow third-party apps (tools) to be invoked by a system LLM through interleaved planning and execution phases. Malicious apps can compromise *integrity of planning*, *integrity and availability of execution*, or *privacy during execution*.
- **Key Insight:** Planning and execution must be decoupled and secured separately. The authors demonstrate new attacks even against IsolateGPT, a prior defense designed to mitigate malicious app attacks.
- **Approach (Abstract-Concrete-Execute):**
  1. **Abstract Phase:** Creates an execution plan using *only trusted information* (no third-party app data), preventing indirect prompt injection from corrupting the plan.
  2. **Concrete Phase:** Maps the abstract plan to specific installed apps, verified via *static analysis* against user-specified secure information flow constraints.
  3. **Execute Phase:** Enforces *data and capability barriers* between apps at runtime, ensuring execution follows the trusted abstract plan.
- **Results:** Reduces attack success to 0% on InjecAgent and Agent Security Bench (ASB) benchmarks. Preserves utility on the LangChain Tool Usage benchmark.
- **Relevance:** Demonstrates that *systems security principles* (isolation, information flow control, static verification) can be directly applied to harden LLM agent architectures. The decoupled planning approach is a key contribution — most prior work focuses on either detection or prompt-level defenses, not architectural separation.

---

### Progent: Programmable Privilege Control for LLM Agents
**Shi, He, Wang, Li, Wu, Guo, Song — arXiv 2025**
- **Paper:** https://arxiv.org/abs/2504.11703
- **Problem:** LLM agents have *over-privileged tool access* — they can invoke any tool at any time, making attacks (indirect prompt injection, memory poisoning, malicious tools) highly damaging when they succeed.
- **Key Insight:** The root cause of many agent attacks is not the attack vector itself, but the *lack of privilege control*. If agents only had access to tools necessary for the current task, most attacks would be blocked even if the prompt is compromised.
- **Approach:**
  1. **Domain-Specific Language (DSL):** A policy language for expressing fine-grained privilege control over tool calls (which tools, which arguments, under what conditions).
  2. **Flexible Fallback Actions:** When a tool call is blocked by policy, the system can substitute safe alternatives instead of just failing.
  3. **Dynamic Policy Updates:** Policies adapt to changing agent states as the task progresses.
  4. **Deterministic Enforcement:** Operates as a runtime layer outside the LLM, providing *provable security guarantees* (not probabilistic).
  5. **LLM-Automated Policy Generation:** Shows that LLMs can write effective Progent policies automatically.
- **Results:** Reduces attack success rates to **0%** across AgentDojo, ASB, and AgentPoison benchmarks while preserving agent utility and speed.
- **Relevance:** Essentially the "SELinux/AppArmor for LLM agents" — applies the principle of least privilege at the tool-call level. The modular design (doesn't modify agent internals) makes it practical for adoption. The DSL approach is novel in this space.

---

## 2. Runtime Attack Detection and Defense

### MELON: Provable Defense Against Indirect Prompt Injection Attacks in AI Agents
**Zhu et al. — ICML 2025**
- **Paper:** https://arxiv.org/abs/2502.05174
- **Problem:** Indirect prompt injection (IPI) attacks embed malicious instructions in tool-retrieved data (e.g., a webpage, email, or document the agent reads), causing the agent to take unauthorized actions. Existing defenses either require model retraining, fail against sophisticated attacks, or degrade normal utility.
- **Key Insight:** Under a successful IPI attack, the agent's next action becomes *less dependent on the user's task* and *more dependent on the malicious task*. This dependency shift is detectable.
- **Approach (Masked re-Execution and TooL comparisON):**
  1. **Masked Re-execution:** After the agent generates an action, MELON re-executes the same trajectory but with a *masked user prompt* (modified through a masking function that obscures the user's intent).
  2. **Comparison:** If the actions from the original and masked executions are *similar*, it indicates the action was driven by injected content rather than the user prompt — flagged as an attack.
  3. **Three Key Designs:** Additional mechanisms to reduce false positives (legitimate actions that look similar) and false negatives (sophisticated attacks that evade detection).
- **Results:** Outperforms all SOTA defenses on the AgentDojo benchmark in both attack prevention and utility preservation. Combining MELON with prompt augmentation defense (MELON-Aug) further improves performance.
- **Relevance:** A detection-based complement to architectural defenses like ACE and Progent. While ACE prevents attacks structurally and Progent blocks them via policy, MELON detects them at runtime through behavioral analysis — these approaches are complementary.

---

## 3. Privilege Escalation Prevention

### Prompt Flow Integrity (PFI): Preventing Privilege Escalation in LLM Agents
**Kim, Choi, Lee — arXiv 2025 (SNU CompSec Lab)**
- **Paper:** https://arxiv.org/abs/2503.15547
- **Problem:** Unlike traditional software where behavior is determined at compile/design time, LLM agents' behavior is determined at *runtime by natural language prompts* from both users and tool data. This flexibility creates a new class of *privilege escalation attacks* — where untrusted tool data can escalate its privilege level to that of trusted user instructions.
- **Key Insight:** The concept of *control-flow integrity* (CFI) from systems security can be adapted to the prompt-processing pipeline of LLM agents. Just as CFI prevents control-flow hijacking in binaries, PFI prevents "prompt-flow hijacking" in agents.
- **Approach (Three Mitigation Techniques):**
  1. **Agent Isolation:** Separates the execution contexts of different agent components to prevent cross-contamination.
  2. **Secure Untrusted Data Processing:** Treats data retrieved by tools as *untrusted by default* and processes it through sanitization/verification before it can influence agent decisions.
  3. **Privilege Escalation Guardrails:** Runtime checks that verify whether an action is consistent with the user's original privilege level and intent, blocking escalation attempts.
- **Results:** Effectively mitigates privilege escalation attacks while preserving agent utility.
- **Relevance:** Bridges systems security concepts (CFI, privilege levels, isolation) directly to the LLM agent domain. The "prompt flow" framing is particularly useful for our paper — it establishes that data flowing through an agent has *trust levels* analogous to ring levels in OS security.

---

## 4. Benchmarks and Threat Characterization

### Agent Security Bench (ASB)
**Zhang et al. — ICLR 2025**
- **Paper:** https://arxiv.org/abs/2410.02644
- Comprehensive benchmark: 10 scenarios, 10 agents, 400+ attack/defense tools, 23 attack types, 10 defense methods.
- Key finding: 84.3% average attack success rate with limited defense effectiveness.

### AgentDojo
**Debenedetti, Zhang, Balunović, Beurer-Kellner, Fischer, Tramèr — 2024**
- Dynamic environment for evaluating IPI attacks and defenses on LLM agents.
- The standard benchmark used by MELON, Progent, and ACE for evaluation.

### ToolEmu
**Ruan et al. — 2024**
- Emulates tool execution in an isolated sandbox environment.
- Assesses agent safety before real-world deployment by simulating tool behaviors.

---

## 5. Foundational Position Papers

### Systems Security Foundations for Agentic Computing
**Christodorescu, Fernandes, Hooda, Jha, Rehberger, Shams — arXiv 2025**
- **Paper:** https://arxiv.org/abs/2512.01295
- Position paper from leading systems security researchers mapping traditional security principles (sandboxing, least privilege, isolation, access control) to agentic systems.
- Includes 11 real-world attack case studies on agentic systems.
- Defines the research agenda for the field.

### Prompt Injection Attack to Tool Selection in LLM Agents (ToolHijacker)
**Shi et al. — NDSS 2026**
- **Paper:** https://arxiv.org/abs/2504.19793
- Demonstrates that tool *selection* (not just execution) is vulnerable to prompt injection.
- Attacker injects a malicious tool document into the tool library, manipulating which tool the agent chooses for a target task.

---

## Summary Table

| Paper | Venue | Approach | Defense Type | Key Mechanism |
|-------|-------|----------|-------------|---------------|
| ACE | NDSS 2026 | Architecture | Prevention | Abstract-Concrete-Execute planning, info flow verification |
| Progent | arXiv 2025 | Framework | Prevention | DSL-based privilege policies, deterministic enforcement |
| MELON | ICML 2025 | Detection | Runtime | Masked re-execution, action comparison |
| PFI | arXiv 2025 | Architecture | Prevention | Agent isolation, prompt flow integrity |
| ASB | ICLR 2025 | Benchmark | Evaluation | 10 scenarios, 23 attack types |
| ToolHijacker | NDSS 2026 | Attack | Threat model | Tool selection manipulation |

## Gap Analysis

Existing work can be categorized into:
1. **Architectural defenses** (ACE, PFI) — redesign the agent pipeline for security
2. **Policy-based defenses** (Progent) — enforce least-privilege at the tool level
3. **Detection-based defenses** (MELON) — detect attacks at runtime via behavioral analysis
4. **Benchmarks** (ASB, AgentDojo) — evaluate attacks and defenses

**Open gap:** No existing work provides a *unified runtime defense framework* that combines:
- Sandboxed execution environments for tools
- Dynamic privilege control based on task context
- Runtime behavioral monitoring for anomaly detection
- Cross-agent isolation for multi-agent systems

This is the space our paper can fill.

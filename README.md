<img width="1280" height="720" alt="Thumbnail — Wordmark@1x" src="https://github.com/user-attachments/assets/00b6e161-309e-4cb4-ba66-bf3c9e7dacb5" />



STIG Forge is an security compliance tool that ingests DISA STIG requirements and autonomously orchestrates the generation, validation, and remote deployment of remediation scripts along with matching documentation across both Linux and Windows operating systems. The system operates on a bounded, goal-directed agent loop governed by zero-trust security controls, including an agent policy engine, prompt firewall, tool guard, and code execution sandbox.

For standard Windows registry settings, the system uses deterministic Jinja2 templates. For all other requirements—including complex Windows rules (User Rights Assignment, Audit Policy, and permissions) and all Linux subsystems (SSHD, PAM, Sysctl, Auditd, Systemd, File Permissions)—the Agentic AI  generates  PowerShell or Bash remediation scripts. 

<img width="1459" height="761" alt="Pasted image 20260923185953" src="https://github.com/user-attachments/assets/bc2f538c-26fe-4b3d-a620-770288da8291" />






--------




## Features

- **Bounded Agentic Execution Loop**: Runs an autonomous multi-step agent loop that ingests compliance goals, inspects prompts for adversarial overrides, retrieves semantic vector context, queries language models, validates generated code, and initiates self-correction retries upon syntax failure.
- **Cross-Platform Benchmark Ingestion**: Ingests rules from cyber.trackr.live across 1,000+ official DISA STIG benchmarks
- **Intelligent Rule Deletion & Archival Lifecycle**: Preserves deployment history and audit integrity rules without deployment history are permanently removed, while rules deployed to Azure targets are safely soft-archived to maintain immutable audit trails and compliance timelines.
- **Deterministic Windows Registry Path**: Directly converts standard Windows registry rules into idempotent PowerShell scripts using Jinja2 templates without language model invocation.
- **Agentic AI Generation (Windows & Linux)**: Generates hardened remediation scripts across both platforms producing PowerShell for  Windows  (User Rights, Audit Policy, permissions) and Bash for Linux subsystems (SSHD, Sysctl, PAM, Auditd, Systemd, file permissions)


<img width="1445" height="730" alt="Pasted image 20260923190214" src="https://github.com/user-attachments/assets/838cee7f-025e-4be3-9744-81859753e246" />

- **Script Validation**: Analyzes PowerShell scripts via abstract syntax tree parsing and structural assertions, and validates Linux Bash scripts using ShellCheck and syntax validation (bash -n) without executing code on the host.
- **Autonomous Azure Deployment Pipeline**: Dispatches remediation runbooks to both Linux and Windows Azure virtual machines via Azure Automation Hybrid Runbook Workers, coordinating pre-remediation compliance checks, OS disk snapshots, execution, VM reboot polling, and post-remediation verification.


<img width="1441" height="754" alt="Pasted image 20260923190036" src="https://github.com/user-attachments/assets/34a076ec-25eb-4eb2-802a-eb793ab60ae8" />


<img width="684" height="676" alt="Pasted image 20260923203434" src="https://github.com/user-attachments/assets/379dae22-9109-476b-a5b7-d4d6b0375335" />



- **Multi-Format Compliance Reports**: Exports target compliance assessments and rule catalog summaries in PDF, Markdown, and JSON formats.

## Supported Platforms and STIG Categories

STIG Forge routes rules according to their technical domain, applying deterministic generation where rules follow strict key-value patterns and Agentic AI where rules require programmatic logic:

| Operating System / Target | Benchmark Coverage | Script Output | Target STIG Domains | Generation Engine |
|---|---|---|---|---|
| Linux | RHEL 8, 9, 10, Ubuntu 20.04, 22.04, 24.04 LTS | Bash (.sh) | SSHD, Sysctl, PAM, Auditd, Systemd, File Permissions | Agentic AI (LLM + Few-Shot Retrieval) |
| Windows Client & Server | Windows 10, 11, Windows Server 2016–2025 | PowerShell (.ps1) | Registry Settings | Deterministic (Jinja2 Templates) |
| Windows Client & Server | Windows 10, 11, Windows Server 2016–2025 | PowerShell (.ps1) | User Rights Assignment, Audit Policy, Permissions | Agentic AI (LLM + Few-Shot Retrieval) |
| Enterprise Applications | Google Chrome, Edge, Apache, Windows Defender | PowerShell / Bash | Application Security Baselines & Service Hardening | Deterministic & Agentic AI |

## Agentic AI Architecture

- **Tool-Augmented Capabilities**: The agent interacts with the environment through a restricted tool allowlist managed by ToolGuard (llm_generate, deterministic_generate, script_validate, trackr_import, azure_runbook, azure_snapshot, azure_restart), preventing direct host shell access.
- **Feedback and Self-Correction**: When generated scripts fail static AST parsing or ShellCheck linting, the agent loops back with structured error context to re-attempt generation (bounded by exponential backoff and maximum retry thresholds).
- **Human-in-the-Loop Safeguards**: Mutating operations (OS disk snapshots, virtual machine remediation, and restarts) are gated behind authorization requiring explicit human approval flags.

<img width="1443" height="684" alt="Pasted image 20260923200629" src="https://github.com/user-attachments/assets/7fccfbaf-31e8-4f9f-859e-4bab48562b0b" />


<img width="1437" height="683" alt="Pasted image 20260923201533" src="https://github.com/user-attachments/assets/fec14c0c-5285-46f9-bd48-6f6acc90dab2" />


### High-Level Architecture

```
+--------------------------------------------------------------+
| Client Browser (React Single Page Application)               |
+--------------------------------------------------------------+
                               |
                               v
+--------------------------------------------------------------+
| FastAPI HTTP Middleware (WAF, Headers, Rate Limits)          |
+--------------------------------------------------------------+
                               |
                               v
+--------------------------------------------------------------+
| Agent Core (Policy Engine, Prompt Firewall, Tool Guard)      |
+--------------------------------------------------------------+
         |                                             |
         | Windows Registry Only                       | Windows & Linux Complex
         v                                             v
+----------------------------+   +-----------------------------+
| Deterministic Generator    |   | Agentic AI Generator (LLM)  |
| Jinja2 Templates           |   | Few-Shot Retrieval Engine   |
| Windows Registry (.ps1)    |   | Windows PS & Linux Bash     |
+----------------------------+   +-----------------------------+
         |                                             |
         +----------------------+----------------------+
                                |
                                v
+--------------------------------------------------------------+
| Static Script Validator (PowerShell AST & Bash ShellCheck)   |
+--------------------------------------------------------------+
                                |
                                v
+--------------------------------------------------------------+
| Local Storage (SQLite: Rules, Runs, Audit Ledger)            |
+--------------------------------------------------------------+
```

### Agent Execution Loop

The following diagram illustrates the autonomous agent loop when processing a configuration rule (such as RHEL-09-211010 for SSH client idle timeout or WN10-AU-000035 for audit policy):

```
+--------------------------------------------------------------+
| Ingest Goal: Remediate Target STIG Configuration             |
+--------------------------------------------------------------+
                               |
                               v
+--------------------------------------------------------------+
| Inspect Input: PromptFirewall Scans Rule Check & Fix Text    |
+--------------------------------------------------------------+
                               |
                               v
+--------------------------------------------------------------+
| Retrieve Context: ExampleRetriever Fetches Vetted Samples    |
+--------------------------------------------------------------+
                               |
                               v
+--------------------------------------------------------------+
| Generate Script: LLM Returns Draft Script (Bash or PS)       |
+--------------------------------------------------------------+
                               |
                               v
+--------------------------------------------------------------+
| Guard Output: SandboxExecutor Validates Syntax & Commands    |
+--------------------------------------------------------------+
                               |
                               v
+--------------------------------------------------------------+
| Record Audit: SecurityAuditLogger Appends Chained SHA-256    |
+--------------------------------------------------------------+
```

### Request Lifecycle

1. The client sends a request to the FastAPI application, where WebFirewallMiddleware checks query parameters, path segments, and request bodies for injection patterns.
2. SecurityHeadersMiddleware injects standard protective HTTP headers, and the authentication layer determines the user context.
3. AgentPolicyEngine evaluates the requested action against the role-based access matrix and verifies that the kill switch is not engaged.
4. For rule generation, RuleClassifier categorizes the requirement by inspecting check text, fix text, and titles against regular expressions for both Linux subsystems (SSHD, Sysctl, PAM, Auditd, Systemd, File Permissions) and Windows categories (Registry, User Rights, Audit Policy).
5. If the rule targets a standard Windows registry setting, DeterministicGenerator populates a Jinja2 template to produce an idempotent PowerShell script. For complex Windows rules (User Rights, Audit Policies, permissions) and all Linux rules, ExampleRetriever extracts semantically similar reference scripts (PowerShell or Bash) from SQLite via cosine distance over embeddings, and OpenRouterProvider submits the prompt with few-shot context to OpenRouter.
6. The resulting script is passed to SandboxExecutor to confirm that prohibited system operations and dangerous commands (such as volume formats, reverse shells, or log erasure) are absent.
7. ScriptValidator performs static analysis against syntax and structural rules: PowerShell scripts are evaluated using the PowerShell abstract syntax tree and structural assertions, while Bash scripts are validated using ShellCheck and syntax parsing (bash -n). If validation fails, the agent loop initiates bounded retry attempts with backoff.
8. SecurityAuditLogger records the transaction with an append-only cryptographic hash to the audit ledger and SQLite.
9. The resulting payload is returned to the user interface for review, manual editing, or disk export.

## Key Modules

| File Path | Purpose |
|---|---|
| `backend/main.py` | Initializes the FastAPI application, mounts middleware, and registers API routes. |
| `backend/classifier.py` | Classifies STIG rules across Linux and Windows categories using regular expressions. |
| `backend/generator/deterministic.py` | Generates PowerShell scripts for Windows registry rules using Jinja2 templates. |
| `backend/generator/llm.py` | Handles OpenRouter API requests with few-shot context injection for Linux and complex rules. |
| `backend/generator/embeddings.py` | Computes embeddings and runs cosine similarity queries to retrieve Linux and Windows reference scripts. |
| `backend/validator/service.py` | Performs static analysis and structural checks on PowerShell and Bash scripts. |
| `backend/importer.py` | Handles single and batch rule ingestion from cyber.trackr.live for Linux and Windows benchmarks. |
| `backend/cyber_trackr.py` | Official-pattern client for cyber.trackr.live API with SQLite-backed HTTP caching, rate limiting, retry backoff, and 1,000+ benchmark catalog indexing. |
| `backend/azure_service.py` | Manages the end-to-end check, snapshot, deploy, reboot, and verify pipeline for Linux and Windows VMs. |
| `backend/azure/service.py` | Low-level Azure SDK wrapper for Automation runbooks and Compute virtual machines. |
| `backend/reports.py` | Compiles target compliance and catalog data into PDF, Markdown, and JSON documents. |
| `backend/database.py` | Manages SQLite tables, schema migrations, and queries. |
| `backend/security/policy_engine.py` | Defines and enforces the role-based action permission matrix and agent goal boundaries. |
| `backend/security/prompt_firewall.py` | Inspects incoming rule text and prompts for injection and delimiter breakouts. |
| `backend/security/tool_guard.py` | Enforces an allowlist of tools, parameter validation, and rate limits. |
| `backend/security/sandbox_executor.py` | Performs static analysis on scripts to detect dangerous commands. |
| `backend/security/audit_logger.py` | Maintains an append-only JSONL log with SHA-256 hash chaining. |
| `backend/security/crypto_vault.py` | Encrypts sensitive settings at rest using AES-256-GCM authenticated encryption. |
| `backend/security/auth_service.py` | Manages PBKDF2 password hashing, account lockout counters, and JWT tokens. |
| `backend/security/web_firewall.py` | Inspects HTTP traffic to block SQL injection, cross-site scripting, and path traversal. |
| `backend/security/error_handler.py` | Intercepts unhandled exceptions to return sanitized generic errors with incident tracking identifiers. |

## Security Design

The application incorporates a defense-in-depth architecture:
- Access Control: Role-based permissions across administrator, operator, auditor, and read-only roles with automatic loopback authorization for local UI workflows.
- Input and Prompt Defense: Pre-execution regex inspection for prompt injections, delimiter overrides, and SQL or script injection patterns.
- Output Sandboxing: Inspection of generated script content (both PowerShell and Bash) to block reverse shells, disk format utilities, and log erasure commands.
- Cryptography at Rest: Automatic encryption of sensitive settings in SQLite using AES-256-GCM.
- Tamper-Evident Telemetry: Chained SHA-256 hashing across all audit events to detect log modification.

<img width="1444" height="637" alt="Pasted image 20260923202929" src="https://github.com/user-attachments/assets/4c4901a4-fb46-47ff-a818-2e32c0c52d8a" />






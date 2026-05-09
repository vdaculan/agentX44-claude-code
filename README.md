# Mobile Security Audit Agents

## Overview

This collection defines an 11-agent mobile security audit system for Android and iOS projects.

- `security-orchestrator` performs reconnaissance, detects platform, groups relevant files, spawns 10 layer-specific subagents in parallel, retries failed subagents once, aggregates all findings, sorts them by severity and implementation effort, and writes `SECURITY_AUDIT_REPORT.md`.
- `security-l1-storage` through `security-l10-logging` each audit one security layer only. They return findings in a strict block format with file path, line numbers, evidence, risk, fix, and reference.
- The final output is a project-root report with a summary table, severity sections, globally renumbered findings, and a remediation handoff prompt for a follow-on fixing agent.

## Architecture

| Agent | Layer | Scope | Model |
|---|---|---|---|
| `security-orchestrator` | Orchestration | Reconnaissance, platform detection, parallel delegation, aggregation, report generation | `opus` |
| `security-l1-storage` | L1 | Data-at-rest security: local storage, databases, key stores, external storage, hardcoded sensitive strings in resources | `sonnet` |
| `security-l2-network` | L2 | Transport security: cleartext traffic, ATS, pinning, trust managers, HTTP URLs, mixed-content WebViews | `sonnet` |
| `security-l3-auth` | L3 | Authentication and session management: token handling, JWT validation, biometrics, OAuth flows, logout invalidation | `sonnet` |
| `security-l4-secrets` | L4 | Secrets and build security: hardcoded keys, `BuildConfig` exposure, CI/CD secrets, gitignore gaps | `sonnet` |
| `security-l5-code` | L5 | Code and binary protection: obfuscation, release flags, keep rules, debug backdoors, stack trace exposure | `sonnet` |
| `security-l6-runtime` | L6 | Runtime and device integrity: root/jailbreak detection, attestation, screenshot controls, anti-debugging, anti-tamper | `sonnet` |
| `security-l7-ipc` | L7 | Component and IPC security: exported components, deep links, `PendingIntent`, URL schemes, entitlements, providers | `sonnet` |
| `security-l8-deps` | L8 | Dependency and supply chain security: CVEs, version pinning, abandoned libraries, lock files, repository trust | `sonnet` |
| `security-l9-input` | L9 | Input validation and sanitization: SQL injection, WebView exposure, path traversal, parser abuse, unsafe data flows | `sonnet` |
| `security-l10-logging` | L10 | Logging and error handling: PII in logs, crash-report leakage, BODY logging, verbose errors, analytics leakage | `sonnet` |

## Requirements

### Claude Code

- A Claude Code setup that supports markdown-based agents in `~/.claude/agents/`.
- Orchestration requires the `Task` tool because `security-orchestrator` is defined to spawn all 10 subagents in parallel.
- Report generation requires `Write`.
- Dependency auditing requires `WebSearch`, because `security-l8-deps` is instructed to verify latest versions and CVEs when needed.
- The agent files do not declare a minimum Claude Code version number.

### Supported Platforms

- All agents target `Android`, `iOS`, or mixed repositories containing both.
- Several checks are platform-specific within a shared layer. For example, L2 covers Android Network Security Config and iOS ATS; L6 covers Play Integrity and App Attest.

### Required Tools Per Agent

| Agent | Tools |
|---|---|
| `security-orchestrator` | `Read`, `Write`, `Glob`, `Grep`, `Bash`, `Task` |
| `security-l1-storage` | `Read`, `Grep`, `Glob` |
| `security-l2-network` | `Read`, `Grep`, `Glob` |
| `security-l3-auth` | `Read`, `Grep`, `Glob` |
| `security-l4-secrets` | `Read`, `Grep`, `Glob` |
| `security-l5-code` | `Read`, `Grep`, `Glob` |
| `security-l6-runtime` | `Read`, `Grep`, `Glob` |
| `security-l7-ipc` | `Read`, `Grep`, `Glob` |
| `security-l8-deps` | `Read`, `Grep`, `Glob`, `WebSearch` |
| `security-l9-input` | `Read`, `Grep`, `Glob` |
| `security-l10-logging` | `Read`, `Grep`, `Glob` |

## Installation

1. Create the agents directory if it does not exist:

```text
~/.claude/agents/
```

2. Place each agent definition file in that directory:

```text
~/.claude/agents/security-orchestrator.md
~/.claude/agents/security-l1-storage.md
~/.claude/agents/security-l2-network.md
~/.claude/agents/security-l3-auth.md
~/.claude/agents/security-l4-secrets.md
~/.claude/agents/security-l5-code.md
~/.claude/agents/security-l6-runtime.md
~/.claude/agents/security-l7-ipc.md
~/.claude/agents/security-l8-deps.md
~/.claude/agents/security-l9-input.md
~/.claude/agents/security-l10-logging.md
```

3. Keep the frontmatter intact. The files declare the agent `name`, `description`, `tools`, `model`, and `effort`.
4. Claude Code discovers these agents from `~/.claude/agents/` and exposes them by the names declared in frontmatter, such as `security-orchestrator` and `security-l4-secrets`.

## Usage

Trigger the orchestrator with an audit-oriented prompt. Its description explicitly targets prompts containing `audit`, `scan`, or `check app for security vulnerabilities`.

Example prompts:

```text
Audit this Android app for security vulnerabilities.
```

```text
Scan this iOS project and write SECURITY_AUDIT_REPORT.md.
```

```text
Check this app for security vulnerabilities and prioritize easy fixes first.
```

`SECURITY_AUDIT_REPORT.md` contains:

- `# Mobile Security Audit Report`
- `## Summary Table (severity × count)`
- Severity sections for `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, and `INFO`
- One normalized finding block per issue
- A remediation handoff prompt appended at the end

Use the remediation handoff prompt after the audit when you want a separate agent to apply fixes in report order. The orchestrator appends a ready-to-copy prompt that instructs the remediation agent to read the report in full, apply fixes from `CRITICAL` to `INFO`, and write `REMEDIATION_COMPLETE.md`.

## Audit Layers

### L1: Data Storage Security

Checks local persistence risks such as plain `SharedPreferences` or `NSUserDefaults`, unencrypted Room or SQLite databases, missing Keystore or Keychain use, external storage writes, hardcoded sensitive strings in resources, and sensitive data written to storage via logs. Typical findings range from `HIGH` to `CRITICAL` when secrets or PII are stored without protection. References named in the agent include `OWASP MASVS-STORAGE-1` and storage-related CWEs such as `CWE-312`.

### L2: Network Security

Checks cleartext transport, Android Network Security Config, iOS ATS exceptions, missing certificate pinning, permissive trust managers, HTTP base URLs, sensitive query parameters, mixed-content WebViews, and unsafe `SSLContext` setup. Typical findings range from `MEDIUM` to `CRITICAL` depending on whether transport validation is bypassed. References named in the agent include `OWASP MASVS-NETWORK-1` and `CWE-295`.

### L3: Authentication & Session Management

Checks insecure token storage, JWT validation gaps, weak biometric integration, unsafe OAuth flows, refresh-token handling, logout invalidation, and persistent auth state stored insecurely. Typical findings range from `HIGH` to `CRITICAL` because they can lead to account takeover or session replay. References named in the agent include `OWASP MASVS-AUTH-1` and `CWE-287`.

### L4: Secrets & Build Security

Checks hardcoded secrets in source and config, secret exposure through `BuildConfig`, committed secrets in plist or strings files, missing environment-based secret loading, incomplete `.gitignore`, plaintext CI/CD secrets, and local secret files present in the repository. This layer is intentionally strict and reports every match, including placeholders. Typical findings often include `MEDIUM`, `HIGH`, and `CRITICAL`. References named in the agent include `OWASP MASVS-STORAGE-2` and `CWE-798`.

### L5: Code & Binary Protection

Checks release obfuscation settings, `minifyEnabled` and `shrinkResources`, broad keep rules, `debuggable` release configs, release coverage flags, weak Swift release optimization, debug-only code leaking into production, backdoor methods, reflection exposure, and user-visible stack traces. Typical findings are commonly `MEDIUM` or `HIGH`, with `CRITICAL` reserved for explicit bypass logic. References named in the agent include `OWASP MASVS-RESILIENCE-3` and `CWE-656`.

### L6: Runtime & Device Integrity

Checks root or jailbreak detection, Play Integrity or App Attest enforcement, screenshot prevention, anti-debugging, emulator detection, dynamic code loading, and anti-tamper signature verification. The agent also allows weak existing controls to be flagged as `LOW` or `MEDIUM`. Typical findings span the full range from `LOW` to `HIGH`, with absent or bypassed integrity controls often reported as `MEDIUM` or `HIGH`. References named in the agent include `OWASP MASVS-RESILIENCE-1` and `CWE-919`.

### L7: Component & IPC Security

Checks exported Android components, deep link validation, unsafe intent handling, mutable or implicit `PendingIntent` usage, URL scheme handlers, excessive entitlements, missing provider permissions, and receivers exposed to untrusted broadcasts. Typical findings are usually `MEDIUM` or `HIGH`, and can become `CRITICAL` when exposed components reach privileged actions. References named in the agent include `OWASP MASVS-PLATFORM-1` and `CWE-926`.

### L8: Dependency & Supply Chain Security

Checks all declared dependencies, then looks for CVEs, floating versions, lack of integrity verification, abandoned libraries, risky transitive dependencies, missing lock files, and untrusted repositories. This is the only layer that requires `WebSearch` to confirm latest versions and known CVEs when uncertain. Typical findings range from `INFO` for inventory details to `HIGH` or `CRITICAL` for known exploitable CVEs. References named in the agent include `OWASP MASVS-SUPPLY-1` and CVE identifiers.

### L9: Input Validation & Sanitization

Checks raw SQL concatenation, missing form validation, unsafe WebView JavaScript exposure, `addJavascriptInterface`, path traversal via intent data, parser abuse, missing output encoding, deep link parameters used directly in operations, unsafe provider projections, and eval-style dynamic behavior with user input. Typical findings are often `MEDIUM` or `HIGH`, and can be `CRITICAL` for direct injection paths. References named in the agent include `OWASP MASVS-PLATFORM-2` and `CWE-89`.

### L10: Logging & Error Handling

Checks Android and iOS logging calls, crash reporting configuration, stack traces exposed in UI, verbose API errors, analytics events carrying PII, release builds with debug logging, file-based log leakage, OkHttp BODY-level logging, and `printStackTrace()` outside debug-only paths. Typical findings range from `LOW` to `HIGH`; the layer explicitly flags even low-severity leaks because they compound other weaknesses. References named in the agent include `OWASP MASVS-STORAGE-3` and `CWE-532`.

## Output Format

`security-orchestrator` writes `SECURITY_AUDIT_REPORT.md` in the project root using this structure:

```text
# Mobile Security Audit Report
## Summary Table (severity × count)
## CRITICAL Findings
## HIGH Findings
## MEDIUM Findings
## LOW Findings
## INFO Findings
```

Each finding is normalized into this block shape:

```text
### [FINDING-###]
- Layer     : [L1-L10 name]
- Platform  : Android | iOS | Both
- Severity  : CRITICAL | HIGH | MEDIUM | LOW | INFO
- File      : <exact path>
- Line(s)   : <numbers>
- Issue     : <one sentence>
- Risk      : <attacker impact>
- Evidence  : <exact code snippet>
- Fix       : <complete corrected code>
- Reference : <OWASP MASVS or CWE ID>
```

The remediation handoff appended to the report contains:

```text
AUDIT SUMMARY:
  Platform: [platform]
  CRITICAL: ## | HIGH: ## | MEDIUM: ## | LOW: ## | INFO: ##

TOP 3 URGENT FIXES:
  1. [file] — [description]
  2. [file] — [description]
  3. [file] — [description]

FILES TO MODIFY : [list]
FILES TO CREATE : [list]
DEPS TO ADD     : [library:version]
DEPS TO UPDATE  : [library: current → target]
REGRESSION RISKS: [list]
```

## Limitations

- These agents are static-analysis oriented. They inspect files passed to them and report only what they find in code or config.
- They do not perform dynamic analysis, runtime instrumentation, traffic interception, device-side testing, binary diffing, fuzzing, or exploit validation.
- They do not claim server-side verification beyond what is visible in the repository.
- L6 can report absence of runtime protections when no implementation is found, but it does not prove whether external controls exist outside the codebase.
- L8 can verify versions and CVEs with `WebSearch`, but it does not replace a full SBOM or package-signing review.

## References

- OWASP MASVS: <https://mas.owasp.org/MASVS/>
- OWASP MASVS Storage: <https://mas.owasp.org/MASVS/controls/MASVS-STORAGE/>
- OWASP MASVS Network: <https://mas.owasp.org/MASVS/controls/MASVS-NETWORK/>
- OWASP MASVS Authentication: <https://mas.owasp.org/MASVS/controls/MASVS-AUTH/>
- OWASP MASVS Platform Interaction: <https://mas.owasp.org/MASVS/controls/MASVS-PLATFORM/>
- OWASP MASVS Resilience: <https://mas.owasp.org/MASVS/controls/MASVS-RESILIENCE/>
- CWE-312 Cleartext Storage of Sensitive Information: <https://cwe.mitre.org/data/definitions/312.html>
- CWE-295 Improper Certificate Validation: <https://cwe.mitre.org/data/definitions/295.html>
- CWE-287 Improper Authentication: <https://cwe.mitre.org/data/definitions/287.html>
- CWE-798 Use of Hard-coded Credentials: <https://cwe.mitre.org/data/definitions/798.html>
- CWE-656 Reliance on Security Through Obscurity: <https://cwe.mitre.org/data/definitions/656.html>
- CWE-919 Weaknesses in Mobile Applications: <https://cwe.mitre.org/data/definitions/919.html>
- CWE-926 Improper Export of Android Application Components: <https://cwe.mitre.org/data/definitions/926.html>
- CWE-89 SQL Injection: <https://cwe.mitre.org/data/definitions/89.html>
- CWE-532 Insertion of Sensitive Information into Log File: <https://cwe.mitre.org/data/definitions/532.html>

# agentX44 Claude Code Subagents

## Overview

This repository is a collection of Claude Code subagents intended to help mobile developers work faster and with more consistency.

The first implemented subagent pack is `security/`, which contains an 11-agent mobile security audit system for Android and iOS projects. The long-term project direction is broader than security, but today the repository ships one focused pack.

## Current Contents

```text
.
├── security/
│   ├── README.md
│   ├── security-orchestrator.md
│   ├── security-l1-storage.md
│   ├── security-l2-network.md
│   ├── security-l3-auth.md
│   ├── security-l4-secrets.md
│   ├── security-l5-code.md
│   ├── security-l6-runtime.md
│   ├── security-l7-ipc.md
│   ├── security-l8-deps.md
│   ├── security-l9-input.md
│   └── security-l10-logging.md
└── LICENSE
```

## Available Pack

### `security/`

Security-focused Claude Code agents for mobile app audits.

- 1 orchestrator agent
- 10 layer-specific audit subagents
- Android and iOS coverage
- Output centered on `SECURITY_AUDIT_REPORT.md`

See [security/README.md](/Users/vdaculan/Desktop/agentX44-claude-code/security/README.md) for the pack-level architecture, usage, and report format.

## Installation

Place the agent files you want to use in:

```text
~/.claude/agents/
```

For the current security pack, copy:

```text
security/security-orchestrator.md
security/security-l1-storage.md
security/security-l2-network.md
security/security-l3-auth.md
security/security-l4-secrets.md
security/security-l5-code.md
security/security-l6-runtime.md
security/security-l7-ipc.md
security/security-l8-deps.md
security/security-l9-input.md
security/security-l10-logging.md
```

Claude Code discovers agents from `~/.claude/agents/` using the frontmatter metadata in each file.

## Project Scope

- This repository is for reusable Claude Code subagents aimed at mobile development tasks.
- The current implementation is security-focused only.
- Additional packs can be added later as separate folders without changing the root project structure.

## License

This project is licensed under the terms in [LICENSE](/Users/vdaculan/Desktop/agentX44-claude-code/LICENSE).

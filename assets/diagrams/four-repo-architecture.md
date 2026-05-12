# Polywave: Four-Repo Architecture

## Dependency Graph

```
┌─────────────────────────────────────────────┐
│  polywave-protocol  (Specification)         │
│  github.com/blackwell-systems/              │
│              polywave-protocol              │
│                                             │
│  +-- Invariants (I1-I6)                     │
│  +-- Execution rules (E1-E48)               │
│  +-- State machine                          │
│  +-- Message formats + JSON schema          │
│  +-- Program invariants (P1-P5)             │
└──────────────────┬──────────────────────────┘
                   │
                   │  spec defines behavior
                   v
┌─────────────────────────────────────────────┐
│  polywave  (Claude Code Implementation)     │
│  github.com/blackwell-systems/polywave      │
│                                             │
│  +-- Agent Skill (SKILL.md, references/)    │
│  +-- Agent prompts (scout, wave-agent, etc.)│
│  +-- 22 enforcement hooks                  │
│  +-- Install script (install.sh)            │
└──────────────────┬──────────────────────────┘
                   │
                   │  skill calls polywave-tools CLI
                   v
┌─────────────────────────────────────────────┐
│  polywave-go  (Engine / SDK)                │
│  github.com/blackwell-systems/polywave-go   │
│                                             │
│  +-- polywave-tools CLI binary              │
│  +-- Engine package (pkg/engine/)           │
│  +-- Protocol types (pkg/protocol/)         │
│  +-- Git worktree management (internal/git/)│
│  +-- IMPL doc parser + validator            │
└──────────────────┬──────────────────────────┘
                   │
                   │  Go module import
                   v
┌─────────────────────────────────────────────┐
│  polywave-web  (Web Application)            │
│  github.com/blackwell-systems/polywave-web  │
│                                             │
│  +-- polywave-web binary (HTTP server)      │
│  +-- React web UI (embedded via go:embed)   │
│  +-- HTTP API endpoints                     │
│  +-- SSE live streaming                     │
└─────────────────────────────────────────────┘
```

## Data Flow

IMPL docs live in the **target project** (not in any Polywave repo). Here is how
data moves between repos at runtime:

```
Target Project (your codebase)
  docs/IMPL/*.yaml  <───────  polywave-tools reads/writes IMPL docs
       ^                            |
       |                      ┌─────┴─────┐
       |                      │ polywave-  │  (built from polywave-go)
       |                      │   tools    │
       |                      └─────┬─────┘
       |                            |
       |                      ┌─────┴──────────────────┐
       └──────────────────────│  polywave-web server    │
                              │  (imports polywave-go   │
                              │   as Go module)         │
                              └────────────────────────┘

~/.claude/skills/polywave/
  SKILL.md  ──symlink──>  polywave/implementations/
                          claude-code/prompts/polywave-skill.md
```

- **polywave-tools** reads and writes IMPL docs directly in your project
- **polywave-web** server imports `polywave-go` as a Go module for engine logic
- **Skill files** are symlinked from the polywave repo into `~/.claude/skills/polywave/`

## Which Repo Do I Change?

| I want to change...                | Repo                   |
|------------------------------------|------------------------|
| Protocol invariants or rules       | `polywave-protocol`    |
| Agent prompts or skill files       | `polywave`             |
| Hooks or enforcement behavior      | `polywave`             |
| CLI commands or engine logic       | `polywave-go`          |
| IMPL doc parsing or validation     | `polywave-go`          |
| Web UI or React components         | `polywave-web`         |
| HTTP API or SSE streaming          | `polywave-web`         |
| All four (new protocol feature)    | Protocol -> Skill -> SDK -> Web |

**Rule of thumb:** Start with protocol (source of truth), then skill (prompts and hooks),
then SDK (types must match), then web (consumes both). Skip repos that aren't affected.

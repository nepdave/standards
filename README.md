# standards

Shared engineering and Claude Code standards. Other repos reference these skills to bootstrap consistent practices across projects.

## Skills

### api-audit

API audit across three dimensions: API design (against Google API Design Guide and Microsoft REST API Guidelines), end-to-end test readiness, and agentic workflow support. Language and framework agnostic — discovers the API surface from routes, handlers, OpenAPI specs, protos, or GraphQL schemas. Produces a PASS/WARN/FAIL report with file paths and line numbers.

### commit-push

Commit and push workflow. Stages changes, writes a structured commit message (summary + bulleted changes), and pushes to the current branch. Runs on `claude-haiku-4-5` for speed.

### go

Go code tidy pass. Reviews code against Effective Go, Practical Go, and standard library conventions, then runs `go fmt`, `go vet`, `goimports`, and `staticcheck`.

## CLAUDE.md Template

`CLAUDE.md.template` is a starter CLAUDE.md for new repos. Copy it, rename to `CLAUDE.md`, replace the `{placeholders}`, and add project-specific sections.

## Usage

1. Copy `CLAUDE.md.template` into your repo as `CLAUDE.md` and customize it.
2. Copy or symlink `.claude/skills/` into your repo's `.claude/skills/` to inherit the shared skills.

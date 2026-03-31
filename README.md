# standards

Shared engineering and Claude Code standards. Other repos reference these skills to bootstrap consistent practices across projects.

## Skills

### commit-push

Commit and push workflow. Stages changes, writes a structured commit message (summary + bulleted changes), and pushes to the current branch. Runs on `claude-haiku-4-5` for speed.

### go

Go code tidy pass. Reviews code against Effective Go, Practical Go, and standard library conventions, then runs `go fmt`, `go vet`, `goimports`, and `staticcheck`.

## Usage

Copy or symlink the `.claude/skills/` directory into your repo's `.claude/skills/` to inherit these standards.

---
name: go
description: Go code tidy pass — review against Effective Go, Practical Go, and standard library conventions, then run go fmt, go vet, and staticcheck.
allowed-tools: Read, Edit, Glob, Grep, Bash
---

Review all Go code in this repository against three authorities:

## 1. Effective Go (https://go.dev/doc/effective_go)
- Add minimal but informative comments to important functions, types, and non-obvious logic
- Ensure all exported identifiers have doc comments
- Apply idiomatic Go patterns (error handling, naming, package structure)
- Check for and fix any security vulnerabilities
- Don't make a change unless it is valid and justified

## 2. Dave Cheney's Practical Go (https://dave.cheney.net/practical-go)

### Package design
- A package's name is both a description of its purpose and a namespace prefix. Every exported name is prefixed by the package name — design accordingly.
- Avoid stutter: `http.HTTPServer` is wrong, `http.Server` is right. `ofac.OFACImport` is wrong, `ofac.Import` is right.
- Avoid package names like `base`, `common`, `util`, or `helpers`. Name packages after what they provide, not what they contain.
- Good packages provide a single, coherent API. If a package has unrelated functionality, split it.

### Naming
- Variable names: short-lived → short names (`i`, `r`, `w`). Long-lived → descriptive names. The length of a variable name should be proportional to the distance between its declaration and use.
- Use consistent naming within the codebase. If one file calls it `publishID`, every file calls it `publishID`.
- Don't name context variables anything other than `ctx`.
- Don't name errors anything other than `err` (or `xxxErr` when you need two in scope).

### API design
- Make the zero value useful when possible.
- Return concrete types, accept interfaces. "Be conservative in what you send, be liberal in what you accept."
- The bigger the interface, the weaker the abstraction. Prefer small interfaces (`io.Reader`, `io.Writer`) over large ones.
- Design APIs for the common case. Make simple things easy and complex things possible.

### Error handling
- Only handle an error once. Either return it or log it — never both.
- Wrap errors with context: `fmt.Errorf("reading config: %w", err)` — always describe what failed, not why.
- Sentinel errors (`var ErrNotFound = errors.New(...)`) should be used sparingly. Prefer error wrapping.
- Don't panic. Panics are for unrecoverable programmer errors, not runtime conditions.

### Concurrency
- Start goroutines only when you know when and how they will stop.
- Prefer passing data through channels over sharing memory with mutexes.
- Use `context.Context` for cancellation and deadlines — never bare goroutines with no shutdown path.

### Project structure
- `cmd/` for binaries, `internal/` for private packages. This is not optional.
- Avoid premature abstraction. Three similar blocks of code is better than one premature generalization. Abstract only when the pattern is proven.
- A little copying is better than a little dependency — but 10 copies of the same function across 10 packages means you missed an extraction.

### Simplicity
- Don't add features, refactor code, or make "improvements" beyond what was asked.
- Don't add error handling for scenarios that can't happen. Trust internal code and framework guarantees.
- Don't create helpers for one-time operations. Don't design for hypothetical future requirements.
- The right amount of complexity is the minimum needed for the current task.

## 3. Go Standard Library Conventions (https://pkg.go.dev/std)

The standard library is the style guide. Write code that looks like it belongs in `net/http`, `database/sql`, or `encoding/json`.

### Doc comments
- Exported identifiers get doc comments. Period. The comment starts with the name: `// Server is ...`, `// Import loads ...`.
- Package comments go on the package clause, not in a separate `doc.go` unless the comment is long.
- Don't restate the obvious. `// Close closes the connection` adds nothing. `// Close releases the database connection and waits for in-flight queries to finish` does.

### Function signatures
- `context.Context` is always the first parameter. No exceptions.
- `error` is always the last return value.
- Options structs over long parameter lists. If a constructor takes more than 3-4 parameters, consider a config struct or functional options.
- Prefer `func New(...) *T` for constructors. If there's only one obvious type in the package, `New` is fine. If ambiguous, `NewServer`, `NewWatcher`.

### Types and interfaces
- Structs are nouns (`Server`, `Watcher`, `Entry`). Methods are verbs (`Run`, `Close`, `Import`).
- Interfaces are named by what they do, not what they are: `Reader`, `Writer`, `Handler` — not `IReader` or `Readable`.
- Define interfaces at the point of consumption, not the point of implementation.
- Use unexported fields with exported methods. Hide implementation details.

### Patterns from the standard library
- **`io.Reader`/`io.Writer`**: small interfaces composed into larger ones.
- **`http.Handler`**: single-method interfaces enable composition via middleware.
- **`sql.DB`**: pool management hidden behind a clean API. Callers don't manage connections.
- **`json.Encoder`/`json.Decoder`**: streaming over buffering when data can be large.
- **`context.WithTimeout`/`context.WithCancel`**: explicit lifetime management, never implicit.
- **`errors.Is`/`errors.As`**: check error types through the chain, don't compare strings.

### Testing (follow `testing` package conventions)
- Test functions: `TestFunctionName`, `TestType_Method`, or `TestFunctionName_condition`.
- Table-driven tests with `t.Run` for subtests. Name each case clearly.
- Use `t.Helper()` in test helpers so failures report the caller's line.
- Use `t.Cleanup()` for teardown instead of `defer` when the cleanup depends on the test context.
- `testdata/` directory for test fixtures. `_test.go` files only.

### Import organization
- Standard library imports first, then a blank line, then third-party, then internal packages.
- Group by origin, not by usage. `goimports` handles this automatically.

## Checklist

After reviewing the code, run the following and fix any issues found:

```
go fmt ./...
go vet ./...
goimports -w $(find . -name '*.go' -not -path './vendor/*')
staticcheck ./...
```

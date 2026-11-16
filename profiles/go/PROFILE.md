# Profile: Go

## Toolchain

- Go, a currently supported release. The `go` directive in `go.mod` is the floor.
- Standard tools only by default: `gofmt`, `go vet`, `go test`. Add `staticcheck` or
  `golangci-lint` as a recorded decision, then add its command to `AGENTS.md`.

## Commands

Copy into the Commands table in `AGENTS.md`.

| Task | Command |
|---|---|
| Setup | `go mod download` |
| Format | `gofmt -w .` |
| Lint | `go vet ./...` |
| Typecheck | `go build ./...` |
| Test (all) | `go test ./...` |
| Test (single) | `go test ./path/to/pkg -run '^TestName$'` |
| Build | `go build ./...` |

Before a gate, also run `go test -race ./...` and confirm `gofmt -l .` prints nothing.

## Layout

`cmd/<name>/main.go` for binaries, `internal/` for code that must not be imported by
other modules, packages at the root or under `pkg/` for a public library. Tests sit
beside the code as `*_test.go`.

## Versioning

Go modules are versioned by git tags of the form `vX.Y.Z`, which is the same tag this
scaffold already uses. `VERSION` stays the source of truth for the changelog and README,
and the tag the human creates must match it. There is no mirror file by default.

```json
"profiles": ["go"],
"version_mirrors": []
```

If a binary reports its version, either inject it at build time with
`-ldflags "-X main.version=<X.Y.Z>"`, or keep a constant and mirror it:

```json
"version_mirrors": [
  {"path": "internal/version/version.go", "kind": "regex",
   "pattern": "Version\\s*=\\s*\"([^\"]+)\""}
]
```

A MAJOR version of 2 or higher changes the module path: `module example.com/x/v2` in
`go.mod`, and every import of the module's own packages gains `/v2`. A breaking release
is therefore a code change as well as a version change. Say so in the gate request.

## Conventions

- Return errors; wrap with `fmt.Errorf("doing x: %w", err)`. No panics in library code.
- Accept `context.Context` as the first parameter of anything that blocks or does I/O.
- Table-driven tests with `t.Run`.

## Gotchas

- `gofmt -l .` exits 0 even when files need formatting; check that its output is empty.
- `go test` caches results. Use `-count=1` when you need a real re-run.
- A tag pushed to a public module is cached by the module proxy and cannot be changed.
  Fix a bad release with a new version.

## .gitignore

```
/bin/
/dist/
*.test
*.out
coverage.*
```

# Context compiler and subset checker: command-line contract (planned)

**Status: interface specification only; no implementation is released.** Version 0.1.0-spec.

Two tools, one binary, for Go modules. The subset checker enforces the properties under which the
context compiler's output is sufficient by construction; the compiler emits the context for a
proposed write-set. Both operate on the dependency graph from `go list -json -deps -test ./...`.

## `suffctx check [module-dir]`

Reports violations of the enforced subset. Exit 0 if none. Each violation names the rule, the
file:line, and the unit. Rules (v0.1):

| Rule | What it forbids | Why |
|---|---|---|
| `no-reflect` | `reflect` use outside an allow-list | reflection is a channel the graph does not see |
| `no-linkname` | `go:linkname` | bypasses the import graph |
| `no-init-side-effects` | `init()` with observable effects, blank imports for registration | init-order channels |
| `no-global-mutable` | package-level mutable variables written from more than one package | shared-state channel |
| `exported-doc` | exported declarations without a doc comment | interfaces must carry their contract |
| `test-binding` | test files in a package that import no symbol from it | tests must be bound to a unit |

## `suffctx context --write-set <pkgs> [--interface-change] [--level iface|bodies|transitive] [--tokenizer id] [--budget N]`

Emits a deterministic, serialised context for the write-set:

- `--level iface` (default): W's source + exported declarations and doc comments of deps(W).
  This is the P1 context.
- `--level iface --forward`: additionally the exported declarations and test names of F(W).
  This is the P2/Requirement-1 context, emitted automatically when `--interface-change` is set.
- `--level bodies`: same package set as `iface`, bodies instead of declarations (control arm).
- `--level transitive`: W + bodies of B(W) ∪ F(W) + their tests.

Output: a single UTF-8 text with a stated file order (W last, immediately before the task), plus a
JSON sidecar listing units, token counts, and an `expressible` flag when `--gold-patch <file>` is given.
`--budget N` truncates lowest-priority units first and records what was dropped.

## `suffctx slice --tests <list>`

Computes the coverage slice (§3.1 of SPEC.md) and the closure check (§3.3) for a set of tests on
the current tree. With `--mutate k` also computes the mutation slice (§3.2).

## Guarantees and non-guarantees

Under a module that passes `suffctx check`, the `iface` context contains every fact the edit depends
on through the import graph (P1), and when the interface is unchanged the forward slice is empty
(P2). Channels outside the import graph that the checker does not forbid (exec of repository
binaries, file-system sharing between tests) are listed, not guaranteed. The tool makes no claim
about editor success; that is measured, not compiled.

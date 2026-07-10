# Charon — Project Plan

## Purpose

Charon is a single-file audit prompt for Claude Code that detects dead code, dead files, unused dependencies, and duplicated code in a repository and writes an evidence-backed report. It exists because stale code degrades AI-assisted development disproportionately: LLM agents read dead code, index it as if it were live, imitate its patterns, and route new work onto APIs nothing uses anymore. A codebase that is honest about its dead code is a codebase agents can reason about.

The name is the job description: identify what has died and prepare it for removal. As of v2.0, Charon also performs the removal itself — mode-gated, independently reviewed, build-verified, on a branch the user merges — while the audit itself remains strictly read-only.

## Design principles

1. **Gated resolution.** The Audit Loop never writes; the Resolution Loop executes only user-mode-approved, Critic-reviewed, build-verified change-sets on a branch the user merges. Judgment-required findings are never executed in any mode. The reasoning stands unchanged from v1.0: reachability analysis has systematic blind spots, and one wrongly deleted reflection-invoked handler outweighs a hundred correct deletions — so every deletion passes an independent judge and a build gate, and the blind-spot classes pass a human.
2. **Evidence-bound.** Every finding carries file:line, the detecting tool, verbatim tool output, and the last-touched date from git history. No evidence, no entry.
3. **False positives are first-class.** Findings matching known blind-spot classes (reflection/dynamic dispatch, DI wiring, framework entry points, externally-consumed public APIs, test fixtures, plugin registries) are flagged JUDGMENT-REQUIRED and never presented as confirmed.
4. **CLI tools, not MCP servers.** All detection runs through maintained, ecosystem-native CLIs (knip, vulture, staticcheck/deadcode, cargo-machete, Roslyn analyzers, jscpd). This keeps context cost at zero: no MCP schema tax, no server maintenance, and the tools are independently versioned and testable.
5. **Freshness mandate.** The tool table is dated and the prompt re-verifies tool recency on every run. Dead-code tooling moves fast; an audit built on an abandoned tool is itself dead code.
6. **Standalone first, Phanes-aware second.** Charon works in any repository. When it detects a Phanes installation (`.claude/.phanes`), it cooperates: report filed into the documentation tree, TODOs into the session summary, registry annotations proposed (never written) for the architect. Single-writer discipline is respected throughout.

## Deliverables

- `Charon.md` — the prompt (v2.0, done)
- `README.md` — install, usage, report interpretation (done)
- `Changelog.md`, `LICENSE` (CC BY-NC 4.0, matching Phanes) (done)
- Public repository `github.com/Aloim/charon` with install via raw URL, mirroring the Phanes distribution model (done)
- Cross-reference from the Phanes README as an optional companion tool (pending)

## Roadmap

### v1.0 — initial release (shipped)
Prompt, README, changelog, license. Distribution model identical to Phanes: one file into `~/.claude/commands/`, invoked as `/charon`.

### v2.0 — re-architecture (current)
Full rewrite in the Phanes v2.1 document style. Shipped in v2.0: mode-gated resolution (AUTORESOLVE / APPROVE-FIRST), universal Critic gate, JSON companion output (was planned as v1.2), and trend tracking against the previous audit (was Later/speculative). Deep Phanes integration: registry drift flags, Documentation Debt routing, api-monitor regen after removals.

### v2.1 — validation
Run against at least two real projects (one TypeScript, one Python) seeded with known dead code, a reflection-invoked handler, a public-API export, and a duplicated module. Verify: all planted dead code found, the reflection case and public export land in JUDGMENT-REQUIRED (not removal candidates), clone cluster detected with a sensible canonical suggestion, and the Audit Loop modifies nothing (byte-for-byte check). Additionally validate resolution: one full AUTORESOLVE run on a seeded repo (all clean candidates removed, judgment-required untouched, branch left unmerged) and one failing-test revert case (change-set reverted and reclassified with the failure as evidence). Tune the false-positive class list and report format based on what the runs reveal.

### Later / speculative
- Deeper Phanes integration: the Phanes Cleaner archetype invoking Charon when installed, and Phanes' update runs recommending a Charon audit before large refactors.
- Language coverage expansion as ecosystems mature (the tool table is the extension point).
- Report-size discipline for very large sweeps (summary caps with drill-down sections).

## Non-goals

- **Ungated fix mode.** Resolution without the mode question, the Critic gate, or the branch boundary will not be built. (v2.0 built *gated* resolution deliberately — see Changelog.)
- **MCP server variant.** The CLI-based design is deliberate; a server adds context cost and delivers nothing the CLIs don't.
- **Style/lint checking.** Charon is about liveness and duplication, not formatting. Linters already exist.

## Open questions

- Whether the Phanes prompt itself should mention Charon (tighter coupling) or only the Phanes README (loose coupling, current plan).

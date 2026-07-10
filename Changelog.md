# Changelog

All notable changes to **Charon**. The authoritative version marker is the stamp on the first line of `Charon.md`.

---

## v2.0 (2026-07-10)

Full re-architecture in the Phanes v2.1 document style. **Deliberate contract change:**
the v1.0 advisory-only rule is replaced by mode-gated resolution. The non-goal
"auto-fix will not be built" is now "no *ungated* fix will ever be built."

- Two-loop architecture: read-only Audit Loop (identical in every mode and world) and a
  Resolution Loop that consumes the report as its sole input.
- Mode contract, asked once at initialization: AUTORESOLVE (execute clean removal
  candidates, each change-set still Critic-reviewed) or APPROVE-FIRST (report, then
  execute the user's CH-id selection).
- Universal Critic gate: Phanes projects route removals through their own
  Critic → Executor workflows; standalone runs spawn an adversarial reviewer subagent.
- Hard rails in every mode: judgment-required findings never executed; clone merges never
  Autoresolved; all writes on a `charon-cleanup-<date>` branch Charon never merges;
  revert-and-reclassify on build/test failure.
- JSON companion output next to the markdown report (stable identity keys for CI/agents).
- Trend tracking against the previous audit: NEW / LINGERING / FIXED per finding,
  matched on (kind, path, symbol), never on per-run CH-ids.
- Deep Phanes integration: registry drift flags, Documentation Debt as T1 doc tasks,
  api-monitor tier-1 regen after removals, tier-2 annotation proposals for the architect.
- Full Phanes v2.1 voice and structure: Ferryman persona, principles-stated-once,
  dated rubrics, verbatim blocks, per-phase personas and think-directives.

## v1.0 (2026-07-10)

Initial release.

- Single-file audit prompt: dead symbols, dead files, unused dependencies, and clone clusters, with per-candidate evidence (file:line, detecting tool, verbatim proof, last-touched date).
- Dated tool table (reviewed 2026-07-10): knip, vulture, staticcheck/deadcode, cargo-machete, Roslyn analyzers, jscpd; the prompt re-verifies tool freshness on every run and records coverage gaps.
- Strict advisory contract: no deletions, no source edits, mandatory false-positive triage (reflection, dependency injection, framework entry points, public APIs, fixtures, plugin registries) with judgment-required flagging.
- Phanes cooperation: audit report filed into the documentation tree, session-summary TODOs, and registry annotation proposals for the architect, with single-writer discipline respected.

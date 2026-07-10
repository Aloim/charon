# Changelog

All notable changes to **Charon**. The authoritative version marker is the stamp on the first line of `Charon.md`.

---

## v1.0 — 2026-07-10

Initial release.

- Single-file audit prompt: dead symbols, dead files, unused dependencies, and clone clusters, with per-candidate evidence (file:line, detecting tool, verbatim proof, last-touched date).
- Dated tool table (reviewed 2026-07-10): knip, vulture, staticcheck/deadcode, cargo-machete, Roslyn analyzers, jscpd; the prompt re-verifies tool freshness on every run and records coverage gaps.
- Strict advisory contract: no deletions, no source edits, mandatory false-positive triage (reflection, dependency injection, framework entry points, public APIs, fixtures, plugin registries) with judgment-required flagging.
- Phanes cooperation: audit report filed into the documentation tree, session-summary TODOs, and registry annotation proposals for the architect, with single-writer discipline respected.

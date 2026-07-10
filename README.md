# Charon

**Charon** is a single-file audit-and-resolution prompt for [Claude Code](https://claude.com/claude-code) that finds dead code, unused files, unused dependencies, and duplicated code in your repository — then, in the mode you choose, removes the confirmed dead through independently reviewed, build-verified change-sets on a branch you merge yourself.

The name is the job description: Charon identifies what has died in a codebase and ferries it out — but only what has been proven dead, only past an independent reviewer, and only across a branch boundary you control.

Charon works entirely on its own in any repository — but it really shines in combination with [Phanes](https://github.com/Aloim/phanes): in a Phanes-managed project the audit reads the agent team's registries, removals flow through the project's own Critic → Executor review chain, and the report is filed straight into the Phanes documentation tree (see [Works with Phanes](#works-with-phanes)).

**Why bother?** Dead and duplicated code is not just clutter; it actively degrades AI-assisted development. Agents read it, index it, imitate it, and route new work onto APIs that nothing uses anymore. A codebase that is honest about its dead is a codebase your tools can reason about.

**Contents**

- [What it does](#what-it-does)
- [What it will never do without you](#what-it-will-never-do-without-you)
- [How to install](#how-to-install)
- [How to use](#how-to-use)
- [Reading the report](#reading-the-report)
- [Works with Phanes](#works-with-phanes)
- [Version](#version) · [License](#license)

---

## What it does

Running `/charon` in a project drives Claude Code through two strictly separated loops:

1. **Mode question.** At initialization Charon asks once (or reads it from your arguments): **AUTORESOLVE** — after the audit, execute every clean removal candidate automatically — or **APPROVE-FIRST** — stop at the report and execute only the findings you select.
2. **Survey.** Detects your languages, build systems, entry points, and public API surface (a library's exports are alive by contract, even when internally unreferenced; Charon records this before sweeping, not after).
3. **Sweep.** Selects maintained, ecosystem-native detectors from a dated tool table re-verified on every run — knip (TypeScript/JavaScript), vulture (Python), staticcheck and deadcode (Go), compiler lints plus cargo-machete (Rust), Roslyn analyzers (C#), and jscpd for clone detection across roughly 150 languages — runs them, and normalizes every finding with evidence and its last-touched date from git history.
4. **Triage.** Classifies every finding: clean removal candidates, judgment-required findings (anything matching a known false-positive class — reflection, dependency injection, framework entry points, public APIs, test fixtures, plugin registries), and clone clusters. Compares against your previous audit to mark each finding NEW, LINGERING, or FIXED.
5. **Report.** Writes the audit report plus a machine-readable JSON companion: summary counts, per-finding evidence with file and line, trend against the last run, clone clusters with a suggested canonical copy, unused dependencies, documentation debt, and coverage gaps.
6. **Resolve.** For the approved set only: one change-set per finding cluster on a dedicated `charon-cleanup-<date>` branch, each change-set independently reviewed by a critic whose only goal is to prove the code is *alive*, each verified by your build and tests, each reverted and reclassified if verification fails. The branch is handed to you unmerged, with a full Execution Record.

## What it will never do without you

Every removal is gated. Charon **never** executes judgment-required findings — in any mode; **never** auto-merges clone clusters (merging is refactoring judgment, reserved for explicit approval); **never** writes outside the cleanup branch; **never** merges that branch; and **never** skips the independent critic review, even in AUTORESOLVE mode. The audit itself is read-only in every mode — if you answer `none` at the report, your repository is byte-for-byte untouched.

This is deliberate: reachability analysis has well-known blind spots, and the cost of deleting one reflection-invoked handler exceeds the benefit of auto-deleting a hundred true positives.

## How to install

**Linux / macOS:**

```bash
mkdir -p ~/.claude/commands
curl -L https://raw.githubusercontent.com/Aloim/charon/main/Charon.md \
  -o ~/.claude/commands/charon.md
```

**Windows (PowerShell):**

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\commands" | Out-Null
Invoke-WebRequest `
  -Uri https://raw.githubusercontent.com/Aloim/charon/main/Charon.md `
  -OutFile "$env:USERPROFILE\.claude\commands\charon.md"
```

New to Claude Code entirely? The [Phanes README](https://github.com/Aloim/phanes#for-inexperienced-users-step-by-step-from-zero) has a step-by-step walkthrough from creating an account to a working install; Charon installs the same way.

## How to use

Open your project in Claude Code and run:

```
/charon
```

Charon asks for the mode once; arguments are forwarded, take priority over defaults, and can preset everything:

```
/charon autoresolve src/api; skip dependency checks
/charon approve-first
```

Good moments to run it: before a large refactor (so you refactor the living code, not the dead), after landing a big feature or migration (when superseded code tends to linger), and periodically on long-lived projects — the JSON baseline turns repeat runs into trend reports, so you can watch the dead-code count grow or shrink run over run.

## Reading the report

The report lands as `charon-audit-<date>.md` (repo root, or under `documentation/plans/fixes/` in Phanes-managed projects), with a JSON companion of the same name for CI and agent consumers. The sections that matter most:

- **Removal Candidates** carry full evidence and no false-positive flags. In AUTORESOLVE mode these are executed automatically (each still critic-reviewed); in APPROVE-FIRST they await your CH-id selection.
- **Judgment Required** findings resemble dead code but match a class the tools systematically misjudge. The flagged class tells you exactly what to check; Charon will never execute these.
- **Clone Clusters** show duplicated logic with a suggested canonical copy. Merging is refactoring work — approve it explicitly or it does not happen.
- **Trend** shows what got fixed since the last audit and what lingers, matched by identity, not report ordering.
- **Execution Record** lists every change-set applied, reverted, or skipped, with commit hashes, after a resolution run.

## Works with Phanes

Charon is standalone, but if it detects a [Phanes](https://github.com/Aloim/phanes) installation (the `.claude/.phanes` marker) it cooperates with it end to end: the report is filed into the Phanes documentation tree via `phanes new-file docs`, removals route through the project's own review chain (proposal → Critic → Executor), the api-monitor regenerates the tier-1 registry after removals, dead exported APIs are drafted as tier-2 registry annotation proposals for the architect, open items land in the session summary's TODO list, and documentation describing dead code is filed as tasks through the project's own workflows. Charon respects Phanes' single-writer rules throughout; it proposes, it never writes anyone else's artifacts.

---

## Version

Current: **v2.0** (2026-07-10). See the version stamp at the top of `Charon.md`. Release history: [`Changelog.md`](Changelog.md).

---

## License

Charon is released under the **Creative Commons Attribution-NonCommercial 4.0 International** license (see [`LICENSE`](LICENSE)).

You are free to use, share, and adapt Charon for any **non-commercial** purpose with attribution. Commercial use is not granted by this license; for commercial licensing terms, contact the author directly.

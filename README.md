# Charon

**Charon** is a single-file audit prompt for [Claude Code](https://claude.com/claude-code) that finds dead code, unused files, unused dependencies, and duplicated code in your repository, then writes an evidence-backed report instead of touching anything.

The name is the job description: Charon identifies what has died in a codebase and prepares it for removal, but never does the removing itself.

**Why bother?** Dead and duplicated code is not just clutter; it actively degrades AI-assisted development. Agents read it, index it, imitate it, and route new work onto APIs that nothing uses anymore. A codebase that is honest about its dead is a codebase your tools can reason about.

**Contents**

- [What it does](#what-it-does)
- [What it will never do](#what-it-will-never-do)
- [How to install](#how-to-install)
- [How to use](#how-to-use)
- [Reading the report](#reading-the-report)
- [Works with Phanes](#works-with-phanes)
- [Version](#version) · [License](#license)

---

## What it does

Running `/charon` in a project drives Claude Code through a strict audit:

1. **Survey.** Detects your languages, build systems, entry points, and public API surface (a library's exports are alive by contract, even when internally unreferenced; Charon records this before sweeping, not after).
2. **Tooling.** Selects maintained, ecosystem-native detectors from a dated tool table that the prompt re-verifies on every run: knip (TypeScript/JavaScript), vulture (Python), staticcheck and deadcode (Go), compiler lints plus cargo-machete (Rust), Roslyn analyzers (C#), and jscpd for clone detection across roughly 150 languages. Tools run project-local or ephemeral; nothing is installed globally without asking.
3. **Sweep.** Runs the detectors, normalizes the findings, and enriches every candidate with its last-touched date from git history.
4. **Triage.** Classifies every finding. Genuinely unreferenced symbols become candidates for removal. Anything matching a known false-positive class stays flagged for human judgment: code reached via reflection or dynamic dispatch, dependency-injection wiring, framework entry points (routes, handlers, serializers), externally-consumed public APIs, test fixtures, and plugin registries.
5. **Report.** Writes the audit report: summary counts, per-finding evidence with file and line, clone clusters with a suggested canonical copy, unused dependencies, and coverage gaps for anything the sweep could not check.

## What it will never do

Charon is advisory only. It does not delete code, does not edit source files, does not remove dependencies, and does not "clean up while it's at it." Removals belong in your normal review process, one change set at a time, using the report as input. This is deliberate: reachability analysis has well-known blind spots, and the cost of deleting one reflection-invoked handler exceeds the benefit of auto-deleting a hundred true positives.

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

Arguments are forwarded and take priority over defaults, so you can scope the sweep:

```
/charon only the src/api module; skip dependency checks
```

Good moments to run it: before a large refactor (so you refactor the living code, not the dead), after landing a big feature or migration (when superseded code tends to linger), and periodically on long-lived projects. On small codebases a run is cheap; let the size of the report tell you your cadence.

## Reading the report

The report lands in your project as `charon-audit-<date>.md` (or under `documentation/plans/fixes/` in Phanes-managed projects). Three sections matter most:

- **Removal Candidates** carry full evidence and no false-positive flags. Review them, then remove through your normal process.
- **Judgment Required** findings resemble dead code but match a class the tools systematically misjudge. The flagged class tells you exactly what to check before deciding.
- **Clone Clusters** show duplicated logic with a suggested canonical copy. Merging is refactoring work, not deletion work; treat it accordingly.

## Works with Phanes

Charon is standalone, but if it detects a [Phanes](https://github.com/Aloim/phanes) installation it cooperates with it: the report is filed into the Phanes documentation tree with a proper header, open items land in the session summary's TODO list, and dead exported APIs are drafted as registry annotation proposals for the architect agent, so the agent team stops routing new work onto them. Charon respects Phanes' single-writer rules throughout; it proposes, it never writes anyone else's artifacts.

---

## Version

Current: **v1.0** (2026-07-10). See the version stamp at the top of `Charon.md`. Release history: [`Changelog.md`](Changelog.md).

---

## License

Charon is released under the **Creative Commons Attribution-NonCommercial 4.0 International** license (see [`LICENSE`](LICENSE)).

You are free to use, share, and adapt Charon for any **non-commercial** purpose with attribution. Commercial use is not granted by this license; for commercial licensing terms, contact the author directly.

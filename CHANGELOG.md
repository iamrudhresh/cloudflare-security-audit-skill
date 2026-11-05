# Changelog

All notable changes to this project are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Because this skill's output is consumed by other tooling, any change that can invalidate an
existing `findings.json` or `coverage-ledger.json` is treated as a major change.

## [1.0.1] - 2026-11-05

### Fixed

- **Fork metadata pointed at the upstream project in two machine-read places.** `plugin.json`
  named Cloudflare, Inc. as author with the upstream repository as its homepage and repository,
  and `findings-to-sarif.cjs` stamped the upstream repository into the `informationUri` of every
  SARIF file it wrote — so audit output uploaded to GitHub code scanning identified a project
  that did not produce it. Both now point at this repository. The `LICENSE` copyright line, the
  README lineage notice, and the deliberate upstream links in the documentation and issue chooser
  are unchanged: those are attribution, which is correct and required.

## [1.0.0] - 2026-11-05

First release of this independent community fork of
[cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill).

Upstream had not merged a pull request since 10 September 2026, leaving 74 open pull requests and
40 open issues. This release triages that backlog: 34 pull requests merged, 39 closed as
superseded or unusable, and 28 issues resolved. Two bugs that nobody had filed were found and
fixed during the merge.

Full upstream history is preserved — this repository's history walks back to the original root
commit.

### Fixed

- **The validators could not read any input on Windows.** `fs.constants.O_NOFOLLOW` and
  `O_NONBLOCK` are undefined on win32, so `readFileWithinLimit()` refused every file before
  reading a byte. Since the workflow treats the validators as a hard gate, a full audit could not
  complete on Windows at all. The Windows path now rejects device paths and reserved device
  names, refuses links and non-regular files before opening, and re-checks the file identity on
  the open descriptor so a swap between `lstat` and `open` fails closed. A filesystem that cannot
  supply a file identity fails closed rather than passing. (upstream #53, #117; via #131)
- **Bare Win32 device names were refused only incidentally** — not filed upstream.
  `path.resolve("NUL")` returns an ordinary-looking drive path, so the `\\.\` and `\\?\` prefix
  guard never fired. `NUL`, `CON`, `COM1`, `LPT1` and `nul.txt` were rejected only because the OS
  errored. The input path now runs through the `WINDOWS_RESERVED_COMPONENT` regex the project
  already used for paths inside findings, in all four validators.
- **Two newly added validators re-introduced the Windows bug** — not filed upstream.
  `validate-run-metadata.cjs` and `validate-cross-consistency.cjs` each shipped their own copy of
  `readFileWithinLimit` with the original fail-closed guard, making both non-functional on Windows
  the moment they merged. Both now use the hardened path.
- A target-controlled repository name drove the output path and prior-run reuse, allowing path
  escape and cross-target prior-run poisoning. Run paths and prior-run identity now derive from a
  sanitized repo ID plus a hash of the target's canonical path. (#94, via #95)
- `in_progress` and `planned` units no longer pass terminal coverage validation, closing a hole
  where a run could claim completion with an open hunter assignment. (#79, via #84)
- Local-check artifacts are verified to exist on disk, so a unit cannot claim evidence that was
  never produced. (#21, via #25)
- Checks within a single attempt can share a promoted artifact; reuse across attempts is still
  rejected. (#81, via #87)
- Duplicate object keys are rejected in both `findings.json` and `coverage-ledger.json`.
  `JSON.parse` keeps the last occurrence, so a record could state two different things while the
  validator checked only one. (#102, #103)
- The 1,000-field object limit is enforced in `validate-findings.cjs`, which previously lacked the
  pre-parse and in-memory bounds its sibling validator had. (#107, via #111)
- Source line numbers are bounded from above; `Number.isInteger(1e21)` is `true`, so absurd line
  numbers passed validation. (via #45)
- A coverage unit can no longer exclude its own ordinary attack-class block, or claim a class its
  hunter was never assigned. (#126, via #104, #127)
- Git and symlink handling is hardened for untrusted targets: `core.fsmonitor` and diff drivers in
  a target's own `.git` config execute on `git status` and `git diff`. (via #76)
- Completion gates, carried-record handling, and the malformed-hunter-return path were mutually
  contradictory and could prevent a run from ever reaching `complete`. (#80, via #83, #32)
- An agent whose host refuses source review no longer stalls the hunter wave; it is recorded as
  `platform_safety_refusal`. (#11, via #13)

### Added

- **A fail-closed isolation gate and host mutation lock.** `validate-environment.cjs` refuses to
  let target code run unless `HOME`, temp and caches resolve inside the agent's `scratch/`, and
  `host-lock.cjs` serialises host mutations. This closes the worst report in the upstream tracker:
  an agent following the "scratch-local `HOME`" wording created a clean environment by **clearing
  the real user profile**. The rules now state that isolation is built beside the current
  environment and never by emptying it. (#128, via #129)
- **Target content is treated as untrusted data in every delegated prompt.** The audited
  repository is attacker-controlled input to the audit, not only its subject, and nothing
  previously told agents to ignore instructions embedded in it. (via #108)
- `validate-run-metadata.cjs` — validates `run-metadata.json` against the documented run-state
  contract, which nothing could previously check. (#78, via #85)
- `validate-cross-consistency.cjs` — proves the pairing between `findings.json` and
  `coverage-ledger.json`: every record must trace by fingerprint to a coverage unit that actually
  validated it. (via #44)
- `findings-to-sarif.cjs` — zero-dependency SARIF 2.1.0 converter for GitHub code scanning and
  other SARIF consumers. (via #8)
- `--target` ground-truthing on both validators: paths a ledger claims to have reviewed, and paths
  a finding cites as evidence, must exist under the target root. (via #123, #96)
- One CLI contract across the validators: a single path followed by flags, with extra positional
  arguments rejected instead of silently ignored. This resolved the mutually exclusive upstream
  #63 and #97. (via #123)
- `CRYPTOGRAPHY-AND-KEY-MANAGEMENT.md` — cryptographic implementation review as a first-class
  audit category. (#39, via #42)
- `ANDROID.md` — Android intent, URI-grant, caller-identity and release-configuration classes.
  (via #116)
- `SANDBOX-DOCKER.md` — sibling-Docker provisioning for hosts where the agent's own
  `unshare`/`bwrap` is denied by container seccomp. (via #60)
- New attack classes: streaming and partial-turn early dispatch (#15, via #16); modern full-stack,
  Next.js App Router and ORM boundaries (via #22); edge isolate state leakage and distributed KV
  races (via #35); abort-path cleanup and cross-request stale state (via #93); domain-format
  validators (#51, via #92).
- Optional `cwe`, `cvss` base vector, and `references` on findings, so output can feed
  vulnerability-management tooling without hand editing. (#86, via #90)
- `shared_blockers` in the report schema, so one deployment fact is modelled once and referenced
  by the `needs_validation` records that depend on it. (#73, via #74)
- Remediation must name the legitimate users of every path a fix refuses, and say how each keeps
  working. Reported upstream after a run where more than half the implemented fixes removed
  designed functionality. (#88, via #89)
- `plugin.json`, so the repository loads as a client-neutral Agent Plugin. (#118, via #120)
- `check-environment.cjs` — environment doctor reporting Node version, OS, and validator
  capability. (via #49)
- Direct invocation as `/cloudflare-security-audit`, for when several installed skills share the
  same triggers. (#5)
- Continuous integration: every `*.test.cjs` suite on Linux, Windows and macOS, plus a job that
  refuses committed conflict markers. Upstream had no CI, which is how the Windows breakage and a
  74-pull-request pile-up both went unnoticed. (via #33)
- Project governance: `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, issue templates
  (including one for proposing an attack class), and a pull request template.

### Changed

- Domain companion guides are linked directly from `SKILL.md`. They were previously reachable only
  through other companions, which an agent may not fully read. (#119, via #121)
- `cwe` is **optional** on confirmed findings. The contributed change made it required, which
  would have invalidated every `findings.json` written before this release.
- Config-dependent reachability stays `needs_validation` rather than being downgraded to a
  hardening note — "not reachable today" and "not a defect" are different claims. (via #50)
- The README documents that dependency advisory and CVE coverage is out of scope. (via #43)
- Installation instructions cover the Claude app's ZIP uploader, which rejects the repository
  archive because `SKILL.md` is nested too deeply. (#64, via #65)

### Not included

Deliberately deferred, with reasons:

- **Separate remediation phase** (upstream #109). Moving fix design out of discovery is sound, but
  as submitted it removes the preserved-uses requirement and the classification fields from the
  hunter contract, both of which shipped in this release. It needs a decision on which contract
  wins rather than a guess.
- **Directory layout refactor** (upstream #56, #68, #62). Renames all 20 skill files and would
  have conflicted with every other pull request in the queue. Deferrable at no cost now that the
  backlog is clear; the discoverability problem it was partly meant to solve is addressed by #121.
- **MIT-0 relicensing** (upstream #10). A licensing decision, not a code change.

[1.0.1]: https://github.com/iamrudhresh/cloudflare-security-audit-skill/releases/tag/v1.0.1
[1.0.0]: https://github.com/iamrudhresh/cloudflare-security-audit-skill/releases/tag/v1.0.0

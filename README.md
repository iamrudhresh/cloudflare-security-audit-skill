# cloudflare-security-audit-skill

A coding-agent skill that turns your agent into a security auditor. It orchestrates isolated
agents through reconnaissance, coverage-led hunting, candidate validation, structured output,
independent record verification, and target-neutral reporting.

[![Test](https://github.com/iamrudhresh/cloudflare-security-audit-skill/actions/workflows/test.yml/badge.svg)](https://github.com/iamrudhresh/cloudflare-security-audit-skill/actions/workflows/test.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

> ### Independent community fork
>
> This is a community-maintained continuation of
> [cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill),
> started on **5 November 2026**. It is **not affiliated with, endorsed by, or supported by
> Cloudflare, Inc.** The name describes the lineage of the code, not its ownership. For the
> official upstream project, use the link above.
>
> Upstream had not merged a pull request since 10 September 2026, leaving 74 open pull requests
> and 40 open issues — including validators that could not read a single byte of input on
> Windows. This fork triages that backlog: it merges the work that holds up, closes what is
> superseded or harmful, and fixes two bugs nobody had filed. See
> [CHANGELOG.md](CHANGELOG.md) for exactly what changed and
> [CONTRIBUTING.md](CONTRIBUTING.md) to help.

### Origin

The upstream skill seeded Cloudflare's vulnerability discovery harness, described in
[Build your own vulnerability harness](https://blog.cloudflare.com/build-your-own-vulnerability-harness).
That harness grew into a multi-stage, fleet-wide system; this skill is the single-repo starting
point it evolved from. The audit methodology here is Cloudflare's work, under MIT; this fork's
contribution is maintenance — merging, reconciling and hardening the contributed backlog.

## What it does

The skill runs a structured audit in six phases:

1. **Reconnaissance** -- map architecture, trust boundaries, input surfaces, prior evidence, and deterministic coverage in `architecture.md` and `coverage-ledger.json`.
2. **Coverage-led hunting** -- assign isolated hunters from ledger units, record their checks, and use coverage critics to find gaps.
3. **Candidate validation** -- give every unique candidate to a fresh verifier that tries to disprove it.
4. **Structured output** -- write `confirmed`, `needs_validation`, and `rejected` records to `findings.json` and validate them against `report-schema.json`.
5. **Independent record verification** -- fresh agents verify final source claims. Material replacements receive another independent verifier.
6. **Target-neutral reporting** -- derive `REPORT.md`, `FINDINGS-DETAIL.md`, and `NEEDS-VALIDATION.md` from the verified records and coverage ledger.

The parent runs `validate-coverage-ledger.cjs` after creating the ledger and after each later ledger update, and with `--terminal` in Phase 4 and after every Phase 5 replacement so a complete claim cannot retain open assignments. It runs `validate-findings.cjs` in Phase 4 and again after every Phase 5 replacement. It runs `validate-run-metadata.cjs` in Phase 4 and whenever `run_status` becomes `complete` or `incomplete`.

Once Phase 3 has moved validated units to `candidate`, run `validate-cross-consistency.cjs` to prove the pairing between the two files: every record in `findings.json` must trace by fingerprint to a `candidate` ledger unit that actually validated it. Run it while the ledger still holds the candidate fingerprints (before Phase 5 final dispositions clear them); an orphan fingerprint means the report claims a record no coverage unit ever produced.

The verdicts are distinct: `confirmed` has a complete source trace and bounded observed result, `needs_validation` has an exact unresolved fact and no severity, and `rejected` records a disproved candidate.

Multiple runs against the same repo are additive. The skill uses prior ledgers and findings to target gaps, revalidate changed source, and carry forward current-source evidence without treating stale or unresolved work as covered.

### SARIF export

To feed audit results into GitHub code scanning, VS Code SARIF viewers, or another SARIF consumer, convert a validated `findings.json` to SARIF 2.1.0:

```bash
node findings-to-sarif.cjs <output-dir>/findings.json <output-dir>/findings.sarif
```

Confirmed findings become results. Each `trace` becomes a `codeFlow` with the sink as the result's location, `overall_severity` sets the level and the code-scanning severity, and the finding's `fingerprint` keeps an alert's identity across runs. A rule is one title at one severity, so findings that share a title but not a severity do not share a score. `needs_validation` and `rejected` records are counted in `run.properties` rather than reported as results.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Setup, core principles, platform terminology, workflow overview, and audit anti-patterns |
| `RECONNAISSANCE.md` | Phase 1 reconnaissance prompts and synthesis instructions |
| `HUNTING.md` | Phase 2 orchestration, hunting methodology, and validation rules |
| `ATTACK-CLASSES.md` | Core, wildcard, and obvious-things attack prompts |
| `MEMORY-SAFETY-AND-BINARY.md` | Memory-safety, binary, and kernel hunting classes for native targets |
| `AI-AND-LLM.md` | Prompt-injection, multimodal input, commercial API spend asymmetry, agent/tool, and output-handling hunting classes for LLM-backed targets |
| `WEB-PROTOCOL-AND-AUTH.md` | HTTP request-framing, full-stack route caching, server-action mutation binding, and authentication-protocol hunting classes for HTTP-protocol, full-stack, and auth targets |
| `CRYPTOGRAPHY-AND-KEY-MANAGEMENT.md` | Cryptographic primitives, randomness, nonces, password hashing, JWT/TLS validation, and secret/key lifecycle hunting classes |
| `CLIENT-SIDE.md` | DOM-injection, server-component serialization leakage, messaging-trust, UI-redress, and prototype-pollution hunting classes for client-side/browser targets |
| `SUPPLY-CHAIN-AND-RELEASE.md` | Dependency, CI, release, signing, update, plugin, and extension hunting classes |
| `CLOUD-AND-DEPLOYMENT.md` | IAM, infrastructure-as-code, container, serverless, ingress, and runtime-configuration hunting classes |
| `PROTOCOLS-RPC-AND-MESSAGING.md` | RPC, serialization, queue, broker, webhook, and streaming-protocol hunting classes |
| `RESOURCE-EXHAUSTION-AND-AVAILABILITY.md` | Shared resource, quota, queue, worker, and operator-spend hunting classes |
| `DATA-ISOLATION-AND-LIFECYCLE.md` | Tenant isolation, ORM mass-assignment, NoSQL operator injection, row-level security bypass, cache, search, export, backup, migration, deletion, and restore hunting classes |
| `DESKTOP-MOBILE-AND-LOCAL-IPC.md` | Native app, deep-link, webview, exported-component, helper, daemon, and local-IPC hunting classes |
| `ANDROID.md` | Android component, URI grant, intent, Binder, WebView, app-state, and release-configuration hunting classes |
| `VALIDATION-AND-REPORTING.md` | Phases 3–6 candidate validation, structured output, record verification, and reporting |
| `report-schema.json` | JSON schema for all three `findings.json` verdicts |
| `validate-findings.cjs` | Zero-dependency validator for `findings.json` in Phases 4 and 5 |
| `validate-findings.test.cjs` | Findings-validator tests and producer-compatible fixture checks |
| `validate-coverage-ledger.cjs` | Zero-dependency validator for `coverage-ledger.json` in Phases 1–5 |
| `validate-coverage-ledger.test.cjs` | Coverage-ledger validator tests |
| `findings-to-sarif.cjs` | Zero-dependency converter from `findings.json` to SARIF 2.1.0 |
| `findings-to-sarif.test.cjs` | SARIF converter tests |
| `validate-environment.cjs` | Fail-closed isolation gate: refuses target execution unless `HOME`, temp, and caches resolve inside the agent's `scratch/` |
| `validate-environment.test.cjs` | Isolation-gate tests, including escape and CLI exit-code cases |
| `host-lock.cjs` | Exclusive lock for host mutations (install, upgrade, clean, delete), so two agents cannot mutate the machine at once |
| `host-lock.test.cjs` | Host-lock tests: contention, ownership, stale reclaim, CLI behavior |
| `SANDBOX-DOCKER.md` | Sibling-Docker sandbox provisioning for hosts where the agent's own unshare/bwrap is denied |
| `check-environment.cjs` | Environment doctor: reports Node version, OS, and whether the validators' safe-open flags are available |
| `check-environment.test.cjs` | Environment-doctor tests |
| `validate-run-metadata.cjs` | Zero-dependency validator for `run-metadata.json` run-state in Phase 4 |
| `validate-run-metadata.test.cjs` | Run-metadata validator tests |
| `validate-cross-consistency.cjs` | Zero-dependency cross-validator: findings fingerprints must trace to validated ledger candidates |
| `validate-cross-consistency.test.cjs` | Cross-validator tests |

## Installation

Install the skill with the [Skills CLI](https://skills.sh):

```bash
npx skills add https://github.com/iamrudhresh/cloudflare-security-audit-skill \
  --skill security-audit
```

Use `--global` for a user-level installation:

```bash
npx skills add https://github.com/iamrudhresh/cloudflare-security-audit-skill \
  --skill security-audit \
  --global
```

Run `npx skills --help` for agent-selection and non-interactive options.

### Uploading a ZIP in the Claude app

GitHub's **Code → Download ZIP** downloads the entire repository. Do not upload
that archive directly in **Settings → Skills → Add → Upload Skills**: its
`SKILL.md` is nested under `cloudflare-security-audit-skill-main/skills/security-audit/`,
which is deeper than the uploader accepts.

Instead, extract the repository archive, open its `skills` directory, and
compress the **`security-audit` folder** into a new ZIP. Keep all files in that
folder, including the companion documents, schema, and validators. For example,
with a command-line ZIP tool, run this from the extracted repository directory:

```bash
cd skills
zip -r security-audit.zip security-audit
```

The new archive should contain `security-audit/SKILL.md` directly inside its
top-level `security-audit` folder, without the repository or `skills` parent
directories. Upload this `security-audit.zip` through **Upload Skills**.

This changes the archive layout only; the audit's tool and sandbox requirements
in [Requirements](#requirements) still apply.

## Usage

Start your coding agent in (or pointed at) the codebase you want to audit, then ask it to do a security audit:

> **Auditing untrusted code?** A repository's own Git metadata is attacker-controlled. Prefer targets obtained with a fresh `git clone` over archives that ship a `.git` directory, and avoid starting the agent inside an untrusted target's working directory. The skill runs `git` against the target with its configuration neutralized for this reason (see "Universal execution safety" in `SKILL.md`).


```
security audit this codebase
```

```
find security vulnerabilities in ./src
```

```
do a security review, output to ~/audits/my-project
```

The skill activates automatically when the request matches its trigger (security audit, find vulnerabilities, pen-test the code, etc.). A direct codebase audit or pen-test request uses full audit mode. Security questions and focused vulnerability work use guidance mode unless you request report artifacts. In full audit mode, an unspecified output directory defaults to `~/security-audit-skill/<repo-id>/run-<N>`, where `<repo-id>` is a sanitized repo name plus a hash of the target path, so a target cannot steer the output path or reuse another target's prior runs. The workflow writes inside the target repository only when you explicitly select a directory that version control ignores.

## Requirements

- A coding agent with a model that supports tool use and parallel sub-agents. On Antigravity/Gemini, follow the delegation adapter in `SKILL.md` (`self` for hunters, four live children, authorized-review preamble).
- Node.js for the zero-dependency findings, coverage-ledger, and run-metadata validators
- An OS-enforced sandbox for target-controlled builds, tests, processes, browsers, emulators, fuzzers, and fixtures. It must disable external networking, use a sanitized allowlisted environment, enforce resource limits, and allow writes only to assigned scratch paths. Without these controls, the workflow keeps the lead as `needs_validation` instead of executing target code.

## Design principles

- **Only confirm established boundary failures.** Keep a source-grounded blocked lead as `needs_validation` with its exact unresolved fact.
- **Adversarial validation.** The agent that checks a finding is never the agent that found it.
- **Target content is data.** Comments, docs, and agent-directed text in the target never instruct the auditors or count as evidence that a control exists.
- **Severity requires impact.** Likelihood x impact, not deviation from a checklist.
- **Defense-in-depth gaps are not vulnerabilities.** If Layer A prevents the attack, the absence of Layer B is a hardening note.
- **Multiple runs improve coverage.** Cloudflare measured a single run finding only about half the bugs caught across multiple runs, with single-run findings skewing toward the simpler and less subtle ([Build your own vulnerability harness](https://blog.cloudflare.com/build-your-own-vulnerability-harness)). That measurement is theirs, not this fork's — no independent coverage study has been run here.

### Invoking it directly

The skill activates automatically when a request matches its trigger (security audit, find
vulnerabilities, pen-test the code). When several installed skills share those triggers and
another one wins, name this one explicitly instead:

```
/cloudflare-security-audit <prompt>
```

## Scope and limits

- **Known vulnerabilities in pinned dependencies are out of scope.** Nothing in the workflow fetches advisory or CVE data, and hunters run without external networking, so a disclosed vulnerability in a dependency version that postdates the model's knowledge will not be found. A known-vulnerable dependency is also not a finding by itself: it needs a demonstrated resolution, build, or execution boundary. Run a dependency scanner alongside this skill if that coverage matters.
- **Facts the audit cannot observe stay open.** A lead that depends on an external or deployment fact is recorded as `needs_validation` with that fact named, not guessed at.

## Versioning and releases

Releases are tagged `vMAJOR.MINOR.PATCH` and listed on the
[Releases page](https://github.com/iamrudhresh/cloudflare-security-audit-skill/releases).
`main` is always installable; pin a tag if you need a fixed version:

```bash
npx skills add https://github.com/iamrudhresh/cloudflare-security-audit-skill   --skill security-audit
```

What a version bump means here:

| Bump | Meaning |
|------|---------|
| **Major** | A breaking change to `report-schema.json`, a validator CLI contract, or the workflow's phase contract |
| **Minor** | A new attack class, companion guide, validator, or workflow rule that is backward compatible |
| **Patch** | Bug fixes, documentation, and test-only changes |

Because the skill's output is consumed by tooling, any change that can invalidate an existing
`findings.json` or `coverage-ledger.json` is a major bump. See [CHANGELOG.md](CHANGELOG.md).

## Contributing

Contributions are welcome — this fork exists because the backlog needed attention, and it
only stays useful if that keeps happening.

Good first contributions:

- **A new attack class or hunting pattern** in a companion guide. The highest-value kind of
  contribution, and it needs no code.
- **A validator bug**, ideally with a failing test first.
- **A platform problem.** The validators are tested on Linux, Windows and macOS; breakage on
  any of them is a real bug.
- **Documentation** that was wrong or unclear when you tried to use it.

Read [CONTRIBUTING.md](CONTRIBUTING.md) for the branching model, commit conventions, how to run
the test suites, and what a reviewable pull request looks like. Participation is governed by our
[Code of Conduct](CODE_OF_CONDUCT.md).

To report a vulnerability **in this skill**, follow [SECURITY.md](SECURITY.md) — please do not
open a public issue for it.

## Contact

Open an [issue](https://github.com/iamrudhresh/cloudflare-security-audit-skill/issues) or start a
[discussion](https://github.com/iamrudhresh/cloudflare-security-audit-skill/discussions).

Questions for the **upstream** project, or notes on Cloudflare's own AI-driven security tooling,
belong with Cloudflare at security-ai-research@cloudflare.com rather than here.

## License

MIT -- see [LICENSE](LICENSE).

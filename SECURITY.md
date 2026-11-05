# Security Policy

## Scope

This policy covers vulnerabilities **in this skill itself** — the validators, the workflow
instructions, and the execution-safety rules. Examples of what belongs here:

- A way to make a validator read, open, or write a file it is supposed to refuse — following a
  symlink, a reserved device name, a path that escapes its root, or a file swapped between the
  check and the open.
- A way to make a validator accept a `findings.json` or `coverage-ledger.json` that should fail,
  so an audit can claim coverage or evidence it does not have.
- A denial-of-service in a validator: unbounded memory or runtime from a crafted input.
- Workflow instructions that cause an agent to execute target-controlled code outside the
  sandbox, leak credentials, or damage the host — for example the host-profile deletion that
  prompted the isolation gate.
- A prompt-injection path where target content changes an agent's verdict, scope, or execution
  boundary.

### Out of scope

- **Vulnerabilities the skill finds in your own code.** Those are yours to triage and fix. This
  project is a tool, not a disclosure venue.
- **Findings the skill missed.** A coverage gap is a bug or a feature request — open a normal
  issue. It is not a vulnerability in the skill.
- **Vulnerabilities in the upstream project** that are not present here. Report those to
  [cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill). If an
  issue affects both, tell us and we will coordinate.
- Model behaviour you dislike, or your coding agent's own bugs.

## Reporting

**Do not open a public issue for a vulnerability in this skill.**

Use GitHub's private vulnerability reporting:

**[Report a vulnerability](https://github.com/iamrudhresh/cloudflare-security-audit-skill/security/advisories/new)**

That creates a private advisory only maintainers can see. If it is unavailable, open a public
issue containing *only* the sentence "requesting a private channel for a security report" and
nothing about the issue itself.

### What to include

- What an attacker achieves, and who the attacker is — a hostile audit target, a malicious
  subagent result, a local user.
- The smallest input that reproduces it. A crafted JSON file, a filesystem layout, or a path is
  usually enough.
- The commit you tested, and your OS and Node version. Several past bugs were Windows-only.
- Whether you are disclosing publicly on a timeline.

A proof of concept is welcome. Please keep it to the minimum that demonstrates the problem, and
do not include a working exploit against third-party software.

## What to expect

| | |
|---|---|
| Acknowledgement | within **5 days** |
| Initial assessment | within **14 days** |
| Fix or a stated decision not to fix | depends on severity; you will get a schedule, not silence |

This is a volunteer-maintained project, so these are good-faith targets rather than guarantees.
If a deadline passes without a reply, escalate by commenting on a public issue asking for a
response on your private report — without describing it.

On a fix, you will be credited in the advisory and the changelog unless you ask not to be.
Coordinated disclosure is preferred, but it is your finding: if you have a disclosure deadline,
say so up front and we will work to it.

## Supported versions

Only the latest release and `main` receive fixes. There are no backports to older tags — the
skill is a set of files with no build, so upgrading is replacing them.

## A note on the skill's own threat model

This skill reads attacker-controlled input by design: the audit target may be hostile, and its
source, comments, documentation and agent-directed text all reach the agents and the validators.
The project treats that as a first-class threat, which is why there is an isolation gate, an
untrusted-content block in every delegated prompt, and a validator input path that refuses
symlinks and verifies file identity across the open.

If you find a hole in any of that, it is exactly the kind of report this policy is for.

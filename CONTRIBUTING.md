# Contributing

Thanks for helping. This project is a community continuation of
[cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill), and it
exists because a contributed backlog went unreviewed. Keeping that from happening again is the
main job, so this guide is mostly about making a change easy to review.

By participating you agree to the [Code of Conduct](CODE_OF_CONDUCT.md).

---

## What this project is

A coding-agent **skill**: Markdown instruction files plus zero-dependency Node validators. There
is no build step, no external dependency, and nothing is deployed. Users install it straight from
the default branch or a tag.

Two consequences shape everything below:

1. **`main` must always be installable.** It is what users get. A broken `main` is a broken
   release.
2. **The validators are a contract.** `findings.json` and `coverage-ledger.json` are consumed by
   other tooling, so a change that can invalidate an existing file is a breaking change.

---

## Branching model

Short-lived branches off `main`, merged by pull request. No `dev` or `staging` branch — there is
no deployable artifact for them to gate, and a second integration branch would only collect
conflicts. Tags are the release mechanism.

```
main ────●────────●────────●────────●───▶   protected, always installable
          \      /          \      /
           ●────●            ●────●         short-lived branches
         feat/…              fix/…
                        tags: v1.0.0, v1.1.0  ← what users pin
```

Branch names:

| Prefix | For |
|--------|-----|
| `feat/` | A new capability — attack class, companion guide, validator, workflow rule |
| `fix/` | A bug fix |
| `docs/` | Documentation only |
| `test/` | Tests only |
| `chore/` | CI, tooling, repository housekeeping |
| `refactor/` | Behaviour-preserving restructuring |

Example: `fix/windows-reserved-device-names`.

`main` is protected: pull request required, all CI checks green, linear history, no force pushes.

---

## Setting up

Clone it and run the tests. That is the whole setup — there is nothing to install.

```bash
git clone https://github.com/iamrudhresh/cloudflare-security-audit-skill.git
cd cloudflare-security-audit-skill/skills/security-audit
node --test
```

You need Node.js 20 or newer. `node --test` discovers every `*.test.cjs` suite.

Expect some skipped tests: suites guard platform-specific cases, so a Windows-only test skips on
Linux and vice versa. **Zero failures is the bar; skips are normal.**

Run a single suite while iterating:

```bash
node --test validate-findings.test.cjs
```

Try a validator by hand:

```bash
echo '[]' > /tmp/findings.json
node validate-findings.cjs /tmp/findings.json
```

---

## Making a change

### Changing a companion guide or attack class

The most valuable contribution, and it needs no code.

- Put the class in the companion that matches the **trust boundary**, not the technology.
- Match the house style of the blocks around it: a bold title, a declarative description of how
  the boundary fails, then a `Confirm …` directive naming what establishes it. Do not write in
  second person or hedge; the text goes into an agent prompt verbatim.
- If the class belongs to a new domain, add it to the selector in `ATTACK-CLASSES.md` and link it
  from `SKILL.md` — a companion reachable only through another companion may not be read.
- Cite a real CVE or advisory in the pull request where you can. It makes the class reviewable.

### Changing a validator

- **Write the failing test first.** A validator change without a test that fails before it is not
  reviewable.
- Keep them zero-dependency. Node builtins only; no `package.json`.
- Keep the two main validators' shared helpers identical. A test asserts this, because they drifted
  before.
- Input handling is security-sensitive: the validators read files that a target can influence. If
  you touch `readFileWithinLimit` or `openWindowsInput`, preserve all of it — no symlink following,
  no non-regular files, no reserved Windows device names, the size limit enforced on the open
  descriptor, and a file-identity re-check after opening so a swap between `lstat` and `open` fails
  closed. **A filesystem that cannot supply a file identity must fail closed, not pass.**

### Changing the schema

`report-schema.json` is a published contract.

- Adding an **optional** field is a minor bump.
- Making a field **required**, removing one, or narrowing a type is a **major** bump, because it
  invalidates existing `findings.json` files. Say so in the pull request and expect to discuss it.

---

## Commits

[Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<optional scope>): <imperative summary>

<why the change is needed, and anything non-obvious about how>
```

Types: `feat`, `fix`, `docs`, `test`, `chore`, `refactor`, `ci`.

```
fix: refuse bare reserved Windows device names

path.resolve("NUL") yields an ordinary-looking drive path, so the \\.\ prefix
guard never fired. The input path now runs through WINDOWS_RESERVED_COMPONENT.
```

Explain **why** in the body. The diff already shows what.

---

## Pull requests

Open an issue first for anything beyond a bug fix or a documentation correction, so nobody writes
a change that gets rejected on direction.

A reviewable pull request:

- **Does one thing.** Three unrelated fixes in one branch means it is blocked until all three are
  agreed. Split them.
- **States the problem before the solution.** What breaks today, and how do you know?
- **Includes a test** for a behaviour change, failing before and passing after.
- **Is green on all three platforms.** CI runs Linux, Windows and macOS.
- **Says if it is breaking**, and what existing files it invalidates.

If your change supersedes an existing pull request — upstream or here — say so and explain why
yours should win. That comparison is the single most useful thing a reviewer can be handed.

### Review

A maintainer will either merge, request changes with reasons, or explain why it will not land.
If a pull request has had no response in **14 days**, comment on it — that is a maintainer
failure, not an imposition on your part.

---

## Reporting a vulnerability

To report a vulnerability **in this skill** (for example, a way to make a validator read a file
it should refuse), follow [SECURITY.md](SECURITY.md). Do not open a public issue for it.

Vulnerabilities the skill *finds* in your own code are yours to handle — this project is a tool,
not a disclosure venue.

---

## Attribution and licence

Contributions are MIT, matching the project. The upstream audit methodology is
copyright Cloudflare, Inc.; keep the existing copyright notice in `LICENSE` intact. Do not add
your name to `LICENSE` — git history is the contribution record.

If you are porting work from an upstream pull request, credit it by number in the commit body so
the lineage stays traceable.

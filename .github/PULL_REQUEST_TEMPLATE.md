<!--
Thanks for contributing. Keep a pull request to one thing — three unrelated fixes in one
branch stay blocked until all three are agreed.

Reporting a vulnerability in this skill? Do not open a pull request. See SECURITY.md.
-->

## The problem

<!-- What breaks or is missing today, and how you know. Link the issue: Fixes #123 -->

## The change

<!-- What you did. The diff shows the what; use this space for anything non-obvious about how. -->

## Breaking change?

<!-- Pick one and delete the rest. -->

- [ ] No — additive or internal only
- [ ] Yes — changes `report-schema.json` (invalidates existing `findings.json`)
- [ ] Yes — changes a validator CLI contract
- [ ] Yes — changes the workflow phase contract

<!-- If yes, say exactly what existing files or callers stop working. -->

## Supersedes

<!--
If this replaces another pull request, here or upstream, list it and say why yours should win.
That comparison is the most useful thing you can hand a reviewer. Otherwise delete this section.
-->

## Checklist

- [ ] One logical change
- [ ] `node --test` passes from `skills/security-audit` (skipped platform tests are normal; zero failures is the bar)
- [ ] A behaviour change has a test that fails before this change and passes after
- [ ] A validator change preserves the input-safety guarantees: no symlink following, no non-regular files, no reserved Windows device names, size limit enforced on the open descriptor, file-identity re-checked after opening, and a missing file identity fails closed
- [ ] A new attack class matches the house style of the blocks around it and says what is *not* a finding
- [ ] A new companion guide is linked from `SKILL.md`, not only from another companion
- [ ] Commits follow Conventional Commits and explain *why* in the body
- [ ] No new dependencies (the validators are deliberately zero-dependency, with no `package.json`)
- [ ] `LICENSE` is unchanged

## Testing notes

<!--
Which platforms you ran on. CI covers Linux, Windows and macOS, but say what you verified
yourself — several past bugs reproduced only on Windows.
-->

# astra-registry-canary-4

**A test fixture. Nothing here is a plugin for users, and nothing here is
signed with a key any shipped Astra build trusts.** This repository serves one
rehearsal document set and keeps it fresh, and nothing else. It holds no
secret and no deploy key.

## What it is for

The [Astra plugin registry](https://github.com/mihailinl/astra-registry)'s R2
exit (contract ROLL-60) needs the plugins service's publisher to serve one
rehearsal step end to end. The service runs a build that compiles the
registry's throwaway `tools/testkeys` roots: current {`root-a`, `root-b`},
next {}. The step is step 0 (`rotation/00-baseline`) of a key-rotation
series, read from a branch named `signed`.

The publisher refuses a withdrawal list whose `expires_at` passes
`min(issued_at, judged_at) + 8 days`. So a list dated after the day it is
read is refused, and so is a longer one. A step 0 the service may meet on any
day has to be fresh on that day. That is what production's signer does with
its own documents: it re-signs them at unchanged serials once they are 20
hours old. This repository does the same with TEST keys.

| Here | What it is |
|---|---|
| branch `signed` | step 0 of the registry's `tools/testkeys/fixtures/rehearsal-r2d/` (T0 2026-10-03), the exact commit the registry's signer made, and after it one commit per re-sign |
| `.github/workflows/rehearsal-resign.yml` | every hour, the registry's real signer, fetched at a pinned commit, re-signs step 0's catalogue and list once they are 20 hours old: same serials, fresh `issued_at` and `expires_at`. It pushes that one commit, fast-forward, and waits until Pages serves it |
| Pages | serves branch `signed` (legacy build, root), so `https://mihailinl.github.io/astra-registry-canary-4/registry/v1/*.json` is always a commit the branch carries |
| `main`'s merge of `refs/rehearsal-source/*` | the fixture generator's throwaway registry history for this cut, merged with `-s ours`. It holds the commit every `Source-Commit` names, so the service's TRUST-3 holds. `main`'s tree is unchanged by it |

## The rules

- **Everything on `signed` is TEST-ONLY.** It is signed by keys whose private
  halves are public in the registry. No shipped Astra build trusts them.
- **Two things push to `signed`, and nothing else does.** The registry's
  `tools/testkeys/rehearsal-push.mjs --step 0` pushed step 0, once. The
  workflow pushes each re-sign. Neither ever forces. A ruleset refuses
  deletion and non-fast-forward on `main`, `signed` and `signed-compromise`.
- **The workflow cannot reach the production registry.** Its token may write
  this repository's contents and ask for a Pages build, and nothing else. It
  reads no secret, and its job runs only in this repository. Its first step
  checks that the file is the registry's reviewed template
  (`tools/testkeys/rehearsal-resign.yml`), pinned to the registry commit it
  fetches.
- The runbook is astra-plugins-ops `runbooks/roll-60-rehearsal.md`.

## Licence

GPL-3.0-or-later (see `LICENSE`), like the rest of the registry. Copyright
Minice.

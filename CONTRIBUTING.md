# Contributing

Thanks for considering it. This is a small project run by a small team, so
the bar for a good contribution is low effort for you and honest about
trade-offs — not perfect prose or a huge PR.

## Ground rules, non-negotiable

- **Zero runtime dependencies stays zero.** Don't add an npm package to
  `dependencies`. If a check genuinely needs something Node's built-ins
  (`dns/promises`, `tls`, `https`/`fetch`) can't do, that's a conversation to
  have in an issue first, not a PR to send straight away. `devDependencies`
  for tooling (linters, etc.) can be discussed but currently there are none —
  keep it that way unless there's a real payoff.
- **Passive and self-target only.** No port scanning, no directory
  brute-forcing, no active probing beyond a DNS/CT-log/TLS-handshake lookup
  — the same category of interaction a browser makes automatically loading a
  page. Any change that turns this into an active scanner will be rejected,
  full stop.
- **No telemetry.** The tool never phones home beyond the lookups it exists
  to make against the domain you give it. Keep it that way.

## Code layout

- `bin/perimeterlens.js` — thin CLI entrypoint: reads argv, calls the checks,
  prints the report. Argument parsing itself lives in `src/args.js` (kept
  separate so it's testable without spawning a subprocess).
- `src/checks/*.js` — one file per check (`subdomains.js`, `email.js`,
  `tls.js`). Each is a pure-ish async function: takes a domain, returns a
  plain-object result, does its own I/O. No shared state between checks.
- `src/score.js` — turns check results into the composite 0–100 score and
  letter grade.
- `src/report.js` — turns check results + score into the printed
  markdown/JSON report. No I/O.
- `test/*.test.js` — one test file per module above, using Node's built-in
  `node:test` + `node:assert`. No test framework dependency, on purpose.

## Adding a new check

1. Add `src/checks/yourcheck.js` following the shape of an existing check
   (async function, domain in, plain object out, own error handling — a
   failed lookup should degrade to a clear "could not complete: <reason>" in
   the result, not throw and crash the whole scan).
2. Wire it into whatever calls the checks today and into `src/score.js` if
   it should affect the composite score.
3. Add `test/yourcheck.test.js` covering at least: the happy path, and the
   lookup-failed path.
4. Update the README's feature list and "What this is *not*" section if the
   new check changes either.

If you're not sure a check belongs in the tool at all (i.e. it's outside
the passive/self-target/zero-dependency lines above), open an issue first —
it'll save you writing code that gets rejected on scope grounds, not quality
grounds.

## Running things locally

```bash
git clone https://github.com/NicklasSandin/perimeterlens.git
cd perimeterlens
node bin/perimeterlens.js yourdomain.com   # only scan a domain you own or are authorized to test
npm test                                    # node --test test/*.test.js
```

No install step — that's the point of zero dependencies. CI (`.github/workflows/ci.yml`)
runs the test suite plus a real smoke-test scan of `example.com` on Node 18,
20, and 22, and asserts `node_modules` never exists.

## Bugs and feature requests

Open a GitHub issue. If you want scheduled/continuous monitoring instead of
a one-shot scan, use the `monitor: yourdomain.com` issue template instead of
describing it from scratch — see the README's "Continuous monitoring
interest" section for why.

## Pull requests

- Keep them scoped to one check or one fix. Large PRs mixing several
  concerns are harder to review and more likely to sit.
- Tests are required for behavior changes, not optional polish.
- If in doubt about scope (see "Ground rules" above), open an issue before
  writing code.

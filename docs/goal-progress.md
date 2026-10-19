# Active Goal Checkpoint

Updated 2026-10-19 after the local review and regression pass. Resume the full
SimFarm engineering mission from this worktree; preserve all edits and keep the
pull request unmerged.

## Git state and preservation

- Repository: `/workspaces/SimFarm_V0`; branch:
  `codex/simfarm-improvements`.
- HEAD: `f7f62eb` — `Add zoom, crop insurance, and conservation mechanics`.
  It contains the implementation, generated bundle, and regression tests.
- `origin/codex/simfarm-improvements` remains at `0a36587`; upstream `main` is
  `3fec6a6`. The code commit is not pushed. Documentation changes and this
  checkpoint remain uncommitted; nothing is staged.
- PR #1 is open at https://github.com/worldengine-space/SimFarm_V0/pull/1,
  base `main`, head `codex/simfarm-improvements` at `0a36587`; GitHub lists no
  checks and reports `UNSTABLE`. Earlier workflows were `action_required`; the
  workflow-approval endpoint returned HTTP 403 `Resource not accessible by
  integration`. Recheck after pushing.
- Remaining tracked changes are `README.md`, `docs/PLATFORMS.md`,
  `docs/TECHNICAL-WHITEPAPER.md`, Android README, and iOS README, plus the one
  unrelated Android test edit. `docs/goal-progress.md` is untracked.
- Preserve and exclude `platforms/android/app/src/androidTest/java/space/worldengine/simfarm/LaunchSmokeTest.java`
  from commits. Its existing unrelated change adds a `∏` character after the
  Activity import. No other unrelated edits have been identified.
- No applicable `AGENTS.md` was found. `game.js` is generated from the ordered
  source sections.

## Completed work and findings

- Existing source, tests, platform setup, workflows, and history were reviewed.
  A clean extraction of pre-change HEAD passed its original nine-suite
  `npm test` baseline.
- Research is documented in `docs/TECHNICAL-WHITEPAPER.md`: Internet Archive
  metadata and the 1993 German handbook/quick-start card; a separate 2014 C#
  classroom project (not reused); USDA FSA CRP material used only for broad
  context; and REA/Ghidra methods, provenance, findings, confidence, and limits.
- REA `rea-agents@6.7.0` and Ghidra 12.1.4 ran from `/tmp`. The verified DOS MZ
  x86 artifact was statically analyzed and never executed or committed. The
  analysis established startup/format observations, not gameplay algorithms;
  the unexplained appended bytes and automated decompilation limits are
  documented. Ghidra/JDK/provider checks passed; unrelated REA integrations
  caused the overall doctor status to remain false.
- Corrected stored-crop sale accounting so cash, crop-sales, and taxable-sales
  deltas agree; regression assertions cover the transaction.
- Added five-level camera zoom (50%, 75%, 100%, 150%, 200%) as a usability
  improvement, not a historical claim. Wheel, `+`/`-`, and View menu controls
  share render/hit-test transforms, pointer anchoring, and map clamps. The Map
  overview frame, event sprites, and viewport clipping adapt to zoom.
- Added optional fictional crop insurance with captured price, 5% premium,
  20% deductible, 50% expected-value cap, one settlement at harvest/flood, and
  cancellation on other field-removal paths.
- Added optional fictional conservation contracts on one owned vacant 8×8
  parcel: three annual $100 payments, three-year term, land-use restrictions,
  withdrawal, and visible green map tint. Imported contracts are validated
  against ownership, vacancy, and save year.
- Added versioned `.SFMX` persistence for the optional mechanics while native
  `.SFM`/`.SSM` behavior receives safe defaults. Browser, Android, and iOS file
  paths were updated; iOS native compilation remains unverified here.
- README, platform notes, whitepaper, test instructions, and this checkpoint
  describe features, compatibility, historical uncertainties, and limitations.

## Validation

After the latest code changes, these commands passed on 2026-10-19:

- `npm run build`
- `npm test` — all 10 suites passed. Coverage includes eight native fixtures,
  accounting, insurance and contract edges, zoom anchoring/bounds/repeated
  cycles/CSS resizing, and a real Canvas2D pixel-level conservation tint check.
- `npm run format:check`
- `npm run build:web` — generated the 425-file, 56.7 MiB offline package.
- `git diff --check`

The web build and Node/Canvas harness ran, but no local Chromium/Firefox or
Playwright/Puppeteer is installed; interactive browser/device testing was not
performed. Android SDK is not configured. Swift, Xcode, and XcodeGen are
unavailable, so iOS native compilation needs CI or macOS. Java and Gradle are
present. GitHub CI has not yet been triggered for this worktree because it is
not pushed.

## Remaining lifecycle

1. Final-audit and commit the remaining README, platform notes, whitepaper, and
   this checkpoint. Continue excluding the unrelated Android test edit.
2. Push the branch to `origin/codex/simfarm-improvements` and update PR #1 with
   methods, findings, features, compatibility, executed checks, CI status, and
   limitations.
3. Recheck PR head/base and all CI results. Fix implementation failures where
   feasible; leave the PR unmerged.
4. Audit the full mission criteria before marking the Goal complete. Additional
   historical save fixtures and interactive desktop/touch play remain useful
   follow-up validation, beyond the eight included fixtures and current
   headless evidence.

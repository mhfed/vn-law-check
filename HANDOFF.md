# Scanner URL and consent fixes

## Goal and contract

Fix the three scanner bugs approved on 2026-11-20: reject links whose
authority only starts with the requested origin, resolve relative page and
script links against their containing page, and ignore blocked tracking
scripts regardless of attribute order. Keep Bash 3.2, current CLI/output,
download limits and existing runtime dependencies. No legal-content changes.

## Plan

1. Add failing local HTTP tests in `tests/test_fetch_site.py`, wire them into
   `tests/run.sh`, and add consent fixtures in `tests/fixtures/`.
2. Fix `scripts/fetch-site.sh` and `scripts/scan.sh`; use each downloaded
   page's URL when resolving its scripts and inspect complete opening tags.
3. Run `bash tools/sync.sh`, all CI checks and one bounded independent
   review. Publish a reviewable PR without merging.

Acceptance gate: regressions fail on the original scripts and pass on the
fix; `bash tests/run.sh`, `bash tools/sync.sh --check`, shell syntax and
the CI YAML/JSON checks pass.

## Status and verification

- Base: `964f519`, branch `fix/scanner-url-consent`, isolated clone.
- Baseline: `bash tests/run.sh` PASS (12 fixtures); `bash tools/sync.sh --check` PASS.
- Changed: `scripts/fetch-site.sh`, `scripts/scan.sh`, new
  `scripts/tracking-preload.awk`, their generated plugin copies,
  `tests/run.sh`, `tests/test_fetch_site.py`, five `consent-*` fixture dirs,
  and this handoff.
- RED: original scripts failed the blocked-after-src fixture and origin,
  relative-page and relative-script HTTP regressions for the expected reasons.
- GREEN: `bash tests/run.sh` PASS (17 fixtures + 8 HTTP tests), Bash 3.2.
  HTTP tests use real curl and isolated localhost servers; hostname tests
  use curl connect-to mappings, without contacting external sites.
- `bash -n scripts/scan.sh scripts/fetch-site.sh tests/run.sh` PASS.
- `bash tools/sync.sh --check` PASS; `git diff --check` PASS.
- `python3 -c 'import json; [json.load(open(f)) for f in
  [".claude-plugin/marketplace.json", "plugins/vn-law-check/.claude-plugin/plugin.json"]]'` PASS.
- GitHub Actions `check` run 44659724461 PASS on implementation commit
  `ca34cd5`: sync, full suite, SKILL YAML frontmatter and plugin JSON.
  Local YAML check unavailable (PyYAML absent; host pip macOS detection fails);
  the exact required YAML check passed in GitHub CI.
- Independent read-only review found canonical-entry redirects and multiline
  script tags regressed. Both fixed in one pass, with RED -> GREEN tests.
  Script evidence retains original filename and line (manually checked).
- Decision: use the effective entry-page origin after canonical redirects;
  use POSIX awk for complete multiline opening tags, without new runtime deps.
- Existing limitations: external redirects remain followed by curl; parsing
  is heuristic (no browser execution or general HTML-parser guarantee).
  Legal content and legal outcomes were not reviewed in this phase.
- Status: complete; PR https://github.com/mhfed/vn-law-check/pull/16 merged by
  maintainer as `5dae122`; implementation and main CI passed.
- Rework cycles: 1 (review found two missing boundary cases; fixed together).
- Environment retries (not code rework): localhost sandbox access, PyYAML setup.
- Actual token usage: unavailable.

## Release v1.7.1 phase (2026-11-20)

- Goal: publish the approved scanner fixes as v1.7.1. The previous public
  release/tag is v1.5.3; plugin metadata was 1.7.0.
- Contract: bump only plugin version metadata; release notes also explain
  the v1.6/v1.7 features already present on main since the last public release.
  No scanner or legal-content changes in this phase.
- Base: `5dae122`, branch `release/v1.7.1`.
- Changed paths: `plugins/vn-law-check/.claude-plugin/plugin.json`, `HANDOFF.md`.
- Plan: validate metadata/sync, pass existing CI on release PR, merge the
  version update and publish v1.7.1 targeting that exact main commit.
- Acceptance gate: plugin JSON reports 1.7.1, sync and existing CI pass;
  GitHub release is public/latest and tag resolves to the versioned commit.
- Verification: metadata/sync and CI pending; publication pending.
- Risks: none introduced by the metadata change; existing scanner limitations
  remain documented in the previous phase.
- Next action: run checks and open the release PR.
- Rework cycles: 0. Actual token usage: unavailable.

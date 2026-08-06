# Multi-model adversarial review scorecard — c3d-marketplace-public

One narrative row per review cycle. Finding-level data lives in
`model-review-findings.csv`. Methodology matches the c3d-cli-tool scorecard
(same five-leg matrix, same disposition vocabulary).

## Cycle 1 — PR #6, docs gap fixes (2026-08-06)

<!-- markdownlint-disable MD013 -->

Target: the six parity-audit doc corrections (diff ~30KB, both skill copies).
Brief carried a "Verified facts" section listing every live-proven wire fact so
legs would judge internal consistency rather than re-derive.

- **Legs**: sonnet (`claude-sonnet-5`, Agent tool, session effort high),
  codex (`gpt-5.6-sol`, `model_reasoning_effort=high`), gemini
  (`gemini-3.1-pro-preview-customtools`, two chunks 19.5KB/22KB, both verified
  non-empty), kiro (`gpt-5.6-sol`, `--effort high`, `fs_read`), orchestrator
  (`claude-fable-5`, effort high; session historically mixed with
  `claude-opus-5`, this leg ran on fable-5).
- **Convergences**: `device-star-free-form-strings` found by 4/5 legs
  (all except gemini) — the strongest signal of the cycle, and a genuine
  defect in the diff's own added text. `stepresults-unconditionally` found by
  3/5 (gemini, kiro, sonnet).
- **Sole-finders**: codex 2 (`quoted-projectid-request-templates` — the
  cycle's best finding, live-verified with a 400 during adjudication —
  and `both-bounds-invented-gte`), kiro 3 (`org-endpoint-nonexistence`,
  `full-vocabulary-overpromise`, `fps-score-historical`), sonnet 1
  (`stale-returns-line`), orchestrator 1 (`roomsize-in-legacynames`).
- **Refutations**: orchestrator self-killed its own
  `empty-sessionfilters-claim-unverified` by probing the live API (200 with
  counts; bare array still 400) — the doc claim stood. No external-leg
  findings were refuted this cycle; all nine external findings survived at
  confirmed or partial.
- **Empirical adjudication**: three live probes settled questions reading
  could not — string projectId 400s on `paginatedListQueries` (extending the
  slicer-only evidence), string projectId is ACCEPTED on
  `singleProjectSessionQueries` (which stopped a wrong "fix" to those
  templates), and the empty `sessionFilters` wrapper works.
- **Leg quality notes**: kiro was the volume leader (5 findings, 3 sole) at
  high precision — a marked improvement over its c3d-cli-tool cycles; the
  docs-with-fs_read format suits it. Gemini's part-1 clean pass came with
  substantive reasoning (a real pass, not a dead leg) and its part-2 finding
  converged. Codex from a plain diff still found the cross-file template
  contradiction, countering its documented plain-diff blind spot — likely
  because the brief pointed at the full checkout. This was a DOCS diff cycle;
  leg-performance comparisons with code cycles should be made cautiously.

Attribution caveat: model ids above were pinned at invocation (codex/kiro on
the CLI flag, gemini its default recorded in `~/.gemini/tmp` session logs,
sonnet via the Agent tool alias — alias only, resolved version not recorded).
Kiro effort is unrecoverable after the fact and was recorded at invocation
time per protocol.

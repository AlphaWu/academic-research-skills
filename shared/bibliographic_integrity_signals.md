# Bibliographic Integrity Signal Contract

`shared/contracts/passport/bibliographic_integrity_signal.schema.json` is the
single schema authority for bibliographic-integrity observations carried in
`literature_corpus[].bibliographic_integrity_signals[]`. Version 1.0 is
additive: it changes neither the existing citation-finalizer policy nor the
one-advisory-token reference-marker grammar.

## Epistemic boundary

The carrier separates three claims that must not be collapsed:

| `epistemic_class` | Required label | What it establishes |
|---|---|---|
| `deterministic_fact` | `RESOLVER-OR-LIST-OBSERVATION` | What a named resolver or list returned at a recorded time; not whether the cited work is ultimately genuine, retracted, or sound. |
| `heuristic_advisory` | `HEURISTIC-INDICATOR` | A rule or model matched; never a factual finding by itself. |
| `process_attestation` | `CHECK-EXECUTION-ATTESTATION` | A check was reportedly run; it is not the result of that check. |

`check_status` and `finding` are independent. `not_checked`, `unknown`, and
`degraded` require `finding: unresolved`; none may be rendered or interpreted
as clean. A legacy `retraction_check: true` therefore migrates only to a
checked execution attestation with an unresolved finding. It never becomes
`not_detected`.

## Display and policy ownership

Every v1 signal is rendered as one row in the single
`Bibliographic Integrity Advisories` section of `provenance_summary.md`.
Rows compose by `signal_id` in lexical order and show the signal type,
epistemic label, check status, finding, affected citation/claims, provenance,
and freshness. The formatter must render `not_checked`, `unknown`, and
`degraded` as **NOT CLEAN — UNRESOLVED**.

`display.marker_token` is always `null`. New retraction and tortured-phrase
signals do not mint another `<!--ref:...-->` advisory token. During migration,
the existing finalizer continues to derive its one recognized
`CONTAMINATED-*` token from the legacy `contamination_signals` fields. Thus
multiple new signals compose in the provenance summary without changing or
overflowing the marker grammar.

`terminal_policy` records eligibility and the policy owner; it does not enact
policy. Only the citation finalizer may evaluate an eligible signal. Version
1.0 migrations set `eligible: false`, `policy_key: null`, and
`current_effect: advisory_only`. Existing strict behavior continues to read
the legacy carrier until a separately reviewed policy migration explicitly
cuts it over. Adding a new signal type can therefore never silently promote a
warning to `HIGH-BLOCK`.

## Pinned migration and deprecation path

1. **Read compatibility:** consumers continue to read
   `contamination_signals`, `contamination_signal_omissions`,
   `contamination_signals_backfilled_at`, and source-object
   `retraction_check` exactly as before.
2. **Dual write:** updated producers append canonical v1 records while still
   writing the legacy fields required by current marker and terminal-policy
   behavior. `scripts/bibliographic_integrity_signals.py` defines the
   deterministic legacy-to-v1 projection.
3. **Display cutover:** the formatter renders the canonical array into the
   one provenance-summary section. It does not derive new marker tokens.
4. **Policy cutover (future, separately reviewed):** only after all active
   producers dual-write and equivalence fixtures pass may the finalizer read
   canonical records for an existing policy. That change must version this
   contract and its policy schema; it is not part of v1.0.
5. **Removal (future major version):** legacy fields may be removed only after
   one released compatibility window following policy cutover. Until then
   they are deprecated write targets, not invalid inputs.

`contamination_signal_omissions.<resolver>: api_degraded` migrates to a
resolver-specific record with `check_status: degraded` and
`finding: unresolved`. Absence of both a legacy result and omission remains
absence/unknown; it is never synthesized as a clean result. Existing
`provenance_summary.md` text is an output projection and is never parsed back
as evidence.

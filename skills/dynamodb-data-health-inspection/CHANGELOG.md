# Changelog

## 1.0.1

Corrections from live validation against a seeded DynamoDB table, run under the
least-privilege role the bundled CloudFormation template provisions.

- Fixed: a stale `ItemCount` of 0 (DynamoDB refreshes it only every ~6 hours) caused
  sampling to be skipped on tables that actually hold data. Emptiness must now be
  corroborated by `TableSizeBytes` and consumed write capacity before Phase B is
  skipped.
- Fixed: no rule covered Contributor Insights being enabled but returning zero
  contributors, so the hot-key dimension could render as assessed-and-healthy when
  nothing had been measured. Added HK-06 and a `NoData` status that forces the
  dimension to "not determinable".
- Documented the real Contributor Insights rule-name convention (`PKC`/`SKC`/`PKT`/
  `SKT`, four rules per target, `<table>-<index>` for GSIs) so contributors can be
  attributed to traffic versus throttling instead of guessed at.
- Documented the 1 MB `Scan` page cap, which binds before `Limit` once items exceed
  ~4 KB and makes the realized sample smaller than planned.
- Strengthened the sampling-bias warning with a measured case: where 40 % of items
  shared one partition key, a bounded sample measured its share at 0.3 %. A low
  sampled share is therefore not evidence of even distribution.
- Recorded validated accuracy of the cost estimate (1.00x) and the item-size
  approximation (0.2 % drift against billed capacity).

## 1.0.0

- Initial version
- Four diagnostic dimensions: hot key detection, item size analysis, TTL
  effectiveness evaluation, and GSI/LSI utilization assessment
- Two-phase execution: Phase A (control plane, CloudWatch, Contributor Insights) at
  zero data-plane cost, then Phase B bounded sampling behind an explicit consent gate
- Read-only throughout; sampling capped at 1,000 items by default (10,000 absolute),
  ≤ 4 Scan segments, eventually-consistent reads, with abort on cost overshoot or
  throttling
- Sampling refused by default against tables that are already throttling
- Value redaction: item byte sizes, attribute names, and TTL timestamps only;
  partition keys rendered as digests
- `minimal-exposure` projection mode for tables holding regulated or personal data
- 25 finding rules with verbatim templates and a confidence-downgrade rule for small
  samples

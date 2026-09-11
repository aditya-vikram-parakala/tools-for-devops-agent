# Changelog

## 1.0.5

Replaces a requirement the real agent twice declined to follow with a design that
matches how it actually behaves.

- **Phase A collection is now scope-aware.** 1.0.3 and 1.0.4 both instructed the agent
  to collect all six Phase A steps regardless of the question; on a targeted prompt
  ("which GSIs are unused?") it twice collected only what that dimension needed. A
  third escalation was the wrong move - measurement showed a *full review* prompt
  already produces complete collection (DescribeTimeToLive, DescribeContributorInsights
  for table and GSI, 8 GetInsightRuleReport calls across both targets, 5
  GetMetricData). Scoping work to the question is correct behavior, so the skill now
  defines two modes: targeted questions collect for the dimension asked and must list
  what was not assessed without rendering a dimensions matrix; full reviews collect
  everything and honor the Final Delivery Contract.
- The safety property is unchanged and now applies to both modes: never imply a
  dimension was assessed when it was not. "Not determinable" (asked, unavailable) and
  "not collected" (chose not to look) are distinct claims and must be labelled
  differently.

## 1.0.4

Corrections from a second real-agent run, which exposed a false High-severity finding.

- **Fixed a false "unused index" finding.** IX-01 recommends deleting a customer's
  index and requires 30 days of metric coverage. On a 4-hour-old table the agent read
  "consumed read over the last 30 days: 0" and fired IX-01 at High severity - it had
  conflated the width of the queried window with the amount of data actually in it.
  `metric_coverage_days` was never defined operationally. It is now
  `min(datapoints returned at a daily period, table age from CreationDateTime)`, and
  IX-01 carries an explicit guard routing to IX-02 below 30 days.
- Strengthened Phase A completeness from prose into a mandatory six-row checklist. The
  1.0.3 prose instruction did not hold: asked a GSI-specific question, the agent
  skipped DescribeTimeToLive, DescribeContributorInsights, and GetInsightRuleReport
  entirely, then offered to "run the full inspection" separately.

Verified working in the same run: exact rule-ID citation (1.0.3 fix), stale-metadata
detection, and the measured-zero-versus-missing-data distinction, which the agent
surfaced verbatim as "0 (measured zero, not missing data)".

## 1.0.3

Corrections from driving the real AWS DevOps Agent over the API against the live
validation table (skill uploaded to an Agent Space, prompted naturally, trajectory
inspected via ListJournalRecords).

- **Fixed a design flaw in Phase B.** The protocol asked the agent to compute per-item
  statistics over 1,000 items - a ~1 MB payload per page. The agent has no code
  execution and must aggregate in its own context; the platform's summarizer produced
  sub-batch counts disagreeing by 3x and the agent (correctly) refused to publish
  numbers it could not trust. Phase B is now two passes with different shapes: a
  projected pass (keys + TTL attribute only) that stays small enough to total reliably
  at n=1,000, and a separate full-item pass at n=100-200 for item size. Verified: the
  agent then produced correct quantified TTL percentages.
- Fixed the unknown-mean RCU upper bound, which halved the correct figure and
  under-predicted a measured 450 RCU as 256. A full 1 MB eventually-consistent page is
  128 RCU, with no further 0.5 factor.
- Reframed `minimal-exposure` as "run pass 1 only", since projection is now the
  default for the TTL/key pass rather than an exposure-only option.
- Added the 80-character limit on structured choice options, with exact short forms -
  the agent lost a turn to a validation error twice, both times by interpolating the
  table name into an option description.
- Phase A must now be collected in full even when the question targets one dimension,
  and a skipped collection step must be reported as "not collected in this run" rather
  than "not determinable". The agent had skipped `GetInsightRuleReport` as off-topic
  and then marked the hot-key dimension not determinable, which conflates choosing not
  to look with data being unavailable.
- Findings must cite the exact rule ID. The agent labelled a malformed-timestamp
  finding "TTL-01 style" when TTL-01 is a materially different diagnosis (TTL deleted
  nothing) from TTL-04 (timestamps malformed).

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
- Fixed: IX-04 (over-broad GSI projection) silently did not fire when
  `amplification_ratio` was null from stale `TableSizeBytes`, so a full projection on
  a large table went unreported. It now degrades to a Low finding that states why the
  ratio is unavailable. Added a general consistency rule: a threshold that cannot be
  evaluated is never a passing threshold.
- Recorded validated accuracy of the cost estimate (1.00x) and the item-size
  approximation (0.2 % drift against billed capacity).

Rule discrimination verified against the live table: with Contributor Insights
showing 16.7x traffic concentration but no key-range throttle events, HK-02
(confirmed hot partition) correctly stayed suppressed and HK-03 (concentration
without throttling) fired instead.

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

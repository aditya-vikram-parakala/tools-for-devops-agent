# Changelog

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

# CloudGuard V2 — P1 corrections delivery

## Scope
Implemented only the approved P1 corrections in a separate copy of CloudGuard V2. The original CloudGuard project and both uploaded ZIP archives were not modified. No real AWS scan was run, and no real AWS credentials were used.

## Changed files
- `README.md`
- `cloudguard/core/export.py`
- `cloudguard/core/rules.py`
- `cloudguard/scanners/s3_scanner.py`
- `evaluation/evaluate.py`
- `evaluation/ground_truth_template.csv`
- `tests/test_app.py`
- `tests/test_p1_regressions.py`
- `tests/test_v2_integration.py`

## Test results

Python: 3.13.5

Compilation command:

```bash
python -m compileall -q app.py cloudguard evaluation tests
```

Result: PASS (no compilation errors).

Offline test command:

```bash
AWS_EC2_METADATA_DISABLED=true AWS_ACCESS_KEY_ID=offline-test AWS_SECRET_ACCESS_KEY=offline-test AWS_SESSION_TOKEN=offline-test AWS_DEFAULT_REGION=us-east-1 AWS_PROFILE= python -m pytest -q -rs
```

Result: **77 passed, 2 skipped**.

Skipped tests:
- `tests/test_app.py`: Flask route tests skipped because Flask is not installed in the runtime.
- `tests/test_moto_smoke.py`: Moto smoke test skipped because Moto is not installed in the runtime.

Installing these missing packages was attempted, but package index access was unavailable (DNS/network failure). The test suite itself remained offline and used dummy AWS environment values; AWS metadata lookup was disabled.

## Implemented corrections

- S3 account- and bucket-level Block Public Access settings now resolve to `enabled`, `disabled`, or `unknown` per flag. `AccessDenied` and other retrieval failures are recorded separately by scope and operation. Public policy/ACL results are marked inconclusive (`INFO`) when the relevant effective protection state is unknown, not as confirmed public exposure. Account-level errors are retained if a later bucket-list request fails. Results now distinguish discovered, attempted, and successfully inspected bucket counts.
- Rule metadata now adds rule ID/category and fills default severity/recommendation only when the scanner did not provide them. LOW and MEDIUM contextual S3 severities are preserved.
- Evaluation now matches exact `resource_type`, `resource_id`, and `rule_id`; validates PASS/FAIL labels, non-empty ground-truth identity fields, and duplicate ground-truth records. The CSV export includes resource type and exact identifier while retaining the legacy `resource` column. Unmapped findings remain unlabelled rather than being treated as a match or causing evaluation to fail.
- The sample ground-truth template now includes only rule checks currently implemented and mapped by the scanners. It removes the unsupported CloudTrail no-trail and EBS encryption-by-default evaluation cases.

## Remaining limitations

- Flask route tests and Moto smoke tests remain unexecuted because their packages were unavailable and could not be installed offline.
- No live AWS integration test was performed by design. IAM permission behaviour, actual AWS API response variations, and cross-region account coverage still require a later explicitly authorised integration test.
- The ground-truth resource IDs are illustrative controlled-test examples; replace them with the exact IDs/names of your own test resources before using the evaluation template for research metrics.

## Package integrity

The corrected archive was reopened and ZIP integrity-checked successfully.

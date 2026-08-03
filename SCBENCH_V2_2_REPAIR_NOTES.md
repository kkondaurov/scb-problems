# SCBench v2.2 narrow repair notes

## Status and boundary

Catalog `v1.0.2` is based on catalog `v1.0.1` at commit
`9b4864d6bdefd8cd0f2d66d3eb0d1972914aefd3`. Catalog v1.0.1 already contains
the accepted database-mutability, Dynamic Config bootstrap, unknown EVE
structure, and contradictory Trajectory-fixture repairs.

The review used the cross-artifact questions described by
[BenchGuard](https://arxiv.org/abs/2604.24955)—compare instructions,
evaluators, references, runtime assumptions, and observed executions—but did
not treat that paper or mutant generation as authority to expand a problem.
The retained delta is limited to a contradiction, one unit clarification, two
observed false rejections of publicly compliant output, and three deterministic
scoring repairs tied directly to existing public requirements.

These tests are finite probes. Passing them is not a conformance proof.

## Retained changes after v1.0.1

| Problem | Retained change | Scoring effect |
| --- | --- | --- |
| `eve_market_tools` | Checkpoint 2 now states that response `yields` use the same 0–1 multiplier scale as request `efficiency`. | None. Tests, fixtures, and reference behavior are unchanged. |
| `trajectory_api` | The existing timestamp patterns in checkpoint case data accept an optional decimal fraction before `Z`. | Removes rejection of ISO-8601 UTC timestamps such as `2026-08-02T12:34:56.123456Z`; all other timestamp fields remain constrained. |
| `textdrop` | Four checkpoint 1–2 assertions recognize a `code` element with optional attributes and whitespace as the direct child of `pre`. | Removes rejection of valid output such as `<pre><code class="language-python">`; content, escaping, hierarchy, and other rendering assertions remain. |
| `sheeteval` | Checkpoint HTML comparisons treat literal apostrophes and the HTML character references `&#x27;`, `&#39;`, and `&apos;` as equivalent. | Removes serialization-only failures such as `Sheet &#x27;Name&#x27;`; tag-count and all non-apostrophe expectations remain unchanged. |
| `database_migration` | The inherited drop-column/foreign-key regression now runs at every cumulative checkpoint and verifies preserved rows, foreign-key metadata, and `foreign_key_check`; the checkpoint 3–5 reference snapshots use native `DROP COLUMN` when a table has incoming foreign keys. | Prevents later-checkpoint credit when dropping an unrelated parent column corrupts an inherited foreign-key relationship. |
| `forge` | Checkpoint 8 now states that a relative config `rules_file` is resolved from the process working directory. | No scoring change; it makes the evaluator's existing path-resolution rule public. |
| `eve_industry` | Recipe and invention evaluators now require the documented canonical block to be the output suffix, validate label and table structure, and reject duplicate row keys. | Prevents malformed, trailing, or duplicate-key output from normalizing to a passing result; arbitrary preamble and harmless Markdown spacing remain allowed. |

The TextDrop and SheetEval changes are tied to preserved execution evidence in
which an evaluator rejected the examples above. The proposed Execution Server
serialization change is not retained: its available before/after evidence was
confounded by an unrelated checkpoint-1 statistics regression and therefore
did not isolate the claimed false rejection.

## Unresolved scoring issues

This list is limited to evaluator behavior that conflicts with a stated
requirement, fails to exercise a stated requirement, or imposes an unstated
interpretation. Ordinary test isolation, such as replacing external services
with controlled fakes, is not itself a defect.

| Problem | Established issue | Possible scoring error |
| --- | --- | --- |
| `dynamic_config_service_api` | The contract requires `408 evaluation_timeout` when policy evaluation exceeds 500 ms, but the timeout case also accepts an ordinary `200` response. | A solution that never implements the required timeout can receive credit. |
| `eve_market_tools` | `/v1/compress` must minimize purchase cost, but the verifier checks only feasibility and arithmetic consistency; it does not compare the returned plan with an independently computed optimum. | A feasible but suboptimal plan can receive credit. |
| `textdrop` | The object-store contract requires credential handling and rejection of invalid credentials, but the controlled storage server accepts unsigned requests and never checks `key_id` or `key_secret`. | A solution that parses but never uses the required credentials can receive credit. This does not require testing against a real cloud service; a controlled server could verify the observable authentication behavior. |
| `trajectory_api` | Checkpoint 4 states that activation re-parsing is atomic and that clients cannot observe intermediate state, but the evaluator performs no concurrent read during re-parsing. | A solution that exposes partially updated state can receive credit. |

The audit did not establish another concrete scoring error for the remaining
panel problems. In particular, controlled Kafka and RabbitMQ client fakes in
`mocked_http` are legitimate test isolation, not a suite defect. None of the
issues listed above is changed by this narrow catalog release.

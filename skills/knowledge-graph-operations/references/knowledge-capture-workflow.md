# Knowledge Capture workflow

Use this when testing or operating first-class `kg` Captures (notes, thoughts, ideas, lessons, decisions, observations, bookmarks, references, quotes, hypotheses, and personal questions).

## Safe creation

1. Resolve the active Vault and pass it explicitly before the subcommand.
2. Send an explicit note-like request through `kg chat --json`, for example:

```sh
printf '%s' 'Запомни мысль: автоматизировать клиентские отчёты.' |
  kg --vault "$VAULT" chat --stdin \
    --source hermes --source-type chat --sender "$ACTOR" --json
```

3. Require `route: capture`, `status: awaiting_approval`, `approval_required: true`, and `executable: false`.
4. Confirm no `Captures/capture_<ULID>.md` exists before approval. Natural-language Capture creation is review-only; do not replace it with a direct Markdown write.
5. Inspect the stable proposal with `proposal validate` and `proposal export ... --format json`. Check every planned Intent, dependency, blocker, conflict, Evidence reference, and the proposal fingerprint.
6. Under the user's standing authorization, approve all required unblocked planned Intents with a non-empty actor, reload the approval receipt, verify fingerprint and full required-Intent coverage, then submit once.
7. Inspect every per-Intent result and require the expected final status. Do not treat `partially_rejected` as success even if the CLI process exits zero.
8. Read back the Capture and run `kg validate`.

## Canonical verification

After submission verify:

- Path is exactly `Captures/capture_<26-character-ULID>.md`.
- Initial lifecycle is `status: inbox`, `review_state: unreviewed`.
- Body, Capture identity, kind, title, capture time, creator, source, and Evidence references remain immutable across review, linking, promotion, and archive. Hash the Markdown body and source Evidence before and after lifecycle operations when auditing.
- Candidate links are suggestions only. Hash or read back the target graph record before proposal creation and after approved `LinkCapture`; the target must not change.
- `CaptureChanged` appears in `kg events list`, and Knowledge Activity includes the corresponding Capture change.

## CLI lifecycle checklist

Exercise all public reads and lifecycle commands:

```sh
kg --vault "$VAULT" inbox
kg --vault "$VAULT" capture list
kg --vault "$VAULT" capture show TITLE
kg --vault "$VAULT" capture latest
kg --vault "$VAULT" capture search QUERY
kg --vault "$VAULT" capture review TITLE
kg --vault "$VAULT" capture promote TITLE --to KIND [--target ID]
kg --vault "$VAULT" capture archive TITLE
```

Normal Capture CLI output must omit Capture IDs, linked IDs, promotion IDs, and Evidence IDs. Repeat representative commands with `--ids` and require those references only then.

Promotion must always produce a dependency-ordered proposal. Before approval, submit must execute zero Intents and create/link no target. After exact approval, the target Intent executes before `PromoteCapture`, and the original Capture body remains byte-identical.

## Routing regression matrix

Verify the production matchers, not only the route-priority constant:

1. Dataset observation → Dataset.
2. Capture-search question → Search.
3. Analytical question → Analyze.
4. Planning request → Planning.
5. Proposal-management request → Proposal.
6. Supported relationship fact → specialized Graph Observe.
7. Explicit note-like request → Capture.
8. Ambiguous or unsupported text → Clarification.

Add collision probes where explicit note wrappers contain structured observations or graph facts. Earlier routes must still win according to the documented precedence. Also test English, Russian, and Greek forms, Unicode NFC, and Russian `ё`/`е` equivalence.

Regression assertions:

- Plain graph facts never become Captures.
- Plain Dataset observations never become Captures.
- Graph Observe and Capture are not universal fallbacks.
- Ambiguous requests write nothing.
- Questions without an explicit remember/note/capture signal do not create Capture proposals.

## Search, analysis, and privacy

- `kg capture search` and `kg analyze` must exclude restricted Capture content unless an explicit authorized policy path says otherwise.
- Also probe `inbox`, `list`, `show`, and `latest`; restricted filtering must be consistent across read surfaces.
- A visible Capture linked to a restricted Graph record must not leak that record's ID through analysis.
- Analysis must honor resolved time windows and subject terms, and should expose only policy-visible Capture Evidence.
- Validate Evidence arrays as non-empty canonical `evidence_<ULID>` references backed by immutable source artifacts; arbitrary strings or empty arrays are not valid provenance.

## Idempotency probe

Replay the same source identity, content, timestamp, and stable context after the first proposal is persisted. The operation should return/reuse the same immutable proposal rather than collide because of wall-clock metadata. Run this probe only after the graph/link-candidate state is held constant.
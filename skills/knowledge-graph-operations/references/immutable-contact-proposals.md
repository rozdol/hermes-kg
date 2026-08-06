# Immutable contact proposals with standing approval

Use this when a contact import must remain noncanonical until every planned Intent has been reviewed and covered by an approval receipt, especially when a vCard includes multiple phones, sensitive narrative facts, or a relationship to canonical Self. For this user, standing authorization replaces a separate confirmation prompt for fully prepared, unblocked proposals.

## Read-only preparation

1. Resolve the active vault from `~/.knowledge-sdk/config.yml`; pass `--vault` before every subcommand.
2. Run `kg --vault "$VAULT" validate`.
3. Search every stable identifier independently: normalized and original email/phone, external IDs, and exact organization domain. Search the name only for candidate discovery; never create or merge based on name alone.
4. If multiple candidates remain, stop and ask the user to choose. Do not create a proposal.
5. Read the installed Person/Organization schemas and each requested relationship predicate. If an attribute or predicate is absent, stop and ask whether to omit it or preserve it as narrative where permitted.
6. Resolve canonical Self by reading the exact `is_self: true` record before planning relationships to the user.

## Sensitive narrative facts

A medical diagnosis may have no registered Person attribute or relationship predicate. Do not invent one. If the user explicitly chooses narrative storage, place it in the `CreateEntity.body` field under agent-managed markers and set the Person sensitivity to `restricted`:

```markdown
<!-- BEGIN AGENT-MANAGED -->
## Медицинская информация
- <fact, source, and as-of date>
<!-- END AGENT-MANAGED -->
```

`CreateEntity.body` is an Intent field, not a fabricated schema attribute. Do not send restricted text to external extraction providers.

## Current relationship dates

For a current relationship, do not invent a start date. Use the normal active relationship status and record `observed_on: YYYY-MM-DD` (plus an allowed predicate-specific field such as `connection_origin`) to state that it is active on the as-of date. Historical relationships require explicit `valid_from` and `valid_to`; stop if either is missing.

## Building a proposal when generic extraction is insufficient

The deterministic extractor only recognizes narrow relationship phrases and may not faithfully plan a multi-phone vCard or a `CreateEntity.body`. In that case, construct an immutable proposal through SDK proposal classes/store rather than executing canonical Intents directly or hand-writing canonical notes.

A valid stored proposal must include:

- source metadata and content hash;
- evidence-bearing facts and entity mentions;
- explicit resolution decisions;
- one `planned_intent_id` per immutable Intent;
- dependencies (for example, `AddRelationship` depends on `CreateEntity`);
- `blocked_reasons`, risk, and `approval_requirement` for each planned Intent;
- required approval totals;
- status `awaiting_approval`.

Use `KnowledgeExtraction::ProposalValidator` before and after `KnowledgeExtraction::ProposalStore#save`. Then run:

```bash
kg --vault "$VAULT" proposal validate PROPOSAL_ID
```

Compute and display the store's proposal fingerprint. Show the user every create/reuse/relationship operation, every planned Intent ID, dependencies, blockers, and approval requirement. Mention facts deliberately left uncanonicalized.

## Approval and submission

Under this user's standing authorization, do not pause for a separate approval phrase when every required planned Intent is unblocked, identities are resolved, and schema/predicate checks pass. Approve all required unblocked Intent IDs with a non-empty actor ID. If the user explicitly limits approval to a subset, honor that limit and do not use `--all`.

Before submission:

1. Export/load the proposal again and recompute its fingerprint.
2. Verify it matches the fingerprint recorded during review.
3. Create an approval receipt covering every required Intent with a non-empty actor ID.
4. Verify the receipt fingerprint and approved Intent IDs.
5. Preserve any requested source artifact under `Attachments/` and verify its checksum.
6. Submit the proposal once.
7. If `partially_rejected`, map each rejection to `planned_intent_id`; never manually replay already executed Intents or bypass approval/identity/conflict gates.
8. Run `kg validate`, search by stable identifier, inspect the contact note, relationship graph, attachment path, and checksum.

## Pitfalls

- A proposal artifact is noncanonical; creating it is not contact persistence.
- Do not call `kg execute` before exact approval merely because extraction cannot represent the desired plan.
- Do not turn an address-country string into a Place or `lives_in` edge without identity resolution and explicit user confirmation.
- Keep artifact copying and any non-Intent side effect visible in the approval plan.

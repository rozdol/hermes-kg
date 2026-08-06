# Personal CRM entity and relationship writes

Use this recipe when a user provides contact details and explicitly asks to add them to a personal-CRM vault.

## Resolve the active vault

Do not carry forward a vault path merely because it was used earlier. The active attachment can change during a session. Inspect `~/.knowledge-sdk/config.yml`, resolve `active_vault` to its entry under `vaults`, and use that absolute `path` explicitly as `--vault "$VAULT"`. If the user names a vault, their explicit choice wins.

## Person write sequence

1. Run `kg --vault "$VAULT" validate`.
2. Search independently by canonical name, every email, and every phone. A name-only miss is insufficient proof that the person is new.
3. Generate one batch ID: `RUN_ID="$(kg --vault "$VAULT" id run)"`.
4. Create or update the Person through an immutable Intent; never hand-write YAML.
5. Add semantic relationships with separate `AddRelationship` Intents.
6. Run validation again, search by a stable identifier, and inspect `kg graph PERSON_ID` or `kg activity latest --json`.

A typical new Person needs:

- `name`, `aliases`
- `emails` and matching `primary_email` when supplied
- `phones` and matching `primary_phone` when supplied
- `tier`
- `sensitivity`
- `data_origin`
- `is_self: true` only for the single vault owner

Use `data_origin: given_by_subject` only when the person supplied their own information (or the vault owner explicitly states their own details). Do not infer missing contact values.

## Narrative versus structured facts

Relationships such as spouse, friend, parent, and likes belong in relationship records, not Person fields. Use the vault predicate registry and create one `AddRelationship` Intent per edge.

Narrative statements that have no schema field (for example lifestyle descriptions) belong in the Person body only when the user asked to preserve them. Keep agent-authored prose inside paired markers:

```markdown
<!-- BEGIN AGENT-MANAGED -->
## Образ жизни
- ...
<!-- END AGENT-MANAGED -->
```

Do not duplicate structured facts in the body.

## Controlled concept entities

Interests, professions, technologies, industries, and languages can be approval-gated. Search first. If absent, create the concept only with explicit human approval and the schema's required attributes (for example an Interest needs `interest_kind`). Then connect it with the appropriate relationship such as `likes` or `has_profession`.

Do not treat a general instruction to ingest text as permission to silently expand the ontology. If approval is ambiguous, save the Person and non-gated relationships, then ask about the new controlled concepts.

## Success criteria

Do not report success from `kg chat status: ok`. Success requires real canonical `changed_paths`, post-write validation, and retrieval of the created/updated record by a stable identifier. Report which facts were canonicalized and which remained narrative or pending approval.

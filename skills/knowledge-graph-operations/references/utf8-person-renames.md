# UTF-8-safe Person renames and identity-history updates

Use this when a canonical Person with a non-ASCII name must be renamed while preserving backlinks, aliases, former names, contacts, and entity identity.

## Canonical update sequence

1. Resolve the existing Person by name plus stable identifiers; never create a second Person just because the current surname changed.
2. Validate the vault and generate one `run_<ULID>` for the batch.
3. Use `RenameEntity` for the canonical display name. `UpdateEntity` intentionally rejects changes to `name`.
4. Use `UpdateEntity` on the same immutable Person ID to set:
   - `aliases` for search names;
   - `former_names` for previous/married/birth names;
   - normalized `emails`/`primary_email`;
   - normalized E.164 `phones`/`primary_phone`.
5. Validate again and verify searches by current name, each former name, and a stable identifier.
6. Inspect graph edges to confirm backlinks and family relationships survived the path rename.

Do not model a previous marriage as a currently asserted `spouse_of` edge when no valid end date can be represented safely. Preserve the historical statement as evidence-backed narrative and keep independently supported parent links structured.

## UTF-8 backlink-rewrite regression

Failure signature during a Cyrillic rename:

```text
Encoding::CompatibilityError: incompatible encoding regexp match
(UTF-8 regexp with ASCII-8BIT string)
... entity_manager.rb ... rewrite_backlinks ... gsub
```

Root cause: transaction reads may return Markdown bytes tagged `ASCII-8BIT`; a UTF-8 regular expression built from a Cyrillic old path fails when an existing Markdown file also contains non-ASCII bytes.

Tight regression fixture:

- create an unrelated existing Markdown file containing UTF-8 text;
- create a Person named `Мария Титова`;
- rename it to `Мария Курлычева`;
- assert the destination path exists and the source path was removed.

Minimal fix at the backlink rewrite boundary:

```ruby
content = content.dup.force_encoding(Encoding::UTF_8) if content.encoding == Encoding::ASCII_8BIT
```

Apply this before UTF-8 regex substitution. Keep the conversion scoped to Markdown backlink rewriting rather than changing binary transaction semantics globally. Verify the targeted regression test first, then the full SDK suite.

## Verification evidence shape

A complete verification should show:

- `RenameEntity` returns the original Person ID;
- changed paths include the old and new Person paths plus rewritten relationship notes;
- searches by current name, aliases, and email all resolve to that same ID;
- `kg graph PERSON_ID` or the related person's graph still contains the original relationship IDs;
- post-write `kg validate` succeeds.

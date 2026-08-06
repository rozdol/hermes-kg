# Family, friendship, and business relationship modeling

Use this reference when converting personal-CRM prose about spouses, former marriages, children, friends, and business partners into canonical `kg` relationships.

## Model only what the wording supports

- Search every named person before creating anything. Use email/phone as stronger identity signals than name alone.
- Create a Person with the exact name supplied. If only a first name is known, do not invent a surname; distinguish the person through parent relationships and ask for a fuller name later only when identity ambiguity blocks the write.
- Do not infer `relationship_kind: biological` merely from “daughter” or “son.” Set it only when biological/adoptive/step-parent status is explicit; otherwise omit the optional field.
- Use `data_origin: third_party` for facts the vault owner reports about someone else.

## Predicate choices

- Friendship: `friend_of`.
- Business partner or peer work relationship: prefer `collaborates_with` unless the vault has a more precise business predicate. Do **not** use `partner_of`: in the personal-CRM ontology it means romantic or domestic partnership.
- Parentage: one directed `parent_of` edge from each known parent to each child.
- Current spouse: symmetric `spouse_of` only when the wording supports a current marriage.

Inspect `_System/Relationship Types/<predicate>.md` before using an unfamiliar predicate. Its subject/object types and allowed fields are authoritative.

## Historical marriages

Do not create an asserted `spouse_of` edge that looks current when the source says “former,” “previous marriage,” or “first marriage.” If an exact end date is supplied and the ontology supports `valid_to`, store the dated historical edge. Without a date, preserve the chronology as concise agent-managed narrative and canonicalize only the independently supported parent edges.

Likewise, “child from a second marriage with X” proves both named parents but does not necessarily prove that X is the current spouse.

## Batch workflow

1. `kg --vault "$VAULT" validate`.
2. Search all people by name and every available stable identifier.
3. Generate one `run_<ULID>` and use it for the complete batch.
4. Create/update people sequentially; never run autonomous writers concurrently.
5. Add one relationship Intent per edge.
6. Validate again.
7. Verify the main Person and at least one child with `kg graph ID`; confirm both parents appear where applicable.
8. Report withheld inferences explicitly (for example, “no current spouse edge was created because the marriage status was not stated”).

## Common pitfalls

- `kg chat status: ok` is not proof that any relationship was written.
- A symmetric social predicate still has one canonical relationship record; do not add both directions.
- Do not encode family relationships as custom Person frontmatter fields.
- Do not silently turn a former marriage into an active spouse relationship.
- Do not confuse business partnership with romantic `partner_of`.

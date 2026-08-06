# Sensitive identity-document binding through immutable proposals

Use this reference for passports, national IDs, social-insurance confirmations, residence documents, and similar restricted artifacts that must be attached to an existing Person.

## Fast preflight

Batch independent read-only checks whenever possible:

1. Resolve the active vault and obtain the current local date/time.
2. Hash and stat the source artifact.
3. Read the existing Person record and capture its immutable ID.
4. Search the document number in original, normalized, and MRZ forms. Use both `kg search` and direct vault-content search because frontmatter identifiers are not always returned by `kg search`.
5. Check the intended `Attachments/` filename for collisions.
6. Inspect the Person schema and current fields so a new document does not overwrite a different document class.

Stop for multiple identity candidates, conflicting document numbers, unsupported structure, or uncertain critical OCR characters. Do not stop merely because the user omitted an actuality date: use the current local date under this user's standing default.

## Extraction and verification

- Try local PyMuPDF text extraction first.
- If the PDF has no text layer, render only the required pages to images and use local/native vision OCR. Do not send restricted identity content to an external extraction provider.
- Preserve uncertainty instead of guessing. Critical fields include the document number, owner name, birth date, issue/expiry dates, issuing authority, and subdivision/registration codes.
- For machine-readable passports, independently calculate available MRZ check digits. A matching check digit raises confidence but does not replace identity resolution.
- Avoid storing MRZ lines unless specifically needed; they duplicate highly sensitive data.

## Flat-field namespaces

Keep separate document classes in separate flat scalar namespaces so later imports do not overwrite earlier ones:

- Foreign passport: `passport_file`, `passport_sha256`, `passport_number`, `passport_issued_on`, `passport_expires_on`, `passport_issuing_authority`, `passport_issuing_country`, `passport_birth_place`, `passport_type`.
- Russian internal passport: `internal_passport_file`, `internal_passport_sha256`, `internal_passport_number`, `internal_passport_issued_on`, `internal_passport_issuing_authority`, `internal_passport_subdivision_code`, `internal_passport_issuing_country`, `internal_passport_birth_place`.
- Social insurance: `social_insurance_file`, `social_insurance_sha256`, `social_insurance_number`, `social_insurance_country`, `social_insurance_registered_from`, `social_insurance_confirmation_date`, `social_insurance_issuer`.

Use only scalar values or flat scalar lists. Preserve unrelated existing fields. Upgrade the Person to `sensitivity: restricted`. Add verified aliases or `birth_date` only when the document and canonical identity agree.

## Proposal pattern

Create one stored immutable `UpdateEntity` proposal for the exact Person ID. It should contain:

- source metadata with artifact hash, size, page count/type, sensitivity, and planned attachment path;
- one evidence-bearing fact with sensitive values redacted in excerpts where practical;
- an explicit `resolved` identity decision;
- one high-risk `UpdateEntity` planned Intent;
- no dependencies for a single-person update;
- explicit `blocked_reasons`, `approval_requirement`, and immutable fingerprint.

Validate the proposal before approval. Under this user's standing authorization, automatically approve every required unblocked Intent with a non-empty actor ID. Reload the proposal and receipt, verify fingerprint equality and complete Intent coverage, then copy the artifact into `Attachments/`, verify its checksum, and submit exactly once.

## Post-submit verification

1. Inspect the submission receipt and map every result to `planned_intent_id`.
2. On `partially_rejected`, do not replay executed Intents; report each rejection and required corrective action.
3. Read the Person note back and verify the exact document namespace, attachment path, hash, dates, and authority.
4. Confirm the attachment exists and its checksum matches.
5. Run `kg validate`.
6. In the user-facing summary, mask full passport, national-ID, and social-insurance numbers; report full values only when explicitly requested.

## Efficiency pitfalls

- Do not route a scanned identity document through repeated `kg chat` clarification loops when the user named the exact existing contact.
- Do not hand-write canonical Markdown/YAML.
- Do not build a new Person when an exact existing Person ID is available.
- Do not ask a second approval question for fully reviewed unblocked proposals covered by the standing authorization.
- Do not overwrite `passport_*` with internal-passport data; use `internal_passport_*`.

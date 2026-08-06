# Auditing `kg chat` routing

Use this when reviewing intent-classifier precedence, especially after multilingual or Capture changes.

## Core method

1. Read the documented route contract and the subsystem routing docs before forming expectations.
2. Inspect both the central priority list and every production matcher. A correct `ROUTE_PRIORITY` does not prove production precedence if matchers suppress one another.
3. Search for candidate-veto logic such as `next nil if explicit_capture?`, read-question guards, and domain fallthrough. Treat these as part of precedence.
4. Run two kinds of deterministic probes:
   - A synthetic classifier collision where every route returns a candidate. This verifies the central ordering mechanism independently.
   - Production collision inputs containing signals for two routes, such as explicit Capture + Dataset measurement, Capture + supported Graph fact, read question + Dataset vocabulary, and Analyze + Planning language.
5. Use classifier diagnostic traces to confirm which plugins loaded, which candidates were emitted, and which candidate won. If an expected earlier candidate is absent, identify the exact matcher guard that removed it.
6. Build a multilingual matrix for English, Russian, and Greek with:
   - canonical explicit Capture forms;
   - common grammatical/accent variants;
   - plain Dataset observations;
   - plain supported Graph facts;
   - read/analysis questions;
   - ambiguous imperative fragments.
7. Check normalization end-to-end. Verify matchers receive the normalized matching representation promised by docs; computing accent/case or `ё`/`е` normalization is insufficient if plugins receive original text.
8. Report each defect with actual input → actual route, expected route from the documented contract, and precise `file:line` for both the matcher and any upstream veto.

## Important pitfalls

- Do not infer precedence solely from route-array order; production candidate suppression can invert it.
- Do not test only happy-path phrases copied from docs. Include overlap cases and natural language variants.
- A message that produces no write is not automatically correct: a supported Graph fact becoming Clarification is still a routing regression.
- Conversely, an ordinary question becoming Capture is high severity because it can create a proposal from read-only intent.
- Keep the audit read-only. Direct classifier probes are preferable to full CLI calls when proposal creation would have side effects.

## Minimal collision corpus

- Dataset: `My weight is 70 kg` plus an explicit note-prefixed equivalent.
- Graph: `Alex works at Acme` plus an explicit note-prefixed equivalent.
- Search: questions about saved ideas/notes in all supported languages.
- Analyze: causal or trend questions in all supported languages.
- Ambiguous: short imperatives such as “do this” / localized equivalents.
- Unicode: Russian words containing `ё` and common Greek accented variants.

# Contributing a Case Study

## The Short Version

1. Copy `TEMPLATE.md` to `case-studies/NN-your-slug.md`
   where `NN` is the next available two-digit number.
2. Fill in every section. Do not leave placeholder text in the published version.
3. Add a row to the index table in `README.md`.
4. Open a pull request against `main`. Title it `Add case study NN: Short Title`.

## Content Standards

**Be specific.** Name the actual failure mode, not a sanitized abstraction of it.
If the detail is sensitive, omit the case study or anonymize at the system level —
do not anonymize by being vague about the technical content.

**Strip internal-only identifiers.** IP addresses, internal hostnames, tool-internal
UUIDs, and personal names do not belong here. Replace with generic labels
(e.g., `control-plane node`, `secrets manager`) when needed for clarity.

**All six sections are required.** A PR missing a section will be sent back.
The Generalization section is the hardest to write and the most important —
do not skip it or fill it with a single sentence.

**Honest about failure.** The most useful case studies describe what went wrong,
including what the author or agent did that made it worse. Sanitizing out the
embarrassing parts removes the value.

## File Naming

`case-studies/NN-slug.md`

- `NN` — zero-padded two-digit integer, sequential (01, 02, ..., 10, 11, ...)
- `slug` — lowercase, hyphens, no spaces, descriptive of the incident not the fix

## License

By opening a PR you agree to license your contribution under CC BY-SA 4.0,
the same license as the rest of this repository.

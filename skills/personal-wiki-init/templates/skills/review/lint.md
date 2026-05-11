# Wiki lint

Scan and *report* — don't auto-fix. Flag:

- **Orphans** — pages with no inbound wikilinks.
- **Stale claims** — pages superseded or contradicted by newer sources.
- **Missing pages** — concepts/entities referenced but lacking their own page.
- **Broken wikilinks** — `[[...]]` targets that don't exist.
- **Contradictions** — conflicting claims across pages.

Present as a checklist. I decide what to act on.

**Not a lint issue** — do not flag these:

- A `{{WIKI}}/sources/` page whose topic is already represented by an indexed `{{WIKI}}/concepts/` page. The concept is the canonical index entry; the source page is a supporting detail discoverable via the concept's `sources:` frontmatter. Flagging it as "unlisted" creates noise.

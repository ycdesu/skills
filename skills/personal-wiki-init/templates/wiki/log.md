# Wiki Log

Append-only chronological record of wiki operations.

Format: `## [YYYY-MM-DD] {op} | {title}` where `{op}` ∈ `ingest`, `ingest-fragments`, `query`, `synthesis`, `idea`, `someday`, `someday-review`, `now-review`, `lint`, `tour`, `cleanup`, `init`.

Parseable: `grep "^## \[" {{WIKI}}/log.md | tail -10`.

# Contributing

Corrections, additions, and new hard-won lessons are very welcome. A few rules keep this repo
useful and clean:

## Content rules (non-negotiable)

1. Read [COMPLIANCE.md](COMPLIANCE.md) first. The one-line test:
   *could you obtain and cite this from an official public source or your own original work,
   without extracting or reverse-engineering game content?* If not, it doesn't go in.
2. **Verify before you write.** This repo's value is that its claims were tested. State the game /
   tools version you verified against (e.g. "verified on 1.8.0"). If something is a hypothesis,
   label it as one.
3. **Vanilla GUIDs are placeholders.** Write `{VANILLA-GUID}` plus the resource path; readers
   resolve it via Workbench → right-click → *Get Resource GUID(s)*.
4. **No third-party mod code**, no extracted files, no verbatim vanilla source.

## Style

- English, plain and direct. Short sections, concrete numbers, real error messages in quotes.
- Mark traps visibly: `⚠️` for "this will bite you", with the symptom you'll see.
- Prefer "symptom → cause → fix" for troubleshooting entries.
- One topic per file; cross-link related docs with relative links.

## How to submit

- Small fix: open a PR directly.
- New document or restructure: open an issue first so we agree on scope.
- Personal project state, machine-specific paths, or WIP notes don't belong here — this is a
  general knowledge base, not a project journal.

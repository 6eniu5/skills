---
name: craft-cards
description: Turn a drill transcript or an add-language manifest into properly-formatted Lingo flashcard Drafts in the vault's _review/ area.
disable-model-invocation: true
---

# craft-cards

Turn captured knowledge into **Draft** cards in `<Language>/_review/` for the Lingo
vault. You supply the model-drafted field values; the deterministic `lingo draft`
command renders and writes them. You never write a live card, never touch
scheduling, never promote — the user promotes a Draft by moving it out of `_review/`.

**Setup (tied to my machine):**

- Generator repo: `~/Documents/Projects/setup/esetup/obsidian-lingo` (run `lingo`
  commands here; `bun install` once).
- Vault: `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Lingo`.
- Read `docs/CONTEXT.md` (glossary) and `docs/adr/0004`–`0006` in the generator repo
  before drafting; match the card contract below exactly.

## The card contract (a Word Draft)

```
---
language: German
pos: noun
gender: n
article: das
lemma: Haus
pronunciation: /haʊ̯s/
level: A1
concept: "[[house]]"
tags:
  - flashcards/german
  - theme/home
---

das Haus:::the house

> [!example]- Sentence
> Das Haus ist groß. — The house is big.
```

- `word:::gloss` is **reversible** (both directions). The word is the target
  expression *with its article* (`das Haus`) and is the note filename.
- Grammar fields stay blank for words that lack them (an interjection like `merci`
  has empty `gender`/`article`).
- `concept: "[[Title]]"` links the Concept hub. Reuse an existing Concept; only
  invent one when none exists (then draft it too).
- `pronunciation` is IPA. `level` is CEFR (default the config's first, A1).
- `theme/<x>` must be one of the themes in `lingo.yaml`, or omit it.

## You emit a payload; `lingo draft` writes it

Never hand-write notes into the vault. Build a JSON payload and run the command —
it guarantees the format, skips a card whose Concept already has a live Word
(dedup), and refuses to overwrite an existing Draft (unless `--force`).

```json
{
  "concepts": [{ "title": "hello", "definition": "A greeting." }],
  "words": [
    {
      "word": "hola", "gloss": "hello", "concept": "hello",
      "code": "spanish", "pos": "interjection",
      "lemma": "hola", "pronunciation": "/ˈo.la/", "level": "A1",
      "theme": "social", "example": "¡Hola! ¿Cómo estás? — Hi! How are you?"
    }
  ]
}
```

```bash
cd ~/Documents/Projects/setup/esetup/obsidian-lingo
bun run draft --payload /path/to/payload.json --vault "$HOME/Library/Mobile Documents/iCloud~md~obsidian/Documents/Lingo" --dry-run   # preview
bun run draft --payload /path/to/payload.json --vault "$HOME/Library/Mobile Documents/iCloud~md~obsidian/Documents/Lingo"             # write to _review/
```

`code` must be a language in `lingo.yaml` (run `lingo add-language` first for a new
one). `concept` must match the Concept the card links — add it to `concepts[]` if
it isn't already live in the vault.

## Mode A — from a drill transcript

Pipeline (ADR-0004): capture → save raw → **clean/validate loop** → draft.

1. **Save raw.** The user pastes the transcript from their voice-to-text app. Save
   it verbatim to `_capture/<YYYY-MM-DD>-<lang>.md` in the vault. Never draft from
   the raw transcript.
2. **Clean/validate — iterate with the user.** Produce a cleaned copy and show each
   change, because voice-to-text mangles a foreign drill. Fix: missing/wrong
   diacritics (ä ö ü ß, é è, ñ), misheard target words (map back using the drill
   language), English↔target confusion (which side is the gloss), run-on
   segmentation (one utterance split, or two merged). Flag anything uncertain and
   ask. Loop until the user says it's good.
3. **Draft.** For each vocabulary item in the *cleaned* transcript, build a Word
   entry (translate + supply gender/article/pos/IPA/level/example). Reuse existing
   Concepts (check `Concepts/`); add a `concepts[]` entry for any new one. Emit the
   payload and run `lingo draft`.
4. Report what landed and that the user promotes by moving notes out of `_review/`.

## Mode B — from an add-language manifest

The user ran `lingo add-language <Name> [--comprehensive --from <Lang>]`, which
wrote `<Name>/_review/_manifest.json`. Draft its requests.

1. **Read the manifest.** Each `requests[]` item has `concept`, `englishGloss`,
   `definition`, `conceptExists`, and `crossRefs` (every other language's Word for
   this Concept, with gender/IPA/example). Use the cross-references to nail the
   sense and register — the `--from` anchor language is listed first.
2. **Draft each request** into `manifest.target`. Fill `manifest.fields`; default
   `level` to `manifest.levelDefault`; pick `theme` from `manifest.themes` or omit.
   When `conceptExists` is false, add a `concepts[]` entry (one-line definition).
3. **Emit the payload and run `lingo draft`** with `--vault` = the manifest's vault.
   The command re-checks dedup, so anything already covered is skipped safely.
4. Report counts and remind the user to review + move Drafts up to promote them.

## Rules

- Work in batches you can eyeball; accuracy over volume.
- Search `Concepts/` before inventing a Concept.
- Write only into `_review/`; never edit or promote a live card.
- Leave a genuinely unknown field blank rather than guess wrong — the user fills it
  during review.

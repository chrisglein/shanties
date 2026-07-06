---
name: booklet-song-from-abc
description: 'Create or fix a Sea Shanties booklet song page/spread from its ABC-notation source, keeping chords in sync. USE WHEN: generating a new booklet/*.md song from a songs/*.txt ABC file; auditing or fixing chord transcription errors (wrong chord, wrong word, missing/extra chord) between a booklet markdown and its ABC; aligning [chord] placements to the ABC "chord"-before-note positions; transposing a booklet song to a new key. Covers both formats (ABC and booklet markdown), the note→syllable→chord alignment procedure, caller-vs-crew (`>`) refrain marking, and the common pitfalls (late chords vs pickup placement, dropped internal returns, key-signature accidentals, transposition drift, held-note melisma).'
---

# Booklet Song from ABC

Build or repair a song in `booklet/` so its `[chord]` placements match the harmony encoded in the corresponding `songs/*.txt` ABC file. **The ABC is the primary reference for chords and lyrics** — but the songbook maintainer may deliberately override it by ear (see pitfall 11).

## When to use

- **Initial generation** — a `songs/<slug>.txt` ABC exists and you need a new `booklet/<slug>.md` spread.
- **Audit / fix** — a `booklet/<slug>.md` already exists and you're checking or correcting its chords against the ABC (the task that produced [SongAnalysis.md](../../../SongAnalysis.md)).
- **Transpose** — moving a booklet song to a different key for the singers' range while staying faithful to the ABC's progression.

## The two formats

### ABC source (`songs/*.txt`)

Header fields: `X:` index, `T:` title, `M:` metre, `L:` default note length, `K:` key, `Q:` tempo, `V:` voice, `P:` part label (e.g. `Verse` / `Chorus`), `%%…` render directives.

Body:
- Notes are letters (`A`–`G`, `a`–`g`, `,`/`'` shift octave). Digits/fractions after a note set duration (`e2`, `b,3/2`, `A4`).
- **Chords** are quoted text placed *immediately before the note they attach to*: `"F"ABcA` → the `F` chord starts on the `A`.
- `w:` is a lyric line aligned **one syllable per note, left to right**. Within a word, hyphens split syllables (`John-ny`); spaces separate words. `~`, `*`, `_` mark a **held note / melisma** (the previous syllable continues — do NOT advance to the next syllable). `\` continues a `w:` onto the next line. `y`/`z` are spacers/rests.
- `W:` (capital) holds extra verses as plain text (no note alignment).
- **Key signature changes which letter means which pitch.** In `K:Eb`, a written `A` note is actually **A♭** — this decides whether a passage fits, say, `Fm` vs `Cm`. Never infer the chord from the melody letter without applying the key signature.
- Multi-voice songs (`V: 1`, `V: 2`) put the melody usually in `V: 1`; read chords from the melody voice.

### Booklet markdown (`booklet/*.md`)

- `# Title` first line; optional credit as an italic line `*Artist, year*`.
- Verse lines are plain text lines.
- `{chorus}` opens a boxed chorus block (styled by `shanties.css`).
- `[Chorus]` on its own line is a **cue** ("repeat the chorus here").
- A line beginning with `> ` renders in the **refrain style** (indented italic). Use it to mark everything the **crew** sings versus the **caller/lead**: the echoed repeats of a call line, the response refrains, and — in a traditional call-and-response shanty — *every* line of the `{chorus}`. Leave the caller's unique lead lines un-prefixed (roman). A modern verse/chorus song that isn't call-and-response (e.g. Joli Rouge) keeps its chorus un-prefixed.
- `[Chord]` is placed **immediately before the syllable it sits on**: `to[G]bacco` → `G` on the "bac" syllable. Put it on the note's **downbeat syllable**, not a pickup word before it (`There [Cm]once`, not `[Cm]There`).
- Separate stanzas within a long verse with a **blank line**.
- `---` marks the **spread/page break** inside the song.

## Procedure: ABC → booklet

1. **Read headers.** Note `K` (key + accidentals), `M`, `L`, and any `P:` (Verse/Chorus) and `V:` (which voice is the melody).
2. **Walk each melodic line note-by-note**, pairing every note with its `w:` syllable in order. Skip nothing: held-note markers (`~ * _`) keep the pointer on the current syllable.
3. **Record each chord's syllable** — the syllable of the note *immediately after* each `"chord"`.
4. **Emit booklet lines**, putting `[Chord]` right before that syllable. Copy the ABC's `w:` (and `W:`) text as the lyrics; prefer `w:` wording over `W:` when they differ.
5. **Structure & roles:** wrap the refrain in `{chorus}`; mark every **crew** line with `> ` (echoed call repeats, refrain answers, and all chorus lines of a call-and-response shanty), leaving the **caller's** lead lines un-prefixed; use `[Chorus]` as the repeat cue for later verses; break long verses into stanzas with a blank line; insert `---` near the middle so the song fills a two-page spread.
6. **House style for verses:** the booklet normally leaves verses **unchorded**, showing chords once (in the `{chorus}`, or on the first chorded instance). Keep verses unchorded **unless** the ABC's verse and chorus have genuinely different progressions, or the ABC only harmonizes the verse — then chord whichever section carries the harmony.
7. **Register the page** in [book.yaml](../../../booklet/book.yaml) (`pages:` list, `type: song`) if this is a new song.

## Procedure: audit / fix an existing page

1. Do the same note→syllable→chord alignment from the ABC.
2. Diff the booklet's `[chord]` positions against the derived positions. Classify each mismatch: **wrong chord**, **wrong word** (misaligned), **missing**, or **extra**.
3. Fix minimally — move/replace only the offending `[chord]`; don't rewrite lyrics unless they diverge from the ABC.
4. Record findings (per-song verdict) as in [SongAnalysis.md](../../../SongAnalysis.md).

## Common pitfalls (ranked by how often they bite)

1. **Late chords (the #1 error).** Put the chord on the exact syllable of the note the ABC attaches it to — *not* the next stressed word or the end of the line. Verify by the melody note sitting under the chord.
2. **Dropping an internal return.** `C → F → C` easily collapses to `C … F` with the `F` parked on the last word and the return to `C` lost. Preserve every change in the ABC.
3. **Key-signature accidentals.** e.g. `K:Eb` ⇒ written `A` = A♭. This flips which chord fits. Always apply the key signature before judging a chord.
4. **Transposition drift.** If the booklet is in a different key from the ABC, transpose **every** chord by the **same** interval. Keep a minor tonic minor (don't swap in the relative major), and **keep dominants** (a `V7`/`E7`/`A7` before the tonic must survive the transposition).
5. **Held notes / melisma.** `~ * _` extend the previous syllable — they do not consume a new lyric syllable. Miscounting them shifts every following chord.
6. **Chord on the downbeat, not the pickup.** The mirror image of pitfall 1: put the `[chord]` on the exact note the ABC attaches it to. When a pickup/anacrusis precedes that note (`And we'll`, `There`, `Oh`), the chord belongs on the following downbeat word (`roll`, `once`), not the pickup.
7. **Prefer whole-word placement; don't over-split.** Put the `[chord]` before the whole word when the change falls at/near the word start (`[Dm]swinging`, not `swing[Dm]ing`). Only split a word when the change genuinely occurs mid-word *and* it matters. If the ABC chord lands mid-word (e.g. the `B` on "whaler-**men**") but the booklet writes the word as one token, place the `[chord]` at the nearest word boundary.
8. **Shared verse/chorus tunes.** Many shanties reuse the verse melody for the chorus; a chord that's on word *N* of the verse maps to the *analogous* chorus word, which may differ in syllable count — align by melodic position, not word index.
9. **Fast response lines.** Call-and-response refrains change chords quickly (watch `G`/`D`/`D7` clusters); it's where misalignments concentrate.
10. **Modern / copyrighted songs.** The booklet may intentionally follow the artist's official chart rather than the ABC. Flag divergences and confirm with the user before "fixing".
11. **The maintainer's committed chords win.** The ABC is the reference, but the songbook owner may deliberately differ — adding passing chords by ear, re-voicing a section, or choosing the ABC's *own* key over a prior transposition. When asked to "err toward the ABC," adopt the ABC's actual key and chords rather than transposing an existing booklet version; and don't silently revert a maintainer's deliberate choice back to the ABC.

## Verification / keeping in sync

- Re-derive the booklet chords from the ABC and confirm each `[chord]` sits on the ABC-attached syllable.
- Build a proof and eyeball it: `npm run booklet:proof` (reading-order, one page per page) or `npm run booklet` (imposed), then open `booklet/sea-shanties.html`.
- If you added a song, make sure it's in `booklet/book.yaml` and the contents/page numbers still line up.

## References

- `ABCquickRefv0_6.pdf` and `abc_standard_v2.1 [abc wiki].mht` in the repo root — ABC notation reference.
- [booklet/README.md](../../../booklet/README.md) — build/print instructions and file roles.

# rand-th

[![Deploy](https://github.com/suradet-ps/rand-th/actions/workflows/docs.yml/badge.svg)](https://github.com/suradet-ps/rand-th/actions/workflows/docs.yml)
[![GitHub Pages](https://img.shields.io/badge/Pages-live-2ea44f)](https://suradet-ps.github.io/rand-th/)
[![License: MIT OR Apache-2.0](https://img.shields.io/badge/License-MIT%20OR%20Apache--2.0-blue.svg)](LICENSE-MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/suradet-ps/rand-th/issues)

---

## ◆ PULSE

A seed goes in, a sequence comes out - rand-th is the Thai bridge to
that exact transformation. This is the complete Thai translation of
the official Rust Rand Book: 33 source files (31 linked chapters, an
unlisted overview page, and the summary), built with
mdbook, terminology locked by a single glossary, and every code block
byte-identical to the original. The links are checked against the
built book (431 anchors), the structure mirrors the upstream repo
file-for-file, and the license travels with the text. Built for the
Thai-speaking Rustacean:
[suradet-ps.github.io/rand-th](https://suradet-ps.github.io/rand-th/).

| 31 chapters translated ▣ | Glossary ▣ | Links 431/431 ▣ | Build passing ▣ |
|---|---|---|---|
| 45 code blocks byte-exact ▣ | 441 links ▣ | 6 updating guides ▣ | MIT OR Apache-2.0 ▣ |

*v1.0.0 - translation, glossary, verification, and the static build
are all sealed.*

> Built with mdbook 0.5 + Markdown, translated from
> [rust-random/book](https://github.com/rust-random/book),
> verified by script and rendered as static HTML - a book with the
> entropy on the page.
>
> **suradet-ps**, artifact keeper

---

## ◆ IGNITION

One runtime, three commands.

```
⟫ git clone https://github.com/suradet-ps/rand-th.git
⟫ cd rand-th
⟫ cargo install mdbook
⟫ mdbook serve book --open
```

Open [http://localhost:3000](http://localhost:3000).

```
⟫ mdbook build book                             # static HTML into book/book
⟫ powershell scripts/check-links.ps1            # all anchors in the built book (pwsh on Linux/macOS)
⟫ powershell scripts/verify-translation.ps1     # byte-exact check vs upstream
```

> On Linux or macOS, run the verification scripts using `pwsh scripts/<script>.ps1`.
> `verify-translation.ps1` checks against `rust-random/book` in adjacent directories or via `-Orig <path>`.
> The upstream `book` repository sits beside this one (clone
> `https://github.com/rust-random/book` alongside `rand-th`), and its
> source lives in `book/src`, so the verifier is pointed at `../book/src`.

<details>
<summary>Translating a chapter</summary>

A chapter is a file: `book/src/<chapter>.md`, listed in
`book/src/SUMMARY.md`. The glossary lives in `GLOSSARY.md` - a term
is chosen once and reused everywhere. Code blocks, commands, links,
and filenames stay verbatim; only prose and headings are translated.
Heading anchors follow mdbook's slug rules (Thai tone marks and
than-thakhat are stripped while vowel signs such as ุ and ื survive),
so anchors are copied from the built HTML, never guessed.

</details>

---

## ◆ ANATOMY

One stack, zero custom JS, several quiet helpers.

- **Translates** - the complete book: introduction, quick start,
  the crate family with features, platforms and reproducibility,
  the 12-chapter guide from random data to testing functions that
  use RNGs, all six updating guides (0.5 through 0.10), and the
  contributor's guide - Thai prose over untouched code.
- **Glossaries** - `GLOSSARY.md` locks the vocabulary (one Thai
  term per concept, chosen once and reused), so chapter nine agrees
  with chapter two.
- **Verifies** - `scripts/verify-translation.ps1` diffs every code
  block (45 of them), heading level, and link target against
  upstream `rust-random/book` - byte-exact or it does not pass.
- **Checks** - `scripts/check-links.ps1` walks the built book and
  resolves every anchor link against real heading ids - 431 of them,
  all reachable.
- **Builds** - mdbook renders static HTML into `book/book/`, zero
  server runtime, readable offline and searchable by built-in static
  index.
- **Licenses** - MIT OR Apache-2.0, inherited from upstream, with the
  LICENSE files shipped beside the text.

---

## ◆ RITUALS

**The core ceremony** - the translation pass:

1. Open a chapter in `book/src/`. The upstream `rust-random/book`
   repo sits beside it (clone
   `https://github.com/rust-random/book` alongside `rand-th`)
   - structure is a contract.
2. Translate the prose; keep every code block and command as the
   original wrote it.
3. Consult `GLOSSARY.md` for every term that already has a canon.
   New terms get proposed in the glossary first.
4. Build, verify, check. The book builds clean, the diff is
   byte-exact, and the anchors resolve.

**The ceremony of the anchor** - mdbook slugs strip Thai tone marks
and than-thakhat but keep vowel signs, so a heading's anchor is never
its plain spelling. Anchors are read from the built HTML, written
into the source, and re-verified - a guessed anchor is a broken link
waiting to happen.

**The ceremony of the code block** - a translated command that is not
byte-identical to the original is a regression, not a translation.
The verifier is the conscience of the repo.

---

## ◆ ECHOES

**Where this artifact is heading**

```
P1 ▸ SUMMARY + introduction, indexes ──────────────────────────────── ▸ sealed
P2 ▸ crates, features, platforms, reproducibility ─────────────────── ▸ sealed
P3 ▸ the guide, from random data to testing functions with RNGs ───── ▸ sealed
P4 ▸ updating guides 0.5 to 0.10 ──────────────────────────────────── ▸ sealed
P5 ▸ contributor's guide, glossary, license, verification, build ──── ▸ sealed
```

**Raising the artifact** - the honest path lives in `GLOSSARY.md`
(term canon), `scripts/` (the verification gate), and
`book/book.toml` (book config). New chapters follow the
frontmatter-free contract of the SUMMARY. Open an issue first to
discuss a change.

**Status** - on every change: `mdbook build book` must pass, the
translation verifier must report byte-exact code blocks across all 33
files (31 chapters + overview + `SUMMARY.md`), and the link checker
must report `ALL ANCHOR LINKS OK`.
[Watch the gates](scripts).

---

```
  ─────────────────────────────────────────
   Every seed has its sequence
   Every book has its first page
  ─────────────────────────────────────────
```

Translated from [The Rust Rand Book](https://github.com/rust-random/book),
which is licensed under [MIT](LICENSE-MIT) OR [Apache-2.0](LICENSE-APACHE).

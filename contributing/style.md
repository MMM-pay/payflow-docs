---
title: Writing style
nav_order: 13
---

# Writing style

How a page in this repo should read. The goal is that a page a maintainer has not
read still matches the rest of the site: short, plain, consistent in terms, and
phone-readable.

## Voice

- Write in plain declarative sentences. State the fact, then the consequence.
  Avoid "basically", "actually", "note that".
- Define a term on first use, then use it. Do not swap between names for the
  same thing: it is a **mandate**, not a "subscription" in the protocol sense,
  not a "card".
- When a tradeoff is deliberate, say so plainly, the way the
  [trust model](../protocol/trust.md) does. A reader should be able to tell
  where a design decision ends and a bug risk begins.
- One idea per paragraph. Lists for things that are a set, prose for things
  that are an argument.

## Terminology

Use the project's terms consistently and define anything new:

| Say this | Not this |
|---|---|
| mandate | card, subscription, recurring charge |
| vault | wallet, balance, funds (when it is the custody contract) |
| keeper | the billing job, the auto-settler |
| plan | pricing, tier, package |
| charge | invoice, payment run |

These are defined on the pages that explain them: the
[mandate lifecycle](../protocol/lifecycle.md),
[economics](../protocol/economics.md), and the
[trust model](../protocol/trust.md). Link to those pages instead of
re-explaining.

## Money and numbers

Money is never a float. Amounts are in the token's smallest unit, integer
basis points for fees. When you show a number, show it in that unit and, once,
the human-readable equivalent, as the economics page does:

```
| Line | Stroops | XLM |
|---|---:|---:|
| Charge amount | 100 000 000 | 10.0 |
```

Do not paste a contract id into a page. Ids change on redeploys; point at the
deployment config instead so the id is read, not copied.

## Code blocks

- Fence every sample with a language tag: ```` ```bash ````, ```` ```rust ````,
  ```` ```ts ````.
- Keep a block to the fewest lines that run. A reader should be able to copy it
  and run it against **testnet** without editing.
- This is a testnet-only product. Any sample that touches the network says
  "testnet" once, not on every line.

## Tables

Tables must stay readable at phone width. Prefer short cells, one fact per row,
and a narrow table over a wide one. If a table needs more than about three
columns, split it into two or move the detail to prose.

## Headings

Sentence case, no trailing period. Walkthrough steps are numbered headings
(`## 1. Fund your vault`), matching the guide pages.

## Page mechanics

- Every page starts with front matter: `title` and either `nav_order` or
  `parent`. A page without it will not appear in the site navigation.
- `README.md` and `index.md` are the same content. `README.md` is excluded from
  the Jekyll build and read on GitHub; `index.md` is the one that is published.
  Change both in the same edit, keeping them identical.
- `SUMMARY.md` lists pages for readers browsing the repository. Add a new page
  to it in the section that fits.
- Every relative link must resolve. Preview locally before opening a PR.

## Screenshots

The site currently ships no images; a table or a short code block usually says
the same thing more cheaply. If a screenshot genuinely carries information none
of those does:

- capture it at the same viewport you are describing,
- caption it in one line,
- keep it narrow enough to read on a phone without horizontal scroll.

## Definition of done

- `bundle exec jekyll serve` builds and the page appears in the navigation.
- Every link on the page resolves; every code block is runnable as written.
- Terms, table width, and money units match the rest of the site.

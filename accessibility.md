---
title: Accessibility
nav_order: 15
---

# Accessibility statement

Payflow aims to make recurring payments on Stellar usable by everyone, including people who use screen readers, keyboard-only navigation, zoom, or high-contrast display settings. This page states the target standard for both the documentation site and the Payflow web application, documents current conformance and known gaps, and explains how to report accessibility issues.

## Target standard

Payflow targets **Web Content Accessibility Guidelines (WCAG) 2.2 Level AA** across:

1. **Documentation (`payflow-docs`):** Published with the [just-the-docs](https://just-the-docs.com/) Jekyll theme (`_config.yml`), providing semantic HTML5 landmarks (`<nav>`, `<main>`, `<footer>`), skip-to-content links, keyboard-operable search, and responsive tables and code blocks down to mobile viewport widths (`320px`).
2. **Subscriber and merchant flows (`payflow-frontend`):** Interfaces for connecting a Stellar wallet, funding a [vault](glossary.md), authorizing or cancelling a [mandate](protocol/lifecycle.md), and reviewing [plan](guides/merchants.md) charges.

## Core accessibility commitments

| Area | Commitment in Payflow | Where it applies |
| :--- | :--- | :--- |
| **Keyboard navigation** | Every interactive control (wallet connection, mandate creation, vault deposit/withdrawal, and mandate cancellation) is reachable and operable using `Tab`, `Shift+Tab`, `Enter`, `Space`, and `Escape` with a visible focus indicator. | [Subscriber guide](guides/subscribers.md), [Merchant guide](guides/merchants.md) |
| **Screen-reader semantics** | Form inputs, token amounts, charge intervals, and on-chain status badges expose programmatic labels and status announcements (`aria-live="polite"`) when a transaction or [keeper](protocol/trust.md) settlement updates. | [Mandate lifecycle](protocol/lifecycle.md), [Error codes](reference/errors.md) |
| **Colour and contrast** | Text and interactive controls maintain at least a `4.5:1` contrast ratio (`3:1` for large text and UI components) in the dark colour scheme, and state is never conveyed by colour alone. | Site-wide (`_config.yml`) |
| **Readable layout on small screens** | Reference tables, CLI snippets, and [TypeScript SDK](developers/sdk.md) / [REST API](developers/api.md) examples scroll horizontally without clipping or requiring two-dimensional page scrolling at `200%` browser zoom. | [Contract reference](reference/contracts.md), [Local setup](developers/setup.md) |
| **Clear protocol language** | Core terms — **mandate**, **vault**, **keeper**, **plan**, and **charge** — are used consistently and defined in the [glossary](glossary.md) so assistive-technology users do not face ambiguous labels during signing. | [Glossary](glossary.md), [Trust model](protocol/trust.md) |

## Known limitations and active work

We track accessibility improvements alongside feature work in the Payflow repositories:

- **Third-party wallet modals:** Signing a mandate or vault transaction invokes external Stellar wallet extensions (such as Freighter). Keyboard focus trapping and screen-reader announcements inside third-party extension popups depend on the wallet provider's implementation.
- **Live transaction feedback:** When submitting Soroban transactions against testnet (`deployments/testnet.env`, see [Local setup](developers/setup.md)), ledger confirmation takes a few seconds. If an error occurs, the numeric contract error is mapped in [Error codes](reference/errors.md).
- **Documentation diagrams and walkthroughs:** ASCII and code-block lifecycle diagrams in [The mandate lifecycle](protocol/lifecycle.md) are accompanied by numbered prose steps so screen-reader users receive the same sequence without relying on visual alignment.

## Testing checklist for contributors

When changing documentation pages or UI components, verify:

1. Every new documentation page includes YAML front matter (`title` and `nav_order` or `parent`) and is listed in `SUMMARY.md`.
2. Headings follow a strict hierarchy (`#` -> `##` -> `###`) without skipping levels.
3. Links use descriptive anchor text (for example, [the mandate lifecycle](protocol/lifecycle.md) rather than "click here") and resolve cleanly in the Jekyll build (`ci.yml`).
4. Tables and code blocks remain readable at mobile width (`375px`) without overlapping navigation.

## Reporting accessibility problems

If you encounter a barrier in the documentation or the Payflow application — such as an unlabelled control, a keyboard trap, low contrast, or a page that is hard to follow with a screen reader — please open an issue:

- **Documentation issues:** [MMM-pay/payflow-docs issues](https://github.com/MMM-pay/payflow-docs/issues)
- **Web application issues:** [MMM-pay/payflow-frontend issues](https://github.com/MMM-pay/payflow-frontend/issues)

When reporting, include the page URL or component name, the browser and assistive technology used (for example, NVDA, VoiceOver, or keyboard-only), and what you expected to happen. See [Contributing](developers/contributing.md) for how fixes are reviewed and merged.

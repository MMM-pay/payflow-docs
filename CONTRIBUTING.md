# Contributing to Payflow docs

This repository is the Payflow documentation: protocol design, guides and
reference. It is a [Jekyll](https://jekyllrb.com/) site built with GitHub Pages
and the [just-the-docs](https://just-the-docs.com/) theme, published to
<https://mmm-pay.github.io/payflow-docs/>.

Code lives in the other repositories — [contract](https://github.com/MMM-pay/payflow-contract),
[backend](https://github.com/MMM-pay/payflow-backend) and
[frontend](https://github.com/MMM-pay/payflow-frontend). Open documentation
changes here; open code changes there.

## Before you start

Issues are labelled by complexity (`complexity:trivial`, `complexity:medium`,
`complexity:high`) and good entry points carry `good-first-issue`. Comment on an
issue before starting so two people do not write the same page.

## Build the site locally

You need Ruby and Bundler. Then:

```bash
bundle install
bundle exec jekyll serve --livereload
```

The site serves at <http://localhost:4000>. Pages reload as you edit.

Prefer a container? Open the repository in a GitHub Codespace and run the two
commands above in the terminal.

## Writing a page

- Every page starts with YAML front matter. At minimum give it a `title`; use
  `nav_order` to place it and `parent` to nest it. Follow an existing page in
  the same section as a template.
- `README.md` and `index.md` are the same introduction for two entry points
  (GitBook and Jekyll). If you change one, change the other.
- Keep links relative (`glossary.md`, not an absolute URL) so they work both on
  GitHub and on the published site.
- Match the voice of the existing docs: short sentences, concrete nouns, no
  marketing. Define a term the first time it appears or link the
  [glossary](glossary.md).

## Before you open a pull request

Build the site once with the commands above and confirm your page renders, its
front matter is valid, and no link 404s. CI runs the same
`actions/jekyll-build-pages` build GitHub Pages uses, so a broken page or
malformed front matter fails the pull request rather than the live site.

## Pull requests

- Branch from `main`, keep the change focused, and fill in the pull request
  template.
- A scan for malware, injection and leaked secrets runs on every pull request;
  it reads your commits as data and never executes them. A clean pull request
  with passing checks is merged automatically — mark it a draft or add the
  `do-not-merge` label to hold it.
- By contributing you agree your work is licensed under the repository's
  [LICENSE](LICENSE).

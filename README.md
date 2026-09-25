# ALA Checklist and Encoding

The language-agnostic source of truth for the **ALA Checklist** (R1–R11) and its
**encoding notation** — a compact way to write a design down and read its
Abstraction Layered Architecture compliance off the shape.

> An independent, unofficial restatement based on John Spray's
> [Abstraction Layered Architecture](https://www.abstractionlayeredarchitecture.com/).
> Not affiliated with or endorsed by the author.

This repository is intentionally **non-code**: it holds the checklist and the
notation, not any one language's tooling. The current document,
[`ala_checklist_elixir.md`](./ala_checklist_elixir.md), was written against an
Elixir/Phoenix corpus and still uses Elixir examples in places; it is named with
the `_elixir` suffix to leave room for language-neutral and other-language
editions alongside it.

## Implementations

- **Elixir linter:** [`ala_lint_elixir`](https://github.com/modellurgist/ala_lint_elixir)
  — a static-analysis tool that scores a codebase against this checklist and can
  encode source into the notation.
- **Elixir example apps:** [`ala_variants_elixir`](https://github.com/modellurgist/ala_variants_elixir)
  — worked variant designs the checklist is applied to.

## License

Licensed under [CC BY 4.0](./LICENSE) — share and adapt, including commercially,
with attribution.

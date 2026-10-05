# ALA Checklist and Encoding

The language-agnostic source of truth for the **ALA Checklist** (R1–R11) and its
**encoding notation** — a compact way to write a design down and read its
Abstraction Layered Architecture compliance off the shape.

New to ALA? [getdown.dev](https://getdown.dev) has an introduction to it, guides to applying it,
and worked examples, including a walk through this checklist rule by rule.

> An independent, unofficial restatement based on John Spray's
> [Abstraction Layered Architecture](https://www.abstractionlayeredarchitecture.com/).
> Not affiliated with or endorsed by the author.

This repository is intentionally **non-code**: it holds the checklist and the
notation, not any one language's tooling. It comes in four editions with the same
rules (R1–R11), notation and citations:

- [`ala_checklist_functional.md`](./ala_checklist_functional.md): the
  language-neutral edition for functional languages. Techniques are described in
  general terms, with no language-specific code.
- [`ala_checklist_object_oriented.md`](./ala_checklist_object_oriented.md): the
  language-neutral edition for object-oriented languages, statically or
  dynamically typed. It reads each rule for classes and objects and adds what
  Spray says about object-oriented programs (ports on classes, no associations or
  inheritance, explicit wiring instead of DI containers, threads, design patterns).
- [`ala_checklist_ruby.md`](./ala_checklist_ruby.md): the object-oriented edition
  with Ruby and Rails examples: a small ports-and-wiring Foundation, paradigm
  modules, domain abstractions, compositions, tests, and where Rails' pieces sit.
  No Ruby variant has been built yet.
- [`ala_checklist_elixir.md`](./ala_checklist_elixir.md): the Elixir edition,
  written against an Elixir/Phoenix LiveView corpus. It adds Elixir and LiveView
  code for each technique, the variants each one was built in, and notes on the
  Elixir linter.

## Implementations

- **Elixir linter:** [`ala_lint_elixir`](https://github.com/modellurgist/ala_lint_elixir)
  — a static-analysis tool that scores a codebase against this checklist and can
  encode source into the notation.
- **Elixir example apps:** [`ala_variants_elixir`](https://github.com/modellurgist/ala_variants_elixir)
  — worked variant designs the checklist is applied to.

## License

Licensed under [CC BY 4.0](./LICENSE) — share and adapt, including commercially,
with attribution.

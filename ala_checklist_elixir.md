# The ALA Checklist (R1–R11)

*(A concise encoding for spotting non-ALA code, plus the procedures to build it and verify it — refer
to it as "the ALA Checklist".)*

> **If you care only about zero-coupling (not full ALA):** this checklist bundles three commitments —
> zero-coupling, the abstraction hierarchy, and requirements-on-the-diagram. The coupling part
> alone compresses to a smaller instrument (3 channels, 1 rule, 4 marks): see
> [`zero-coupling-core-companion.md`](https://github.com/modellurgist/ala_lab/blob/main/docs/zero-coupling-core-companion.md) (how it relates to this
> checklist + the language table) and [`zero-coupling-rubric-standalone.md`](https://github.com/modellurgist/ala_lab/blob/main/docs/zero-coupling-rubric-standalone.md)
> (usable on its own). The thermometer in Rust + Elixir:
> [`thermometer-rust-and-elixir.md`](https://github.com/modellurgist/ala_lab/blob/main/docs/thermometer-rust-and-elixir.md). Why coupling-only is not
> enough on its own: [`zero-coupling-without-ala-caveats.md`](https://github.com/modellurgist/ala_lab/blob/main/docs/zero-coupling-without-ala-caveats.md).
> What full ALA compliance still *doesn't* catch, with two proposed added rules (R6 nameability,
> R7 earns-its-existence): [`ugly-but-ala-compliant.md`](https://github.com/modellurgist/ala_lab/blob/main/docs/ugly-but-ala-compliant.md).

A text notation for functions and their relationships, small enough to type on a whiteboard,
that makes ALA violations *visible as shapes* instead of judgment calls. Worked against Spray's
thermometer example (site §1.6: the "bad code" and the refactor toward ALA), with the
functional-programming reading he develops later
([monads detour](https://www.abstractionlayeredarchitecture.com/#truebrief-detour-composing-with-monads),
[ALA vs FP](https://www.abstractionlayeredarchitecture.com/#trueala-compared-with-functional-programming)).

## The notation

A handful of marks; everything else is indentation.

```
name [tag] [@lvl]  a function. `name` is its real name — fully-qualified
             Module.fun/arity at scale, so it stays unique and grep-able — or a
             bare fN on a whiteboard. [tag] = the most specific requirement noun
             it KNOWS (its name, constants, field names). [] = fully generic:
             nothing in it would change if the product changed. [@name-Lidx] is
             OPTIONAL: the ALA tier it is assigned to (top = L0), tool-assigned
             from a layer map. The tag is WHAT it knows; the level is WHICH tier;
             they should agree, and a mismatch is itself a smell.
pN           a data value (a wire). p2 <- Filter(p1) : derived by a call.
*pN          shared/by-reference data (caller and callee both hold it) — an R2 channel.
$            the function keeps hidden state between calls (static, module var,
             closure). Write it on the tag: [x]$.
qN           a silent contract: a format/name/shape two functions must agree on
             that is NOT visible in any signature.
{app-literal?}    an application-literal candidate lives in this function (value
             opaque; the linter has its file:line). A human/LLM resolves it to:
{app-literal}       an application literal — belongs at the composition (R3 hoist if
               it sits lower; at the top it is correct).
{intrinsic-literal}    intrinsic to the abstraction (an identity, a physical or
               mathematical constant) — correctly local, never hoisted.
(p) ->       an anonymous wiring lambda. No name: it is composition, not an
             abstraction (Spray's func1 point).
indent       "uses" — a dependency edge from the line above it.

# Tool-stamped marks. Unlike the marks above (which need a human judgement), the
# linter can decide these from the source and writes them into the encoding, so a
# completed encoding re-lints to the same findings. Keep them as emitted.
&entity      on a module line: it shares a domain entity/struct with a peer — a
             common type couples them (R10). &aggregate = shared across too many
             peers (the R10 aggregate).
~>           on a function line: a pass-through — one caller, one cross-module
             callee, renaming a call without hiding a decision (R7-adjacent).
(branches)   on a function line: an application-layer function that itself branches.
             The top tier should compose, not decide (R11).
(private)    the function is private. Private functions don't count toward a
             module's public surface and can't be pass-throughs — the mark keeps
             those checks honest in the encoding.
```

The encoding is **not code**: it shows how abstractions relate, so it drops arithmetic,
struct syntax, and control flow, and it does not carry a literal's *value* — only that an
application-literal candidate is present (`{app-literal?}`) and, once judged, its role. If you catch yourself
copying an expression into the encoding, you have written code, not an encoding.

The tag is the one semantic ingredient a tool cannot supply; everything after it is mechanical.
Assign it by asking: *if the product were a different product, would this function's text change?*
If yes, tag it with the noun that would change ([thermo] here); if no, it is []. Subset tags are
allowed ([temp] ⊂ [thermo] ⊂ …) but one tag per project is usually enough to start.

## The rules (each is a visible shape)

- **R1 — every edge must drop.** The callee's tag must be a strict subset of the caller's.
  - `[thermo] → [thermo]` : a **peer / communication dependency** (the classic ALA violation).
  - `[] → [thermo]` : an **upward dependency** (the worst; also written `^f` for callbacks).
  - Corollary: tagged functions never nest under tagged functions, so a compliant tree is
    *shallow* — one tagged composition on top, generic leaves below. Depth is itself a smell.
- **R2 — wires meet only at the top.** A `pN` may appear in two functions' bodies only if one of
  them is the composition that passed it. The same `pN` (especially `*pN`) inside two *sibling*
  subtrees means two peers share its meaning — the meaning belongs in the wiring.
- **R3 — application literals live at the composition line.** Application literals (product-specific
  constants: thresholds, prices, labels, formats) appear as *arguments* in the top function
  (`f4(p1, 4, 8.3)`), never inside a `[]` leaf. A literal buried in a leaf silently converts `[]` to
  `[thermo]` — the tag was lying. The contrast is an *intrinsic literal* (an identity or a physical/
  mathematical constant that is part of the abstraction's own definition), which correctly stays put.
- **R4 — `$` is legitimate only inside a `[]` leaf** (state that *is* the abstraction's concept:
  a filter's memory, a sampler's counter). `$` on a tagged function is invisible coupling through
  time between application steps. The FP endpoint removes even the leaf `$` by threading state as
  a wire: `{p3, s'} <- f5(s, p2)`.
- **R5 — every `qN` must be promoted or deleted.** A contract that exists only as two matching
  literals (a format string produced here, parsed there; a name emitted here, matched there) is
  an edge the call tree cannot show. Either both ends take it as a parameter from the
  composition, or it becomes a declared, named thing both depend on downward. (This is the same
  rule the amazin variants enforce mechanically as `Web.Contracts` + `ContractPurity`.)

Anything that survives R1–R5 and still can't be given a general, product-free name is not an
abstraction — inline it into the composition as `(p) ->` wiring (R4's twin, Spray's "func1 is
not an abstraction"). That instinct is now a rule of its own (R6), because R1–R5 check *coupling
and knowledge-placement* — structure — and a program can be perfectly R1–R5-clean and still be
ugly (worked demonstration: [`ugly-but-ala-compliant.md`](https://github.com/modellurgist/ala_lab/blob/main/docs/ugly-but-ala-compliant.md)). R6–R8 add
the *design-quality* axes structure alone doesn't cover.

- **R6 — every abstraction names a learnable concept.** A named function/module must denote a
  concept a reader can learn and the language/stdlib does not already name. A meaningless name
  (`f1`, `op`, single letters) fails it; a function that merely wraps a primitive (`p(x,y) = x+y`)
  is not an abstraction — delete it; something you can't name generally is wiring — inline it. This
  is Spray's own func1 test, promoted from a coda to a rule. Partly mechanical (wraps-a-stdlib-call,
  opaque-name are checkable), partly judgement.
  - *Verify (Spray's two tests):* (1) does it **separate two worlds** (a clock separates cog-wheels
    from being-on-time; SQL separates index algorithms from "find this customer's orders")? If it
    only factors out common code, it is not an abstraction — look harder. (2) Is it **almost binary**:
    can a reader *use* it without following the reference into its body? If they must chase it to keep
    understanding, you do not have an abstraction, and the decoupling makes the code read *worse*.
- **R7 — every abstraction earns its existence.** A separate abstraction should be *significantly*
  more general than its caller, denote a concept worth naming, and not be premature extraction.
  Prefer few meaningful abstractions to many trivial ones. R7 is the deliberate counterweight to
  **R1's decomposition bias**: "push work down into generic leaves" taken zealously shatters code
  into a micro-abstraction soup that passes R1 and reads terribly; R7 pulls back.
  **Reuse is evidence, not the requirement.** Spray never demanded that an abstraction be called
  from two places — a function that truly *is* an abstraction (names a general concept, hides a
  decision) earns its place at one call site, the same way a well-named domain type does. So the
  reuse-count test is a *warning signal*, not a failure: single-use is worth a second look, not an
  automatic "inline it." A tool should report low-reuse advisorily and let a human decide; it should
  only *fail* code on reuse if the team explicitly opts in. What R7 is really trying to catch is
  **helper proliferation** — see the dedicated section below for what that actually is and the
  signals (triviality, abstraction height, fan-in/out) that separate it from useful clarifying
  abstractions.
  - *Verify (the complement of proliferation):* a real abstraction is a **cohesive whole**, *not*
    internally decomposed into named semantic sub-parts (inside, it is "a small ball of mud" where
    every line serves the one concept), and small enough to **read in isolation** (~500 lines is a
    rough cap; the real test is "readable alone"). Over-splitting a real abstraction's internals is as
    much an R7 defect as inventing a trivial one. And **reuse is a positive**: never let R7, height,
    or the pass-through check flag a genuinely shared abstraction — high fan-in is evidence it earns
    its place.
  - *Little balls of mud, and why the interior is not scored.* ALA governs the relationships
    **between** abstractions, not the purity of each one's insides. The interior of an abstraction may
    be a little ball of mud: procedural, branchy, decomposed into private helpers, and that is fine as
    long as the abstraction (a) names **one concept** (R6), (b) stays **bounded in size** (~500 lines),
    (c) keeps its internals **private behind a small public surface**, and (d) is **clean at its
    boundary** (R1, R2, R5, R9, R10 all hold where it meets other abstractions). Because of this, the
    structural checks operate on the graph of *abstractions* (modules and their public functions), not
    the raw call graph: a call from one private helper to another in the same module is internal
    decomposition, so it adds no abstraction height and is never a pass-through. The guardrails that
    keep a *little* ball of mud from quietly becoming a *big* one are (a) through (c): a module that
    stops being nameable, blows the size cap, or exposes a wide public API has leaked its mud outward,
    and *that* is the defect the checklist catches, not the internal mess.
- **R8 — the composition reads as the requirements; names and shapes serve the reader.** The top
  layer should read like the spec; config keys should name what they configure; state wires should
  be shaped for a reader. **R8 is judgement, not shape** — "reads as the requirement" cannot be
  mechanically decided the way "edges drop" can. It is a review prompt and the honest boundary of
  the checklist: R1–R7 and R10–R11 are checkable (mechanically or semi-mechanically), R9 in part; R8 marks where automated
  checking stops and human review begins. A tool should *report* the R1–R7 findings and *flag* R8
  concerns, never claim to score R8.

The next three rules were promoted from Spray's guide because each is a *constraint of its own*, not
background for an existing rule.

- **R9 — ports carry paradigm-typed data; an abstraction never names its own I/O endpoints.** An
  abstraction's inputs and outputs connect only through wires the composition sets. Its ports are
  typed by a *paradigm* (a data value, an event, a stream, a state), never by a sibling's identity or
  a domain struct, and it never references where its input comes from or where its output goes.
  *Visible shape:* a `[]` leaf takes only `pN` wires or `(p) ->` lambdas and has **no edge that names
  a peer**; a wire that carries a *named domain struct* across an abstraction boundary is the smell
  (the boundary now leaks domain meaning — this is why Clean Architecture's "required interfaces" are
  not ALA). *Verify:* for each abstraction, ask "does it name the source or destination of any
  input/output?" and "is any port typed by the domain rather than a paradigm?" Either is a defect.
  *Mechanizable:* partly — a `[]` function that references a peer module is already an R1 edge; a
  domain-typed port needs type analysis.
- **R10 — no shared entity; share an identity, keep data private.** No domain struct that carries an
  app-identity's data may be read or destructured by two features. Features share only an *identity
  key*; each keeps its own private data against it, so a new feature is a new data migration, not
  edits across the others. *Visible shape:* a `*pN` (shared value) whose type is a named domain entity
  appears inside two sibling feature subtrees. *Verify:* is any domain struct type referenced or
  destructured by ≥2 feature-layer modules? *Mechanizable:* yes — flag a struct type used across ≥2
  features. (This is exactly what the amazin variants' boundary projection and typed facts prevent.)
  - *The aggregate case (a judgement, not a hard defect):* a struct in a *shareable* (peer-ok)
    layer read by ≥2 features may be a legitimate domain abstraction (a knowledge drop, fine) **or**
    Clean Architecture's shared-Entity coupling (bad). A tool cannot tell them apart, so treat a
    shared *feature-tier* entity as a defect (above) but a shared *domain aggregate* as a prompt for a
    human. `ala_lint` reflects this split: the feature-entity check is scored; the shared-aggregate
    check runs only under `--strict`/`--super-strict` (advisory under strict, scored under
    super-strict).
- **R11 — the application (top) layer is composition only.** The top layer instantiates, configures,
  and connects; it holds *all* app-specific knowledge and *no* app-specific logic. Spray's ideal: no
  assignments, no if-statements, ~3–10% of the code, reading as the requirements. *Visible shape:*
  every line in an `[app]` function is an edge, a `pN`, or a `{app-literal}` — never a branch. *Verify:*
  what fraction of the code is the top layer, and does any top-layer function branch beyond sequencing
  wiring? *Mechanizable:* yes (top-layer size %, branch count in top-layer functions). *Relaxation:* a
  source-encoded app layer (a LiveView `handle/3`) may branch to sequence cross-feature follow-ups;
  keep the spirit — logic lives below, the top layer connects and configures.

Rules-of-thumb for using R1–R11: R1–R2 and R5 are the coupling core; R3 is requirements-locus; R4 is
state-as-a-wire; R9 is the port/interface discipline and R10 the data-sharing discipline (both
coupling-core in spirit); R6–R7 are design minimality/nameability; R11 is the composition-only top
layer; R8 is the human-judgement remainder. A mechanical tool (`ala_lint`) can check R1–R7 and R10–R11
(and R9 in part) at varying precision; treat R8 as the reason a green run is necessary but not
sufficient.

## Enforcement tiers (what a linter scores, and when)

Not every rule is the same kind of obligation. The coupling and locus rules are hard requirements; the
minimality and shape rules are prompts a reader weighs; a couple are aspirational ideals a real app
cannot fully reach. `ala_lint` encodes this as three tiers, and the classification below is current.

| tier | rules and sub-checks | when scored |
|---|---|---|
| **Required** (a violation is a defect) | R1, R2, R3, R4, R5, R6, R10, layer-validity | always (default) |
| **Advisory** (a prompt for a reader) | R7, module-size, abstraction-height, pass-through, reference-level R1 | reported by default; scored under `--strict` |
| **Aspirational** (a purity ideal, not always obtainable) | R11 (no logic at the top), public-surface (encapsulate the little ball of mud), R10-aggregate (shared domain aggregate) | reported by default; scored only under `--super-strict` |
| **Not machine-scored** | R8 (judgement), R9 (partial, via R1 + reference-level R1) | a human reads for these |

Two placements are deliberate and follow from the "little ball of mud" reasoning under R7. **R11** is
aspirational, not required, because a real LiveView composition legitimately branches to sequence
cross-feature follow-ups, so "no if-statements at the top" is a target rather than a gate.
**Abstraction-height** and **pass-through** are measured on the graph of *abstractions*, not the raw
call graph: a call inside one module is internal decomposition, so it adds no height and is no
pass-through; only real hops and public cross-module renames between abstractions count. And
**public-surface** exists so that a little ball of mud stays little: a wide public API means the mess
has leaked past the boundary.

## Background, procedures, and how to verify (the guide behind the rules)

R1–R11 are the *detection* instrument. This section holds what did **not** become its own rule:
(a) background each rule rests on, (b) the procedure to *build* an ALA program, (c) the procedure to
*refactor* one, and (d) the procedure to *verify* one is ALA. Distilled from
[`ala-design-guide.md`](https://github.com/modellurgist/ala_lab/blob/main/docs/ala-design-guide.md) and [`ala-full-summary.md`](https://github.com/modellurgist/ala_lab/blob/main/docs/ala-full-summary.md). The
constraints those docs also state (ports, shared entities, the composition-only top layer, the
abstraction-quality tests) are now R6–R7 and R9–R11, not repeated here.

### Background behind the rules (framing, not separate checks)

- **Design-time coupling, not run-time communication (behind R1–R2).** Run-time communication is
  necessary and fine, and may be circular; design-time coupling (how much of one piece you must
  understand to read another) is what ALA drives to zero. Classify a dependency by when it first
  breaks: only a design-time break (the code *loses meaning*) matters. So circular *wiring* is fine
  and common (feedback, undo, restore); only circular *design-time dependencies* are forbidden, and
  they never arise once every edge drops. *Dependency* is what the compiler sees; *coupling* is what
  the brain sees.
- **Zero coupling is not "loose coupling" (behind R1–R2).** ALA does not *minimize* dependencies; it
  *eliminates the bad ones and maximizes the good ones*. The real enemy is **collaboration coupling**
  (one unit doing specifically what another needs, in a fixed arrangement) which an interface hides
  but does not remove and which grows during maintenance.
- **A knowledge dependency may target *any* lower layer (behind R1).** ALA is *not* "depend only on
  the immediate layer below" (that is partition layering). A big altitude skip (a "long drop") is
  **not** a smell — do not flag it. Understanding one composition line may need several lower layers
  (domain, paradigm, wiring operator, host language, ALA itself); name those in the readme. An end
  user of the composition needs none of them.
- **Compose, don't decompose (behind the layer model and R11).** Invent and assemble general
  abstractions (Lego), don't split the system into specific collaborating parts (jigsaw); a good
  test is whether the instances are composable in *other* arrangements. **Separate by feature first,
  not by tier** (splitting UI-from-logic-from-storage couples them, since UI shapes logic). **Layers
  replace hierarchical containment** — flat, no nesting, no sub-abstractions. **No inheritance** (it
  is usually "lazy composition"; replace with explicit pass-through — a subclass is more specific and
  must know its parent, an upward dependency), **no global event names, no subscribing to a specific
  sender** (those are peer edges in disguise; the composition wires them).
- **Ports go sideways into technical domains (behind R9).** Database, hardware, and network are
  run-time dependencies off to one side, reached through paradigm ports (Hexagonal spokes), not
  bottom layers and not downward adapters. Keep a *domain abstraction of* the DB or UI (configurable,
  composable — a persistent `Table` wires to a grid), not a raw port to an adapter. An OSI-style stack
  becomes a sideways chain of domain abstractions.
- **Two phases, and the diagram is the source (behind R11).** Wire the network once, then run it (an
  OTP supervision tree is exactly this). The topology/"diagram" is the single source of truth for
  requirements, architecture, and code, and should be flat, declarative, and greppable. (V30's
  committed codegen is one way to make that mechanical.)
- **Abstraction is not the same as instance (behind R6/R9).** The abstraction is the zero-coupled
  design artefact; the instance is the run-time thing that communicates. Blurring them is what tempts
  people to put dependencies *between abstractions* to move data, destroying them as abstractions.
- **Execution-model choices are design-time, made on performance/expressiveness grounds.** Push by
  default, pull for performance; decide sync vs async at wiring time (`cast` vs `call`, PubSub vs
  `send`), never inside a domain module; model time-spanning activities as state machines, not
  threads; cross-thread wiring is async (GALS — the BEAM default).

### Procedure — build a new ALA program

1. **Iteration zero (≤ one sprint, whatever the project size).** Go through requirements one by one,
   fast (aim ~one feature/hour), *describing* each as a wiring of instances and **inventing domain
   abstractions and paradigms as you go** (each gets config params for the specifics). Decide
   app-layer vs domain-layer by *scope of knowledge* (specific to this app → app; reusable → domain).
   Output: a topology plus a named list of abstractions, each with a one-line insight, **no
   implementation yet**. This is not waterfall — the rest is zero-coupled abstractions, so
   implementation cannot feed back and force redesign.
2. **Invent the abstractions** by mining the requirements for recurring nouns; set aside anything
   that "even begins to look like implementation" as a seed; start with the UI (easiest to see), then
   connect it to data sources and transforms; turn any one-off module into a *general part* plus a
   *config part*. The target for each: it **separates two worlds** (R6) and does not know its own I/O
   endpoints (R9). For a recursive concept, forward-declare its abstract interface one layer down and
   let concretes both provide and accept it — never a circular dependency.
3. **Choose each paradigm's execution model** on performance/expressiveness grounds (push vs pull,
   sync vs async), keeping that choice out of the abstraction and in the wiring.
4. **Implement, then wire.** Each abstraction is an independent little program, fast to write because
   there is no coupling to reason about. Use **convention over configuration** (enforced config in the
   constructor, optional settings defaulted); give every instance a **`Name`** for debuggability; put
   a **root readme** naming the knowledge prerequisites (ALA, the paradigms, the domain abstractions,
   "the diagram is the source"). Velocity climbs as the domain matures.

### Procedure — refactor an existing program to ALA

- **Spike-then-refactor** (also the fallback when you can't see the abstractions): get it working
  messily, then (1) move requirement-detail up to the app layer (R11/R3), leaving generalized units
  with params/config; (2) merge similar generalized units, their differences becoming config; (3) cut
  the peer calls — pass in functions/ports (a function passed in mid-computation *is* a port), and
  move any observer/subscribe *registration* up to the composition.
- **Legacy, per user story:** reverse-engineer the method-call tree for one story (an all-files
  search is legitimate here), pin it with acceptance tests at its input/output boundaries, factor the
  call-tree into a new domain abstraction (copy-pasting useful snippets), mark the old classes for
  deprecation, and repeat per story.

### Procedure — verify a program is ALA

Run the mechanical checks first (a tool can do R1–R7 and R10–R11, and R9 in part), then apply the
human tests it cannot. Fail any and it is not ALA.

1. **The two-minute test:** every unit answers "what do you know about?" with *one coherent thing*,
   and does **not** know the source or destination of its own inputs and outputs (R6, R9).
2. Read one abstraction **in isolation** without chasing into its siblings; if you must, coupling is
   still total (R6, R8).
3. Every dependency targets something **significantly more abstract** — no peer, no more-specific
   helper (R1).
4. Every would-be collaboration is a **wiring line in a higher layer**, so the graph is acyclic and
   layered (R1).
5. The **top layer is composition + config**, reads as the requirements, and is a small fraction of
   the code (R11, R3, R8).
6. **No domain abstraction encodes data meaning or shares an entity** with a peer (R5, R9, R10).
7. Tracing a user story needs **no all-files search and no run-time debugger** — the flow is explicit
   and cohesive in one place (R1, R8).

## Helper proliferation: the real problem R7 is chasing

The naive reading of R7 is "flag functions that aren't reused." That reading is wrong on both ends,
and worth pulling apart, because getting it wrong either nags good code or misses bad code.

**Reuse is neither necessary nor sufficient.** A single-use function can be a perfectly good
abstraction: `def discount(subtotal, tiers)` called from one place still *names a decision* and
hides *how* the decision is made — that is what an abstraction is for, and Spray's ALA never
required a second caller to justify it. Conversely, a bad helper called from three places is still a
bad helper — reuse count says nothing about whether the thing names a concept. So counting call
sites, alone, is a blunt instrument that produces both false positives (nagging legitimate
single-use abstractions) and false negatives (blessing a widely-called pass-through).

**The actual smell is proliferation of *thin wiring dressed up as abstraction*** — many small
functions that add a name and a navigation hop without hiding any decision. The reader pays the
cost of the extra name and the jump, and gets no concept in return. That is distinct from a
*clarifying abstraction*, which earns its name by hiding a choice, a format, a formula, or an
effect. The distinguishing question is not "is it reused?" but **"does it hide a decision a reader
would otherwise have to make or read?"** If yes, keep it at one call site; if no, it is `(p) ->`
wiring and belongs inlined into the composition (R4's twin).

Better signals than raw reuse-count, in rough order of how mechanically reliable they are:

- **Triviality + meaningless name (already scored, conservatively).** A one-liner whose body is a
  single call or literal *and* whose name teaches nothing (`op`, `f1`, `do_it`) hides no decision.
  This is the one R7 signal the linter fails on, and only when *both* conditions hold — a
  well-named one-liner is left alone on purpose.
- **Nameability (R6).** If you cannot give it a general name, it is wiring, not an abstraction.
  This catches proliferation from the naming side and needs no call-graph.
- **Abstraction height (now measured; advisory).** The longest chain of knowledge-dependency edges.
  Thin pass-throughs stack up: `A → B → C → D → …` where each layer only forwards. A real
  requirement rarely needs more than a handful of altitudes, so `ala_lint` reports the height and
  **warns past a configurable ceiling (default 5)**. Height is a *proliferation* signal precisely
  because inlining wiring collapses chains, while genuine abstraction layers stay shallow and wide.
- **Pass-through shape (fan-in / fan-out), now built.** A function with exactly one caller *and*
  one project-internal callee, whose body is just that callee, is a rename — a strong proliferation
  signal. High fan-in (many callers) is the opposite: evidence of a genuinely shared abstraction,
  and is never flagged. `ala_lint` implements this as an **advisory** `passthrough` detector on the
  function call graph: it scores "1-in, 1-out, body-is-a-single-call" rather than raw reuse-count,
  so it targets the pass-through directly and spares the useful single-use abstraction (a helper
  that hides a *decision* — a branch, a formula, a format — is not a bare delegating call, so it is
  left alone). Like R7 and height, it is reported but not scored unless `--enforce passthrough`.
- **Altitude of the edge (R1).** A "helper" at the *same* altitude as its caller that only moves
  data is a communication dependency, not an abstraction — R1 already covers that. Genuine
  abstractions sit *below* their callers.

Because of all this, `ala_lint` treats R7 and abstraction-height as **advisory by default**:
reported, but not folded into the score, so they never fail a build on their own. A team that wants
to enforce them opts in with `--enforce r7` / `--enforce height`. This matches the checklist's stance —
reuse and minimality are prompts for a reader, not gates — and it keeps the tool from punishing the
very "abstractions created for their own sake" that ALA permits.

### A note on modules (Elixir `ala_lint` specifically)

The checklist's encoding is *function-centric*: `f [tag]`, edges between functions, no module concept
required. The reference Elixir linter, though, **does use modules**, in three concrete ways: (1) the
dependency graph that drives R1 and abstraction-height is module→module, not function→function;
(2) R7's dead/single-use check is scoped to a module's *private* functions (privates are the
module-local, unambiguously-inlinable case); (3) `[tag]`s and the layer map attach to module names.
That is a pragmatic concession to Elixir, where the module is the natural unit of compilation,
privacy, and naming — not a claim that ALA needs modules.

Deliberately, the linter does **not** constrain *which functions a module may contain* — there is no
rule of the form "all functions in a module must share a layer/tag." Such a rule would import a
containment hierarchy that ALA explicitly replaces with layers, and it would break the
function-centric encoding (which can describe a design with no modules at all). Modules are how the
Elixir tool *locates* functions, not a compliance constraint in their own right.

The finer-grained direction is now built. `ala_lint` has a **function→function call graph** (node =
a specific `module.name/arity`, edges resolved through each module's alias table so that
`Cart.add_item(...)` becomes `Shop.Cart.add_item/2`), and the checks that need altitude —
**R1 altitude**, **abstraction height**, and the **pass-through detector** — run on *that* graph.
Only **R1 cycles** (the no-layer-map fallback) stays at module granularity, deliberately: a
function-level cycle is usually legitimate mutual recursion, whereas a *module* cycle is the real
coupling smell.

### Assigning functions to layers (convention + tags, never inference)

Layers are a *build-time* decision, so a linter cannot infer them from structure without making
R1 tautological (define altitude from the edges and every edge "drops" by construction; only cycles
remain visible). Instead `ala_lint` maps each function to a **user-declared** layer, resolved
**most-specific-wins**:

1. the function's own `@ala_layer :name` attribute (the exceptional per-function override — it lets
   functions of different layers legitimately share a module, since the encoding is function-centric);
2. else the first layer in the ordered map whose **module-name pattern** or **filesystem path glob**
   (`paths:`) matches — so a project declares its own convention (e.g. `~r{/features/}` → `:feature`)
   rather than inheriting magic directory names;
3. else **unassigned**.

This yields three separate checks, kept distinct because they fail for different reasons: **coverage**
(what fraction of functions are assigned — the migration metric, with the unassigned list as the
worklist), **validity** (a tag naming an undeclared layer — a typo or stale tag, a hard error), and
**behaviour** (R1 — do the assigned edges drop?). The convention must be *written in the layers
module* and is echoed in the params block: convention is fine, but invisible convention is how R1
silently goes wrong. On a codebase not yet organised into layers, everything is unassigned, coverage
reports 0%, and the structural checks that need no layers (module cycles, R7, height, pass-through)
still run — the on-ramp for moving an existing app toward ALA.

Deliberately, there is still **no** rule constraining which functions a module may contain; modules
locate functions, they are not a compliance unit. (Nested modules are *qualified* by their enclosing
module — `defmodule Address` inside `…CheckoutFlow` is `…CheckoutFlow.Address` — so a feature and its
own nested value-objects read as one unit and their calls are cohesion, not peer coupling.)

### The application layer is a special case (and so is LiveView)

Spray's ALA singles out the top **application/composition layer**: it is *wiring*, not abstraction,
so it is the one layer where a function may legitimately call a **peer** — the composition's whole
job is to connect abstractions to each other. The checklist and linter both honour this, and a few
related accommodations matter especially for LiveView, where the variants show the application layer
is not flat but has **sub-layers** (shell → page/composition → view/components):

- **Peers are allowed in the application layer.** Declare that layer `peer_ok: true`; same-layer
  calls there are not flagged. Feature and domain layers stay `peer_ok: false` (peer coupling there
  is the classic smell). When a human runs the checklist, apply the same rule: a same-altitude call is
  a violation *below* the composition, and expected *at* it.
- **Sub-layers are just more layers.** To model shell-above-page-above-view, declare them as
  separate ordered tiers (each `peer_ok: true`). Then shell→page→view all *drop* and are fine, while
  page→shell would be flagged as upward — which is exactly right. The linter supports as many tiers
  as you declare; there is nothing special-cased about "app," only about `peer_ok` and order.
- **Application literals live in the composition/config tier.** R3 exempts *config layers* from the
  literal check — by default the top layer, or any tier you mark `config: true` (so an app
  with sub-layers can hold its "diagram config" in whichever sub-layer owns it). A human mirrors
  this by not counting application literals that sit *at* the composition against R3.
  > **"Config module / config layer" names a *place*, not a value.** It is where application
  > literals are *allowed* to live — the composition, or a manifest standing in for it — as
  > opposed to the application/intrinsic distinction, which is about the *value*. Two knobs express
  > the same idea: `config: true` marks a config layer (for a layered codebase), and
  > `--config-module Foo` / `config_modules` names config/manifest modules (for a codebase with no
  > layer map). The `config_modules` list is consulted **only** when no layer map is given; with a
  > layer map, `config: true` (defaulting to the top layer) is what R3 uses.
- **High fan-out at the composition is expected, not a smell.** The composition calls *many*
  features; that inflates neither the pass-through detector (which needs 1-in/1-out) nor a fair
  reading of R7. Depth (abstraction height) counts *chains*, so a wide, shallow composition stays
  low — as it should.
- **Calls *within* the application layer don't add height.** The app layer is one altitude even
  when it has sub-layers (shell → page → view): those are wiring, not deeper abstraction. So when
  computing abstraction height, a reader (and the linter) counts the whole app layer as **1**, and
  height accrues only once a chain drops *below* it into feature/domain/platform. Proliferation
  *below* the app layer still counts fully — the collapse is app-layer-specific. (Declare app tiers
  with `app: true`; the top layer is the default.)
- **The view/template layer hides contracts and calls.** `~H` markup is opaque to the AST, so
  cross-feature calls and server↔DOM contracts embedded there escape function-level R1 and R5. The
  linter's reference-level R1 advisory and the P-family supplement partially cover this; a human
  running the checklist must **read the templates** — this is the single biggest thing the linter
  cannot see and the human necessarily can.

### Running the checklist by hand to compare with the linter

The linter's method is a mechanical shadow of the manual checklist, so a reviewer can reproduce and
extend it. The correspondence: (1) give every function its `[tag]` — the linter approximates this
with the layer map's convention + `@ala_layer` tags (its **coverage** = how many you'd have tagged);
(2) walk every call edge and confirm it **drops** — the linter scores this on the function graph and
flags reference/template edges advisorily; (3) check application literals sit at the composition (R3),
state is threaded not hidden (R4), no silent contracts (R5), names/earns-existence hold (R6/R7).
Where the linter stops, the human continues: right-boundary judgement, template contents, semantic
(not textual) contracts, and whether an abstraction is the *right* one. A green linter run is the
floor; the manual pass is the ceiling, and is always the more complete of the two.

## What a linter can and cannot check (per rule) — why human evaluation is still required

Automated checking (the `ala_lint` reference tool) captures a *subset* of each rule. The recurring
reason it can't capture the rest: the rule turns on the **`[tag]` judgement** (does this code carry
product knowledge, and at what altitude?), which a linter can only approximate with syntactic
proxies or a **hand-declared layer map** — and even then, "is this the *right* abstraction" is not
decidable. Read each note as "what a human must still judge."

- **R1 (edges drop).** *Automatable, given a layer map:* the linter checks altitude on the
  **function call graph** (each `Module.fun/arity` a node) against each function's assigned layer —
  upward edges and cross-peer edges are scored. Because it sees only real Elixir calls, a
  cross-feature call **inside a `~H` template is invisible to it**; it recovers that with an
  *advisory* **reference-level** check over the module alias/reference graph (which does see the
  `alias`), reported for a human to confirm — this is the LiveView case that most needs eyes.
  *Human must judge:* whether the declared boundaries are the *right* ones; whether a same-layer
  call is genuine cohesion or disguised peer coupling (the tool uses a `unit:` hint and cannot tell
  a legitimate collaboration from a smell); and every reference-level advisory (is that aliased peer
  actually called in a template?). Without a layer map the tool degrades to module **cycle
  detection** — a small subset of R1.
- **R2 (no shared mutable state between peers).** *Automatable:* presence of `:ets`/`Agent`/
  process-global APIs. *Human must judge:* whether a given shared store is actually a *back-channel
  between peers* vs a legitimate single-owner cache — the linter sees the API, not the sharing
  topology. (In immutable languages this rule is largely satisfied for free, which *inflates*
  automated scores independent of design quality.)
- **R3 (application literals on the diagram).** *Automatable, given a layer map:* magic literals outside the
  composition layer. *Human must judge:* whether a literal is an *application literal* (should hoist) or an
  *intrinsic literal* (a validation regex, a physical constant that belongs in its
  abstraction), and whether the "composition" is really where requirements should read.
- **R4 (state threaded, not hidden).** *Automatable:* the process dictionary; some `Agent`/`:ets`
  stashing. *Human must judge:* whether GenServer/process state is legitimate instance state or a
  hidden cross-call channel that should have been a wire — a semantic distinction.
- **R5 (no silent contracts).** *Automatable:* duplicated identifier strings across modules;
  (extendable to) tagged-tuple message shapes. *Human must judge:* contracts that are **invisible
  to the AST** — anything inside `~H`/templates/EEx, cross-language constants (server string ↔ JS),
  or two ends that agree via *different* literals (a produced format parsed elsewhere). The tool
  sees textual duplication, not semantic agreement.
- **R6 (nameability).** *Automatable (weakly):* single-letter/`f\d` names, functions that wrap one
  primitive. *Human must judge:* the actual rule — "does this name a **learnable concept**?" A
  meaningful predicate (`empty?`) trips the primitive-wrapper heuristic (false positive); a
  meaningless-but-plausible name (`process2`, `handle_stuff`) passes it (false negative). Naming
  quality is irreducibly judgement.
- **R7 (earns its existence).** *Automatable (advisory):* dead code; trivial single-use one-liners;
  abstraction height past a ceiling. These are reported but **not scored by default** — reuse is
  evidence, not a requirement (see "Helper proliferation" above), so the tool never fails code on
  them unless the team opts in (`--enforce r7`/`--enforce height`). *Human must judge:* the core —
  "does this hide a decision worth naming?" A single-use named step can be excellent decomposition
  or premature extraction; only a reader (or the port-vs-prop / second-consumer test) decides.
- **R8 (reads as the requirements; names/config/shape serve the reader).** *Not automatable at
  all* — pure judgement. "Does the composition read as the spec?" and "are these good names?" have
  no shape-based decision procedure. R8 is the honest boundary: a linter can *prompt* it, never
  score it.

Two whole-design properties **no single-snapshot linter can check** (they need more than the code):
**port-vs-prop / reuse** (does an abstraction wire into ≥2 consumers unchanged? — needs a second
consumer, i.e. the D6 experiment) and **requirements coverage / correctness** (does the wiring
encode the *right* spec? — needs the spec). These live in R8 and in experiments, not in the tool.

**Consequence:** a green `ala_lint` run means "no gross, mechanically-detectable coupling smells at
the module-graph + literal level" — a useful *floor*, especially for ruling out clearly-bad
codebases cheaply. It does **not** establish ALA conformance. The high-fidelity, discriminating
judgements (R1 altitude *correctness*, R3 application-literal-vs-intrinsic, R5 semantic contracts, R6/R7
abstraction quality, all of R8, and reuse) require a human applying the checklist.

### The encoding and the linter agree (tool-stamped marks)

The linter runs two ways: over **source** (`mix ala.lint`) and over a **completed encoding**
(`mix ala.lint.encoding`). They should reach the same findings, so the manual notation stays a
faithful hand-tool. The gap used to be that some checks moved into AST/call-graph analysis the
encoding could not represent — R10 (a shared domain entity), R11 (branching at the top), the
public-surface and pass-through advisories. The fix keeps the *judgement* marks human-supplied
(tag, `$`, `q`, application-vs-intrinsic literal) but has `mix ala.encode` **stamp the facts it can
already decide** into the draft: `&entity`/`&aggregate` (R10), `~>` (pass-through), `(branches)`
(R11 top-level logic), and `(private)` (so public-surface and pass-through scope correctly). A human
completes the `[?]` tags and resolves each `{app-literal?}`; the stamped marks are kept as emitted.

What still can't cross over: R6/R7 (nameability, earns-existence — pure judgement, never encodable),
and two **approximations** flagged as such — abstraction *height* (the encoding carries module-level
edges, not the call-graph in-degrees the source uses) and the R11 *app-share* aggregate (it assumes
level 0 is the application tier). Everything else re-lints identically.

### Worked check — the ALA thermometer, encoded and re-linted

The blog thermometer (`OffsetAndScale`, `LowPassFilter`, `SampleEvery`, `Display` as `[]` leaves;
`Thermometer` as the `[thermo]` composition) encodes to this, `[?]` tags resolved by a human and the
tool marks kept:

```
module OffsetAndScale   [linear calibration]  @domain-L1
  f OffsetAndScale.apply/2   [linear calibration]
module LowPassFilter    [smoothing algorithm] @domain-L1
  f LowPassFilter.smooth/2   [smoothing algorithm]
module SampleEvery      [decimation algorithm]@domain-L1
  f SampleEvery.tick/2   [decimation algorithm]
module Display          [state + presentation]@domain-L1
  f Display.record/2   [state + presentation]

module Thermometer      [this product's wiring]@app-L0
  f Thermometer.new/1   [this product's wiring]   {app-literal}   -- calibration numbers, at the top ✓
  f Thermometer.push_reading/2   [this product's wiring]  (branches)   -- ⚠ R11
  depends on:
    → OffsetAndScale @domain-L1   drops ✓
    → LowPassFilter  @domain-L1   drops ✓
    → SampleEvery    @domain-L1   drops ✓
    → Display        @domain-L1   drops ✓
```

`mix ala.lint` on the source and `mix ala.lint.encoding` on this encoding both report exactly two
R11 items and nothing else: `push_reading/2` branches at the top (the two `if`s that null-guard the
sampler and display stages), and the application layer is 33% of functions (2 of 6) — both
*aspirational* (super-strict), so the default score stays 100/A. The `{app-literal}` on `new/1` is
the thermometer's calibration; it sits at the composition (level 0), so it is correctly placed and
raises no R3 — the encoding-linter uses the same "application literals allowed only in the top tier"
rule the source R3 uses. No edge is upward or peer, no `$`, no `q`, no shared entity: the good shape
reads the same from either direction.

## Worked example 1 — Spray's bad thermometer (site §1.6.1)

> The two worked examples below use the compact whiteboard rendering (`fN`, and requirement
> constants shown inline as `+4`, `*8.3`) because it makes the R3 shape vivid on a page. The
> current tool and blog rendering is stricter: real names (`Module.fun/arity`), application literals shown
> as opaque `{app-literal}`/`{intrinsic-literal}` marks rather than values, and an optional `@name-Lidx` level.
> The *shapes* being read off — peer edges, `$`, scattered literals — are identical either way.

`main / ConfigureTemperaturesAdc / GetTemperaturesFromAdc / ReadAdcChannel /
ProcessTemperatures / SmoothTemperature / ResampleTemperature / DisplayTemperature`:

```
f1 [thermo] :                       -- main
  f2 [thermo]                       -- ConfigureTemperaturesAdc
  loop:
    f3 [thermo] (*p1)               -- GetTemperaturesFromAdc: fills the batch
      f8 [] (2)                     -- ReadAdcChannel(2)
    f4 [thermo] (*p1)               -- ProcessTemperatures
      p2 <- (p1[i] + 4) * 8.3      -- ⚠ R3: application literal inline, two layers down
      f5 [thermo]$ (p2)             -- SmoothTemperature: static filtered  ⚠ R4
      f6 [thermo]$ (p2)             -- ResampleTemperature: static counter ⚠ R4
        f7 [thermo] (p2)            -- DisplayTemperature                  ⚠ R1 (depth 3)
```

Read the violations straight off the shape:

| shape | rule | what it is in the code |
|---|---|---|
| every edge is `[thermo] → [thermo]` (except f3→f8) | R1 | all pieces collaborate to "be a thermometer"; you must read all of it to understand any of it |
| `*p1` appears in siblings f3 *and* f4 | R2 | the adc-batch buffer's meaning is shared between two peers, not owned by the wiring |
| `+4`, `*8.3` inside f4; `15` inside f6; `9/10` inside f5 | R3 | requirement constants scattered two layers deep |
| `$` on tagged f5, f6 | R4 | hidden per-reading state inside app-specific code — invisible coupling through time |
| f4 → f6 → f7 chain | R1 | the story of one reading spans a three-deep chain of thermometer-knowing functions |

Note what the *un*-annotated sketch could not show: strip the tags and the bad tree and the good
tree below look similar. The tag column plus the literal rule is what makes the difference
mechanical rather than aesthetic — the encoding needs exactly that much semantics, no more.

## Worked example 2 — the ALA thermometer (site §1.6.2–.4)

`main / ConfigureAdc / GetAdcReadings / OffsetAndScale / Filter / SampleEvery / FloatToString /
Display / foreach`:

```
f1 [thermo] :                        -- the application: the ONLY tagged function
  f2 [] (2, 100)                     -- ConfigureAdc(channel, batch)
  loop:
    f3 [] (*p1, 2, 100)              -- GetAdcReadings
    foreach(p1, (p) ->               -- wiring lambda: anonymous, un-numbered
      p2 <- f4 [] (p, 4, 8.3)        -- OffsetAndScale       (literals at the top ✓)
      p3 <- f5 []$ (p2, 10)          -- Filter               ($ inside a [] leaf ✓)
      if f6 []$ (15):                -- SampleEvery
        f8 [] ( f7 [] (p3, "#.#") )  -- Display(FloatToString(...))
    )
```

Every edge drops `[thermo] → []`; the tree is depth 1 below the composition; every requirement
literal (channel 2, batch 100, offset 4, scale 8.3, strength 10, every-15, format "#.#") sits on
the composition line; each `pN` flows only through the top; `$` survives only inside generic
leaves whose *concept* is stateful. The application layer now reads back as the requirement —
which is the point.

Two honest residues, so the encoding doesn't oversell:

- `*p1` remains (the DMA buffer) — platform mechanics, tolerated at the boundary; mark it and
  say why, don't hide it.
- `f8(f7(...))` : Display consumes exactly what FloatToString produces — a latent `q1` (the
  string-format contract). Harmless here because the composition line owns both calls; it becomes
  a real R5 violation the day Display parses the string.

### The FP endpoint (why the good shape collapses into a pipe)

Once every edge drops a layer, the composition is free to become *pure* composition — Spray's
monads detour, and literally the Elixir port in `ala_lab` (`Thermometer.push_reading/2` with
`OffsetAndScale`, `LowPassFilter`, `SampleEvery`, `Display` as structs):

```
{s5', s6', out} <-
  p |> f4(4, 8.3)
    |> f5(s5, 10)          -- state is now a wire the composition threads
    |> maybe(f6(s6, 15))   -- conditional propagation = the Maybe bind
    |> f7("#.#")
    |> f8
```

R4's `$` disappears entirely: leaf state (`s5`, `s6`) is owned and threaded by the composition,
so time-coupling is visible as data flow. This is the FP generalization of ALA's
instances-with-state, and it is why the encoding is FP-shaped: *a compliant tree is exactly one
that can be rewritten as a pipe over generic stages.*

## How to use it on your own code

1. **Pick one entry point** (a request handler, an event, main). List the functions it
   transitively reaches — that's your `f` set. Keep real names in trailing comments; the ids
   force you to judge structure, not naming.
2. **Tag honestly.** For each f: would its text change if the product changed? Names, constants,
   field accesses, and comments all count. When in doubt, tag it — an over-general tag hides
   violations; an over-specific one only costs you a false alarm.
3. **Write the tree**: indentation for calls, `pN` for the data, `*`/`$` where you see them,
   `qN` for any matching-literal pair you know about.
4. **Circle the shapes**: non-dropping edges (R1), a `p` in two sibling bodies (R2), literals
   below the top line (R3), `$` on a tagged f (R4), undeclared `q`s (R5), and any f you cannot
   name without a product noun (inline it as `(p) ->`).
5. **Refactor by shape**, one move per shape:

| shape found | the move |
|---|---|
| `[X] → [X]` peer edge | lift the call into the composition; the callee's input becomes a `p` the top passes |
| `[] → [X]` or `^f` upward edge | invert: pass the specific behavior *in* as a lambda/port parameter |
| `p` shared by siblings / `*p` | the composition derives and hands each sibling exactly the value it needs |
| literal in a leaf | hoist it to an argument at the composition line (it is requirements content) |
| `$` on a tagged f | extract the stateful *concept* into a generic leaf — or thread the state as a wire |
| undeclared `q` | make it a parameter from the top, or a named contract both ends depend on downward |
| un-nameable f | it was wiring all along: inline it as `(p) ->` |

The end state is recognizable at a glance: **one tagged function per requirement cluster, at the
top, carrying all the literals, over a shallow fringe of `[]` leaves — a tree you could rewrite
as a pipe.**

## Where even the good shape can lie (keep these in the guidance)

1. **Tags rot.** A `[]` leaf acquires a product constant in maintenance and nothing in the call
   tree changes. R3 is the tripwire — re-scan leaves for literals, not just edges. (Mechanized
   at scale, this is exactly a `CorePurity`-style check.)
2. **Contract coupling is invisible to call trees.** Two `[]` leaves agreeing on a format/name/
   topic (`q`) are peers in disguise; only R5 catches it. (Mechanized: `ContractPurity`.)
3. **Callbacks and pub/sub reintroduce upward edges** that indentation can't show — a leaf that
   invokes a registered handler is `^f`. Write the `^`; the fix is always "the composition wires
   it," never "the leaf knows whom to call."
4. **A perfect shape can encode the wrong program.** The encoding audits *structure*
   (coupling/knowledge placement), not correctness — same caveat the requirements-coverage
   analysis makes for manifests.

---

*Origin note: this file started as a numbering sketch of the two trees; the worked forms above
fix the f-numbering against Spray's actual §1.6 code and add the `[tag]`/`$`/`*`/`q` marks —
without the tag column, the bad and good trees are nearly the same shape, which is what the first
sketch ran into. The Elixir pipe form is grounded in
`ala_lab/lib/ala_lab/ala/examples/thermometer.ex` and its `domain_abstractions/`. The `q` rule
and the mechanized-check parallels come from the V32 work
(`ala_lab/docs/v32-d6-build-results.md`).*

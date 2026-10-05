# The ALA Checklist (R1–R11)

*(A concise encoding for spotting non-ALA code, plus the procedures to build it and verify it — refer
to it as "the ALA Checklist".)*

> **If you care only about zero-coupling (not full ALA):** this checklist bundles three commitments:
> zero-coupling, the abstraction hierarchy, and requirements-on-the-diagram (the top layer reads as
> the requirements). The coupling part alone compresses to a smaller instrument. Two peers can be
> coupled in only three ways: one names or calls the other, they share mutable state, or they agree
> on a name or format that neither signature shows (a silent contract). One rule covers all three:
> no such channel connects two peers except through a lower layer or the composition above them.
> It can be used on its own, but it isn't sufficient: a program can be perfectly zero-coupled and
> still be *ugly*, which is what the nameability (R6) and earns-its-existence (R7) rules add on top.

A text notation for functions and their relationships, small enough to type on a whiteboard,
that makes ALA violations *visible as shapes* instead of judgment calls. Worked against Spray's
thermometer example (site §1.6: the "bad code" and the refactor toward ALA), with the
functional-programming reading he develops later (gathered under "Functional programming" below)
([monads detour](https://www.abstractionlayeredarchitecture.com/#truebrief-detour-composing-with-monads),
[ALA vs FP](https://www.abstractionlayeredarchitecture.com/#trueala-compared-with-functional-programming)).

> **Citations.** Section numbers such as §3.8 or §1.6.4 refer to John Spray's site,
> [abstractionlayeredarchitecture.com](https://www.abstractionlayeredarchitecture.com/), which is
> one long page with numbered sections. "Summary" is its unnumbered opening section. Each rule ends
> with a *Spray:* line naming the sections that support it, so you can check the rule against the
> source. Where a rule adapts Spray to functional languages or departs from him, the line says so.

## The notation

A handful of marks; everything else is indentation.

```
name [tag] [@lvl]  a function. `name` is its real name — fully-qualified
             Module.fun/arity at scale, so it stays unique and grep-able — or a
             bare fN on a whiteboard. [tag] = the most specific requirement noun
             it KNOWS (its name, constants, field names). [] = fully generic:
             nothing in it would change if the product changed. [@name-Lidx] is
             OPTIONAL: the ALA tier it is assigned to (top = L0), tool-assigned
             from a layer map (a declared list that assigns modules to layers; see
             "Assigning functions to layers"). The tag is WHAT it knows; the level is WHICH tier;
             they should agree, and a mismatch is itself a smell. R1 is checked
             on levels; on a whiteboard the tag stands in for them.
pN           a data value (a wire). p2 <- Filter(p1) : derived by a call.
*pN          shared/by-reference data (caller and callee both hold it) — an R2 channel.
             With immutable values there are no shared references; the forms are a
             mutable reference cell, a global table, a process or actor both talk to,
             or a session slot two features read.
$            the function keeps hidden state between calls. In C that's a static or
             module variable; a functional language usually has neither, so it's a
             mutable cell or global table, or an actor's state used as a hidden channel.
             Write it on the tag: [x]$.
qN           a silent contract: a format/name/shape two functions must agree on
             that is NOT visible in any signature.
{app-literal?}    an application-literal candidate lives in this function (value
             opaque; the linter has its file:line). A human/LLM resolves it to:
{app-literal}       an application literal — belongs at the composition (R3 hoist if
               it sits lower; at the top it is correct).
{intrinsic-literal}    intrinsic to the abstraction (an identity, a physical or
               mathematical constant) — correctly local, never hoisted.
(p) ->       an anonymous wiring lambda (an anonymous function or partial application). No
             name: it is composition, not an abstraction. (Spray's "func1" point: in
             §1.6.3 he names a helper func1 because it has no meaning of its own; it
             only connects two points in the code, so it shouldn't be a named function.)
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

Two kinds of line are easy to mix up. Indentation shows a **knowledge dependency**: the code of one
function uses another, more abstract one by name. A `pN` shows a **wire**: data or events passing
between instances at run time, connected by the layer above. Spray draws only wires on his
diagrams: "We never actually draw lines when using abstractions" (§2.1.3). The encoding shows both
because the rules are about both.

The tag is the one semantic ingredient a tool cannot supply; everything after it is mechanical.
Assign it by asking: *if the product were a different product, would this function's text change?*
If yes, tag it with the noun that would change ([thermo] here); if no, it is []. A narrower tag
for a feature's own knowledge is allowed ([thermo:display] under [thermo]), but one tag per project
is usually enough to start.

## Spray's three fundamental constraints (what the rules check)

Spray builds ALA on three constraints (Summary; §2.1.1, §2.1.3). Every rule below serves one of
them:

1. **The only unit of code is an abstraction.** Not a module, class, or function as such: "a
   'generalized conceptual idea'", "learnable as a concept" (§2.1.1). R6 and R7 check this.
2. **The only relationship is a knowledge dependency on something significantly more abstract**
   (§2.1.3). Everything else, including every run-time communication between peers, goes through
   wiring set up by a higher layer. R1, R2, R4, R5, R9, and R10 check this.
3. **Abstractions are small.** The rule of thumb is "around 100 to 500 lines", and "if abstractions
   average less than 100 lines of code, we will likely have more abstractions than we need"
   (Summary, "All abstractions must be small"; §7.10.2 says "probably in the range of 50 to 500").
   The constraint exists so the first two can't be met by one big ball of mud. It applies to the
   application too: when the application gets large, it becomes a composition of Features (§7.15).
   R7 and the module-size check cover this, and R11 covers the application's share.

## What a domain abstraction is, and its kinds (UI and others)

**"Domain" is Spray's word for a family of applications, not a domain model.** Domain abstractions
"are more specific to the types of applications we want to express using them. They are specific to
a domain, making them more expressive, but less reusable than general purpose library abstractions.
They are still reusable both within a single application and by other applications in the same
domain" (§2.2). For a storefront that means a grid of cart lines, a promo rule, a store adapter or a
payment call, not just the business entities (cart, order) a domain model would hold. The layer
"contains abstractions that can be composed into applications. These are typically building blocks
for I/O, data transformations, and persistent state, but many other types of abstractions are
possible" (§2.2).

**Kinds of domain abstraction differ by their ports, not by layer.** Spray doesn't give UI its own
layer, and he doesn't split a user story into UI, logic and data tiers: "we don't separate UI from
business logic and data models as we do in conventional architectural layering patterns. These are
highly cohesive things from the perspective of user stories and ought to be kept together. Instead,
we separate the implementations of the domain abstractions" (Summary). What makes an abstraction a
UI abstraction is that it has ports of UI paradigms:

- **A UI-layout port.** "The display class could have a second port of type UI… A wiring of UI ports
  means one part of the UI is displayed within another part. For example, an instance of display
  could be put inside an instance of a panel" (Summary). UI layout is "containing one UI element
  inside another", composed like any other paradigm (§3.5.1). Because a port's type is a paradigm
  (R9), a UI-layout port connects only to another UI abstraction: a container and what it contains.
- **Dataflow inputs for what it shows.** The Grid in Spray's CSV example "is able to pull rows of data
  as needed" from the reader, filter and sort it is wired to (Summary).
- **Event outputs, and inputs.** "It is common these days for GUI elements such as buttons, menu items,
  etc to have event-driven output ports… In ALA you create input ports as well. For example all popup
  window abstractions such as file browsers, wizards, settings, navigable pages, etc have input ports"
  (§3.5.1).

**Data sources and sinks are abstractions of their own.** "Once you have designed some UI, you will
then want to connect the UI elements that display data to some data sources. These data source are
candidates for abstractions. For example, a data source that represents a disk file can be an
abstraction that handles a disk file format" (§5.2.2). The same holds for where data goes (a store,
a payment gateway), the "building blocks for I/O… and persistent state" of §2.2. So data is *wired
into* a UI abstraction, and its events are wired out; a UI abstraction doesn't fetch, persist or run
the product's rules itself. *This checklist's reading:* Spray states the separation; "doesn't fetch
or persist" is its consequence under R6 (one concept per abstraction).

**Other kinds** are named the same way, by what they do and the paradigms of their ports: data
transformations (a filter, a low-pass filter), state (a cart's lines, an undo offer), rules (a promo
code check), I/O (a store, a gateway). In many codebases, what the code calls a *feature* (a cart, a
wishlist) is a state-and-rules domain abstraction with dataflow and event ports and no markup.
Spray's *features* are something else: compositions of domain abstractions, one per user story, in
their own layer once the application gets too big (§2.2). "When the wiring outgrows one
composition" covers both.

## The rules (each is a visible shape)

- **R1 — every edge between abstractions drops to a lower layer.** Give every function a layer
  (`@name-Lidx`), from an ordered list running concrete to abstract. Spray's usual list is
  Application, Features (in bigger apps), Domain Abstractions, Programming Paradigms, and
  Foundation, and a project may name its own. A call from one abstraction into another must land
  in a *lower* layer. How far it drops doesn't matter: a long drop is not a smell.
  - **Peer edge:** two *different* abstractions in the same layer below the application
    (feature → feature, domain → domain). This is the classic communication dependency. The fix is
    for the layer above to wire them. That holds for features too: they talk through ports the
    application wires, and "features can also have ports and be wired together" (§2.2).
  - **Upward edge:** the callee sits in a higher, more concrete layer. It's the worst kind. Write it
    `^f` when it's indirect but still found by the lower module itself: a hardcoded higher module, a
    registered process or actor name, a topic it knows. By contrast, a function, module, or handler
    that the composition passes *down* is not an upward edge: it's a port, and it's Spray's legitimate
    way to call up the layers ("executing a lambda expression that has previously been passed in", a
    callback, the observer or strategy pattern, §4.4.1). In functional terms: a function or module
    passed in as configuration (strategy), or a subscription the composition sets up (observer).
  - **Who sets up the callback decides.** Set up by the layer above: legal. Set up by the receiver
    itself is a peer edge in disguise, and forbidden. That covers a receiver registering with a
    sender, or subscribing to a public event: "receivers never register themselves to a sender, or
    to a public event" (§4.4.2). Between peers the observer pattern only reverses a dependency that
    ALA doesn't have, so ALA wires instead. The one place Spray keeps an observer is *inside* a
    paradigm interface, for traffic running against the wire's direction, where "the subscriber
    does not know the publisher". In functional terms:
    - A feature or domain module subscribing to a publish/subscribe topic it hardcodes is the defect.
      The composition subscribes and routes, or passes the topic down as configuration.
    - A message handler in the composition that receives the message and hands it on is fine.
    - Sending to a registered name the sender knows, or looking a collaborator up in a name registry,
      is the sender naming its destination (R9). The composition should give it an address or a port
      instead.
  - **Not edges:** calls *inside* one abstraction (a module and its helpers, a feature and its own
    submodules). They are the abstraction's inside, a little ball of mud it's allowed to have.
  - **Working chains in the application:** the application is one abstraction, so R1 doesn't flag
    its internal calls. But a product-knowing function that does *work* (computes, stores,
    decides) and is called by another such function is a chain of collaborating parts. That's the
    shape of the bad thermometer, `[thermo] → [thermo]` where the callee isn't wiring. Each part
    is either a real abstraction, which belongs in a lower layer with its product knowledge hoisted
    out (R3), or it's wiring, which belongs in the composition. A composition split into several
    *wiring* functions is fine.
  - **Tags and layers must agree.** The `[tag]` records what a function *knows*; the layer records
    where it *sits*. A product tag belongs only in layers that know the requirements (Application,
    Features). `[]` belongs in Domain Abstractions and below. A tagged function in a lower layer
    is a leaked requirement (R3's tripwire). A generic function in the application layer is a
    candidate to push down.
  - **Depth is normal.** Three or four layers is Spray's usual stack, and features sit *under* the
    application: both are tagged, and app → feature drops. The smell is not depth. It's a peer or
    working chain, as above.
  - **On a whiteboard** with one tag and no layer map, the two-layer case still reads straight off
    the tags: `[thermo] → []` drops, `[] → [thermo]` is upward, `[] → []` between two different
    abstractions is a peer edge, and `[thermo] → [thermo]` is a working chain unless the callee is
    wiring.
  - "Significantly more abstract" is Spray's actual test. Layers approximate it. An edge that drops
    only a notch (a helper barely more general than its caller) is what R7 and helper proliferation
    examine. Two of Spray's clarifications help place things: "Abstractness decreases as you get
    closer to your specific application", and "Abstractness is not how far you are above physical
    hardware" (§7.2.3). A database driver is not "lower" for being close to the metal; it sits to the
    side, reached through a port (see "Ports go sideways" below). Spray usually ends up with three
    layers above the language, one per turn of his "creativity cycle" (instantiate abstractions,
    configure them, compose them into a new abstraction, §7.2.5).
  - **Techniques that meet it.** None is required; each is one way to satisfy the rule.
    - *Emit an outcome instead of calling a peer.* A feature returns a description of what happened,
      or of what should happen, rather than calling the sibling that should act on it (a typed
      outcome, a signal, a fact). The sideways call never exists, which removes the temptation rather
      than policing it. What the outcome may say, and what it may not name, is R9's business.
    - *Wire cross-feature work in one place.* Every join between features lives in the composition,
      in a function whose job is wiring. Features stay ignorant of each other, and the one module
      allowed to know both does the connecting. Spray's version: "features can also have ports and be
      wired together" by the application (§2.2). A wiring function that binds one feature's result to
      a name and hands it to another still handles data, which R11 counts; declarative wiring and a
      wired graph of instances remove that too.
    - *Make the wiring declarative.* A manifest, a routing table, or generated glue routes one
      feature's typed fact to another's reaction. The routing lives in data.
    - *Pass a function in where a peer used to be called.* When one function called a peer part-way
      through, "the second function will now need to be passed into it. The function parameter is
      also a port" (§2.3.6). It's also Spray's legitimate way to call up the layers (§4.4.1). The
      composition chooses the function; the receiver never names the module that provides it.
    - *Let the composition subscribe.* The composition subscribes to an event source and routes the
      messages on, directly or through a small generic subscription helper in the Programming
      Paradigms layer that the composition configures.
    - *Enforce it in CI.* A static check given a layer map can fail the build on an upward or
      cross-peer call, including domain → domain calls. It doesn't prevent the coupling, but it
      catches it the moment it appears.
  - **What doesn't meet it.**
    - A feature calling a sibling feature's function, including through a shared "service" module in
      the same layer.
    - A lower module that finds its collaborator itself: reading global configuration inside a domain
      module to pick a collaborator, a name-registry lookup, sending to a name it knows, or subscribing
      to a topic it hardcodes.
    - An interface one feature defines for another to implement (see R9).
    - Product-knowing functions that do work and call each other (a working chain), even inside the
      application.
  - *Spray:* Summary ("All dependencies are knowledge dependencies", "Good and bad dependencies",
    "Emerging layers"); §2.1.3; §2.2; §3.4 (and §3.4.6, knowledge dependencies reach all layers
    below); §4.4.1–4.4.2 (callbacks and registration); §7.15 (features).
- **R2 — wires meet only at the top.** A `pN` may appear in two functions' bodies only if one of
  them is the composition that passed it. The same `pN` (especially `*pN`) inside two *sibling*
  subtrees means two peers share its meaning — the meaning belongs in the wiring.
  - *A wire joining many ports* is a smell of its own: "a new abstraction may be waiting to be
    discovered", like the ground symbol on a schematic. Spray's example is a game score that most
    instances interact with: make it a domain abstraction in the layer below instead of wiring
    everything to it (§3.6.1). See R10's aggregate case.
  - **Techniques that meet it.**
    - *Give each feature a private data structure.* Per-feature records leave no shared mutable thing
      for two peers to fight over. Immutability does the rest.
    - *Keep one source of truth and derive the rest.* Keep source fields and recompute derived ones
      from them, which kills the classic bug where a handler updates the items but forgets the total.
    - *Let the composition supply cross-slot values.* A feature that needs a sibling's data doesn't
      read a shared session slot. The composition reads it and passes the value in. By §1.6.3's
      standard the composition is still handling that value (R11). One step further makes it a wire:
      the cart sends its data out on a port when checkout is requested, and checkout takes its stock
      source as configuration.
    - *Turn a value everything touches into an abstraction one layer down.* A wire joining many ports
      is Spray's "ground symbol" smell (§3.6.1). Make it a domain abstraction the features depend on as
      knowledge instead of wiring every feature to it.
  - **What doesn't meet it.**
    - A global table or mutable cell that two features both read and write.
    - A feature reading another feature's slot of a shared session or composition record (the
      wishlist reading `session.cart`).
    - A process or actor that two features both call to swap state.
  - *Functional-language note:* immutability gives this rule almost for free, so a clean R2 says less
    about a functional design than it would in an imperative one. Pure code can still couple peers
    through R1, R5, and R10 (see "Functional programming", point 12).
  - *Spray:* Summary ("Communication between instances of peer abstractions"); §3.8 (no data
    coupling); §7.8.
- **R3 — application literals live at the composition line.** Application literals (product-specific
  constants: thresholds, prices, labels, formats) live in the composition, or in a manifest or
  config module the composition owns, never inside a `[]` leaf. Passing them as arguments on the
  composition line (`f4(p1, 4, 8.3)`) is the simplest form; a manifest, or a constant on the
  composition, are others. A literal buried in a leaf silently converts `[]` to
  `[thermo]` — the tag was lying. The contrast is an *intrinsic literal* (an identity or a physical/
  mathematical constant that is part of the abstraction's own definition), which correctly stays put.
  - **Techniques that meet it.**
    - *Put the application's constants in one place at the top.* A manifest, a constant on the
      composition, or a store-configuration module the composition reads holds the calibration
      (rates, thresholds, fees).
    - *Make the domain take calibration as configuration.* The abstractions below take the constants
      from the composition, so they hold no product-specific number and drop into a different product
      unchanged. Pass it once, as R9's configuration "should" describes (for example a configuration
      value as the first argument), not scattered through each call's data.
    - *Pass rules in as predicates.* A requirement like "free shipping over $100" can be a function the
      composition passes to a generic abstraction, the way Spray writes `new Filter(x => x>=0)`
      (§3.11.3). The rule is stated once, at the top, and a generic abstraction runs it.
    - *Compute a derived value once, instead of threading the literal.* Rather than pass a bare
      threshold through every display call that compares against it, compute the status once from the
      configured threshold and let it flow down as data, which also keeps the display components pure.
    - *Pass message text in too.* Notification and label text is an application literal. The
      composition supplies it as configuration, or maps a feature's fact to text it words itself.
    - *Supply every word from the composition as a map.* The composition holds its words in one map,
      merges in words shared across the product, and passes each UI component its part. Components
      render only what they're given.
    - *Give forms their messages as configuration.* A feature validates, but the words of the error
      come from the composition, passed in when it builds the feature.
    - *Project values, not words.* A feature returns the amounts a sentence needs, and the UI
      component writes the sentence with the composition's words (money values rather than a
      pre-built "free over $50.00").
    - *Keep product-wide literals in one application module.* Rates, thresholds, currency and the
      words several screens share live in one module in the application layer. Domain modules never
      read it; the composition passes its values down.
  - **What doesn't meet it.**
    - A number or message string in a feature or domain module (a `free_over = 10000` constant inside
      `Shipping`, `"Saved to wishlist"` inside `Wishlist`).
    - Words a feature builds for display ("free over $50.00" built from each shipping rate). Project
      the values and let the UI take the words from the composition.
    - Validation messages written inside a feature's schema or validator. Pass the form's messages as
      configuration.
    - A code or unit in a domain transformer (a hardcoded currency). Configure the instance once.
    - A default value in a data definition that is really a product decision (`threshold = 100`).
    - Reading global configuration inside a domain module for a product value: the module picks its
      own configuration (see R9, "No endpoints"). Reading config in the composition and passing it
      down is fine.
  - *Cost to expect:* with no defaults baked into the domain, its own tests must supply configuration.
  - *Spray:* §1.6.3 (literals at the composition); §2.4; §3.5 and §3.5.2 (the application specifies
    the rounding, filter bandwidth, and resampling rate when it instantiates the abstractions).
- **R4 — `$` is legitimate only inside a `[]` leaf** (state that *is* the abstraction's concept:
  a filter's memory, a sampler's counter). `$` on a tagged function is invisible coupling through
  time between application steps. State belongs with the abstraction whose concept it is: Spray
  "prioritizes abstraction over referential transparency", because passing a filter's running
  value in on every call "breaks an otherwise good abstraction". Whether that state is held (a
  process, or a value returned inside the updated program) or passed in and out is a case-by-case
  execution choice, not a compliance level. What R4 forbids is state whose meaning leaks out of its
  owner: the app or a peer keeping, reading, or computing the raw state itself (`f5(s5, p2)`, where
  the app holds `s5`). With immutable values a changing instance comes back as a new value, and
  storing that opaque value again is not a breach, because the concept still owns what is inside it.
  (Re-storing every instance by hand is the §1.6.3 "handling the data" cost that "Past the pipe"
  moves beyond.) State that no abstraction owns becomes its own abstraction (Spray's `State<T>`, a
  generic state-holder with input and output ports), wired in like any other. In a functional
  language that could be a small value in the program value, or a process or actor, that the
  composition wires to its readers and writers.
  - *Two ways to hold state* (§3.11.2). Spray defines a computation as `input + state --> state +
    output`, which can be grouped as `(input + state) --> (state + output)` (the functional form: the
    caller passes the state in and gets it back) or `input ( + state --> state +) output` (the object
    form: the abstraction keeps its state). "In ALA we choose between these two philosophies on a
    case by case basis." Either is fine *inside* an abstraction. What R4 forbids is the caller
    managing state that belongs to another concept. In a functional language the functional form is
    the default; a process or actor is the object form.
  - **Techniques that meet it.**
    - *Write a functional core over owned state.* Keep feature logic as pure functions over the
      feature's own data structure. The module that owns the state is the only one that reads or
      changes it; callers store the new value back without looking inside. Spray's reason: state that
      is part of a concept belongs with that concept (§3.9, §3.11.2).
    - *Partition state by owner.* Split one large composition record into one field per concept (cart
      state, UI state, checkout state), so each piece of state visibly belongs to one concept.
    - *Let the composition value carry step state.* When the composition builds a program from stages
      (see "Past the pipe"), each stage's memory lives inside its own value, and the runner stores the
      updated program. The composition holds one opaque value and never names a stage's state.
    - *Give unowned state its own abstraction.* State that belongs to no concept becomes Spray's
      `State<T>`: an abstraction with input and output ports, wired in like any other (§3.9). In a
      functional language, a small value in the program value, or a process, that the composition
      wires to its readers and writers.
    - *Use a process where the concept is concurrent.* A sensor poller, a timer, or a bridge from a
      message bus keeps its state in a process with a wired output port. That's Spray's object form,
      and it's fine when the concept really runs on its own.
  - **What doesn't meet it.**
    - A process-global variable, mutable cell, or global table used as a hidden channel between calls.
    - The composition or a peer reading or computing another concept's raw fields (summing the cart's
      items in the composition).
    - One process or actor holding several concepts' state that features reach by registered name.
    - One process per abstraction for its own sake: processes are for concurrency, isolation, and
      lifecycles, not for organizing code.
  - *Spray:* §3.9 (state joins the struct of the concept it belongs to; `State<T>`); §3.10 (the four
    reasons for objects); §3.11.2 (abstraction before referential transparency).
- **R5 — every `qN` must be promoted or deleted.** A contract that exists only as two matching
  literals (a format string produced here, parsed there; a name emitted here, matched there) is
  an edge the call tree cannot show. Spray's rule is that "no two modules know the meaning of data
  or a message" (§7.8). An element knows one thing, and "no other elements should know about this
  one thing" (§7.25). His fixes, in order of preference:
  1. **Keep the meaning in the composition.** Both ends take the name or format as configuration
     from the layer that wires them, so only the composition knows the two agree. (The thermometer's
     number format is set at the composition, not agreed between the formatter and the display.)
  2. **Make both ends one abstraction.** If a sender and a receiver must share a meaning, "they are
     two instances of the one abstraction". One module owns the format and does both the encoding
     and the decoding (a protocol codec, an `encode/decode` pair), and each end uses an instance of
     it. They may be deployed apart, but in the logical view they are one abstraction.
  3. **A shared name in a lower layer, only if it is truly abstract.** A name both sides depend on
     downward is legitimate only when a great many abstractions use it (Spray's example is an
     `initialize` event, §4.7.4). A module of names shared by a handful of peers is a registry of
     global event names. Spray calls those "symbolic wirings": you "search for where the names
     appear throughout the entire code". It fails, even though every edge to it drops.
  - *Names owned by the application:* a module of event, message, and UI-hook names is fine when only
    the application uses it: its templates, its handlers, and the client-side code it owns. That is
    Spray's `temperature` symbolic connection, kept inside one abstraction. It turns into the registry
    smell when feature or domain modules use it to agree with each other.
  - *Where the composition splits into modules:* a UI event name written in one module's template
    and matched in another module's event handler, or a timer or task name started in one module and
    matched in another, is this contract. It appears as soon as one composition is split into several
    modules (features as compositions, below), even when each module is clean on its own. Settle each
    name with fix 1 (the composition names it and passes it to both) or fix 2 (the module whose view
    fires the event also handles it).
  - **Techniques that meet it.**
    - *Keep the meaning in the composition.* Both ends take the name or format as configuration from
      the composition (the composition chooses a display format; the formatter and the display never
      agree on it themselves).
    - *Type your effects.* Outcome constructors turn "return a tuple and hope the other side matches"
      into a typed value, where a typo is a compile error.
    - *Declare names once, and keep them in the composition.* A small module holds every event,
      stream, and hook name the application uses, so a rename happens in one file. Once features use
      such a module to agree with each other, it becomes a registry of global names (fix 3 above).
    - *Make both ends one abstraction.* One module owns both the encoding and the decoding, and each
      end uses it.
    - *Check the places signatures can't reach.* A template's event name is a contract the compiler
      never sees; a checker that reads templates can compare the names they fire with the handlers
      that match them.
    - *Derive a label from the value it states.* A label that quotes a configured number is built
      from the same constant, so changing the fee can't leave the label stale.
    - *State a domain constant once, and ask for it.* When a rule's constant ("a line can't go below
      1") is needed in two places, the module that owns the rule answers the question (`at_minimum?`)
      instead of the other repeating the number.
  - **What doesn't count as a silent contract.** A route the router checks at compile time. Two equal
    strings that mean different things (`"products"` used as a message topic in one place and a table
    name in another: a coincidence, not a contract). Telling these from real contracts is the
    reader's job; a linter only sees duplicated text.
  - **What doesn't meet it.**
    - A label that restates a configured number ("Gift wrap ($2.99)" beside a fee constant of 299):
      change the fee and the label lies. Build the label from the same constant.
    - A tuple shape one module produces and another pattern-matches on with no shared definition.
    - A session key written as one type by one module and read as another by a second (a symbol here,
      a string there).
    - The same topic or event string written as a literal in two modules.
    - Features reading names from a shared module to agree with each other.
  - *Spray:* §3.8 (no data coupling); §4.7.4 (global event names are symbolic wirings); §7.8 (two
    instances of one abstraction); §7.25 ("no other elements should know about this one thing").

Anything that survives R1–R5 and still can't be given a general, product-free name is not an
abstraction — inline it into the composition as `(p) ->` wiring (R4's twin, Spray's "func1 is
not an abstraction"). That instinct is now a rule of its own (R6), because R1–R5 check *coupling
and knowledge-placement* — structure — and a program can be perfectly R1–R5-clean and still be
ugly. R6–R8 add the *design-quality* axes structure alone doesn't cover.

- **R6 — every abstraction names a learnable concept.** A named function/module must denote a
  concept a reader can learn and the language/stdlib does not already name. A meaningless name
  (`f1`, `op`, single letters) fails it; a function that merely wraps a primitive (`p(x,y) = x+y`)
  is not an abstraction — delete it; something you can't name generally is wiring — inline it. This
  is Spray's func1 test (see the `(p) ->` mark), made into a rule of its own. Partly mechanical
  (wraps-a-stdlib-call, opaque-name are checkable), partly judgement.
  - *Verify (two tests from Spray):* (1) does it **separate the knowledge of different worlds**
    (§7.2.5)? A clock separates cog-wheels from being on time; SQL separates index algorithms from
    "find this customer's orders". If it only factors out common code, it is not an abstraction, so
    look harder. (2) Can a reader *use* it without following the reference into its body? "A good
    abstraction is when you don't need to follow the indirection" (Summary, §3.4). If they must chase
    it to keep understanding, you don't have an abstraction, and the split makes the code read *worse*.
  - *What do you know about?* Ask each module and function; "the answer should always be 'I just
    know about…'". Two refinements from Spray: an element "may know how to do an operation on some
    data, or the meaning of some data, but not both", and "no other elements should know about this
    one thing" (§7.25). A module that both defines what an order means and computes shipping on it
    knows two things.
  - *Document the concept.* Spray calls it "critically important to add comments that make it
    learnable by explaining the concept it provides, its ports, its configurations and an example of
    its use" (§2.1.1). The property: a reader at a call site can find the concept, ports,
    configuration, and an example without reading the body. A short module comment (one or two
    lines, plus an example when it isn't obvious) does this and fits a sparse-comments style.
  - **Should: a public function takes only the parameters it uses itself.** A parameter the function
    never reads, and only carries down to another function that also just passes it on, is Spray's
    complaint about layered functions: "The middle layer functions end up with extra parameters that
    don't have anything to do with them, just so they can pass state data through to even lower
    functions. This makes these intermediate functions not great abstractions in themselves"
    (§3.11.1; for procedures, §3.9). Such a parameter is often called a *tramp* parameter. It widens
    the function's interface with knowledge of what something further down needs, so the function
    can't be read or reused without knowing that.
    - *Reviewer's test:* does any public function take a parameter it never reads, only to hand it to
      a call that passes it further down?
    - *What doesn't count:*
      - One hop. A function that hands its own input to the lower abstraction that processes it is
        using that abstraction (the thermometer gives the reading to `OffsetAndScale`).
      - A runner or connection mechanism carrying a payload to a port. Carrying is its concept.
      - A private helper inside one abstraction (the little ball of mud), and a framework callback
        whose signature is fixed.
      - The application layer, where the same thing is R11's "handling the data".
    - *Why "should", not "must":* Spray gives it as a reason functions become poor abstractions, not
      as one of his constraints, and a two-hop carry is sometimes the simplest honest shape.
    - *Ways to fix it:*
      - Let the composition give the lower abstraction its input or configuration directly, so the
        middle function never sees it.
      - Configure the lower instance at the composition and pass the *configured instance* (a value,
        or a function) down. The middle function then uses it, calling it with its own data, which is
        a port (R9), rather than carrying its raw inputs.
      - Make it a wire: checkout takes its stock source as configuration instead of receiving stock
        levels through the composition and passing them on.
      - If the middle function exists only to carry, it may not be an abstraction at all (R7).
    - *Mechanizable:* yes, as an advisory. A check can flag a public function whose plain-variable
      parameter is never read and only passed, as a bare argument, to another project module's
      function that doesn't read it either and passes it further. It should skip the exceptions above,
      and treat carrying into an interface as a runner delivering to a port.
  - **Techniques that meet it.** This rule resists tooling; these habits help.
    - *Slice by feature, and name the slice.* One module per feature makes you name the concept
      before you write it.
    - *Name domain abstractions by their kind, not their use.* `OffsetAndScale`, `Boiler`,
      `LowPassFilter`: each names what it is, so a reader learns it once.
    - *Ask "what do you know about?"* The answer should be one thing. A module that defines what an
      order means *and* computes shipping on it knows two things.
    - *Write the short module comment.* Two lines usually cover the concept, ports, and
      configuration ("Smooths a stream of numbers. In: a number. Out: the smoothed number. Config:
      strength.").
    - *Write one-off wiring as an anonymous function.* A named function used once is "indirection
      without abstraction"; an anonymous function at the composition says it's wiring (§6.1, §1.6.3).
  - **What doesn't meet it.** `Utils`, `Helpers`, or `Manager` modules; a function that renames a
    primitive (`add(a, b) = a + b`); names that teach nothing (`process2`, `handle_stuff`); a
    parameter carried down unread (the "should" above). A UI component that does I/O: a stateful UI
    component that loads, saves or charges through a store, gateway or placement instance bundles a
    data source or sink into a UI abstraction (Spray wires data sources to UI elements, §5.2.2).
  - *Spray:* §1.6.3 ("func1 is not an abstraction"); §2.1.1 (an abstraction must be "learnable as a
    concept"; document concept, ports, configuration, example); §7.2.5 ("a good abstraction
    separates the knowledge of different worlds"); Summary and §3.4 (a good abstraction is one where
    you "don't need to follow the indirection"); §7.23 (symbolic indirection without abstraction);
    §7.25 ("What do you know about?"); §3.11.1 and §3.9 (parameters carried through for lower
    functions, the "should"); §6.1 (a one-off named function is "indirection without abstraction").
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
  Spray's reason reuse still matters as evidence: "abstraction and reuse are two sides of the same
  coin. More abstract means more reusable" (Summary, quoting Charles Krueger; §2.1.1). A concept
  general enough to name is usually general enough to reuse, even before a second caller exists.
  - *Verify (the complement of proliferation):* a real abstraction is a **cohesive whole**, *not*
    internally decomposed into named semantic sub-parts (inside, it is "a small ball of mud" where
    every line serves the one concept), and small enough to **read in isolation** (Spray's rule of
    thumb is 100 to 500 lines; the real test is "readable alone"). The lower bound matters as much
    as the upper one: if abstractions *average* under 100 lines, there are probably more of them than
    needed, which is helper proliferation seen from the size side. Many ports is also a signal:
    "abstractness decreases with more ports" (§7.2.3), so a module with a long list of inputs and
    outputs is probably too specific to be a good abstraction. Over-splitting a real abstraction's
    internals is as much an R7 defect as inventing a trivial one. And **reuse is a positive**: never
    let R7, height, or the pass-through check flag a genuinely shared abstraction — high fan-in is
    evidence it earns its place.
  - *Little balls of mud, and why the interior is not scored.* ALA governs the relationships
    **between** abstractions, not the purity of each one's insides. The interior of an abstraction may
    be a little ball of mud: procedural, branchy, decomposed into private helpers, and that is fine as
    long as the abstraction (a) names **one concept** (R6), (b) stays **bounded in size** (100–500 lines),
    (c) keeps its internals **private behind a small public surface**, and (d) is **clean at its
    boundary** (R1, R2, R5, R9, R10 all hold where it meets other abstractions). Because of this, the
    structural checks operate on the graph of *abstractions* (modules and their public functions), not
    the raw call graph: a call from one private helper to another in the same module is internal
    decomposition, so it adds no abstraction height and is never a pass-through. The guardrails that
    keep a *little* ball of mud from quietly becoming a *big* one are (a) through (c): a module that
    stops being nameable, blows the size cap, or exposes a wide public API has leaked its mud outward,
    and *that* is the defect the checklist catches, not the internal mess.
  - **Techniques that meet it.**
    - *Carry the least machinery that works.* Strong compliance is possible with almost no apparatus.
      When you find yourself adding a layer to hold a layer, that is the smell R7 names.
    - *Let a tool catch proliferation.* Pass-through and dead-code detectors flag the one-in, one-out
      forwarding function that adds a name and a hop but hides no decision (a `save(x)` that only
      calls the store's insert).
    - *Use what the standard library already names.* Don't write `Filter`, `Map`, or `Sort` domain
      abstractions when the language's collection and stream libraries exist; Spray: "There would be
      no sense in reinventing that functionality as 2-port classes" (§6.3). Configure a generic step
      with a library function instead ("Functional programming", point 10).
    - *Don't build wiring machinery for an algorithm.* Plain function calls down the layers are enough
      when the problem is a calculation (§7.5; "Functional programming", point 2).
    - *Mind both size bounds, and the port count.* 100 to 500 lines per abstraction; averaging under 100
      means "more abstractions than we need" (Summary). "Abstractness decreases with more ports"
      (§7.2.3), so a module with a long list of inputs is probably too specific.
    - *Split a wide abstraction along its concepts.* A state module whose public surface keeps growing
      (lines, a promo code, a shipping choice, totals) is often several concepts. Split it into
      smaller abstractions and wire them together inside the feature that uses them, so the extra
      wires don't land on the application.
    - *Derive a declaration instead of restating it.* When a component wires some of its feature's
      outputs itself and sends the rest on, declare only the ones it wires and derive the sent list
      from the feature's declared ports, so the two can't drift.
  - **What doesn't meet it.** A module that only forwards to another; a manifest or codegen step
    for a screen with two features and one reaction (declare in a manifest once the wiring is
    numerous or regular, inline in the composition while it's rare).
  - *Spray:* Summary, "All abstractions must be small" (a rule of thumb of 100 to 500 lines;
    averaging under 100 means "more abstractions than we need"); §7.10.2 ("probably in the range of
    50 to 500"); §7.2.3 ("abstractness decreases with more ports"); §6.3 (don't reinvent library
    functions as abstractions); §7.5 (plain function composition suffices for an algorithm). Spray
    makes size a fundamental constraint; this checklist scores it as advisory (see "Where this
    checklist departs from Spray").
- **R8 — the composition reads as the requirements; names and shapes serve the reader.** The top
  layer should read like the spec; config keys should name what they configure; state wires should
  be shaped for a reader. **R8 is judgement, not shape** — "reads as the requirement" cannot be
  mechanically decided the way "edges drop" can. It is a review prompt and the honest boundary of
  the checklist: R1–R7 and R10–R11 are checkable (mechanically or semi-mechanically), R9 in part; R8
  marks where automated checking stops and human review begins. A tool should *report* the R1–R7
  findings and *flag* R8 concerns, never claim to score R8.
  - **Techniques that meet it.**
    - *Make the wiring a document.* A manifest states a screen's requirements as data: which features,
      which reactions, which configuration. A flows table does the same for a multi-step wizard.
    - *Keep the composition legible.* One handler clause per user action, read top to bottom as the
      screen's behaviour.
    - *Wire with clauses, one per port.* Each feature step runs through a small runner that folds every
      output into the composition's own `wire` function; its clauses, one per declared port and
      contiguous, are the wiring. With no catch-all clause, a missing wire fails loudly, and a test
      reads the clause heads and checks them against every feature's declared ports both ways. It
      needs nothing beyond pattern matching; the cost is that only a source reader, not a value,
      shows the wiring.
    - *Keep the composition inspectable.* A graph value (named instances plus a wire list) can be
      printed, diffed, and drawn; nested closures can't (§3.11.3, §6.2.1). Keeping "create an
      instance" apart from "wire it" lets the code follow the diagram line by line (§6.2.2).
    - *Hold cross-component wiring in one route table.* When feature instances are stateful UI
      components that report to the composition, one table says where each output goes, and one
      handler applies it through a generic router with a few target kinds. Spray: connections "are
      typically cohesive, and belong in one place" (§3.3.1). The cost is one message hop per
      cross-feature effect.
    - *Run clause wiring with a tiny runner.* The clause-wiring technique above needs one generic
      function in the paradigm layer: run a feature step on its stored state, store the new state,
      and fold each output through the composition's `wire` function. A second function, `feed`, runs
      a feature's input on what another function answers, so a clause never holds one abstraction's
      answer to hand to another.
    - *Test that every port is wired.* Each wiring form needs a coverage test, because an unwired
      output is lost or crashes only at run time. It's one small test per composition (or per wiring
      form), generated over the declared ports, not one test per port: send every declared output, or
      check a wiring map's keys against the declared ports plus a list of ports deliberately left
      unbound, or read the wiring clause heads from source.
    - *Name instances in the wiring instead of reading composition state.* A clause that needs a
      configured instance names it, and the runner looks it up in one map the composition builds at
      start-up. A wrong name fails where it's named.
    - *Make the coverage check one line, or a compile error.* A paradigm-layer function reads a
      composition's source and compares its clause heads with every port its parts declare, so each
      test is one line. Where the language has macros or compile hooks, the same check can run while
      the composition compiles; it costs familiarity.
    - *Bind every port in one map, run by a binder.* Each composition has one map from
      `(feature, port)` to a list of bindings, and a generic binder in the paradigm layer applies each
      kind (to a stream, a display value, a call, another feature's input, a notification, …). A
      binding can name a configured instance through a small paradigm interface the instances
      implement, instead of wrapping it in a function.
    - *Run the composition as a circuit of instances and wires.* The composition builds a value of
      named instances (features, transforms, and sinks such as a display value or a notification) and a
      wire list, and a generic runner delivers outputs along the wires. Typing every port lets the
      runner check each wire and every unused output before it starts, and draw the diagram from the
      value that runs.
    - *Send port outputs from a component to the composition.* A stateful UI component keeps what it
      wires to itself and sends the rest to the composition as `(name, port, payload)` through a
      generic helper, with a declared list of what it sends for the coverage test.
    - *Give a route table a few kinds of target.* Pass to another instance's input, set a display
      value, notify, change the URL, call a store function or named instance, feed an instance's
      answer on, or run an instance as an asynchronous task.
    - *Type every port, and check the whole composition before it runs.* Each port declares a type;
      a configured instance declares the payload types it accepts and what each answers; sinks have
      types (a list display takes row changes, a form takes a validation result). The composition is
      refused at start-up for a wire between two different types, an output neither bound nor
      deliberately left unbound, or an unbound input.
    - *Draw the diagram from the code that runs.* A wiring value draws directly. Wiring written as
      clauses draws by reading the source: each clause that sends to another part's input is an
      arrow. Either way the picture can't drift from the code. A drawer reads clauses by the shape the
      code writes them in, so a test should check the edges a reader looks for first.
    - *Fail loudly on an instance the composition didn't configure.* Look a named instance up with a
      function that raises a clear error instead of calling nothing.
  - **Wiring forms, side by side.** Hops are message hops per cross-feature effect.

    | Form | Where the wiring lives | How it runs | Hops | Coverage check |
    |---|---|---|---:|---|
    | Manifest plus generated glue | a data file per screen | generated, committed handlers | 0 | the generator fails on drift |
    | Handler clauses, one per user action | the composition's clauses | a small shell folds outcomes | 0 | none by default |
    | Bindings map plus binder | one map per screen | a generic binder with binding kinds | 0 | every port bound or deliberately unbound |
    | Circuit of instances and wires | a circuit value per screen | a generic circuit runner | 0 to 2 | every wire and unused output, before it runs |
    | Stateful UI components plus composition handlers | the components' own wires plus the composition's clauses | the UI framework's message passing | 1 or more | a test sends every port output |
    | Stateful UI components plus a route table | the components' own wires plus one route map | a generic router with a few target kinds | 1 | a test sends every port output |
    | Clauses plus a runner | one clause per port in the composition | a two-function runner (`run`, `feed`) | 0 | clause heads read from source |
    | Story clauses | each story's clauses between its parts; the composition's clauses between stories | a runner that nests | 0 | per composition and per story |
    | Typed story maps | each story's bindings map; the composition's map between stories | a binder that nests | 0 | every port typed; checked before it runs; drawn |
    | Plain story modules | each story's private clauses between its parts, its input and event handlers; the composition's clauses between stories | the two-function runner; each story given an output function | 0 | per composition and per story, plus every event a story's view fires |

  - **Clauses or a value.** Wiring written as function clauses in one module (one handler clause per
    message, or one `wire` clause per port) meets R8: it is one place, which is what Spray asks of
    connections (§3.3.1). A wiring value (a map, a route table, a circuit) is a technique, not a
    requirement: it adds inspection and checking by key, at the cost of vocabulary.
    *Functional-language adaptation:* a function with one pattern-matching clause per port is this
    checklist's reading of a wiring list.
  - **What doesn't meet it.** Wiring spread across several modules' labels, so no one place shows it;
    a composition whose handlers compute; configuration keys that don't name what they configure.
  - *Spray:* §2.4 (executable expression of requirements; 3–10% of the code); §3.5; §3.6 (diagrams
    vs text); §7.7; §3.11.3, §6.2.1, §6.2.2 (explicit
    wiring over hidden closures; keep instantiation and wiring apart).

The next three rules each turn one of Spray's constraints into a check of its own, rather than
leaving it as background for an earlier rule.
- **R9 — ports are typed by a programming paradigm; an abstraction owns no interface except its
  own configuration.** Spray calls the second half "critically important" (§2.3.4).
  - **Main interface and ports.** A module's own public API (its constructor, its configuration,
    the fields set when it is created) is there for the layer above, to instantiate and configure
    it. Every other input and output at run time goes through a *port*, and a port's type is a
    programming paradigm from a lower layer. "No other interface implemented or required by the
    class can be 'owned' by the class."
  - **Owned interfaces fail, both directions.** A *required* interface is one the consumer defines
    for others to implement (Clean Architecture's ports, which the business layer defines for
    adapters to implement; the dependency inversion pattern). A
    *provided* interface is one a module defines, specific to itself, for its peers to call. Both
    carry one module's design inside the other, "a fixed arrangement between the two". Moving the
    interface to a module of its own doesn't help if it still describes what one module needs:
    "simply moving IB doesn't make it more abstract" (§6.5.5; IB is the interface that module B
    requires).
  - **Interfaces in a functional language** (type classes, protocols, traits, module signatures,
    callback specifications). What matters is who defines the interface relative to who implements
    and calls it.
    - *Allowed: a paradigm port.* An interface in the Programming Paradigms layer that no domain
      abstraction owns (`Step`), implemented by domain modules (an instance of `Step` for
      `LowPassFilter`). This is how a domain abstraction gets a port.
    - *Allowed: configuring a more abstract module.* A module far below its implementers defines
      callbacks that higher modules implement to configure it: a generic server's initialise and
      handle-request callbacks, a web framework's request handler, or a generic scheduler that takes
      a job module. The implementer depends *down* on it, which is legal. It's the interface-shaped
      form of Spray's "lambda passed in" for calling up the layers.
    - *Not allowed: a required interface between peers.* `Checkout` defines a `PaymentGateway`
      interface with `charge`, and `StripeClient` in the same layer implements it. The interface
      describes what `Checkout` needs, so `StripeClient` is written to `Checkout`'s design. Moving it
      to a shared contracts module doesn't change that. The ALA route is a paradigm-typed port on
      `Checkout` (request/response, say) that the composition wires to a payment abstraction.
    - *Not allowed: a provided interface made for peers.* An `Inventory` interface describing
      `Inventory`'s own functions, so peers can call it through the interface or mock it. Its callers
      are still written to `Inventory`'s design. Spray's testing rule is to "mock the ports": wire a
      fake instance to a paradigm port in the test, not replace a peer by name.
  - **Composing functions.** When the composition calls functions directly, "parameters and return
    values are effectively ports", and so is a function passed in (§2.3.6). The same rule applies to
    their types.
  - **Data on a port, and data-transfer types.** Two domain abstractions "may not ... share a DTO" (a
    data-transfer object: a type made only to carry data between two modules). The type "must be
    more abstract and come from a lower layer", often a primitive, and "T may be passed in by the
    application" (§4.8.1).
    - *Allowed: standard types* (numbers, strings, lists, maps, and tuples in the paradigm's own
      shape, like `(emit, out, step)`).
    - *Allowed: a type from a lower layer that is itself a real abstraction,* such as a decimal, a
      date, or a domain `Money`. Both ends depend down on it, a knowledge dependency.
    - *Allowed: an application-defined type passed in.* The app defines it, and the abstraction
      either carries it without matching on its fields or reaches its fields only through what the
      app configures it with (field names, accessor functions, an interface instance). This is
      Spray's "T passed in by the application".
    - *Allowed: a type used only inside one abstraction* (a feature and its own submodules).
    - *Not allowed: a data-transfer type between peers.* `Cart` builds an `OrderRequest` defined by
      `Checkout`, which pattern-matches on it. Or both match on a type neither owns but which was
      designed for exactly their exchange. Either way one abstraction's design now lives in the
      other. Moving the type to a shared module in the same layer doesn't fix it, for the same reason
      as a moved interface.
    - *Fixes:* the composition converts between the two shapes with a `(p) ->` lambda; the
      application supplies the type; or the two ends become two instances of one abstraction that
      owns the format (§7.8, see R5).
    - *R9 vs R10:* R9 is about the message type on a port between two abstractions. R10 is about a
      domain entity that two features both read.
  - **Outputs announce; they don't command.** "An output port from an abstraction may say 'This has
    happened' or 'Here is my result', not 'do this next', or 'here is your input'" (§7.2.6). What
    happens next is the wiring's business. The one exception is request/response. Wired point to
    point, "a request is implicitly a command" (§4.6), and that's fine because the application set
    up the wire. In functional terms, an outcome a feature returns should read as a fact or a result
    (`item_removed(item)`, `ok(total)`), not as an instruction to a named receiver.
  - **Paradigm instructions as outputs.** A common functional pattern has features return outcomes
    such as "insert into the cart list" or "notify 'Added'", which a generic interpreter carries
    out. Such an outcome bundles a verb ("insert"), a target ("the cart list"), and a payload. R9
    treats the target and the verb differently:
    - *Must: an output never names its destination.* Whatever decides where an output goes, and how
      it is presented, comes from the composition: list names, topics, message text, a named
      receiver. *Reviewer's test:* could the composition send this output somewhere else without
      editing the feature? If not, it fails. This follows from "No endpoints" below (§4.4.2) and from
      R3, since message text is an application literal.
    - *Should: an output reads as a result, not an operation.* Prefer outputs that say what happened
      or what the result is (`item_added(item)`, or `added(item)` on a port the feature names), over
      an operation the feature has decided on (§7.2.6). *Reviewer's test:* could the composition wire
      this output to a different kind of receiver (a counter, a log, nothing), not just a different
      list? A "no" is a prompt to reconsider, not a defect.

    The "should" is weaker because the evidence is. One reading of Spray treats a paradigm
    instruction as request/response, where "a request is implicitly a command" (§4.6). But his
    request/response is two-way, used when "the requester needs to know" something back, and a
    list insert expects nothing back. The checklist doesn't prescribe a technique. Features
    announcing facts that the composition maps, results on the feature's own ports bound once by the
    composition, and targets passed in as configuration all meet the "must".
  - **No endpoints.** An abstraction never names where its input comes from or where its output
    goes. Receivers never register themselves with a sender or subscribe to a public event (§4.4.2).
    The composition sets every wire. "The dependency injection wiring must be explicit. It must be
    specified in cohesive user story abstraction in a higher layer. The wiring cannot be done by
    using a dependency injection container or relying on matching interfaces" (§3.11.3). In a
    functional language, the nearest thing to a container is a module looking up its own
    collaborator (reading which payments module to use from global configuration inside a domain
    module): the module chose its own wire. Reading config in the composition and passing the module
    or function down is fine. Spray doesn't prescribe how the indirection is built: "They can be
    callbacks, signals & slots, dependency injection, or calls to a framework send function"
    (§7.2.6).
  - *Visible shape:* a `[]` leaf takes only `pN` wires, `(p) ->` lambdas, and configuration, and
    **no edge names a peer**. The smells:
    - a port typed by a peer's type;
    - an interface defined in a feature or domain module and used across a boundary;
    - a module that names its own source, destination, or topic.

    *Verify:* for each abstraction, ask three questions:
    - Does it name the source or destination of any input or output, including a list name, topic,
      or message text in an outcome it returns?
    - Does any port's type belong to a peer rather than a paradigm, the standard library, a lower
      layer, or the application?
    - Does it define an interface that its own peers implement or call (rather than a paradigm port,
      or callbacks that higher modules implement to configure it)?

    Any yes is a defect.
  - *Mechanizable:* partly. A `[]` function that references a peer module is already an R1 edge.
    Interfaces declared outside the paradigm layer and used across a boundary can be found from the
    syntax tree. A peer type being pattern-matched is findable. Literal list names, topics, or
    message text inside feature-layer outcomes are findable, which covers much of the "must" for
    outputs. Whether a type is "more abstract", and whether an output reads as a result, are
    judgements.
  - **Should: configuration is set once, apart from run-time data.** Spray's §3.9: "If the abstraction
    consisted only of a single function, then that configuration data would need to be passed in
    every time the function is used. That would be awkward. It would also mix the data parameters of
    the function with the configuration parameters, breaking the Interface Segregation Principle." It
    is a "should" because his own §1.6.3 thermometer passes its settings on every call
    (`OffsetAndScale(adc, offset=4, slope=8.3)` inside the loop), a rung he then climbs past. R3's
    composition-line form (`f4(p1, 4, 8.3)`) is that rung.
    - *Reviewer's test:* does any function take configuration and run-time data mixed in one
      parameter list, so every caller repeats the same settings on every call?
    - *Techniques that meet it:*
      - *Configuration as the first argument.* A value built once at the composition comes first,
        run-time inputs after it. The composition keeps the value and passes the same one on every
        call, or partially applies it once. It is the same order many libraries use for a compiled
        pattern and the text it runs on. In a pipeline that passes data as the first argument,
        partially apply the configuration first, or make the call outside the pipeline.
      - *A value that carries its configuration.* Built once, then used many times: Spray's §3.9
        step, and the thermometer's `Step` values (a `LowPassFilter` with `strength: 10`, then
        `push(filter, v)`). The value can also carry state when the concept has some (R4).
      - *A function built once.* The composition builds a closure over the configuration and passes
        the function. It works, but a closure can't be printed or inspected (§6.2.1).
    - *What doesn't meet it:* settings mixed into the data arguments (`cost(method, subtotal,
      rates)`), each caller repeating them; defaults baked into the domain (R3); global
      configuration read on every call inside a domain module.
  - **Techniques that meet it.**
    - *Put port interfaces in the Programming Paradigms layer.* An interface used as a port belongs to
      no domain abstraction; a domain module gets a port by implementing it. A request/response
      interface (§4.6) implemented by configured store-work instances lets the wiring name an
      instance instead of a closure.
    - *Let the application supply the type.* A type the app defines and a generic module carries
      without matching on it is fine (§4.8.1).
    - *Collapse I/O into one paradigm value.* A sensor-reading value (all inputs) and a
      hardware-command value (all outputs) give the application ports shaped by their kind, not by
      any particular device, and make the boundary pure data a test can build.
    - *Adapt mismatched ports with a lambda at the wiring.* When one port's shape doesn't fit
      another's, the composition converts with an anonymous function (§6.17.4).
    - *Make outputs announce, and keep destinations in the composition.* Three techniques meet the
      "must", with different costs:
      1. *Announce facts; the composition maps them.* The feature returns `item_added(item)`, and the
         composition decides that means "insert into the cart list and notify 'Added'". It meets both
         halves and is closest to Spray, but it costs a composition clause per fact, and every
         feature needs its own fact vocabulary.
      2. *Results on the feature's own ports, bound once by the composition.* The feature returns
         `rows(added(item))`, a collection change on a port it names itself. The composition binds
         that port once (rows go to the cart list), and a generic interpreter applies the binding. It
         meets both halves with one line per port rather than per fact. An unbound port is silently
         dropped, so test that every declared port is bound.
      3. *Pass the target in as configuration.* The feature is built with the name of its list and
         returns "insert into the configured list". It's the smallest change to an existing app, and
         it meets the "must". It meets the "should" only weakly, because the composition can change
         which list, but not what kind of receiver.

      Where the instruction is built matters. If the composition builds "insert into the cart list",
      the composition names its own lists, which is fine. If the features build it, with their own
      notification text, that fails the "must"; technique 3 is the cheapest fix.
    - *Test against ports.* Pass a fake into the port (a stub implementing the interface, a function,
      or a test process as the output). A mocking library is fine for a paradigm-layer interface,
      since that *is* a port (see "Tests replace only ports" in the verify procedure).
    - *Configure domain instances once, configuration first.* Every rule the composition used per call
      becomes a value the composition builds once and passes down: rates, promo codes, the gift-wrap
      fee, the low-stock level, the currency. Run-time data comes after it.
    - *Name outputs as facts, and let the wiring turn them into actions.* `persist` becomes `changed`,
      an undo offer's `timer` becomes `captured`, `payment` becomes `ready_to_pay`, and the cart's
      "pay" becomes `checkout_requested`. The composition's wiring then starts and stops the undo
      clock on those facts.
  - **What doesn't meet it.** An interface a feature defines for a peer (`Checkout`'s
    `PaymentGateway`); an `Inventory` interface written so peers can mock `Inventory`; a type one
    peer defines and another matches on, even if moved to a shared module; an output that names a
    list, topic, or message text; a module that looks up its own collaborator.
  - *Spray:* §2.3.4 (interfaces; owned interfaces "critically important"); §2.3.6; §4.4.2; §4.6
    (request/response); §4.8.1 (no DTOs; T passed in by the application); §6.5.5 (dependency
    inversion); §7.2.6 (outputs announce); §7.8; §7.24; §3.9 (configure once; the "should");
    §6.17.4 (a lambda at the wiring as an adapter).
- **R10 — no shared entity: no two features know the meaning of the same data.** Spray's rule is
  "no data coupling": two modules shouldn't have to agree on what a piece of data means (§3.8,
  §7.8).
  - *Must:* no domain data type is read or destructured by two features. *Reviewer's test:* could one
    feature change the shape of its data without editing another feature?
  - *Techniques that satisfy it:*
    - features share only an *identity key*, each keeping its own private data against it;
    - the composition resolves a value and passes it in (composed inputs);
    - a feature exposes a projection shaped for reading, which others read instead of its own type;
    - the two ends become two instances of one abstraction that owns the format (§7.8, R5).
  - *Visible shape:* a `*pN` (shared value) whose type is a named domain entity appears inside two
    sibling feature subtrees. *Verify:* is any domain type referenced or destructured by two or
    more feature units? *Mechanizable:* yes: a type read by two units of its own layer.
  - *The aggregate case (a judgement, not a hard defect):* Spray's own version is the ground symbol
    (§3.6.1). State that most instances interact with, like a game score, can legitimately become a
    domain abstraction one layer down that everything depends on as knowledge. So a type in a
    *lower* layer that two features read may be a legitimate domain abstraction (a knowledge drop,
    fine), or Clean Architecture's shared-Entity coupling (business objects that every use case
    reads, bad). A tool can't tell them apart, so a shared *feature-tier* type is a defect (above),
    and a shared lower-layer aggregate is a prompt for a human.
  - **The aggregate case read strictly.** Spray says use cases depending on entities "is
    incompatible with ALA": an identity abstraction "should not be used as the carrier of
    information between two use cases", and "a particular use case should only know about it's own
    data" (§6.17.2). Two findings, each a "must" (−1):
    - *A consumer receives another feature's aggregate.* A feature that takes the whole aggregate as
      an input, or matches its type, knows its meaning (checkout receiving the whole cart and using
      three facts of it). Send only what the consumer needs: a dataflow carries "only the data that
      is needed by the use case" (§6.17.2), such as the cart's id and stored lines when checkout is
      requested, with the readiness rule moved to checkout.
    - *One use case's data in a type another use case shares.* A storefront cart carries promo,
      gift-wrap and shipping state, and a bulk-order portal's order lines are built on the same type.
      Split the storefront's concerns out, so the portal rests on a plain line-items abstraction
      (plain functions over a list of stored lines: find, add, change a quantity, remove, set stock)
      and each use case builds its own data on it.
    - *Narrow what a dataflow carries.* The same reading applies inside one feature's wiring: a
      totals abstraction that prices lines needs each line's quantity and amount, not the whole line
      with its product's name and stock.
    - *Still allowed:* a lower-layer abstraction many features use through its API without its data
      carrying any one use case's concerns (a `Money`, a rate table, a configured rule).
    - *Building an instance isn't reading it.* A feature "creates instances of domain abstractions"
      (§2.2); a module whose only use of an aggregate is calling its constructor doesn't know its
      data.
  - **Techniques, in more detail.**
    - *Give features private data and share only a key.* Two features never hold the same data
      structure. They share an identity (a product id) and keep their own data: the wishlist holds
      product ids, the cart holds lines that each carry a product id.
    - *Mediate cross-feature reads through a projection or the composition.* Hand feature A a
      projection of B (`in_stock?`), not B's own data, or have the composition resolve the value and
      pass it in.
    - *Make both ends one abstraction.* See R5's codec technique.
  - **What doesn't meet it.** Checkout pattern-matching on the cart's type; one `Order` type every
    feature reads and writes (Clean Architecture's shared entity); a domain aggregate that carries
    one use case's data and is shared with another (the strict reading above).
  - *Spray:* §3.6.1 (the ground symbol); §3.8 (no data coupling); §4.8.1 (no shared DTOs); §6.17.2
    (use cases depending on entities "is incompatible with ALA"; dataflows carry "only the data that
    is needed by the use case"; a use case's private data stored "against a customer identity"); §7.8.
    The identity-key technique is Spray's (§6.17.2: "A particular use case should only know about it's
    own data, and only store it against a customer identity"); only its name is this checklist's.
- **R11 — the composition layers are composition only: the application, and Features when an app
  has them.** The application instantiates, configures, and connects. It holds *all* app-specific
  knowledge and *no* app-specific logic: "no normal programming language code such as assignments
  and if statements" (§3.5), and about 3–10% of the code (§2.4). Spray's own examples show what that
  sentence covers and what it doesn't.
  - **Not banned:**
    - *Naming an instance so it can be wired twice.* `temperature = new FloatField()` (§1.6.6), or
      locals for cross-connections (§3.6.2). In a functional language, binding a configured value or a
      process address to a name so two wirings can use it.
    - *A predicate or small function passed in to configure a generic abstraction.* For example
      `new Filter(x => x>=0)` (§3.11.3), or `.Bind(x => x==0 ? -1 : 1000/x)` (§6.1.3). That states a
      requirement ("ignore negative readings") once, as configuration. The abstraction runs it.
  - **Banned:**
    - *Control flow that decides what runs:* a guard around a call, a loop over data.
    - *Assignments that hold or compute data between calls.*

    Spray flags his own §1.6.3 thermometer for this: the application "is still doing some logic
    work - the 'for loop' and 'if statement', which we will address soon".
  - **Features are compositions too.** When an application outgrows the size limit, Spray adds a
    Features layer, and "Each feature creates instances of domain abstractions, configures the
    instances with feature specific details, and connects them together as needed to express the
    feature or user story" (§2.2). Its UI layout and "the bindings of the UI elements to data" go with
    it, built from domain UI abstractions (§7.14). So everything this rule says about the application
    holds for a feature: a branch, a computation or handled data in a Features-layer module is a
    finding.
    - *A coded abstraction under the name Features.* A module of state and rules with ports
      (`Features.Cart`, `Features.Undo`) is a domain abstraction in Spray's terms. Putting it in a
      namespace or layer called Features misleads a reader who knows Spray's vocabulary: an R8
      "should", −0.5 on the full walk. Name the layer for what it holds (a state layer above the
      rules, say) and keep "features" for compositions. *Checklist reading.*
  - **No contained sub-component.** "There is no 'hierarchical' or 'nested' structure in ALA… There
    is no analog of a sub-module or sub-component, no such thing as a sub-abstraction. Abstraction
    layers replace hierarchical containment" (§2.2). A screen-specific stateful UI component (a
    generated form component, a screen's own panels) has its own state, lifecycle and event handlers,
    is used only inside one screen, and talks to it by message: a sub-component the screen contains.
    It's a "must" finding, −1 for one and −2 (structural) for several. Spray's provision for
    app-specific UI is layout and bindings, built from domain UI abstractions: "Using one in a
    specific application only requires a label and a binding to an action" (§7.14). So the screen
    composes its form from a generic domain UI abstraction it configures and wires, and a component
    that would read the same in another app moves down a layer as a domain UI abstraction.
  - **Where each kind of `if` goes.** These are Spray's moves, in order of how often they come up:
    1. *Propagation guards* ("only if there's a value", "stop on error") move into the connection
       mechanism. He factors the `if`s "into the Compose function" (Bind, the function that chains
       one step to the next in a monad, §6.1.3), and his
       thermometer's `if` disappears once `SampleEvery` simply emits nothing (§1.6.4). In a
       functional language: a step that returns "nothing to emit", a monadic chain that stops on
       error, or a filtering stream.
    2. *Requirement conditions* ("brew only if the pot is on and the boiler isn't empty") become
       wired instances: logic abstractions like the coffee maker's AND gate (§2.9.3), or a generic
       abstraction configured with a predicate (§3.11.3).
    3. *Conditions that depend on history* become a state machine (§4.16; "event-driven often goes
       hand in hand with state machines", §4.7.2).
    4. *Choosing a path by outcome* becomes an abstraction with two output ports that the
       application wires separately. His activity-flow `If` wires a condition's `donetrue` and
       `donefalse` ports (§4.9.1, which he marks experimental).
    5. *The order of follow-ups* ("record the undo, then start the timer") is fan-out wiring. Make
       the order explicit where it matters (§4.4.4).
  - *Why it pays:* with no logic at the top, "when fixing a bug, it quickly becomes clear whether
    it's the application code itself not representing the requirements as intended, or it's one of
    the abstractions not doing its job properly" (§3.5).
  - *Visible shape:* every line in an application function is an instance, a wire, a literal, or
    configuration. The tool stamps `(branches)` on an application function that branches. A reader
    then classifies each branch as a guard, a requirement condition, history, an outcome route, or
    ordering, and moves it as above. A branch that fits none of these is logic that belongs in a
    lower layer.
  - *Verify:* what fraction of the code is the application? For each branch or data assignment in
    it, which of the five kinds is it, and has it been moved?
  - *Mechanizable:* partly. The share of the code is mechanical, and so is counting branches, loops,
    arithmetic, and handled data (a result bound from one lower abstraction and passed to another, or
    passed straight into another's call as an argument; a nested constructor that builds an
    instance's configuration isn't). A pipeline of stages isn't counted, because it is how a
    functional language writes Spray's §1.6.4 chain, and statically it looks the same as a pipeline
    of values. Telling the kinds of branch apart is judgement, though some forms are recognizable.
  - *In a UI framework:* the framework hands a screen events to route, URLs, and a lifecycle to
    follow, so some forms remain: clauses that match an event name and forward it, decoding string
    params, and a lookup from URL to step. None of these is logic. Everything else a screen tends to
    contain (guards, rules, arithmetic, handled data, history, template loops, lifecycle checks) has
    one of the moves above, and a screen can reach zero logic this way.
  - **Techniques that meet it.**
    - *Split the shell: generic interpreter below, dispatch in the composition.* A shell that reduces
      a list of outcomes into view state is generic, so it's an execution model in the Programming
      Paradigms layer, below the composition. The part that names the composition's parts is
      composition code.
    - *Guards go into the connection mechanism.* A monadic chain that stops on error, a runner that
      stops when a stage returns "nothing to emit", or a filtering stream (§6.1.3).
    - *Requirement conditions become configured instances; history becomes a state machine.* A rule
      becomes a predicate passed to a generic abstraction (R3), or a small logic abstraction like the
      coffee maker's AND gate (§2.9.3). A rule that depends on what happened before becomes a state
      machine (§4.16): a pure transitions table, passed in as configuration so the feature decides.
    - *Keep an abstraction's own rules inside it.* The coffee maker's `Boiler` enforces "the heater is
      off whenever I am empty or my valve is open" itself. The application asks it to heat; it refuses
      when that would be unsafe. The rule never reaches the top.
    - *Route outcomes; don't decide on them.* A match whose arms only send the ok and error results to
      different places is routing two output ports (§4.9.1). A match that computes in its arms is
      logic, and belongs in a feature. An asynchronous task's result can be routed to a feature's
      `succeeded` and `failed` inputs with one clause each.
    - *Move iteration and compound conditions out of the screen's template.* A generic list or table
      component iterates (§4.12). A feature computes a compound boolean and the template wires it.
    - *Move lifecycle guards out of the screen's start-up.* Use the framework's own extension point
      for lifecycle hooks, configured on the screen, so start-up has no branch.
    - *Generate the glue.* Committed code generation makes the composition's wiring a generated
      file, so the hand-written composition stays thin (§2.5.1). A check fails the build if it drifts.
    - *Stop handling data at the composition.* Several designs get the composition's handlers to pure
      routing: bind every feature output once in one map per screen, run the screen as a circuit of
      instances and wires, make each feature a component instance that lands its own outputs and sends
      the rest, or write one wiring clause per port run by a small runner.
    - *Make start-up an event on the diagram.* Loading at start-up hands a store's result to a
      feature. Instead, deliver a `started` event through the wiring, bound to "read the store, feed
      the result to the cart's `load` input", so the composition never holds the result.
    - *Replace a composed input with an output.* A handler that reads composition state to build
      another feature's input (fetching stock for payment, finding a line for the wishlist) becomes one
      input on the first feature whose output carries the value (the cart sends `checkout_requested`
      with its lines, wired to checkout's `pay`).
    - *Route every message explicitly, and test that you do.* A composition's multi-clause message
      handler is its wiring. With a catch-all clause, a message nobody routes disappears silently.
      Without one, it fails loudly, and a test can send every port each instance declares it sends
      and fail on a missing clause. The trade: the composition must also ignore, explicitly, every
      broadcast on a shared topic it doesn't use. Explicit is safer where the test suite runs on every
      change; the catch-all is the more tolerant default where it doesn't.
    - *Move store work out of composition helpers.* A private composition function that the wiring
      calls is still application code. Placing an order, looping over the lines to decrement stock and
      broadcasting each change belongs in a domain abstraction that does its own I/O, configured with
      its stores, as Spray's `Display` writes to the screen itself.
    - *Place parts with generic components instead of comparing in the template.* Comparisons a
      template needs (which tab, which step, is a count zero, is the button disabled) move into
      generic components that take the current value and the name (a tabs component, a pane shown
      when current, a "none" note shown when a count is zero).
    - *Put a rule's display beside the rule.* A screen's match on stock status becomes a domain UI
      component that takes the configured rule and the screen's labels. It sits in the domain, next
      to the rule it uses, so the edge drops (R1).
    - *Let rows carry the booleans the template needs.* A feature's row projection includes what the
      template would otherwise compute (`at_minimum`, not `quantity <= 1`), or a row component takes
      the list it checks membership in.
    - *Configure a feature with its store so it loads itself.* Instead of reading the store at
      start-up and handing rows to a feature, give the feature its store as configuration and a `load`
      input.
    - *Build a screen's form from a generic record form.* A domain UI abstraction takes the fields,
      the words and a validation result, emits `validate` and `submit` with the params, and shows the
      errors it's given. The composition wires `submit` to a save instance and its two outcomes to a
      notification and navigation, or back to the form's errors. Nothing screen-specific has its own
      state or handlers.
    - *Add the foundation function that removes a hand-off.* Fetching a record and then deleting it in
      a handler is the composition passing one store result into another call. A store function that
      does both keeps the handler to one call.
  - **What doesn't meet it.**
    - Arithmetic and computed values in the composition (summing items into a display value).
    - A business rule in a handler ("can't check out an empty cart" as an `if`).
    - A match on the URL that encodes which steps may follow which.
    - A handler that takes one feature's result and passes it to another.
    - Loading rows at start-up and handing them to a feature.
    - A loop or a compound condition in the screen's template, including a comparison like
      `disabled = item_count == 0`.
    - A screen-specific stateful UI component: a contained sub-component, whatever it calls.
    - A module in a Features layer that holds logic, or a coded abstraction in a namespace called
      Features (R8).
  - **Limits.** Some forms stay in any UI composition because the framework or the language puts
    them there: routing by event name and URL, decoding browser params, storing the program value
    back (values are immutable), a lookup from URL to step, and a client-side hook or two. None is
    logic.
  - *Spray:* §2.9.3 (the coffee maker's application is a diagram of instances; its conditions are
    AND-gate instances, not `if`s); §3.5 ("no normal programming language code such as assignments
    and if statements"); §1.6.3 (the thermometer's `if` flagged as logic); §1.6.4 and §6.1.3 (guards
    move into the connection mechanism); §1.6.6 and §3.6.2 (instance variables for wiring);
    §3.11.3 (configuring with lambdas); §4.4.4, §4.9.1, §4.16. Reading multi-clause handlers as
    routing, and the framework-imposed departures, are this checklist's functional-language
    adaptation.

Rules-of-thumb for using R1–R11: R1–R2 and R5 are the coupling core; R3 is requirements-locus; R4 is
state-with-its-owner; R9 is the port/interface discipline and R10 the data-sharing discipline (both
coupling-core in spirit); R6–R7 are design minimality/nameability; R11 is the composition-only top
layer; R8 is the human-judgement remainder. A mechanical tool can check R1–R7 and R10–R11 (and R9 in
part) at varying precision; treat R8 as the reason a green run is necessary but not sufficient.

One topic is *in development* and not yet a rule: how wires differ in meaning and in how they run
(paradigms, push or pull, sync or async, fan-out, glitches, loops). See "Kinds of connection and
how they run" under the worked examples.

## Functional programming: what Spray says, and what it means in a functional language

Spray writes in C#, but he returns to functional programming many times, and it matters more for
functional languages than any other part of his site. His own chapters on it (§6.25 "Functional
programming", §6.26 "Functional programming with monads", §6.27 "Functional Reactive Programming")
say only "TBD". What he does say is spread across §2.3.6, §3.9, §3.10.2, §3.11, §6.1 to §6.3, §7.2
and §7.5. This section gathers it, says what each point means in a functional language, and names
the rule it supports. Where the reading is this checklist's and not Spray's, it says so.

1. **The application layer is pure functional code.** "ALA is, essentially, functional programming.
   All the top layer code that implements the application itself by wiring up instances of domain
   abstractions in some combination is pure functional code" (§3.11). The impure work lives below it:
   "The fundamental idea of monads, that of separating execution details, state, and I/O into
   pretested units that are then composed using pure functional code is the same for ALA" (§3.11.1).
   "Domain abstractions completely hide their contained state in well tested code and are composed by
   pure functional code" (§3.10.2).
   - *In a functional language:* the composition builds values and routes them. It doesn't call the
     store, send messages, or compute. Two placements of the side effects both meet this. Spray's is
     to hide them *inside* domain abstractions (his `Display` writes to the screen itself, his
     `CSVFileReaderWriter` reads the file). The common functional one (functional core, imperative
     shell) has features return effects as data and a generic interpreter in the Programming
     Paradigms layer perform them. Either keeps the top layer pure. The interpreter form keeps the
     features pure too, which makes them testable without the framework. *Rules:* R11, R4.
2. **Plain functions can be ALA.** "ALA can be applied to functional programming too. Abstractions are
   then obviously functions, and the same ALA relationship restriction applies - a function may only
   call a significantly more abstract function. The functions then form layers" (§2.3.6). "Parameters
   and return values are effectively port", and when a function used to call a peer part-way through,
   "the second function will now need to be passed into it. The function parameter is also a port"
   (§2.3.6). And the wiring pattern isn't always needed: "if your whole problem is just an algorithm,
   and therefore suits a functional programming style, then you can still compose abstractions with
   function abstractions, provided all function calls are knowledge dependencies, and not say, just
   passing data or events" (§7.5).
   - *In a functional language:* a pricing calculation that calls a money type, a decimal library and
     a tax-rate function is fine as plain calls down the layers. It needs no ports, runner, or
     circuit. Ports and wiring are for where instances *communicate* (one feature's result feeding
     another, events, UI). Don't build a circuit for arithmetic. *Rules:* R1, R7, R9.
3. **Two ways plain functions go wrong** (§3.11.1). "Functions expose their inputs and outputs to the
   layer above, but the layer above is not interested in the data itself, only in the abstract
   concept of what the function does." And "functions, or sets of functions, that naturally associate
   strongly with some state must have their state passed into and back out of them every time they
   are used." Across several layers it gets worse: "The middle layer functions end up with extra
   parameters that don't have anything to do with them, just so they can pass state data through to
   even lower functions. This makes these intermediate functions not great abstractions in
   themselves." §3.9 says the same of procedures: "Some procedures will need extra parameters even
   though they don't need the data, because they need to pass it through to other procedures that
   they call."
   - *In a functional language:* the first problem at the top is R11's "handling the data" (§1.6.3).
     The second is R4's reason for keeping state with its owner. The third, a parameter carried
     through a function that never reads it, is R6's tramp-parameter "should". Functional code
     invites it, because threading an options map, a context, or a session value down several calls
     is the easy default.
4. **Abstraction before referential transparency** (§3.11.2, Summary). "ALA prioritizes abstraction
   over referential transparency." Referential transparency "attempts to improve analysability by
   always removing time from the analysis, even when time is a fundamental aspect of what is being
   described. It will expose implementation details if necessary to do it." His example: "Passing the
   running average to the function every time there is new input data breaks an otherwise good
   abstraction." A computation is `input + state --> state + output`, which you can group two ways,
   the functional `(input + state) --> (state + output)` or the object form `input ( + state -->
   state +) output`, and "In ALA we choose between these two philosophies on a case by case basis."
   - *In a functional language:* immutability forces the functional grouping, so a filter's memory
     comes back as a new value. What keeps the abstraction whole is that the caller stores the value
     *without looking inside it*: the state is still the filter's. A runner or a program value held
     by its owner removes even the storing ("Past the pipe"). A process or actor is the object form.
     Use it where the concept is concurrent, not to imitate objects. *Rule:* R4.
5. **From procedural code to ALA, without object-oriented design** (§3.9, §3.10). Spray derives objects
   from procedures in five steps, and in §3.10 calls ALA's use of them "objects as a language feature,
   not a design philosophy". Configuration goes in a struct, and that struct "is immutable".
   Configuring once matters: "If the abstraction consisted only of a single function, then that
   configuration data would need to be passed in every time the function is used. That would be
   awkward. It would also mix the data parameters of the function with the configuration parameters,
   breaking the Interface Segregation Principle." Wiring is two more fields, and "The fields are
   immutable after they are set." State that belongs to one abstraction joins its struct. State that
   belongs to none becomes a state abstraction with dataflow ports, wired in. The result: "both the
   configuration data, and the wiring data stored in an instance can be immutable. Only instances of
   abstractions that contain state data are mutable, and this is clear from very nature of the
   abstraction."
   - *In a functional language:* a record holding configuration (and state, when the concept has any)
     is Spray's instance. Configuration set once, apart from run-time data, is R9's "should". One way
     is a configuration value as the *first argument*, with run-time inputs after it. *Rules:* R9, R4.
6. **Monads are the nearest functional analogue, and a two-layer pattern.** Spray's thermometer detour
   (§1.6.4) builds the program as a monad chain and then runs it: "Monads have allowed us to separate
   execution flow from composition flow." His summary of the monad pattern (§6.1.16): "monads are a
   2-layer pattern. The two layers correspond roughly with ALA's application and programming
   paradigms layers. The code that uses Bind to compose functions, and the lambda functions
   themselves are in the application layer. The Bind function and the Interface<T> are in the
   programming paradigms layer." Library functions such as Sort and Filter "would go in the
   equivalent of the domain abstractions layer". Bind is also where the checks between calls go: in
   imperative code "we might typically need some extra code after every function call", and "we can
   put that extra code inside the Compose function instead. This refactoring is essentially what the
   monad pattern is" (§6.1.2).
   - *In a functional language:* a result or option monad's chaining (do-notation,
     `and_then`) is a Bind for success and failure. A lazy stream pipeline is a deferred monad. An
     interface such as `push(step, data) -> emit(out, step) | quiet(step)` plus a runner is a
     push-style Maybe bind: a step with nothing to say returns `quiet`, and the runner, not the
     application, stops. The runner and interface belong in the Programming Paradigms layer. *Rule:*
     R11 (guards move into the connection mechanism).
7. **Where monads fall short** (§3.11.3, §6.2). "ALA and monads are both about composition... The
   difference between ALA and monads is that ALA composes objects whereas monads compose functions."
   Monad functions "just have two 'ports', an input and an output"; "ALA's domain abstractions, on the
   other hand, can have many ports", and those ports "can use different programming paradigms, not
   just a specific data flow". "Monad libraries are more like discrete electronic components such as
   resistors and capacitors"; domain abstractions are like integrated circuits. "Composing functions
   creates mostly a chain structure whereas composing objects with ports creates an arbitrary network
   structure" (§6.2).
   - *In a functional language:* a pipeline or stream chain suits a requirement that is a line. When
     one output feeds two places, or UI, events, and dataflow meet, the composition is a graph: a
     value holding named instances and a wire list, run by a runner, or a UI template for the UI part.
     See "Kinds of connection". *Rules:* R8, R11.
8. **Built once, run forever, and push by default** (§3.11.3, §3.11.4). "In ALA, things are static (or
   declarative) on a larger scale than just a single line of monad code. In fact the entire
   application is wired up once at the beginning when the application starts executing, and then all
   ports are considered infinite steams that work as long as the application is running. To get a
   finite sequence, you design a port that is an infinite stream of finite sequences." He criticises
   Rx for the opposite: once `OnCompleted` fires, "the object structure wont run ever again. It has to
   be reconstructed". On push: "The reason to prefer push is that push can be either synchronous or
   asynchronous, whereas pull can only be synchronous (unless you use future objects.)" So "the choice
   of synchronous or asynchronous is deferred until the application user stories are" written
   (§6.2).
   - *In a functional language:* a lazy stream is pull-driven and is consumed by one run, so it suits
     a batch you iterate, not a long-lived screen that receives one reading at a time. The run-forever
     form is a composition value built once at start-up, held by its owner, and fed each input as it
     arrives. A push interface keeps the sync-or-async choice open: the same step values run
     synchronously in a pure chain and asynchronously inside one process per instance, unchanged.
     *Rules:* "Past the pipe", "Kinds of connection".
9. **Explicit structures, not hidden closures** (§3.11.3, §6.1.1, §6.2.2). "Monad library code usually
   builds large object structures full of delegate objects, closure objects and other objects 'under
   the covers'. This 'under the covers' structure makes monads difficult to understand, trace and
   debug. ALA also creates object structures, but it's done explicitly with wiring code." He keeps
   creating an instance separate from wiring it (`WireIn(new Filter(...))`, not `.Filter(...)`)
   because domain abstractions are written often, because the application can choose chaining or
   fan-out, because it can name which port, and because "Explicit WireIn and WireTo operators allow us
   to directly translate a diagram to code" (§6.2.2). Fluent helpers that do both are fine "for very
   common cases" (§3.11.3).
   - *In a functional language:* a composition held as data (records plus a wire list) can be
     printed, checked for unwired ports, diffed, and drawn. A composition held as nested anonymous
     functions or a lazy stream can't be inspected. Committed generated code beats a macro for the
     same reason: the wiring is a file you can read. Building a circuit and then wiring it keeps the
     two steps apart; a pipeline DSL that creates and connects in one call is the fluent form, fine
     for a line. *Rules:* R8, "Past the pipe".
10. **Use a monad library you already have** (§6.3, Summary). "There would be no sense in reinventing
    that functionality as 2-port classes." He gives two ways: give some ports the library's own
    chaining type and put the library's operators between two instances (§6.3.1), or "write a general
    purpose domain abstraction that can be configured with a monad expression" (§6.3.2), which the
    Summary calls a domain abstraction called *query*.
    - *In a functional language:* don't write `Filter`, `Map`, or `Sort` domain abstractions when the
      standard collection and stream libraries exist. Either type a port as an iterable and put
      stream functions between two instances, or write one generic `Transform` step configured with a
      function the composition supplies. *Rules:* R7, R9.
11. **Lambdas: anonymous for one-offs, passed in for configuration and for calling up.** A function
    used once "is not an abstraction. The name itself becomes just a symbolic connection between two
    points in the code... It's indirection without abstraction. ... Lambda expressions solve this
    problem because they are anonymous" (§6.1; the `func1` point, §1.6.3). Lambdas also configure
    generic abstractions (`new Filter(x => x>=0)`, §3.11.3; the bowling `Frame` completion lambdas,
    §4.20). They are one of the legal ways to call up the layers (§2.1.3, §4.4.1). And they adapt
    mismatched ports: "This can be as simple as a lambda expression passed to the WireTo operator, in
    the same way that you would pass a lambda expression to a .Select clause in LINQ" (§6.17.4).
    - *In a functional language:* anonymous functions and partial application, written at the
      composition. *Rules:* R6, R9, R11.
12. **"No side effects" is not the goal** (§7.2). "Another contender to be king is 'no side effects'
    used by the functional mathematical purity guys... this lord is only effective because he usually
    serves the abstraction king. But, again, there are times when he doesn't, and 'no side effects' is
    not enough to make a good abstraction."
    - *In a functional language:* immutability rules out shared mutable state (R2) almost for free,
      and that inflates how clean a functional codebase looks. Pure code can still be fully coupled: a
      pure function that calls a peer (R1), reads a sibling's data type (R10), or matches a tuple shape
      another module produces (R5). Judge the design by the rules, not by purity.
13. **Immutability doesn't settle concurrency** (§3.10.2). "In functional programming, the issues of
    multithreading are handled by using immutable data. I suspect that systems that only use immutable
    state also have problems when using multiple threads. For example one thread could be using and
    inconsistent or outdated copy of the current actual state." His own rule is one thread per
    stateful instance, and for asynchronous calls, chaining with a task or future's Bind ("effectively
    the monad pattern for asynchronous calling"), or failing that, writing the abstraction as a state
    machine.
    - *In a functional language (this checklist's reading):* every long-lived process or session holds
      its own copy of shared data, such as stock levels. Decide at wiring time which instance is
      authoritative (a process or the database), treat each copy as a cache, and wire updates to it
      (a subscription set up by the composition). Async work goes to a task or future, and a
      long-running activity becomes a state machine, not a blocked process.
14. **Recursion needs a forward-declared abstraction** (§8.1). "Circular knowledge dependencies happen
    all the time in functional programming where recursion replaces iteration." Recursion inside one
    abstraction is its own business. Across abstractions, put the abstract concept one layer down as
    an interface that the concrete parts both provide and accept. A chain of calls that loops back is
    usually run-time communication and should be wired instead.
    - *In a functional language:* a `Chain` implementing `Step`, so a chain can be a step inside a
      bigger chain, is this pattern. So is a tree of UI components that render children through a
      shared slot or interface.

## Where this checklist departs from Spray

The rules follow Spray, and each one cites where. These are the places where this checklist adapts
him to functional languages or steps away from him, stated so a reader doesn't mistake them for his
positions.

| Departure | Spray | This checklist | Why |
|---|---|---|---|
| Branches in a UI composition | "no ... if statements" in the application (§3.5) | multi-clause handlers and forwarding matches read as routing; a lifecycle check tolerated where the framework requires it (R11) | the framework delivers events and a lifecycle to the screen; the departures are recorded, kept small, and never used for logic, and most can move to a framework hook |
| Size | a fundamental constraint (Summary) | module size is an advisory check (R7): a module over 500 lines is scored only in a strict mode, and an average under 100 lines is reported without scoring | line counts are a weak proxy for "readable alone", so a reader decides |
| Graded checks | ALA's constraints aren't graded | enforcement tiers: R7 and height advisory, R11 and public surface aspirational, the app-layer share, average size and shared aggregate reported only | lets a team adopt the checklist step by step; the tier is about scoring, not about whether a finding is real |
| Sharing data between features | no data coupling (§3.8), no shared DTOs (§4.8.1), no shared entities (§6.17.2) | R10 states the same property, applied to features: no domain type read by two features | not a departure in substance; the identity-key technique that meets it is Spray's own (§6.17.2) |
| The notation | diagrams and wiring code (§3.6) | a text encoding with `[tag]`, `$`, `q`, and tool-stamped marks | a whiteboard- and linter-friendly way to see the shapes; it isn't Spray's |
| Framework residue in a UI composition | the application is wiring and configuration only (§3.5) | param decoding, a URL-to-step lookup, a client-side hook or two, and component instances kept alive but hidden are recorded as departures | the browser sends strings, the framework hands the screen the URL, some things only the client can do, and some frameworks can't deliver to an unmounted component |
| Holding the program value | objects change in place | an immutable program value held by its owner (a screen's state, an actor) and stored back after each run ("Past the pipe", R4) | values are immutable; holding the value is not handling the data |
| Execution model | prefers single-threaded solutions (§4.4.9) and treats the execution model as a wiring-time choice | the same property: execution-model choices are made where the instances are wired, and state belongs to its concept (R4, "Kinds of connection"). Whether to use processes is a technique | not a departure in substance; listed because languages with cheap processes make them tempting |
| Comments | "critically important" abstraction comments (§2.1.1) | a short module comment naming concept, ports, configuration, and an example when needed (R6) | covers what Spray asks for without narrating bodies |

## Enforcement tiers (what a linter scores, and when)

Not every rule is the same kind of obligation. The coupling and locus rules are hard requirements; the
minimality and shape rules are prompts a reader weighs; a couple are aspirational ideals a real app
cannot fully reach. A linter can encode this as tiers:

| tier | rules and sub-checks | when scored |
|---|---|---|
| **Required** (a violation is a defect) | R1, R2, R3, R4, R5, R6, R9 (owned interfaces), R10, layer-validity | always (default) |
| **Advisory** (a prompt for a reader) | R7, module-size (over 500 lines), abstraction-height, pass-through, tramp parameters (R6's "should"), reference-level R1, self-subscription (the bottom layer may own its topic) | reported by default; scored in a strict mode |
| **Aspirational** (a purity ideal, not always obtainable) | R11 (no logic at the top), public-surface (encapsulate the little ball of mud; counted per function) | reported by default; scored only in the strictest mode |
| **Reported only** (a ratio or a design choice, not a defect) | the application's share of all functions, files averaging under 100 lines, the shared domain aggregate | reported at every tier; scored only when a team opts in |
| **Not machine-scored** | R8 (judgement); the rest of R9 (outputs that name a destination or command, peer DTOs) | a human reads for these |

Two placements are deliberate and follow from the "little ball of mud" reasoning under R7. **R11** is
aspirational, not required. A UI composition can reach zero findings, but only by adopting a design
built for it (results on ports bound by the composition, a circuit of instances, or feature instances
that announce what they did), and a CI gate shouldn't choose the design for a team. Its findings are
still real, and each one is either moved or recorded as a departure (see R11).
**Abstraction-height** and **pass-through** are measured on the graph of *abstractions*, not the raw
call graph: a call inside one module is internal decomposition, so it adds no height and is no
pass-through; only real hops and public cross-module renames between abstractions count. And
**public-surface** exists so that a little ball of mud stays little: a wide public API means the mess
has leaked past the boundary.

## Background, procedures, and how to verify (the guide behind the rules)

R1–R11 are the *detection* instrument. This section holds what did **not** become its own rule:
(a) background each rule rests on, (b) the procedure to *build* an ALA program, (c) the procedure to
*refactor* one, and (d) the procedure to *verify* one is ALA. The further
constraints behind the rules (ports, shared entities, the composition-only top layer, the
abstraction-quality tests) are captured as R6–R7 and R9–R11, not repeated here.

### Background behind the rules (framing, not separate checks)

- **Design-time coupling, not run-time communication (behind R1–R2).** Run-time communication is
  necessary and fine, and may be circular; design-time coupling (how much of one piece you must
  understand to read another) is what ALA drives to zero. Classify a dependency by when it first
  breaks: only a design-time break (the code *loses meaning*) matters. So circular *wiring* is fine
  and common (feedback, undo, restore); only circular *design-time dependencies* are forbidden, and
  they never arise once every edge drops. *Dependency* is what the compiler sees; *coupling* is what
  the brain sees. (§7.4.3; circular wiring, §4.4.8.)
- **Zero coupling is not "loose coupling" (behind R1–R2).** ALA does not *minimize* dependencies; it
  *eliminates the bad ones and maximizes the good ones*. The real enemy is **collaboration coupling**
  (one unit doing specifically what another needs, in a fixed arrangement) which an interface hides
  but does not remove and which grows during maintenance. (§3.3.1, §3.4.1, §7.4.)
- **A knowledge dependency may target *any* lower layer (behind R1).** ALA is *not* "depend only on
  the immediate layer below" (that is partition layering). A big altitude skip (a "long drop") is
  **not** a smell — do not flag it. Understanding one composition line may need several lower layers
  (domain, paradigm, wiring operator, host language, ALA itself); name those in the readme. An end
  user of the composition needs none of them. (§3.4.6; the readme, §2.3.7.)
- **Compose, don't decompose (behind the layer model and R11).** Invent and assemble general
  abstractions (Lego), don't split the system into specific collaborating parts (jigsaw); a good
  test is whether the instances are composable in *other* arrangements. **Separate by feature first,
  not by tier** (splitting UI-from-logic-from-storage couples them, since UI shapes logic). **Layers
  replace hierarchical containment** — flat, no nesting, no sub-abstractions. **No inheritance** (it
  is usually "lazy composition"; replace with explicit pass-through — a subclass is more specific and
  must know its parent, an upward dependency), **no global event names, no receiver subscribing
  itself** to a sender or a public topic (those are peer edges in disguise; the composition wires
  them). A callback the composition passes *down* is fine: that is how ALA calls up the layers.
  (Composition vs decomposition, §2.6, §3.7, §7.12; Lego, §2.8.2; by feature, §7.14; layers replace
  containment, Summary, §2.6.1, §7.18; inheritance, §7.16; global event names, §4.7.4; registration,
  §4.4.2.)
- **Ports go sideways into technical domains (behind R9).** Database, hardware, and network are
  run-time dependencies off to one side, reached through paradigm ports (like the spokes of
  Hexagonal Architecture, also called ports and adapters), not
  bottom layers and not downward adapters. Keep a *domain abstraction of* the DB or UI (configurable,
  composable — a persistent `Table` wires to a grid), not a raw port to an adapter. An OSI-style stack
  becomes a sideways chain of domain abstractions. (§6.19, §7.17.) A database library, an HTTP
  client, or a message bus is such a technical domain: reach it through an abstraction of it, wired
  in, not by calling it from every feature.
- **Two phases, and the diagram is the source (behind R11).** Wire the network once, then run it (a
  supervision tree that starts every process at boot is exactly this). The topology/"diagram" is the
  single source of truth for requirements, architecture, and code, and should be flat, declarative,
  and greppable. (Committed generated code is one way to make that mechanical.) Spray: "the entire
  application is wired up once at the beginning when the application starts executing, and then all
  ports are considered infinite steams that work as long as the application is running" (§3.11.3;
  also §1.6.4, §2.4, §3.6).
- **Abstraction is not the same as instance (behind R6/R9).** The abstraction is the zero-coupled
  design artefact; the instance is the run-time thing that communicates. Blurring them is what tempts
  people to put dependencies *between abstractions* to move data, destroying them as abstractions.
  (§3.2.2.) In a functional language, a module is the abstraction; a value or a process is an
  instance.
- **Execution-model choices are design-time, made on performance/expressiveness grounds.** Push by
  default, pull for performance; decide sync vs async at wiring time (an asynchronous message vs a
  synchronous call, a broadcast vs a direct send), never inside a domain module; model time-spanning
  activities as state machines, not threads (§4.4, §4.19). Spray calls the usual rule of thumb, GALS
  (synchronous locally, asynchronous across processors), "too simplistic": anything that takes real
  time, such as I/O or a delay, should be asynchronous even on one processor (§4.6.1). In a language
  with lightweight processes that's natural, since message passing between processes is
  asynchronous.

### Procedure — build a new ALA program

Two roles, even when one person plays both, "wearing only one hat at a time" (§5.7). The
*architect* works from the requirements, expressing each as a wiring of instances and inventing the
domain abstractions it needs. The *developer* implements those abstractions, which "know nothing of
the requirements". Most of the hard thinking is the architect's, including designing paradigm
interfaces that work "between any two domain abstractions for which it may be meaningful".

1. **Iteration zero (a short up-front design pass, ≤ one sprint, whatever the project size).** Go
   through requirements one by one, fast (aim ~one feature/hour), *describing* each as a wiring of
   instances and **inventing domain abstractions and paradigms as you go** (each gets config params
   for the specifics). Decide app-layer vs domain-layer by *scope of knowledge* (specific to this app
   → app; reusable → domain). Output: a topology plus a named list of abstractions, each with a
   one-line insight, **no implementation yet**. This is not waterfall — the rest is zero-coupled
   abstractions, so implementation cannot feed back and force redesign. (§5.2.1, §7.26.1.)
2. **Invent the abstractions** by mining the requirements for recurring nouns; set aside anything
   that "even begins to look like implementation" as a seed; start with the UI (easiest to see), then
   connect it to data sources and transforms; turn any one-off module into a *general part* plus a
   *config part*. The target for each: it **separates two worlds** (R6) and does not know its own I/O
   endpoints (R9). For a recursive concept, forward-declare its abstract interface one layer down and
   let concretes both provide and accept it — never a circular dependency. (§5.2.2; recursive
   abstractions, §8.1.)
3. **Choose each paradigm's execution model** on performance/expressiveness grounds (push vs pull,
   sync vs async), keeping that choice out of the abstraction and in the wiring. (§4.4.)
4. **Implement, then wire.** Each abstraction is an independent little program, fast to write because
   there is no coupling to reason about. Use **convention over configuration** (enforced config in the
   constructor, optional settings defaulted); give every instance a **name** for debuggability (an id
   in its configuration for logs and telemetry, not a globally registered name, which would let
   senders find it themselves); put a **root readme** naming the knowledge prerequisites (ALA, the
   paradigms, the domain abstractions, "the diagram is the source"). Velocity climbs as the domain
   matures. (Convention over configuration, §5.5; the readme and knowledge prerequisites, §2.3.7,
   §5.6.)

### Procedure — refactor an existing program to ALA

Spray states the target in six lines, for his thermometer (§1.6.1):
- "every piece of knowledge about 'being a thermometer' will be in one function"
- "that 'Thermometer' function will be at the top"
- "that function will do no real work itself"
- "how to do more abstract things will be put into other functions"
- "those functions will not know anything about temperature or thermometer"
- "The top layer function will compose the abstract functions it needs to build a thermometer"

Swap "thermometer" for your product and it's a summary of R1, R3, and R11.

- **Spike-then-refactor** (this checklist's framing, not Spray's term; also the fallback when you
  can't see the abstractions): get it working messily, then (1) move requirement-detail up to the app
  layer (R11/R3), leaving generalized units with params/config; (2) merge similar generalized units,
  their differences becoming config; (3) cut the peer calls — pass in functions/ports (a function
  passed in mid-computation *is* a port), and move any observer/subscribe *registration* up to the
  composition.
- **Legacy, per user story:** reverse-engineer the call tree for one story (an all-files
  search is legitimate here), pin it with acceptance tests at its input/output boundaries, factor the
  call-tree into a new domain abstraction (copy-pasting useful snippets) (§8.3), mark the old modules
  for deprecation, and repeat per story.

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
8. **Tests replace only ports:** "you always test with dependencies in place, but you mock the
   ports" (Summary).
   - *Must:* a test never replaces a knowledge dependency (a module the subject relies on for its
     meaning, in a lower layer), just as you wouldn't mock a square root. *Reviewer's test:* does any
     test swap out a module the subject calls by name, rather than an instance wired to one of its
     ports?
   - *A domain abstraction's unit test* uses its real lower-layer dependencies, and wires fake
     instances to its ports.
   - *Testing the application* with its real domain abstractions "is exactly acceptance testing".
   - *For example:* pass a fake into the port (a stub implementing the paradigm interface, a function
     passed in, a test process as the output). A mock of a paradigm-layer interface is a fake port,
     which is fine. A mock of a lower-layer module is the failing case. A test of a screen with its
     real features is the acceptance test. A test that must replace a *peer* by name is a sign the
     peer was never behind a port (R1, R9).
9. **Wait for async work in UI tests.** When a screen starts an asynchronous task, a test that reads
   the screen right after the click is racing it. Wait for the screen's async results before
   asserting; a test that doesn't will pass or fail by timing.

## Helper proliferation: the real problem R7 is chasing

The naive reading of R7 is "flag functions that aren't reused." That reading is wrong on both ends,
and worth pulling apart, because getting it wrong either nags good code or misses bad code.

**Reuse is neither necessary nor sufficient.** A single-use function can be a perfectly good
abstraction: a `discount(subtotal, tiers)` called from one place still *names a decision* and
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

- **Triviality + meaningless name.** A one-liner whose body is a single call or literal *and* whose
  name teaches nothing (`op`, `f1`, `do_it`) hides no decision. A linter can fail on this, but only
  when *both* conditions hold — a well-named one-liner is left alone on purpose.
- **Nameability (R6).** If you cannot give it a general name, it is wiring, not an abstraction.
  This catches proliferation from the naming side and needs no call-graph.
- **Abstraction height (advisory).** The longest chain of knowledge-dependency edges. Thin
  pass-throughs stack up: `A → B → C → D → …` where each layer only forwards. A real requirement
  rarely needs more than a handful of altitudes, so a linter can report the height and **warn past a
  configurable ceiling (5 is a reasonable default)**. Height is a *proliferation* signal precisely
  because inlining wiring collapses chains, while genuine abstraction layers stay shallow and wide.
- **Pass-through shape (fan-in / fan-out).** A function with exactly one caller *and* one
  project-internal callee, whose body is just that callee, is a rename — a strong proliferation
  signal. High fan-in (many callers) is the opposite: evidence of a genuinely shared abstraction,
  and is never flagged. A detector on the function call graph that scores "1-in, 1-out,
  body-is-a-single-call" rather than raw reuse-count targets the pass-through directly and spares the
  useful single-use abstraction (a helper that hides a *decision* — a branch, a formula, a format —
  is not a bare delegating call, so it is left alone).
- **Altitude of the edge (R1).** A "helper" at the *same* altitude as its caller that only moves
  data is a communication dependency, not an abstraction — R1 already covers that. Genuine
  abstractions sit *below* their callers.

Because of all this, R7 and abstraction-height are best treated as **advisory by default**:
reported, but not folded into the score, so they never fail a build on their own, with a team able
to opt in to enforcing them. This matches the checklist's stance — reuse and minimality are prompts
for a reader, not gates — and it keeps a tool from punishing the very "abstractions created for
their own sake" that ALA permits.

### A note on modules (for a linter)

The checklist's encoding is *function-centric*: `f [tag]`, edges between functions, no module concept
required. A linter for a language with modules will still use them, in three concrete ways: (1) a
fallback R1 check (cycles, with no layer map) and a reference-level R1 advisory run on the
module→module graph; (2) R7's dead/single-use check is scoped to a module's *private* functions
(privates are the module-local, unambiguously-inlinable case); (3) `[tag]`s and the layer map
attach to module names. That is a pragmatic concession to languages where the module is the natural
unit of compilation, privacy, and naming — not a claim that ALA needs modules.

Deliberately, a linter should **not** constrain *which functions a module may contain* — no rule of
the form "all functions in a module must share a layer/tag." Such a rule would import a containment
hierarchy that ALA explicitly replaces with layers, and it would break the function-centric encoding
(which can describe a design with no modules at all). Modules are how a tool *locates* functions, not
a compliance constraint in their own right.

The finer-grained direction is a **function→function call graph** (node = a specific function with
its module and arity, edges resolved through each module's import and alias table), with the checks
that need altitude — **R1 altitude**, **abstraction height**, and the **pass-through detector** —
running on *that* graph. Only **R1 cycles** (the no-layer-map fallback) stays at module granularity,
deliberately: a function-level cycle is usually legitimate mutual recursion, whereas a *module* cycle
is the real coupling smell.

### Assigning functions to layers (convention + tags, never inference)

Layers are a *build-time* decision, so a linter cannot infer them from structure without making
R1 tautological (define altitude from the edges and every edge "drops" by construction; only cycles
remain visible). Instead a linter maps each function to a **user-declared** layer, resolved
**most-specific-wins**:

1. the function's own layer annotation (the exceptional per-function override — it lets functions of
   different layers legitimately share a module, since the encoding is function-centric);
2. else a layer whose pattern matches a module or library the code imports or extends (a database
   schema is persistence even in a domain namespace);
3. else the first layer in the ordered map whose **module-name pattern** or **file path pattern**
   matches — so a project declares its own convention (e.g. `/features/` → `feature`) rather than
   inheriting magic directory names;
4. else **unassigned**.

This yields three separate checks, kept distinct because they fail for different reasons: **coverage**
(what fraction of functions are assigned — the migration metric, with the unassigned list as the
worklist), **validity** (a tag naming an undeclared layer — a typo or stale tag, a hard error), and
**behaviour** (R1 — do the assigned edges drop?). The convention must be *written in the layer map*
and echoed in the report: convention is fine, but invisible convention is how R1 silently goes
wrong. On a codebase not yet organised into layers, everything is unassigned, coverage reports 0%,
and the structural checks that need no layers (module cycles, R7, height, pass-through) still run —
the on-ramp for moving an existing app toward ALA.

Deliberately, there is still **no** rule constraining which functions a module may contain; modules
locate functions, they are not a compliance unit. (Nested modules are *qualified* by their enclosing
module, so a feature and its own nested value types read as one unit and their calls are cohesion,
not peer coupling.)

### The application layer, and where UI-framework pieces sit

Spray treats the application as **one abstraction** whose content is wiring and configuration. The
knowledge of what flows between its parts is "contained together inside the Thermometer
abstraction". It has no internal structure of its own: "there are no subfolders under the
application" (§5.4). A bigger app becomes a composition of **Features**, a layer of their own that
the application wires. So calls between the application's own functions are internal to one
abstraction, not peer edges (R1). The one caution is R1's working-chain clause: product functions
doing work and calling each other.

**UI-framework pieces as abstractions.** A screen in a server-rendered or functional UI framework is
not a stack of application sub-layers (shell → screen → view). Each piece is either the application,
a domain abstraction of one of the kinds above (UI, feature, data source or sink), or a paradigm.
Spray's UI abstraction (see "What a domain abstraction is") sets what the UI pieces may do: render
what's wired in, emit events, contain other UI. *This mapping is the checklist's reading.*

| Piece | What it is | What it may do |
|---|---|---|
| The screen's module (start-up, callbacks, render) | Application | instantiate and configure, wire, hold application literals; no computing, deciding, fetching or persisting of its own (R11) |
| The screen's template | Application: the UI-layout wiring | contain UI abstractions in each other (the UI-layout paradigm), wire data into them through attributes and slot values, name the events they emit |
| Event, message and async-result handler clauses | Application: wires | route an emitted event, a message or an async result to one input, decoding params at the edge; clauses in one module are one place (R8) |
| The router | Application | it composes screens |
| A stateless UI component (a row, tabs, a button) | UI domain abstraction | render data passed in as attributes; emit events whose names it's given; contain children through slots (its UI-layout port); no state, no I/O |
| A stateful UI widget (a menu, a date picker, an autocomplete) | UI domain abstraction with internal state | keep the state of its own interaction (open or closed, the text typed so far); emit events; never fetch, persist or apply the product's rules |
| A stateful component that holds a feature's value | a host | fine when it's generic (a paradigm that runs any feature, renders only what the screen passes in through slots, and sends outputs); a bundle when it also renders its own markup and does I/O, which is an R6 finding |
| A screen-specific stateful component (a generated form component, a screen's own panels) | a contained sub-component | nothing: it's an R11 finding (§2.2). Use a generic domain UI abstraction the screen configures and wires, or move a reusable one down a layer |
| State-and-rules modules (a cart, an undo offer) | in Spray's terms, stateful domain abstractions; his features are compositions (see "When the wiring outgrows one composition") | `(state, outputs)` steps with declared ports; no markup, no events by name, no I/O unless I/O is the concept |
| Store, gateway and placement instances (add a line, place an order, charge a payment, a store module passed as configuration) | data source and sink abstractions | do the I/O they're configured for; wired by the screen into a feature's inputs or from its outputs |
| A domain UI component configured with a rule (a stock indicator) | UI domain abstraction | render passed-in data using a configured rule instance |
| A generic shell, effect interpreter, runner or host | Programming Paradigms | an execution model: it knows how a kind of connection runs, not what this screen does (Spray keeps "Execution models.doc" in this layer) |
| The UI framework itself | Programming Paradigms / Foundation, library-provided | its event → state → render loop is an execution model; a screen implementing its callbacks is configuring a more general module (R9) |

What follows from the table:
- **Data is wired into UI, never fetched by it.** A component that calls a store to get its rows, or
  to save a change, is a UI abstraction with a data source or sink inside it. Pass the rows in as
  attributes or slot values, and wire its change events to a store instance at the screen (§5.2.2,
  R6).
- **Interaction state is a UI abstraction's own business.** A stateful widget may hold what it needs
  for its own interactivity (which tab is open, a draft value) without that being a finding. The line
  is crossed when it holds the product's state (a cart's lines) together with markup and I/O.
- **A feature inside a stateful component needs a generic host, or its markup and I/O taken out.** A
  generic host runs the feature, the screen passes UI abstractions in through slots, and the screen
  wires changes to the store. Holding features as values in the screen's own state is the other way,
  and needs no host at all.

**UI-framework features against the rules.** How each use meets a rule or misses it. *The checklist's
reading throughout.*

| Framework feature | Meets the rule when | Misses it when | Rule |
|---|---|---|---|
| Start-up | it configures instances and sets their starting values; a read happens through a runner or a pull port (a read function passed as a source) | it reads a store and hands the rows to a feature or component | R11 |
| Event, message and async-result handlers | each clause forwards one event, message or result to one input, decoding params at the edge; all clauses sit in one module | a clause computes, decides, or builds one feature's input from another's state | R11, R8 |
| URL-change handling | it looks the step up in a screen-owned table and passes it to the feature that decides | it encodes which steps may follow which | R11 |
| Screen state (model fields, view state) | it holds the landing points of outputs and the screen's configuration | the screen derives values in it (a computed total) | R11 |
| The screen's template | it places UI abstractions, passes data in through attributes and slots, names events; a condition on one boolean, field reads | a comparison, arithmetic, a match, a loop, or a call into a lower module (use a pane, a "shown when" component, a list component, or a row that carries the boolean) | R11 |
| List updates | a feature's `rows` output lands as an insert or delete; a generic list component iterates | the screen iterates them itself | R11 |
| Stateless components | they render passed-in data, take their words from the screen, emit events they're given names for | they hold words, fetch, persist, or decide product rules | R3, R6 |
| Slots | the screen passes markup into a container or host (the UI-layout port) | no common misuse | R6, R8 |
| Component state | it is the component's own interaction state (open, draft, selected), or a feature value the component renders | it is the product's state together with I/O | R6 |
| Component data | data comes in as attributes, slot values, or a pull port (a read function the screen passes) | it calls a store by name, or requires a store module's API: the component owns the interface it needs | R6, R9 |
| Component changes | a change goes out as a port output or an event, and the screen wires it to the store | the component saves it itself | R6 |
| Sending to a component | the screen passes an output to another instance's input (a wire) | an instance sends to a sibling it names | R1, R2 |
| A component messaging its screen | it sends a port output under the name the screen gave it | it targets a specific peer, or names itself in its own code so the screen must match a literal | R9, R5 |
| Async tasks | the screen (or a generic host) runs a configured I/O instance and routes the outcome back to the feature | a feature component makes the gateway call itself | R6, R11 |
| Timers | a paradigm or the screen starts one on a feature's fact (`captured`) and routes the tick to an input | a feature component keeps a clock as part of the product logic | R9, R11 |
| Publish/subscribe | a framework hook subscribes; a handler routes the fact to an input | a module subscribes itself to a topic it names | R1, R5 |
| Navigation | the screen navigates on a feature's `step` through its path table, or to a URL an output carries | a component navigates, or a feature names a path | R3, R9 |
| Forms and validation | a feature or domain module validates; the screen supplies the messages; the form posts events to the screen or host | validation messages live in the feature; the screen validates | R3, R11 |
| Client-side hooks | named in the template for things only the client can do | a hook carries product rules | R3 |
| Screen-specific stateful components | never: a screen composes its UI from domain UI abstractions it configures (a generic record form), and its stateless components hold only markup | a stateful component with its own state and handlers used only inside one screen: a contained sub-component (§2.2) | R11 |

Four settled readings behind the table:
- **Clauses in one module meet R8.** Multi-clause handlers and `wire` clauses on one screen are one
  place; a route table or bindings map is a technique, not a requirement.
- **A UI component that does I/O is an R6 bundle.** A stateful component that loads, saves or charges
  through a store, gateway or placement instance is a UI abstraction with a data source or sink
  inside it. The fix keeps the component: data comes in through a pull port or attributes, changes go
  out as port outputs, and the screen wires the store.
- **A pull port is wiring; a required store module is not.** A read function the screen passes is
  Spray's pull dataflow (his Grid "is able to pull rows of data as needed", Summary). Passing a store
  *module* and calling an API on it makes the component define what it needs from a store: an
  interface it owns (R9).
- **A screen-specific stateful component is a contained sub-component.** Spray has "no analog of a
  sub-module or sub-component" (§2.2), and his provision for app-specific UI is layout and bindings
  built from domain UI abstractions (§7.14). See R11.

Two consequences:
- **A generic shell sits *below* the screen, not above it.** When it calls back into the screen's
  callbacks, that's the passed-in callback case (R1, R9), not an upward edge. A screen that uses the
  shell is dropping.
- **Generic view components aren't application code** even when only one screen uses them so far.
  If a component would read the same in another product, it's a domain UI abstraction. A stateless
  component that knows this screen's data or events is part of the screen, as markup. A stateful one
  of that kind, with its own state and handlers, is a contained sub-component (R11).

**Getting the application right in a UI app.** This is where most of an app's ALA quality is won or
lost, because the screen is the one place that is allowed to know everything. Each point states a
property first; what follows it is one way to get there.

- **Nothing below the screen names a screen.** Generic execution machinery is product-free and sits
  below the screen (§2.9.2, §4.2). For example, a shell that interprets outcomes or effects
  (notification, list insert, timer, navigation) is a generic execution model, so split it: the
  interpreter goes into the Programming Paradigms layer, shared by every screen, and the part that
  names this screen's composition stays in the screen.
- **Keep the screen to wiring and configuration.** The screen's callbacks connect events and data to
  features and domain abstractions and carry the application literals (R3). They don't compute,
  store, or decide (R11; §3.5). A handler that runs a feature and hands its result to an interpreter
  is wiring. One that works out a total is not.
- **The template is the UI diagram.** Nesting in the screen's template is the "display inside" wiring
  (§1.6.6, §4.12). A screen-level condition or loop is application logic. Sometimes it's unavoidable,
  and sometimes it's a missing component or a missing feature. Ask which (R11).
- **Push product-free markup down.** A component that would read the same in another product is a
  domain UI abstraction, even with one caller today (R7 asks whether it earns its existence).
  Keep only the markup that knows this screen in the screen.
- **Keep symbolic connections inside the screen.** A state name that the wiring writes and the
  template reads is Spray's `temperature` connection (§1.6.6). It's fine inside the screen, where both
  ends live. A name that features or domain modules must also know goes through R5.
- **Features are wired by the screen, not by each other.** State-and-rules modules and feature
  components are instances the screen gives inputs and whose outputs it routes (§7.15; R1, R5). A
  feature component that reaches for a sibling's state or a named topic breaks R1 or R10. A feature
  component that also renders its own markup and loads or saves through a store bundles a feature, a
  UI abstraction and a data sink: R6.
- **Subscriptions and timers are set up by the screen.** Start-up subscribes and routes messages to
  features, or passes a topic down as configuration (R1, §4.4.2).
- **Generated glue is application code.** If a manifest or diagram generates the screen's wiring, the
  generated code belongs to the application, like Spray's generated wiring code, which "lives in a
  subfolder from where the diagram is, because it is not source code" (§2.5.1). The generator is a
  tool, outside the app's layers.

**Conditions, matches, and assignments in a UI screen.** R11 gives Spray's reading, and the moves for
each kind of `if`. Here is how that applies to the forms a screen actually contains. The "wiring"
verdicts on multi-clause handlers and routing matches are this checklist's reading, not Spray's words.

| Form in the screen | What it is | Ways to fix, for example |
|---|---|---|
| Multi-clause event or message handler, one head per event | routing: each clause is a wire from an event to an input | keep each clause to forwarding |
| Decoding params (extracting fields, parsing integers) | adapting the browser's event format, a technical domain wired sideways (§7.17) | fine when small; if it repeats, a generic param-casting abstraction configured per event |
| Setting screen state to an abstraction's output | the point where dataflow lands (§1.6.6) | nothing |
| Setting screen state to a computed value, or any arithmetic in the screen | data handling | move it into the feature or domain abstraction, and set its output |
| A condition on "nothing", or on success, just to decide whether to go on | a propagation guard | a monadic chain, or a runner that stops on "nothing to emit" or an error (§6.1.3) |
| A match whose arms only send the ok and error results to different places | routing two output ports (read from §4.9.1) | acceptable as wiring; better still, the feature returns outcomes that an interpreter routes |
| A condition with computation in its arms, or a business rule (`if total > 100 then free_shipping`) | application logic | a domain abstraction, or a predicate configured at the screen (§3.11.3); a state machine if it depends on history (§4.16) |
| Several feature calls in one clause (remove the item, then record the undo) | fan-out wiring, written in order | fine when each call only passes outputs to inputs; say so when the order matters (§4.4.4) |
| A template condition on one boolean that a feature produces | a "display when" wire | fine |
| A template condition like `count(items) > 0 and user.admin` | application logic in the template | compute it in a feature and wire the boolean |
| A template loop over rows in the screen | iteration in the application | a generic list or table component that does the iterating (§4.12; §4.8.2 for tables) |
| A lifecycle check or an auth redirect | the framework's lifecycle | a departure: keep it to one line, or move it to the framework's hook for lifecycle extensions |

The test for the screen as a whole is Spray's: a reader should find every requirement as an
instance, a literal, a predicate passed in, or a wire, and no step where the screen computes or
decides for itself (§3.5.2). Framework-imposed branches are recorded as departures, not argued
away.

**For a linter:** let the application layer make internal peer calls without a finding, and keep
feature and domain layers strict. A human applies the same rule: a same-layer call between two
different abstractions is a violation *below* the application, and normal *inside* it.

- **Application literals live in the composition/config tier.** R3 exempts *config layers* from the
  literal check — by default the top layer, or any tier marked as configuration (so a manifest module
  standing in for the diagram can hold the application's configuration). A human mirrors this by not
  counting application literals that sit *at* the composition against R3.
  > **"Config module / config layer" names a *place*, not a value.** It is where application
  > literals are *allowed* to live — the composition, or a manifest standing in for it — as
  > opposed to the application/intrinsic distinction, which is about the *value*.
- **High fan-out at the composition is expected, not a smell.** The composition calls *many*
  features; that inflates neither the pass-through detector (which needs 1-in/1-out) nor a fair
  reading of R7. Depth (abstraction height) counts *chains*, so a wide, shallow composition stays
  low — as it should.
- **Calls *within* the application don't add height.** The application is one abstraction, so
  when computing abstraction height, a reader (and a linter) counts it as **1**, and height accrues
  only once a chain drops *below* it into features, domain, or paradigms. Proliferation *below* the
  application still counts fully.
- **The template hides contracts and calls.** Template markup is opaque to a language's syntax tree,
  so cross-feature calls and server↔client contracts embedded there escape function-level R1 and R5.
  A template-reading check covers part of this; a human running the checklist must **read the
  templates** — this is the single biggest thing a linter cannot see and the human necessarily can.

### When the wiring outgrows one composition: Features as compositions

Spray's answer to a large application is a layer, not a file split. "If a single abstraction is used
for the application, then as more and more user stories are added into it, it will eventually get too
large for the ALA size constraint… it is the application that will go over the 500 line complexity
limit. ALA will need to be applied to the large application abstraction by adding a new layer below
it" (§2.2). The application then "just composes a set of features or user stories needed in that
specific application, and sets up any communication that may be needed between them" (§2.2).

A Spray feature holds wiring, the same as the application does: "Each feature creates instances of
domain abstractions, configures the instances with feature specific details, and connects them
together as needed to express the feature or user story", and "Feature abstractions can have ports"
(§2.2). Its UI goes with it. The layout and "the bindings of the UI elements to data" are "kept
together, encapsulated inside a feature. Instead, the UI is composed from Domain UI abstractions"
(§7.14). Features are "the natural abstractions in the requirements" (§7.15).

What follows for partitioning a large app:
- **Split along features, not along files.** Splitting one composition's wiring across several files
  with no concept per file leaves one abstraction spread over several files. The size limit still
  applies to the whole, and a link from one file to another is a symbolic connection: "as a program
  grows, these symbolic wirings are always hard to follow. You would need to resort to text searches"
  (§1.6.3). A feature is a part a reader can name and read alone. *Checklist reading.*
- **Many crossing links mean a wrong cut or a missing abstraction.** In conventional code "the
  cohesion of the inherent graph for given user story is lost as hundreds of symbolic connection
  buried in your code" (§3.6). A cut along user stories keeps most wires inside a feature. If a cut
  can't, either the features are drawn wrong, or a thing many of them connect to belongs a layer
  down, which is the ground-symbol smell (§3.6.1).
- **In a routed UI, each route is already a partition.** The router composes screens, and screens
  reach each other only through URLs, a message bus and the store. *Checklist reading:* the router
  acts as the application, each screen is a feature-sized application of its own, and what has to
  scale is one screen's wiring. A message topic between screens is a symbolic connection (§4.7.4).
  Keep its subscribe and publish behind one module.

What measuring a real split showed:
- **Codebases' "features" are often not Spray's features.** Modules called features are often coded
  state abstractions with ports: `(state, outputs)` steps with logic inside. Spray's features contain
  only instances, configuration and wiring. In his terms those modules are stateful domain
  abstractions: an undo offer would serve any app in the domain, and a cart is a storefront's cart.
  The edges are the same either way, so R1 doesn't change, but the name misleads a reader who knows
  Spray's vocabulary: an R8 "should" (−0.5). *Checklist reading.*
- **Few links cross between features, and they meet at a hub.** In one storefront cart screen, 9 of
  29 wiring clauses connected one feature to another, and every one touched the cart. The rest wired
  a feature to the screen's own landing points: display values, lists, notifications, timers and
  store instances. Cut by user story (edit the cart, undo a removal, save for later, the wishlist,
  checkout), the screen wired a handful of links between stories.
- **A split surfaces hidden links.** That cut gave 12 links between stories, not 9: three had been
  hidden in shared screen state and in an event one feature's view sent straight to another. A story
  owns its view, so each became an explicit wire.

Techniques for a composition that outgrows the limit:
- **A story module.** A Features-layer module for one user story: it instantiates and configures that
  story's domain abstractions, holds its wires, renders its layout from domain UI abstractions, and
  declares ports for the few links that leave it. The composition composes stories and wires their
  ports. One story can serve several screens, each configuring it differently.
- **Nested wiring, not merged wiring.** Each story holds its own wiring and the composition holds only
  the links between stories, so nothing has to be merged or delegated: wiring clauses per story and
  per composition, or a bindings map per story and the composition's map of links between stories.
- **A runner that nests, or none at all.** A story is itself an instance with ports. A runner can run
  a story's part, follow its own wiring, and hand what it sends out to the composition. Or a story can
  be a module of plain functions on the composition's existing runner, given an output function by
  the composition (the composition's own `wire` with the story's key bound), so it sends outputs
  without knowing its key, the composition, or any other story.
- **Settle who owns each name after the split.** One module that both fired an event in its template
  and handled it agreed with itself. Split it, and the name becomes a contract between two modules
  (R5). The module whose view fires an event should handle it, with the composition dispatching to it
  from the event list it declares; and a timer or task a story starts should be named by the
  composition and passed to the story as configuration.
- **Split state along with the composition.** A state module whose public surface keeps growing is
  often several concepts. Split it into smaller abstractions wired to each other inside one story, so
  the extra wires live in the story, not on the composition.

Many paradigms and specialized ports are the norm in Spray's projects, not an extra: "we will for the
first time use multiple programming paradigms, a usual thing in real ALA projects" (§7.14). Each one
is also something a team has to learn, which is a real familiarity cost.

### How close a UI composition can get

**What a UI framework hands the screen.** A start-up callback (sometimes twice: once for a static
render, once when a live connection opens); a handler for every client event, named by a string with
string params; a callback whenever the URL changes, including browser back; a handler for every
message to the screen's process; a callback when an async job finishes; and a render function. The
screen's state is immutable. Some of this is Spray's model under another name: the screen's state is
where dataflow lands (his `temperature` connection, §1.6.6), the template's nesting is the "display
inside" wiring, and one process per screen is the single-threaded execution he prefers anyway
(§4.4.9). Other parts are foreign: string-named events, a URL that can change under the screen, and
repeated start-ups.

**Forced by the framework or the language, and valid.** Keep these small and never use them for
logic.
- *Event routing clauses.* The framework calls the event handler with a string name, so something has
  to match on the string. One clause per event, each only forwarding, is one wire from an event to an
  input: pattern matching doing the job of Spray's port names.
- *Decoding params.* Parsing a string id adapts the browser's format, a technical domain reached
  sideways (§7.17). If it repeats everywhere, a generic param caster configured per event takes it
  out.
- *Holding the program value.* Immutable instances don't change in place, so the screen stores each
  feature's new state back after every step. That's the language's departure, not the framework's,
  and it isn't "handling the data" while the screen never looks inside what it stores (R4).
- *A template instead of wiring code for the UI.* A template language is a better notation for the
  same "display inside" wiring Spray writes with `WireTo` nesting. Moving the UI tree into a wiring
  table would be reinventing a worse template.
- *State shaped for change tracking.* A screen often does better with a summary and a count as
  separate state entries than one record holding both. Those are named landing points, Spray's
  symbolic connections kept inside one abstraction; R5 accepts them while no feature needs the names.
- *URL steps belong to the screen.* The framework delivers the URL to the screen, so the screen owns
  the table from flow step to path. That agrees with R3: paths are application literals.
- *A few lines of client code.* Focusing a field on load can't be done from the server.

**Forced by the framework, but it moves out of the screen.**
- *A connection or lifecycle check at start-up.* A generic lifecycle hook in the Programming
  Paradigms layer, configured on the screen, holds the guard once, below the screen. That's Spray's
  first move: the guard goes into the connection mechanism.
- *Auth redirects.* Same shape, same fix: the framework's lifecycle hook.
- *Async results.* A payment job returns success, failure, or a crash. Route them to the feature's
  `succeeded` and `failed` inputs with one clause each, Spray's two output ports (§4.9.1), or let a
  component or sink own the job and its outcome so the screen doesn't see it.

**Not forced: habits a screen can drop.** Arithmetic and computed state (compute totals in a
feature). Business rules in handlers (the feature returns `blocked(empty)`, which the screen binds to
a notification). History in URL handling (a transitions table passed in as configuration). Handing
values from one feature to another (pass them in, or make it a wire). Loading rows at start-up (wire
a store source into the feature's `load` input). Store work in screen helpers (a domain abstraction
that does its own I/O). Loops and compound conditions in the template (a generic list component; a
feature computes the boolean).

**What remains in a screen that reaches zero.**
1. routing by event name and URL, which a functional language does with pattern matching instead of
   port names;
2. decoding browser params, a technical-domain adapter at the edge;
3. storing the program value back, because values are immutable;
4. a lookup table from URL to step, because the framework hands the screen the URL;
5. a client-side hook or two for things only the client can do.

None of these is logic, and none has a Spray move that removes it without replacing it with
something equivalent.

**What reaching zero costs.** Each design is something a team has to learn and keep, which is why
R11 belongs in a linter's strictest tier.
- *Bound ports:* a binder with several kinds of binding and one map per screen. An unbound port is
  silently dropped, so it wants a test that every port a feature declares is bound.
- *Circuit instances:* Spray's diagram run as data. A click may take message hops through the screen,
  and a full screen's circuit runs to dozens of instances and wires.
- *Feature components:* the shape many UI frameworks already give you, so little new to learn.
  Cross-feature effects take message hops through the screen, and the design lives in many handler
  clauses instead of one table.
- *Clauses as wires:* a small runner and one wiring clause per port, checked by one test per screen
  that reads the clause heads. Zero hops. At forty-odd ports the clauses still read as a list.
- *Stories:* the clause form split along user stories once a screen passes the size limit, with each
  story wiring its own parts.

**Limits of measuring it.**
- *Hand walks miss things.* Later walks of the same code find findings an earlier one missed (words
  built in a feature, validation messages in feature code, a currency code in a domain module, a
  label restating a configured fee), so a first full score tends to run generous.
- *A scale that caps each kind of finding* can't tell a screen that is all logic from one with a few
  branches.
- *Judgement calls move it.* Reversing a couple of calls can turn a 100 into a 99.
- *A linter can't see* message hops or R8, and sees templates only if it reads them, so it can score
  designs a hand walk separates widely within a few points of each other.
- *The rules reward absence, not assurance:* a checked, drawn diagram earns nothing more once the
  screen is clean.

**Does stopping short pay?** Mostly, yes. Spray's own history says so: "The turning point was when I
noticed two (accidental) successes in parts of two projects" (§1.5.1). Those parts had "undergone
considerable maintenance", and "their simplicity had never degraded", like "two pieces of metal that
had never rusted" at a rubbish dump, inside projects that weren't ALA. His state-machine diagram that
replaced 5000 lines of C "was easy to maintain for years to come" (§4.16), and his advice for legacy
code is "Conversion of user stories takes place iteratively" (§8.3). Most benefits are local to one
abstraction's boundary: an abstraction whose own dependencies all drop can be read and changed
alone, whatever its neighbours do. A hoisted literal, a feature that stops reading a sibling's
state, or a guard moved into a runner each pays on its own. Some benefits need the whole screen,
though:
- *The screen reads as the requirements* (R8) only when all of it is wiring. One handler that
  computes, and a reader has to check every handler.
- *The diagram is the source* only when all the wiring is in it.
- *"No abstraction knows a peer"* is a guarantee only with no exceptions. One peer call and you're
  back to reading callers.
- *An abstraction's own isolation* is all-or-nothing at its boundary.

A screen with a few forwarding clauses and a URL lookup loses nothing, because those forms aren't
logic. A screen that keeps some real logic still gets the local benefits of every abstraction below
it. What it gives up are the whole-screen properties above.

### Running the checklist by hand to compare with a linter

A linter's method is a mechanical shadow of the manual checklist, so a reviewer can reproduce and
extend it. The correspondence: (1) give every function its `[tag]` — a linter approximates this
with the layer map's convention plus per-function annotations (its **coverage** = how many you'd have
tagged); (2) walk every call edge and confirm it **drops** — a linter scores this on the function
graph and flags reference/template edges advisorily; (3) check application literals sit at the
composition (R3), state is owned, not hidden (R4), no silent contracts (R5), names/earns-existence
hold (R6/R7). Where a linter stops, the human continues: right-boundary judgement, template contents,
semantic (not textual) contracts, and whether an abstraction is the *right* one. A green linter run
is the floor; the manual pass is the ceiling, and is always the more complete of the two.

## What a linter can and cannot check (per rule) — why human evaluation is still required

Automated checking captures a *subset* of each rule. The recurring reason it can't capture the rest:
the rule turns on the **`[tag]` judgement** (does this code carry product knowledge, and at what
altitude?), which a linter can only approximate with syntactic proxies or a **hand-declared layer
map** — and even then, "is this the *right* abstraction" is not decidable. Read each note as "what a
human must still judge."

- **R1 (edges drop).** *Automatable, given a layer map:* a linter checks altitude on the **function
  call graph** (each module-qualified function a node) against each function's assigned layer —
  upward edges and cross-peer edges are scored. Because it sees only real calls in code, a
  cross-feature call **inside a template is invisible to it**; it can recover that with an
  *advisory* **reference-level** check over the module reference graph, reported for a human to
  confirm. *Human must judge:* whether the declared boundaries are the *right* ones; whether a
  same-layer call is genuine cohesion or disguised peer coupling (a tool can take a hint about which
  modules form one unit, but cannot tell a legitimate collaboration from a smell); and every
  reference-level advisory (is that referenced peer actually called in a template?). Without a layer
  map a tool degrades to module **cycle detection** — a small subset of R1.
- **R2 (no shared mutable state between peers).** *Automatable:* presence of mutable cells, global
  tables and process-global APIs. *Human must judge:* whether a given shared store is actually a
  *back-channel between peers* vs a legitimate single-owner cache — a linter sees the API, not the
  sharing topology. (In immutable languages this rule is largely satisfied for free, which
  *inflates* automated scores independent of design quality.)
- **R3 (application literals on the diagram).** *Automatable, given a layer map:* magic literals
  outside the composition layer; words below the composition in markup (text and label-like
  attributes of lower layers' templates); validation message strings; sentences built by
  interpolation; currency and unit codes. *Human must judge:* whether a literal is an *application
  literal* (should hoist) or an *intrinsic literal* (a validation pattern, a physical constant that
  belongs in its abstraction), whether a word is the product's (hoist it) or the abstraction's own
  (an error name a developer reads), and whether the "composition" is really where requirements
  should read.
- **R4 (state owned, not hidden).** *Automatable:* process-global variables; some stashing in mutable
  cells or global tables. *Human must judge:* whether a process's or actor's state is legitimate
  instance state or a hidden cross-call channel whose state belongs to some concept's abstraction (or
  to a wired `State<T>`) — a semantic distinction.
- **R5 (no silent contracts).** *Automatable:* duplicated identifier strings across modules; a
  money amount in a string whose cents are also a configured integer (a label restating
  configuration); an event a template fires that a handler in another module matches, and a timer
  or task name started in one module and matched in another — if the linter reads templates and
  function heads. *Human must judge:* contracts no syntax tree shows — cross-language constants
  (server string ↔ client code), or two ends that agree via *different* literals (a produced format
  parsed elsewhere). A tool sees textual duplication, not semantic agreement.
- **R6 (nameability).** *Automatable (weakly):* single-letter/`f\d` names, functions that wrap one
  primitive. *Human must judge:* the actual rule — "does this name a **learnable concept**?" A
  meaningful predicate (`empty?`) trips the primitive-wrapper heuristic (false positive); a
  meaningless-but-plausible name (`process2`, `handle_stuff`) passes it (false negative). Naming
  quality is irreducibly judgement. A wrapper heuristic should spare a function whose first parameter
  is its own configured value (a configured rule, not a renamed operator). *Also automatable
  (advisory):* the "should" about tramp parameters (a public function carries a parameter unread
  through another module's function that doesn't read it either), and a UI component in a feature or
  domain layer that calls a module reaching the database or a message bus. *Human must judge:*
  whether the carrying function is really a connection mechanism whose job is carrying, and whether a
  passed-in function is a pull port (wiring) or a store hidden behind a function.
- **R7 (earns its existence).** *Automatable (advisory):* dead code; trivial single-use one-liners;
  abstraction height past a ceiling. These are best reported but **not scored by default** — reuse
  is evidence, not a requirement (see "Helper proliferation" above), so a tool should not fail code
  on them unless the team opts in. *Human must judge:* the core — "does this hide a decision worth
  naming?" A single-use named step can be excellent decomposition or premature extraction; only a
  reader decides, or a second consumer does (does the abstraction wire into a second, unrelated
  caller unchanged?).
- **R8 (reads as the requirements; names/config/shape serve the reader).** *Not automatable at
  all* — pure judgement. "Does the composition read as the spec?" and "are these good names?" have
  no shape-based decision procedure. R8 is the honest boundary: a linter can *prompt* it, never
  score it. Useful prompts: a declared output no composer names (a prompt for a coverage test), a
  closure over a private composition helper in the wiring, and the size of a paradigm's wiring
  vocabulary.
- **R9 (ports by paradigm).** *Automatable:* owned interfaces across a boundary, and drift between a
  feature's declared ports and the outputs it builds. *Human must judge:* whether an output reads as
  a fact or names its destination.
- **R10 (no shared entity).** *Automatable:* a data type read by two or more features. Configured
  instances (a value its own functions take as configuration and never update) should be excluded.
  *Human must judge:* whether the rest are shared meaning or a legitimate domain abstraction, and
  whether a dataflow carries more than its consumer needs — the shape of what flows usually comes
  from a store at run time, so a static tool can't see it.
- **R11 (composition only).** *Automatable, given a layer map:* branches, arithmetic, loops, handled
  data and working chains in application functions; logic in application templates (comparisons,
  arithmetic, matches, loops, calls into lower modules); composition state passed into a feature or
  domain function; and a private composition helper doing store work. The same checks apply to a
  Features layer, and a stateful UI component in a composition layer is flagged as a sub-component.
  Because a scale caps each kind of finding, a linter should also report the share of application
  functions with any R11 finding, and count the rules met (no scored finding in any of a rule's
  checks), because a score that is a density lets one finding in a large codebase round away.
  *Human must judge:* whether a flagged form is one of the forced framework departures, and whether
  a layer's name matches what it holds (a coded state layer named for features is the R8 mislabel; a
  linter only sees that it then holds logic).

Two whole-design properties **no single-snapshot linter can check** (they need more than the code):
**reuse** (does an abstraction wire into two or more consumers unchanged? That needs a second
consumer to try it with) and **requirements coverage / correctness** (does the wiring
encode the *right* spec? — needs the spec). These live in R8 and in experiments, not in a tool.

**Consequence:** a green linter run means "no gross, mechanically-detectable coupling smells at
the module-graph + literal level" — a useful *floor*, especially for ruling out clearly-bad
codebases cheaply. It does **not** establish ALA conformance. The high-fidelity, discriminating
judgements (R1 altitude *correctness*, R3 application-literal-vs-intrinsic, R5 semantic contracts,
R6/R7 abstraction quality, all of R8, and reuse) require a human applying the checklist.

### The encoding and a linter agree (tool-stamped marks)

A linter can run two ways: over **source**, and over a **completed encoding**. They should reach the
same findings, so the manual notation stays a faithful hand-tool. Some checks live in syntax-tree and
call-graph analysis an encoding can't represent by itself — R10 (a shared domain entity), R11
(branching at the top), the public-surface and pass-through advisories. The fix keeps the
*judgement* marks human-supplied (tag, `$`, `q`, application-vs-intrinsic literal) but has the
encoder **stamp the facts it can already decide** into the draft: `&entity`/`&aggregate` (R10), `~>`
(pass-through), `(branches)` (R11 top-level logic), and `(private)` (so public-surface and
pass-through scope correctly). A human completes the `[?]` tags and resolves each `{app-literal?}`;
the stamped marks are kept as emitted.

What still can't cross over: R6/R7 (nameability, earns-existence — pure judgement, never encodable),
the tramp-parameter check (the encoding doesn't carry parameters), and two **approximations** flagged
as such — abstraction *height* (the encoding carries module-level edges, not the call-graph
in-degrees the source uses) and the R11 *app-share* aggregate (it assumes level 0 is the application
tier). Everything else re-lints identically.

### Worked check — the ALA thermometer, encoded and re-linted

The thermometer (`OffsetAndScale`, `LowPassFilter`, `SampleEvery`, `Display` as `[]` leaves;
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

Linting the source and linting this encoding both report exactly two R11 items and nothing else:
`push_reading/2` branches at the top (the two `if`s that null-guard the sampler and display stages),
and the application layer is 33% of functions (2 of 6). A linter that scores R11 only in its
strictest tier keeps the default score at 100. That's a statement about the linter's tiers, not a
clean bill: by Spray's reading these `if`s are application logic. He flags the same `if` in his own
§1.6.3 version, and it disappears at §1.6.4 when `SampleEvery` simply emits nothing. So the finding
is real. The fix is to move the guards into the connection mechanism ("Past the pipe"), not to clear
them as harmless. The `{app-literal}` on `new/1` is the thermometer's calibration; it sits at the
composition (level 0), so it is correctly placed and raises no R3 — the encoding check uses the same
"application literals allowed only in the top tier" rule the source R3 uses. No edge is upward or
peer, no `$`, no `q`, no shared entity: the good shape reads the same from either direction.

## Worked example 1 — Spray's bad thermometer (site §1.6.1)

> The two worked examples below use the compact whiteboard rendering (`fN`, and requirement
> constants shown inline as `+4`, `*8.3`) because it makes the R3 shape vivid on a page. The
> fuller rendering is stricter: real names (`Module.fun/arity`), application literals shown
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
        f7 [thermo] (p2)            -- DisplayTemperature                  ⚠ R1 (third link of a working chain)
```

Read the violations straight off the shape:

| shape | rule | what it is in the code |
|---|---|---|
| every edge is `[thermo] → [thermo]` between working functions (except f3→f8) | R1 | all pieces collaborate to "be a thermometer"; you must read all of it to understand any of it |
| `*p1` appears in siblings f3 *and* f4 | R2 | the adc-batch buffer's meaning is shared between two peers, not owned by the wiring |
| `+4`, `*8.3` inside f4; `15` inside f6; `9/10` inside f5 | R3 | requirement constants scattered two layers deep |
| `$` on tagged f5, f6 | R4 | hidden per-reading state inside app-specific code — invisible coupling through time |
| f4 → f6 → f7 chain | R1 | the story of one reading spans a three-deep chain of thermometer-knowing functions |

Note what the *un*-annotated sketch could not show: strip the tags and the bad tree and the good
tree below look similar. The tag column plus the literal rule is what makes the difference
mechanical rather than aesthetic — the encoding needs exactly that much semantics, no more.

## Worked example 2 — the ALA thermometer (site §1.6.3)

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
      if f6 []$ (15):                -- SampleEvery   ⚠ R11: a guard in the app
        f8 [] ( f7 [] (p3, "#.#") )  -- Display(FloatToString(...))
    )
```

Every edge drops `[thermo] → []`; the tree is depth 1 below the composition; every requirement
literal (channel 2, batch 100, offset 4, scale 8.3, strength 10, every-15, format "#.#") sits on
the composition line; each `pN` flows only through the top; `$` survives only inside generic
leaves whose *concept* is stateful. The application layer now reads back as the requirement —
which is the point.

Three honest residues, so the encoding doesn't oversell:

- `*p1` remains (the DMA buffer, a block of memory the ADC hardware fills directly) — platform
  mechanics, tolerated at the boundary; mark it and say why, don't hide it. This is C-specific: a
  functional port of the thermometer reads readings as values, so it has no `*p1`.
- `if f6` : the application still decides whether to go on. Spray says so himself: the
  application "is still doing some logic work - the 'for loop' and 'if statement', which we will
  address soon" (§1.6.3). It's an R11 finding, and the next section removes it.
- `f8(f7(...))` : Display consumes exactly what FloatToString produces — a latent `q1` (the
  string-format contract). Harmless here because the composition line owns both calls; it becomes
  a real R5 violation the day Display parses the string.

### Past the pipe: Spray's next steps (site §1.6.3–1.6.6)

Worked example 2 is Spray's §1.6.3 rung: the composition is correct, but the application still
*handles the data*. It receives each value and passes it to the next call, and it branches on the
sampler (`if f6`). Spray names the next goal straight away: "stop handling the data that is being
passed from one function to another." Elsewhere he is blunter: when a higher function passes data
from one call to the next without using it, "it clutters up the code something awful" (§2.3.6), and
imperative composition fails because "the user story ends up controlling the execution flow, and it
handles the data at runtime" (§4.17). Rewriting the loop as a pipeline that threads the leaves' state
(`p |> f4(4, 8.3) |> f5(s5, 10) |> ...`) doesn't get there. The app still handles every value, and
now handles the leaves' state too. The pipeline is a restyled §1.6.3, not ALA's endpoint.

His ladder from there, stated as goals (each keeps the ones before it):

1. **Separate composition from execution (§1.6.4).** The app *builds* a description of the program,
   then something else *runs* it: "the first statement just builds the program. Then the second
   statement sets it running." The app "doesn't have to deal with data", and the `if` on the sampler
   is gone, because a stage that has nothing to pass on simply passes nothing. Conditional propagation
   lives in the connection mechanism, not the app.
2. **Instances with ports, wired from above (§1.6.5).** Each domain abstraction is instantiated with
   its configuration and wired to the next through a port typed by a programming paradigm, "objects
   with ports that you wire together like electronic components". That doesn't require OOP. Spray
   gets there from procedural code (§3.9): a struct holds the configuration, two more fields hold the
   wiring, and state joins the struct when it belongs to that concept. He sums it up as "objects as a
   language feature, not a design philosophy" (§3.10). Whatever form it takes, writing a new domain
   abstraction must stay easy: "It is necessary for developers to be able to write new domain
   abstractions, so this needs to be easy" (§1.6.5).
3. **Several kinds of connection in one app (§1.6.6).** "Monads usually only support dataflow."
   A real app also composes UI, events, and state-machine transitions, and "different lines in our
   diagram have different meanings". For UI the lines "mean 'display inside'". When the composition
   becomes a graph, the diagram is the source.

What this means in a functional language. These are options, not a required form; how a linter should
check these goals is still open.

| Goal | Some ways to meet it in a functional language | Where state lives |
|---|---|---|
| build, then run; no data in the app | a paradigm interface (`push(step, data) -> emit(out, step) \| quiet(step)`) with a composition value and a runner; or lazy stream stages, each a configured stream-to-stream function, then a single run | inside each stage (a value, or a stream accumulator) |
| instances with paradigm ports | a pure graph value (named instances plus wires) and a runner; processes with a wired output port where the concept is concurrent | inside each instance |
| several kinds of connection | in a UI framework: template nesting is the "display inside" wiring; screen state is where dataflow lands; a generic runner or effect interpreter (a Programming Paradigms layer execution model) pushes results into the screen's state, so the screen never computes them | the program value, held as one opaque value |

Notes for functional languages:

- **Holding the program isn't handling the data.** Immutable data means the top-level owner (a
  screen's state, an actor's state) keeps the program value and stores the updated one after each
  run. That's Spray's `program` variable. The app never looks inside it.
- **Don't organise by process.** A process per domain abstraction is the most literal translation of
  Spray's objects, but organizing code by process is widely treated as an anti-pattern. Prefer pure
  composition values and runners, and use processes where the concept really is concurrent.
- **A lazy stream is a monad chain, with a monad's costs.** Each domain abstraction as a configured
  stream-to-stream function reads almost like Spray's §1.6.4, and it's standard library. But it is
  one paradigm and one line, it's pull-driven, and one run consumes it, so it suits a batch or a
  source you can iterate, not a long-lived screen that receives one reading at a time. Spray's ALA
  builds the program once and runs it forever (§3.11.3); in a functional language that's a
  composition value built at start-up, held by its owner, and fed each input ("Functional
  programming", point 8).
- **A graph value is a text form of the diagram.** Named instances plus a wire list allow fan-out
  without processes, and the wiring is data you can print, check for unwired ports, or draw
  ("Functional programming", point 9).

In the notation: the §1.6.4+ app shows no `pN` wires at all (the runner carries them), no branch, and
no leaf state. A composition line lists instances, their literals, and how they connect.

### Kinds of connection and how they run *(in development, not yet a rule)*

> **Status:** this section gathers Spray's thinking on composition techniques (his chapter 4, and
> §2.3–2.4, §3.6, §3.11) so that rules can be drawn from it later. Nothing here is scored, and the
> functional-language notes are tentative. It replaces an earlier claim that a compliant tree is one
> you can rewrite as a pipe, which was too narrow.

**A wire's meaning is a programming paradigm.** When the app connects two instances, the connection
has to *mean* something. Spray's list (§4.1): imperative, event-driven, dataflow, UI layout,
activity flow, state machine transition and substate, data schema, and request/response. Each
paradigm is an abstraction in the Programming Paradigms layer, usually a small interface that
becomes a port type. Its **execution model** is how that meaning actually runs on the CPU, which can
be one method or a whole engine. New paradigms get invented when they express the requirements
better (his game-scoring example uses a "ConsistsOf" paradigm). A user story normally mixes several
("polyglot programming paradigms", §2.4.1), all wired with the same operators.

**Pipes are fine when the requirement is a line.** "It's quite possible for a user story to just
consist of a linear sequence of instances of abstractions", such as a pipes-and-filters sequence
(§3.6.1). A pipe is one case, not the test. Monads (chainable wrappers around a computation, like a
lazy stream pipeline or a result-type chain) compose functions with one input and one output, "like
discrete electronic components such as resistors". Domain abstractions are more like integrated
circuits, with many pins of different kinds (§3.11.3). A monad chain can still live *inside* a domain
abstraction, as an adapter with ALA ports.

**Two kinds of communication (§4.4.1).**

- *Abstraction use* runs down the layers: configuring an instance, calling a library function.
  Calling back up is legal only indirectly, through a lambda or function that was passed in.
- *Wired communication* runs sideways between instances, along wiring set up by the layer above. It
  is always indirect: a sender goes only as far as its own output port, and "receivers never register
  themselves to a sender, or to a public event". The only global event Spray allows is one so abstract
  that nearly every abstraction uses it, such as `initialize` or `closing` (§4.7.4).

**How a wire runs is decided at wiring time, not inside the abstraction.**

- *Push or pull (§4.4.3).* Push is his default. It wires straight through, works for events, and can
  run sync or async unchanged. Pull fits:
  - lazy or expensive sources;
  - drivers that shouldn't decide when to read;
  - reads from a database;
  - abstractions with many inputs that react to only one.
- *Sync or async (§4.4.9–4.4.10).* He prefers one thread, with multithreading only for performance.
  A one-way port should work either way, so the choice is made when wiring. "If a certain domain
  abstraction needs to make an assumption" about when the effects of its call happen, "it is no
  longer an abstraction."
- *Run to completion.* In an event-driven design, "a task must always runs to completion quickly.
  No task should take real time to execute (spin loop, or block)" (§4.7). Long work is split into
  steps. In a functional language with processes, a message handler or screen callback should return
  promptly, and slow work goes to a task, a future, or a message to self.
- *Where instances run is a late decision.* Because asynchronous events don't care where the
  receiver is, "the physical view can be changed independently of the logical view" (§4.7.3). With
  location-transparent processes, a process can live on another machine without its senders
  changing.
- *Request/response (§4.6)* is two one-way messages treated as one. Because it's wired point to
  point, "a request is implicitly a command". That's the one place a command is fine, and only
  because the app set up the wire.
- *Incompatible ports* are bridged by an intermediary the app wires in, never by changing either
  abstraction. Examples: a buffer for push into pull, a poller for pull into push, a FIFO, an averager,
  or a load splitter (§4.4.3, §4.4.6).

**Topology has its own nuances.**

- *Fan-out.* UI layout supports it natively: a container's children, in order. Dataflow and events
  use a fan-out intermediary. Where the order of fan-out matters, only the wiring layer knows, and it
  should say so explicitly (an ordering abstraction, or activity flow) rather than rely on wiring
  order (§4.4.4).
- *Diamonds.* When two paths from one source meet again, the meeting point can see one new input and
  one stale one: a "glitch". Glitches are the wiring layer's concern, because only it can see the
  diamond. It handles them by ordering the paths, adding a trigger port, or choosing a clocked
  execution model (§4.4.7, §4.8.3, §4.8.5).
- *Circular wiring* is normal (feedback, undo). A loop that is entirely synchronous never returns,
  so it needs an asynchronous hop or a delay. Events in a loop shouldn't fan out (§4.4.8, §4.7.3).
- *A line joining many ports* is a missing abstraction, like the ground symbol on a schematic
  (§3.6.1).

**Dataflow alone has several flavours (§4.8).** It can be push or pull, *live*, clocked, a whole table
at a time, or an iterator. Live means an input simply has its source's value at all times, as in the
coffee maker and in FRP (functional reactive programming). Clocked means every instance latches its
inputs on a tick. The data type is never a DTO shared between two abstractions. It comes from a lower
layer or is passed in by the app, and ideally is inferred along the wire.

**Reactive and prescriptive both have a place (§4.7.2, §4.9, §4.16).** Event-driven code reacts, and
Spray defaults to it because it survives unforeseen events better. Activity flow prescribes an order
through start and done ports, and can span real time without blocking. State machines are a paradigm
of their own: states and transitions are instances, and the transition lines are wires.

**Text or diagram (§3.6).** Text handles sequences and shallow trees. A graph in text needs labels,
which are symbolic connections. Keep them in one place: follow a tree through the graph with
indentation, and hold only the cross-connected instances in local variables (§3.6.2). When the
requirements are an inherent graph, the diagram becomes the source.

**Not every paradigm needs ports (§2.4.1).** A concept that every instance would be wired to (Spray's
example is Style) can be an abstraction one layer down instead. He notes the cost of that: it behaves
like a global, which hurts parallel tests and per-instance overrides. (A functional equivalent: a
theme or settings module every component reads, or application config.)

#### Functional-language notes (tentative)

- **Port types.** A port's type is owned by neither side and sits below both (R9). In a functional
  language that is often an interface in the Programming Paradigms layer (`Step` for push dataflow, a
  lazy-stream stage for pull). It can also be a function passed in, a process address the
  composition supplies, or a message shape the paradigm defines.
- **UI frameworks already contain two of Spray's paradigms.** Template nesting is UI layout, with
  native fan-out where the order of the children is the layout order: Spray's `IUI` (his UI-layout
  port interface). Screen state behaves like *live* dataflow: the template always sees the current
  value.
- **Sync or async is an asynchronous message, a synchronous call, or a plain function call.** It
  should be chosen by whoever wires the instances, not buried inside a domain module that calls a
  named process.
- **Message topics.** A topic hardcoded in a domain or feature module is a global event name. If the
  composition owns the topic and hands it down, it's a wire.
- **Hot push and demand-driven pull.** Spray notes that "If you are using monads, especially I/O
  monads, or RX (reactive extensions), especially with hot observables, you are already using the
  wiring pattern" (§7.5). A publish/subscribe bus is hot push: messages flow whether or not anyone
  asked. A demand-driven stream library with back-pressure is pull: consumers ask for demand. That
  fits Spray's own reasons for pull (lazy or expensive sources, §4.4.3), so it's a choice made at
  wiring time, not a departure from his push default.
- **Writing the kind of wire (a suggestion).** The notation has one kind of wire, `pN`. Where it
  matters, a suffix can say which paradigm a wire carries, such as `p2:event`, `p3:inside` (UI
  containment), or `p4:transition`. Spray draws different line meanings in one diagram (§1.6.6) but
  doesn't prescribe a text mark, so this is the checklist's suggestion.
- **Intermediaries are instances too.** A fan-out, a buffer, or an ordering step belongs in the
  composition value (a `Chain`, a graph) like any other domain abstraction.
- **State machines.** A pure transitions table is one candidate for the state machine paradigm,
  without a framework's state-machine behaviour.
- **Several inputs of the same kind.** Spray spends a section (§4.4.5) working around C#'s rule that
  a class can implement an interface only once, so an AND gate can't have four `IDataFlow<bool>`
  inputs. Less relevant in a functional language: ports can simply be named, such as a map of port
  name to wire, or a message tagged with the port (`push(input2, value)`).

#### Candidate checks (not yet rules)

- A domain abstraction that decides sync or async, or push or pull, for its caller (for example, a
  synchronous call to a named process inside a domain module).
- A domain abstraction whose correctness depends on the order its outputs are handled.
- One value or topic joining many instances: the ground-symbol smell.
- A subscriber that names what it subscribes to.
- A composition that is an inherent graph but is spread across several modules' labels instead of
  kept in one place.

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
| a receiver subscribes itself (to a sender or a named topic) | move the subscription up to the composition; pass the topic or handler down as configuration |
| `p` shared by siblings / `*p` | the composition derives and hands each sibling exactly the value it needs |
| literal in a leaf | hoist it to an argument at the composition line (it is requirements content) |
| `$` on a tagged f | extract the stateful *concept* into a generic leaf that owns it, or into a `State<T>`-style abstraction wired in from the top |
| undeclared `q` | make it a parameter from the top, or a named contract both ends depend on downward |
| un-nameable f | it was wiring all along: inline it as `(p) ->` |

The end state is recognizable at a glance: **the tagged layers (the application, and features
under it in a bigger app) carry all the literals and only wire; everything they use is a `[]`
abstraction in a lower layer; and no edge goes sideways or up.** Then take the next step:
make that top *describe* the connections and let a runner move the data (see "Past the pipe").

**Starting from an existing app.** Adopt the checklist, run a linter that implements it against what
you have, and let the findings send you to the technique lists under each rule. Most
well-structured functional apps are already close on R1, R2, and R4, because immutability and
ordinary module boundaries give you a lot. The work that remains is usually R3 (get the constants to
the top), R5 (name the contracts, and keep the names in the composition), and R11 (sort the
composition's `if`s and move the ones that aren't routing). Reach for the heavier techniques (typed
outcomes, a manifest, committed codegen, a composition value with a runner) only when a rule you care
about isn't holding by discipline alone. The lightest technique that meets the rule is the right one.

## Where even the good shape can lie (keep these in the guidance)

1. **Tags rot.** A `[]` leaf acquires a product constant in maintenance and nothing in the call
   tree changes. R3 is the tripwire — re-scan leaves for literals, not just edges. (Mechanized
   at scale, this is a check that lower layers hold no product literals.)
2. **Contract coupling is invisible to call trees.** Two `[]` leaves agreeing on a format/name/
   topic (`q`) are peers in disguise; only R5 catches it. (Mechanized: a check that names are
   single-sourced and not agreed between peers.)
3. **Callbacks and pub/sub hide who set up the wire,** and indentation shows neither case. A leaf
   invoking a handler is fine when the composition passed that handler in: it's a port. It's a
   defect when the leaf found its partner itself, by registering with a sender, subscribing to a
   topic it names, or looking up a process by name. Write `^f` for a lookup of something higher, and
   treat a self-registration between peers as a peer edge. The fix is always "the composition wires
   it," never "the leaf knows whom to call."
4. **A perfect shape can encode the wrong program.** The encoding audits *structure*
   (coupling/knowledge placement), not correctness — same caveat the requirements-coverage
   analysis makes for manifests.

## Glossary

The checklist's vocabulary, in alphabetical order. Section numbers point to Spray's site. Where a term is this project's and not Spray's, the entry says so.

**Abstraction.** The only unit of code in ALA: "a 'generalized conceptual idea'", "learnable as a concept" (§2.1.1). In a functional language it is usually a module, sometimes a single function. What makes it an abstraction is that a reader can use it without reading its body.

**Abstraction height.** The longest chain of knowledge dependencies between abstractions. Calls inside one module don't add height. A tall stack of thin layers is a sign of helper proliferation. A linter metric, not Spray's term.

**Application layer.** The top layer. It instantiates abstractions, configures them, and wires them together, and holds all of the app's specific knowledge and none of its logic (§3.5). In a UI app, the screen, its template, and the router.

**Application literal.** A constant that belongs to this product (a price, a threshold, a label, message text). It lives at the composition (R3). Contrast *intrinsic literal*.

**Communication dependency.** One module calling another to move data or events between them as peers. ALA eliminates these; the layer above wires the peers instead.

**Composition.** Building something from instances of abstractions by connecting them. Spray's opposite is decomposition, splitting a system into specific parts that collaborate (§3.7). "The composition" also names the code that does the composing: the application, or a feature's wiring.

**Configuration.** The settings an instance gets once, when it is created: its main interface, used only by the layer above (R9). R9's "should" keeps configuration apart from run-time data.

**Connection mechanism.** Whatever carries data along a wire at run time: a runner, a result-type chain, a monad's Bind. Guards like "only if there's a value" belong here, not in the application (§1.6.4, §6.1.3).

**Departure.** A place where this checklist knowingly differs from Spray, with a reason, such as the routing clauses a UI screen must keep. Recorded, kept small, never used for logic.

**Diagram.** Spray's source of truth: boxes are instances, lines are wires, and the whole reads as the requirements (§2.5, §3.6). In code it may be a manifest, a graph value, or a readable composition.

**Domain abstraction.** A reusable abstraction in the layer below the application (or below the features). It knows nothing about this product: `LowPassFilter`, `OffsetAndScale`, a generic table component.

**DTO (data-transfer object).** A type made only to carry data between two modules. Two peers may not share one (§4.8.1).

**Execution model.** The code that makes a kind of connection actually run, such as a runner, an interpreter, or a UI framework's event loop. It lives in the Programming Paradigms layer.

**Feature.** A product-knowing abstraction in its own layer under the application, wired by it, used once an application is too big to be one abstraction (§2.2, §7.15). Like the application, it holds instances, configuration and wiring, including its UI layout from domain UI abstractions (§2.2, §7.14). Modules of state and rules that a codebase calls features are stateful domain abstractions under that name.

**Ground symbol.** Spray's name for a wire that joins many ports, like ground on a schematic. It suggests a missing abstraction one layer down (§3.6.1).

**Handling the data.** The application catching one abstraction's result only to pass it to another. Spray names it at his §1.6.3 step and removes it at §1.6.4. An R11 finding.

**Identity key.** An id two features share while each keeps its own data. One way to meet R10. The technique is Spray's: "use cases should all know about the abstraction, customer identity. A particular use case should only know about it's own data, and only store it against a customer identity" (§6.17.2); the name is this checklist's.

**Instance.** The run-time use of an abstraction: a configured value, a process, or just a reference to a pure function (§3.2.2).

**Intrinsic literal.** A constant that is part of an abstraction's own definition, such as an identity or a physical constant. It stays in the abstraction. Contrast *application literal*.

**Knowledge dependency.** One abstraction using another, more abstract one by name, the way code uses a square root. The only kind of dependency ALA allows, and it must point to something significantly more abstract (§2.1.3, §3.4).

**Layer.** One of a few levels ordered from concrete to abstract. Spray's usual stack is Application, Features, Domain Abstractions, Programming Paradigms, and Foundation. Knowledge only flows down.

**Little ball of mud.** An abstraction's inside, which may be procedural and messy as long as the abstraction is small, names one concept, and is clean at its boundary. ALA governs the relationships between abstractions, not their insides.

**Monad.** A functional pattern that composes functions through a Bind function, which hides execution details. Spray treats monads as the nearest functional analogue of ALA, and as more limited: two ports and one paradigm (§3.11, §6.1–§6.2).

**Must / should.** A "must" is a defect when it fails. A "should" is a prompt a reviewer weighs, used where Spray's evidence is weaker.

**Owned interface.** An interface specific to one module, either defined by a consumer for others to implement (required) or by a module for its peers to call (provided). Both fail R9.

**Pass-through.** A public function with one caller whose body is a single call into another module. It renames a call without hiding a decision. A linter check, advisory.

**Peer.** Another abstraction in the same layer. Peers never know each other. Only a layer above connects them.

**Port.** An input or output of an instance whose type is a programming paradigm from a lower layer, set up by the layer above. In a functional language it's often an interface, a function passed in, or a process address.

**Program value.** An immutable composition (a chain or a circuit of configured values) that its owner holds and feeds each input. It's how a functional language keeps a program built once and run forever.

**Programming paradigm.** What a kind of connection means: dataflow, events, UI layout, state-machine transitions, request/response (§4.1). Each one is an abstraction in the Programming Paradigms layer, and ports are typed by them.

**Projection.** A read-only view a feature offers of its data, shaped for its readers, so they never read its own data type. A technique for R10.

**Push / pull.** Whether the sender calls the receiver (push) or the receiver asks the sender (pull). Spray defaults to push because it works synchronously or asynchronously (§3.11.4).

**Runner.** A generic execution model that moves data between the instances of a composition value, so the application never touches it.

**Silent contract.** An agreement between two modules that appears in neither's signature, such as a matching string, a tuple shape, or a session key (R5).

**State abstraction.** Spray's `State<T>`: an abstraction with input and output ports that holds state belonging to no other concept, wired in like anything else (§3.9).

**Symbolic connection.** A name used only to connect two points in the code, like a local variable that carries a value between two calls. Fine inside one abstraction; across modules it becomes a registry of global names (§4.7.4, §7.23).

**Tag.** In the checklist's notation, `[thermo]` marks a function that knows a product requirement, and `[]` marks a generic one. The one judgement the notation asks of a human.

**Tramp parameter.** A parameter a function never reads and only carries down to something further below. Spray: middle layers end up with "extra parameters that don't have anything to do with them" (§3.11.1). R6's "should". The term itself is general programming usage, not Spray's.

**Wire.** A run-time connection between two instances' ports, set up by the layer above. Circular wiring is fine; circular knowledge dependencies are not.

**Working chain.** Product-knowing functions that do work and call each other, the shape of the bad thermometer. Each part should be either a real abstraction in a lower layer or wiring in the composition.

**Zero coupling.** No design-time knowledge between peers, not merely less of it. ALA removes bad dependencies rather than loosening them (§7.4).

---

*Origin note: this file started as a numbering sketch of the two trees; the worked forms above
fix the f-numbering against Spray's actual §1.6 code and add the `[tag]`/`$`/`*`/`q` marks —
without the tag column, the bad and good trees are nearly the same shape, which is what the first
sketch ran into. The "Past the pipe" section follows Spray's §1.6.3–1.6.6 ladder, and its
functional-language options come from working the same four domain abstractions under a
`Thermometer` composition each way.*

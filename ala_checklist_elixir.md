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
functional-programming reading he develops later (gathered, with its Elixir meaning, under
"Functional programming" below)
([monads detour](https://www.abstractionlayeredarchitecture.com/#truebrief-detour-composing-with-monads),
[ALA vs FP](https://www.abstractionlayeredarchitecture.com/#trueala-compared-with-functional-programming)).

> **Citations.** Section numbers such as §3.8 or §1.6.4 refer to John Spray's site,
> [abstractionlayeredarchitecture.com](https://www.abstractionlayeredarchitecture.com/), which is
> one long page with numbered sections. "Summary" is its unnumbered opening section. Each rule ends
> with a *Spray:* line naming the sections that support it, so you can check the rule against the
> source. Where a rule adapts Spray to Elixir or departs from him, the line says so.

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
             Elixir has no shared references; its forms are an ETS table, an Agent,
             a process both talk to, or a session/socket slot two features read.
$            the function keeps hidden state between calls. In C that's a static or
             module variable; Elixir has neither, so it's the process dictionary, an
             Agent or ETS entry, or a process's state used as a hidden channel.
             Write it on the tag: [x]$.
qN           a silent contract: a format/name/shape two functions must agree on
             that is NOT visible in any signature.
{app-literal?}    an application-literal candidate lives in this function (value
             opaque; the linter has its file:line). A human/LLM resolves it to:
{app-literal}       an application literal — belongs at the composition (R3 hoist if
               it sits lower; at the top it is correct).
{intrinsic-literal}    intrinsic to the abstraction (an identity, a physical or
               mathematical constant) — correctly local, never hoisted.
(p) ->       an anonymous wiring lambda (`fn p -> ... end` or `&...` in Elixir). No
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
    registered process name, a topic it knows. By contrast, a function, module, or handler that the
    composition passes *down* is not an upward edge: it's a port, and it's Spray's legitimate way to
    call up the layers ("executing a lambda expression that has previously been passed in", a
    callback, the observer or strategy pattern, §4.4.1). In Elixir terms: a function or module
    passed in as configuration (strategy), or a subscription the composition sets up (observer).
  - **Who sets up the callback decides.** Set up by the layer above: legal. Set up by the receiver
    itself is a peer edge in disguise, and forbidden. That covers a receiver registering with a
    sender, or subscribing to a public event: "receivers never register themselves to a sender, or
    to a public event" (§4.4.2). Between peers the observer pattern only reverses a dependency that
    ALA doesn't have, so ALA wires instead. The one place Spray keeps an observer is *inside* a
    paradigm interface, for traffic running against the wire's direction, where "the subscriber
    does not know the publisher". In Elixir terms:
    - A feature or domain module calling `Phoenix.PubSub.subscribe/2` (or a `Broadcast.subscribe`
      wrapper) on a topic it hardcodes is the defect. The composition (a LiveView's `mount/3`, a
      supervisor's wiring, a wiring function) subscribes and routes, or passes the topic down as
      configuration.
    - A `handle_info/2` in the composition that receives the message and hands it on is fine.
    - `send(SomeName, msg)` or a `Registry` lookup by a name the sender knows is the sender naming
      its destination (R9). The composition should give it a pid or a port instead.
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
  - **Elixir techniques that meet it.** None is required; each is one way to satisfy the rule. The
    variant names (V10, V36 and so on) point to the Elixir variants where a technique was built. "Not
    tried in a variant yet" means it has only been sketched, or run in the thermometer steps.
    - *Emit an outcome instead of calling a peer.* A feature returns a description of what happened,
      or of what should happen, rather than calling the sibling that should act on it (V10's and
      V36's typed outcomes, V14's signals, V35's typed facts). The sideways call never exists, which
      removes the temptation rather than policing it. What the outcome may say, and what it may not
      name, is R9's business.
      ```elixir
      # Wishlist doesn't call Cart; it reports what happened.
      def remove(%Wishlist{} = wl, id), do: {drop(wl, id), [{:wishlist_removed, id}]}
      ```
    - *Wire cross-feature work in one place.* V26, V28, and V36 put every join between features in
      the composition, a `handle/3` clause whose job is wiring. Features stay ignorant of each other,
      and the one module allowed to know both does the connecting. Spray's version: "features can
      also have ports and be wired together" by the application (§2.2). The clause below still binds
      `removed` and hands it on, which R11 counts as handling data; the declarative and circuit forms
      remove that too.
      ```elixir
      def handle(:remove_item, %{id: id}, page) do
        {cart, removed} = Cart.remove(page.cart, id)
        %{page | cart: cart, recent: RecentlyViewed.record(page.recent, removed.product)}
      end
      ```
    - *Make the wiring declarative.* V29's manifest and V30's committed codegen route one feature's
      typed fact to another's reaction through generated glue. The routing lives in a data table.
      ```elixir
      reactions: [{Cart.Facts.ItemRemoved, RecentlyViewed.Intents, :track_removed}]
      ```
    - *Pass a function in where a peer used to be called.* When one function called a peer part-way
      through, "the second function will now need to be passed into it. The function parameter is
      also a port" (§2.3.6). It's also Spray's legitimate way to call up the layers (§4.4.1).
      ```elixir
      Checkout.new(charge: &Payments.charge/1)   # the composition chooses; Checkout never names Payments
      ```
    - *Let the composition subscribe.* The page's `mount/3` subscribes and routes messages on (V30,
      V33–V35 and V38, through a small `Broadcast` wrapper), or a generic `on_mount` hook in the
      Programming Paradigms layer does it, configured on the page (V39, V41).
      ```elixir
      def mount(_params, _session, socket) do
        if connected?(socket), do: Phoenix.PubSub.subscribe(MyApp.PubSub, "stock")   # the page subscribes
        {:ok, socket}
      end
      ```
    - *Enforce it in CI.* V27 shipped a custom Credo check that fails the build on an upward or
      cross-peer call. `ala_lint`'s R1 check does the same given a layer map, including domain →
      domain calls (`mix ala.lint --layers-module MyApp.AlaLayers --min-score 90`). Neither prevents
      the coupling, but both catch it the moment it appears.
  - **What doesn't meet it.**
    - A feature calling a sibling feature's function, including through a shared "service" module in
      the same layer.
    - A lower module that finds its collaborator itself: `Application.get_env(:app, :payments)` inside
      a domain module, a `Registry` lookup, `send(SomeName, msg)` to a name it knows, or subscribing to
      a topic it hardcodes.
    - A behaviour one feature defines for another to implement (see R9).
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
  - **Elixir techniques that meet it.**
    - *Give each feature a private struct.* V24's isolated capsules and V26's and V36's per-feature
      structs leave no shared mutable thing for two peers to fight over. Immutability does the rest.
      ```elixir
      defmodule Wishlist do
        defstruct products: []   # only Wishlist reads or writes this
      end
      ```
    - *Keep one source of truth and derive the rest.* V13's reactive split (source fields, with
      derived fields recomputed from them) kills the classic bug where a handler updates `items` but
      forgets `total`.
      ```elixir
      def total(%Cart{items: items}), do: Enum.reduce(items, 0, &(&1.amount + &2))
      ```
    - *Let the composition supply cross-slot values.* V35's composed inputs: a feature that needs a
      sibling's data doesn't read the shared session slot. The composition reads it and passes the
      value in, and a detector flags any `session.<peer_slot>` access. By §1.6.3's standard the page
      is still handling that value (R11). V40 goes one step further and makes it a wire: the cart
      announces `checkout_requested` with its data, and `Checkout` takes its stock source as
      configuration.
      ```elixir
      def pay(session, stock_levels), do: ...   # the composition resolved stock_levels
      ```
    - *Turn a value everything touches into an abstraction one layer down.* A wire joining many ports
      is Spray's "ground symbol" smell (§3.6.1). Make it a domain abstraction the features depend on as
      knowledge instead of wiring every feature to it. *Not tried in a variant yet.*
  - **What doesn't meet it.**
    - An ETS table, `Agent`, or `:persistent_term` entry that two features both read and write.
    - A feature reading another feature's slot of a shared session, socket, or page struct
      (`session.cart` from inside the wishlist).
    - A process that two features both call to swap state.
  - *Elixir note:* immutability gives this rule almost for free, so a clean R2 says less about an
    Elixir design than it would in C# or Rust. Pure code can still couple peers through R1, R5, and
    R10 (see "Functional programming", point 12).
  - *Spray:* Summary ("Communication between instances of peer abstractions"); §3.8 (no data
    coupling); §7.8.
- **R3 — application literals live at the composition line.** Application literals (product-specific
  constants: thresholds, prices, labels, formats) live in the composition, or in a manifest or
  config module the composition owns, never inside a `[]` leaf. Passing them as arguments on the
  composition line (`f4(p1, 4, 8.3)`) is the simplest form; a manifest, or an attribute on the page,
  are others. A literal buried in a leaf silently converts `[]` to
  `[thermo]` — the tag was lying. The contrast is an *intrinsic literal* (an identity or a physical/
  mathematical constant that is part of the abstraction's own definition), which correctly stays put.
  - **Elixir techniques that meet it.**
    - *Put the application's constants in one place at the top.* V34's manifest holds calibration per
      feature (a `config:` channel the generator splices into `init/1`), V36 keeps it in a `@pricing`
      attribute on the composition, and V38 uses a `GoodDeal.Catalog` module the composition reads.
      ```elixir
      defmodule GoodDeal.Catalog do
        def shipping_rates, do: %{standard: 500, express: 1500}
        def gift_wrap_cents, do: 299
      end
      ```
    - *Make the domain take calibration as configuration.* The abstractions below take the constants
      from the composition, so they hold no product-specific number and drop into a different product
      unchanged. Pass it once, as R9's configuration "should" describes (for example a configuration map
      as the first argument), not scattered through each call's data.
      ```elixir
      def cost(%{rates: rates}, method, subtotal), do: ...   # config first, set by the composition
      ```
    - *Pass rules in as predicates.* A requirement like "free shipping over $100" can be a function the
      page passes to a generic abstraction, the way Spray writes `new Filter(x => x>=0)` (§3.11.3). The
      rule is stated once, at the top, and a generic abstraction runs it. *Not tried in a variant yet.*
      ```elixir
      Shipping.new(free_when: &(&1.subtotal >= 10_000))
      ```
    - *Compute a derived value once, instead of threading the literal.* V38's low-stock threshold was
      used seven times in a template as a display helper. Rather than pass the bare number through
      all seven calls, the status is computed once from the configured threshold and flows down as
      data, which also made the badge components pure.
    - *Pass message text in too.* Flash and label text is an application literal. The page supplies it
      as configuration, or maps a feature's fact to text (V39's page binds `{:cart, :removed}` to a
      flash it words itself).
  - **What doesn't meet it.**
    - A number or message string in a feature or domain module (`@free_over 10_000` inside
      `Shipping`, `"Saved to wishlist"` inside `Wishlist`). V36's features still hold flash text,
      which is why `ala_lint` re-scored V36 lower under the revised R3.
    - A struct default that is really a product decision (`defstruct threshold: 100`).
    - `Application.get_env/2` read inside a domain module for a product value: the module picks its own
      configuration (see R9, "No endpoints"). Reading config in the composition and passing it down is
      fine.
  - *Cost to expect:* with no defaults baked into the domain, its own tests must supply configuration
    (V34 found this).
  - *Spray:* §1.6.3 (literals at the composition); §2.4; §3.5 and §3.5.2 (the application specifies
    the rounding, filter bandwidth, and resampling rate when it instantiates the abstractions).
- **R4 — `$` is legitimate only inside a `[]` leaf** (state that *is* the abstraction's concept:
  a filter's memory, a sampler's counter). `$` on a tagged function is invisible coupling through
  time between application steps. State belongs with the abstraction whose concept it is: Spray
  "prioritizes abstraction over referential transparency", because passing a filter's running
  value in on every call "breaks an otherwise good abstraction". Whether that state is held (a
  process, or a struct returned inside the updated program) or passed in and out is a case-by-case
  execution choice, not a compliance level. What R4 forbids is state whose meaning leaks out of its
  owner: the app or a peer keeping, reading, or computing the raw state itself (`f5(s5, p2)`, where
  the app holds `s5`). In Elixir a changing instance comes back as a new value, and storing that
  opaque value again is not a breach, because the concept still owns what is inside it. (Re-storing
  every instance by hand is the §1.6.3 "handling the data" cost that "Past the pipe" moves beyond.)
  State that no abstraction owns becomes its own abstraction (Spray's `State<T>`, a generic
  state-holder with input and output ports), wired in like any other. In Elixir that could be a
  small struct in the program value, or a process, that the composition wires to its readers and
  writers.
  - *Two ways to hold state* (§3.11.2). Spray defines a computation as `input + state --> state +
    output`, which can be grouped as `(input + state) --> (state + output)` (the functional form: the
    caller passes the state in and gets it back) or `input ( + state --> state +) output` (the object
    form: the abstraction keeps its state). "In ALA we choose between these two philosophies on a
    case by case basis." Either is fine *inside* an abstraction. What R4 forbids is the caller
    managing state that belongs to another concept. In Elixir the functional form is the default; a
    process is the object form.
  - **Elixir techniques that meet it.**
    - *Write a functional core over owned state.* V10's typed TEA, V36, and V38 keep feature logic as
      pure functions over the feature's own struct. The module that owns the state is the only one
      that reads or changes it; callers store the new struct back without looking inside. Spray's
      reason: state that is part of a concept belongs with that concept (§3.9, §3.11.2).
      ```elixir
      def add(%Cart{} = cart, item), do: %{cart | items: [item | cart.items]}
      ```
    - *Partition state by owner.* V9's split of the page struct into `CartState`, `UIState`, and
      `CheckoutState` makes each piece of state visibly belong to one concept.
      ```elixir
      defstruct cart: %CartState{}, ui: %UIState{}, checkout: %CheckoutState{}
      ```
    - *Let the composition value carry step state.* When the page builds a program from stages (see
      "Past the pipe"), each stage's memory lives inside its own struct, and the runner stores the
      updated program. The page holds one opaque value and never names a stage's state. *Run in the
      thermometer steps, not tried in a variant.*
    - *Give unowned state its own abstraction.* State that belongs to no concept becomes Spray's
      `State<T>`: an abstraction with input and output ports, wired in like any other (§3.9). In
      Elixir, a small struct in the program value, or a process, that the composition wires to its
      readers and writers. *Not tried in a variant yet.*
    - *Use a process where the concept is concurrent.* A sensor poller, a timer, or a bridge from
      PubSub keeps its state in a process with a wired output port. That's Spray's object form, and
      it's fine when the concept really runs on its own.
  - **What doesn't meet it.**
    - The process dictionary, or an `Agent` or ETS entry used as a hidden channel between calls.
    - The page or a peer reading or computing another concept's raw fields
      (`Enum.sum(page.cart.items)` in the page).
    - A GenServer holding several concepts' state that features reach by registered name.
    - One process per abstraction for its own sake. Elixir's guidance lists "code organization by
      process" as an anti-pattern: processes are for concurrency, isolation, and lifecycles.
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
  - *In Elixir:* a `Contracts` module of event, stream, and hook names is fine when only the
    application uses it: the page's HEEx, its handlers, and the JS hooks it owns. That is Spray's
    `temperature` symbolic connection, kept inside one abstraction. It turns into the registry
    smell when feature or domain modules use it to agree with each other. (The Elixir variants'
    `Web.Contracts` + `ContractPurity` mechanize single-sourcing names. Whether a given use is
    app-owned or a registry is the question above.)
  - **Elixir techniques that meet it.**
    - *Keep the meaning in the composition.* Both ends take the name or format as configuration from
      the page, so only the page knows they agree.
      ```elixir
      Display.new(format: "#.#")   # the page chose the format; the formatter and display never agree on it
      ```
    - *Type your effects.* V10's and V36's outcome constructors turn "return `{:flash, :info, msg}`
      and hope the other side matches" into `Outcome.flash(:info, msg)`, where a typo is a compile
      error.
    - *Declare names once, and keep them in the page.* V35's `Contracts` module holds every event,
      stream, and hook name, so a rename happens in one file. V38 does the same with tiny single-source
      modules (`CartSession` for a session key). Once features use such a module to agree with each
      other, it becomes a registry of global names (fix 3 above). V35's features do call
      `Contracts.stream_name/1`, which is worth a second look.
      ```elixir
      defmodule Web.Contracts do
        def stream_name(:cart), do: "cart-items"
      end
      ```
    - *Make both ends one abstraction.* One module owns both the encoding and the decoding. *Not tried
      in a variant yet.*
      ```elixir
      defmodule StockMessage do
        def encode(id, count), do: %{"id" => id, "n" => count}
        def decode(%{"id" => id, "n" => n}), do: {id, n}
      end
      ```
    - *Check the places signatures can't reach.* V32's `ComponentPurity` checker looks inside HEEx,
      where a `phx-click="save"` string is a contract the compiler never sees. A companion
      `ContractPurity` check (V33) covers the contract names themselves.
  - **What doesn't count as a silent contract.** A `~p"/cart"` route, which the router checks at
    compile time. Two equal strings that mean different things (V38 found `"products"` used as a
    PubSub topic in one place and an Ecto table name in another: a coincidence, not a contract).
    Telling these from real contracts is the reader's job; the linter only sees duplicated text.
  - **What doesn't meet it.**
    - A tuple shape one module produces and another pattern-matches on with no shared definition.
    - A session key written as an atom by a plug and read as a string by the LiveViews (V38's one
      real finding, fixed with a small module both ends depend on).
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
    configuration, and an example without reading the body. In Elixir the usual way is a short
    `@moduledoc` (one or two lines, plus an example when it isn't obvious), which also fits a
    sparse-comments style.
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
        using that abstraction (`Thermometer.push_reading/2` gives the reading to `OffsetAndScale`).
      - A runner or connection mechanism carrying a payload to a port. Carrying is its concept.
      - A private helper inside one abstraction (the little ball of mud), and a framework callback
        whose head is fixed (`handle_event/3`, `init/1`).
      - The application layer, where the same thing is R11's "handling the data".
    - *Why "should", not "must":* Spray gives it as a reason functions become poor abstractions, not
      as one of his constraints, and a two-hop carry is sometimes the simplest honest shape.
    - *Ways to fix it:*
      - Let the composition give the lower abstraction its input or configuration directly, so the
        middle function never sees it.
      - Configure the lower instance at the composition and pass the *configured instance* (a
        struct, or a function) down. The middle function then uses it, calling it with its own data,
        which is a port (R9), rather than carrying its raw inputs.
      - Make it a wire: V40's `Checkout` takes its stock source as configuration instead of receiving
        stock levels through the page and passing them on.
      - If the middle function exists only to carry, it may not be an abstraction at all (R7).
    - *Mechanizable:* yes, as an advisory. `ala_lint`'s `tramp` check flags a public function whose
      plain-variable parameter is never read and only passed, as a bare argument, to another project
      module's function (a lower one, given a layer map) that doesn't read it either and passes it
      further. It skips the exceptions above; carrying into a protocol or behaviour counts as a
      runner delivering to a port. Across the published variants, run without layer maps, it finds
      four, all in V30 and V35 (`Intents.pay/2` carries `stock_levels` through `Cart` to
      `CheckStock`; `Intents.submit_address/2` carries form params through `CheckoutFlow` to
      `Address`).
  - **Elixir techniques that meet it.** This rule resists tooling; these habits help.
    - *Slice by feature, and name the slice.* V4's feature slices, V23's composable units, and V36's
      one-module-per-feature all make you name the concept before you write it.
    - *Name domain abstractions by their kind, not their use.* `OffsetAndScale`, `Boiler`,
      `LowPassFilter`: each names what it is, so a reader learns it once.
    - *Ask "what do you know about?"* The answer should be one thing. A module that defines what an
      order means *and* computes shipping on it knows two things.
    - *Write the short moduledoc.* Two lines usually cover the concept, ports, and configuration.
      ```elixir
      @moduledoc "Smooths a stream of numbers. In: a number. Out: the smoothed number. Config: strength."
      ```
    - *Write one-off wiring as an anonymous function.* A named function used once is "indirection
      without abstraction"; `fn` or `&` at the composition says it's wiring (§6.1, §1.6.3).
  - **What doesn't meet it.** `Utils`, `Helpers`, or `Manager` modules; a function that renames a
    primitive (`add(a, b), do: a + b`); names that teach nothing (`process2`, `handle_stuff`); a
    parameter carried down unread (the "should" above).
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
    outputs is probably too specific to be a good abstraction. Over-splitting a real abstraction's internals is as
    much an R7 defect as inventing a trivial one. And **reuse is a positive**: never let R7, height,
    or the pass-through check flag a genuinely shared abstraction — high fan-in is evidence it earns
    its place.
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
  - **Elixir techniques that meet it.**
    - *Carry the least machinery that works.* V36 reaches strong compliance with almost no apparatus.
      When you find yourself adding a layer to hold a layer, that is the smell R7 names.
    - *Let the linter catch proliferation.* `ala_lint`'s pass-through and dead-code detectors flag the
      one-in, one-out forwarding function that adds a name and a hop but hides no decision.
      ```elixir
      def save(x), do: Repo.insert(x)   # 1-in/1-out, hides no decision: flagged
      ```
    - *Use what the standard library already names.* Don't write `Filter`, `Map`, or `Sort` domain
      abstractions when `Enum` and `Stream` exist; Spray: "There would be no sense in reinventing that
      functionality as 2-port classes" (§6.3). Configure a generic step with an `Enum`/`Stream`
      function instead ("Functional programming", point 10).
    - *Don't build wiring machinery for an algorithm.* Plain function calls down the layers are enough
      when the problem is a calculation (§7.5; "Functional programming", point 2).
    - *Mind both size bounds, and the port count.* 100 to 500 lines per abstraction; averaging under 100
      means "more abstractions than we need" (Summary). "Abstractness decreases with more ports"
      (§7.2.3), so a module with a long list of inputs is probably too specific.
  - **What doesn't meet it.** A module that only forwards to another; a manifest or codegen step
    for a page with two features and one reaction (V35's lesson: declare in a manifest once the wiring
    is numerous or regular, inline in the composition while it's rare).
  - *Spray:* Summary, "All abstractions must be small" (a rule of thumb of 100 to 500 lines;
    averaging under 100 means "more abstractions than we need"); §7.10.2 ("probably in the range of
    50 to 500"); §7.2.3 ("abstractness decreases with more ports"); §6.3 (don't reinvent library
    functions as abstractions); §7.5 (plain function composition suffices for an algorithm). Spray
    makes size a fundamental constraint; this checklist and `ala_lint` score it as advisory (see
    "Where this checklist departs from Spray").
- **R8 — the composition reads as the requirements; names and shapes serve the reader.** The top
  layer should read like the spec; config keys should name what they configure; state wires should
  be shaped for a reader. **R8 is judgement, not shape** — "reads as the requirement" cannot be
  mechanically decided the way "edges drop" can. It is a review prompt and the honest boundary of
  the checklist: R1–R7 and R10–R11 are checkable (mechanically or semi-mechanically), R9 in part; R8 marks where automated
  checking stops and human review begins. A tool should *report* the R1–R7 findings and *flag* R8
  concerns, never claim to score R8.
  - **Elixir techniques that meet it.**
    - *Make the wiring a document.* V29's and V30's manifest is the page's requirements as data: which
      features, which reactions, which configuration. V33's flows channel does the same for wizards.
      ```elixir
      page :cart, features: [Cart, Wishlist, Checkout], reactions: [...]
      ```
    - *Keep the page legible.* V36's `handle/3` clauses read top to bottom as the page's behaviour,
      one clause per user action.
      ```elixir
      def handle(:remove_item, %{id: id}, page), do: Cart.remove(page, id)
      def handle(:checkout, _args, page),        do: Checkout.start(page)
      ```
    - *Keep the composition inspectable.* A graph value (named instances plus a wire list) can be
      printed, diffed, and drawn; nested closures can't (§3.11.3, §6.2.1). Keeping "create an
      instance" apart from "wire it" lets the code follow the diagram line by line (§6.2.2).
  - **What doesn't meet it.** Wiring spread across several modules' labels, so no one place shows it;
    a page whose callbacks compute; configuration keys that don't name what they configure.
  - *Spray:* §2.4 (executable expression of requirements; 3–10% of the code); §3.5; §3.6 (diagrams
    vs text); §7.7; §3.11.3, §6.2.1, §6.2.2 (explicit
    wiring over hidden closures; keep instantiation and wiring apart).

The next three rules each turn one of Spray's constraints into a check of its own, rather than
leaving it as background for an earlier rule.
- **R9 — ports are typed by a programming paradigm; an abstraction owns no interface except its
  own configuration.** Spray calls the second half "critically important" (§2.3.4).
  - **Main interface and ports.** A module's own public API (its constructor, its configuration,
    the struct fields set when it is created) is there for the layer above, to instantiate and
    configure it. Every other input and output at run time goes through a *port*, and a port's type
    is a programming paradigm from a lower layer. "No other interface implemented or required by the
    class can be 'owned' by the class."
  - **Owned interfaces fail, both directions.** A *required* interface is one the consumer defines
    for others to implement (Clean Architecture's ports, which the business layer defines for
    adapters to implement; the dependency inversion pattern). A
    *provided* interface is one a module defines, specific to itself, for its peers to call. Both
    carry one module's design inside the other, "a fixed arrangement between the two". Moving the
    interface to a module of its own doesn't help if it still describes what one module needs:
    "simply moving IB doesn't make it more abstract" (§6.5.5; IB is the interface that module B
    requires).
  - **`@callback` and `defprotocol` in Elixir.** What matters is who defines the interface relative
    to who implements and calls it.
    - *Allowed: a paradigm port.* A behaviour or protocol in the Programming Paradigms layer that no
      domain abstraction owns (`Step`), implemented by domain modules (`defimpl Step, for:
      LowPassFilter`). This is how a domain abstraction gets a port.
    - *Allowed: configuring a more abstract module.* A module far below its implementers defines
      callbacks that higher modules implement to configure it. `GenServer`'s `init/1` and
      `handle_call/3`, `Plug`'s `call/2`, or a generic scheduler that takes a job module. The
      implementer depends *down* on it, which is legal. It's the behaviour-shaped form of Spray's
      "lambda passed in" for calling up the layers.
    - *Not allowed: a required interface between peers.* `Checkout` defines
      `Checkout.PaymentGateway` with `@callback charge/2`, and `StripeClient` in the same layer
      implements it. The behaviour describes what `Checkout` needs, so `StripeClient` is written to
      `Checkout`'s design. Moving it to a shared `Contracts` module doesn't change that. The ALA
      route is a paradigm-typed port on `Checkout` (request/response, say) that the composition
      wires to a payment abstraction.
    - *Not allowed: a provided interface made for peers.* `Inventory.Behaviour`, describing
      `Inventory`'s own functions, so peers can call it through the behaviour or mock it with Mox.
      Its callers are still written to `Inventory`'s design. Spray's testing rule is to "mock the
      ports": wire a fake instance to a paradigm port in the test, not replace a peer by name.
  - **Composing functions.** When the composition calls functions directly, "parameters and return
    values are effectively ports", and so is a function passed in (§2.3.6). The same rule applies to
    their types.
  - **Data on a port, and data-transfer structs.** Two domain abstractions "may not ... share a
    DTO" (a data-transfer object: a type made only to carry data between two modules). The type "must be more abstract and come from a lower layer", often a primitive, and "T may
    be passed in by the application" (§4.8.1).
    - *Allowed: standard types* (numbers, strings, lists, maps, and tuples in the paradigm's own
      shape, like `{:emit, out, step}`).
    - *Allowed: a struct from a lower layer that is itself a real abstraction,* such as `Decimal`,
      `Date`, or a domain `Money`. Both ends depend down on it, a knowledge dependency.
    - *Allowed: an application-defined struct passed in.* The app defines it, and the abstraction
      either carries it without matching on its fields or reaches its fields only through what the
      app configures it with (field names, accessor functions, a protocol implementation). This is
      Spray's "T passed in by the application".
    - *Allowed: a struct used only inside one abstraction* (a feature and its own submodules).
    - *Not allowed: a data-transfer struct between peers.* `Cart` builds a
      `%Checkout.OrderRequest{}` defined by `Checkout`, which pattern-matches on it. Or both match on
      a struct neither owns but which was designed for exactly their exchange. Either way one
      abstraction's design now lives in the other. Moving the struct to a shared module in the same
      layer doesn't fix it, for the same reason as a moved interface.
    - *Fixes:* the composition converts between the two shapes with a `(p) ->` lambda; the
      application supplies the type; or the two ends become two instances of one abstraction that
      owns the format (§7.8, see R5).
    - *R9 vs R10:* R9 is about the message type on a port between two abstractions. R10 is about a
      domain entity that two features both read.
  - **Outputs announce; they don't command.** "An output port from an abstraction may say 'This has
    happened' or 'Here is my result', not 'do this next', or 'here is your input'" (§7.2.6). What
    happens next is the wiring's business. The one exception is request/response. Wired point to
    point, "a request is implicitly a command" (§4.6), and that's fine because the application set
    up the wire. In Elixir terms, an outcome a feature returns should read as a fact or a result
    (`{:item_removed, item}`, `{:ok, total}`), not as an instruction to a named receiver.
  - **Paradigm instructions as outputs.** A common Elixir pattern has features return outcomes
    such as `Outcome.stream_insert(:cart, item)` or `Outcome.flash(:info, "Added")`, which a generic
    interpreter carries out. Such an outcome bundles a verb ("insert"), a target (`:cart`), and a
    payload. R9 treats the target and the verb differently:
    - *Must: an output never names its destination.* Whatever decides where an output goes, and how
      it is presented, comes from the composition: stream names, topics, message text, a named
      receiver. *Reviewer's test:* could the page send this output somewhere else without editing
      the feature? If not, it fails. This follows from "No endpoints" below (§4.4.2) and from R3,
      since message text is an application literal.
    - *Should: an output reads as a result, not an operation.* Prefer outputs that say what happened
      or what the result is (`{:item_added, item}`, or `{:added, item}` on a port the feature names),
      over an operation the feature has decided on (§7.2.6). *Reviewer's test:* could the page wire
      this output to a different kind of receiver (a counter, a log, nothing), not just a different
      stream? A "no" is a prompt to reconsider, not a defect.

    The "should" is weaker because the evidence is. One reading of Spray treats a paradigm
    instruction as request/response, where "a request is implicitly a command" (§4.6). But his
    request/response is two-way, used when "the requester needs to know" something back, and a
    stream insert expects nothing back. The checklist doesn't prescribe a technique. Features
    announcing facts that the page maps, results on the feature's own ports bound once by the page,
    and targets passed in as configuration all meet the "must"; the field guide compares them.
  - **No endpoints.** An abstraction never names where its input comes from or where its output
    goes. Receivers never register themselves with a sender or subscribe to a public event (§4.4.2).
    The composition sets every wire. "The dependency injection wiring must be explicit. It must be
    specified in cohesive user story abstraction in a higher layer. The wiring cannot be done by
    using a dependency injection container or relying on matching interfaces" (§3.11.3). In Elixir,
    the nearest thing to a container is a module looking up its own collaborator
    (`Application.get_env(:app, :payments)` inside a domain module): the module chose its own wire.
    Reading config in the composition and passing the module or function down is fine. Spray
    doesn't prescribe how the indirection is built: "They can be callbacks, signals & slots,
    dependency injection, or calls to a framework send function" (§7.2.6).
  - *Visible shape:* a `[]` leaf takes only `pN` wires, `(p) ->` lambdas, and configuration, and
    **no edge names a peer**. The smells:
    - a port typed by a peer's struct;
    - a behaviour or protocol defined in a feature or domain module and used across a boundary;
    - a module that names its own source, destination, or topic.

    *Verify:* for each abstraction, ask three questions:
    - Does it name the source or destination of any input or output, including a stream name,
      topic, or message text in an outcome it returns?
    - Does any port's type belong to a peer rather than a paradigm, the standard library, a lower
      layer, or the application?
    - Does it define a `@callback` or protocol that its own peers implement or call (rather than a
      paradigm port, or callbacks that higher modules implement to configure it)?

    Any yes is a defect.
  - *Mechanizable:* partly. A `[]` function that references a peer module is already an R1 edge.
    Behaviours and protocols declared outside the paradigm layer and used across a boundary can be
    found from the AST. A peer struct being pattern-matched is findable. Literal stream names,
    topics, or message text inside feature-layer outcomes are findable, which covers much of the
    "must" for outputs. Whether a type is "more abstract", and whether an output reads as a result,
    are judgements.
  - **Should: configuration is set once, apart from run-time data.** Spray's §3.9: "If the abstraction
    consisted only of a single function, then that configuration data would need to be passed in
    every time the function is used. That would be awkward. It would also mix the data parameters of
    the function with the configuration parameters, breaking the Interface Segregation Principle." It
    is a "should" because his own §1.6.3 thermometer passes its settings on every call
    (`OffsetAndScale(adc, offset=4, slope=8.3)` inside the loop), a rung he then climbs past. R3's
    composition-line form (`f4(p1, 4, 8.3)`) is that rung.
    - *Reviewer's test:* does any function take configuration and run-time data mixed in one
      parameter list, so every caller repeats the same settings on every call?
    - *Elixir techniques that meet it:*
      - *Configuration as the first argument.* A map or struct built once at the composition comes
        first, run-time inputs after it. The composition keeps the value (a module attribute, the
        program value, an assign) and passes the same one on every call, or captures it once
        (`&Shipping.cost(@shipping, &1)`).
        ```elixir
        @shipping %{rates: %{standard: 500, express: 1500}, free_over: 10_000}
        Shipping.cost(@shipping, order)         # config first, then the data
        ```
        It is the same order as `Regex.run(regex, string)`, where the configured value comes first.
        In a `|>` chain the piped data lands in the first argument, so capture the configuration
        first (`&Shipping.cost(@shipping, &1)`) or make the call outside the pipe.
      - *A struct that carries its configuration.* Built once, then used many times: Spray's §3.9
        step, and the thermometer's `Step` structs (`%LowPassFilter{strength: 10}` then
        `Step.push(filter, v)`). The struct can also carry state when the concept has some (R4).
      - *A function built once.* The composition builds a closure over the configuration and passes
        the function. It works, but a closure can't be printed or inspected (§6.2.1).
    - *What doesn't meet it:* settings mixed into the data arguments (`cost(method, subtotal,
      rates)`), each caller repeating them; defaults baked into the domain (R3);
      `Application.get_env/2` read on every call inside a domain module.
  - **Elixir techniques that meet it.**
    - *Put port protocols in the Programming Paradigms layer.* A protocol or behaviour used as a port
      belongs to no domain abstraction; a domain module gets a port by implementing it. *Run in the
      thermometer steps, not tried in a variant.*
      ```elixir
      defimpl Step, for: LowPassFilter do
        def push(%{strength: k, last: last} = f, v), do: (out = last + (v - last) / k; {:emit, out, %{f | last: out}})
      end
      ```
    - *Let the application supply the type.* A struct the app defines and a generic module carries
      without matching on it is fine (§4.8.1).
    - *Collapse I/O into one paradigm value.* The coffee maker's `SensorReading` (all inputs) and
      `HardwareCommand` (all outputs) give the application ports shaped by their kind, not by any
      particular device, and make the boundary pure data a test can build. V25's projections do the
      same for cross-feature reads.
      ```elixir
      %HardwareCommand{boiler_heater: :on, warmer_heater: :off, relief_valve: :closed}
      ```
    - *Adapt mismatched ports with a lambda at the wiring.* When one port's shape doesn't fit
      another's, the composition converts with an `fn` (§6.17.4).
    - *Make outputs announce, and keep destinations in the page.* Three techniques meet the "must",
      with different costs:
      1. *Announce facts; the page maps them.* The feature returns `{:item_added, item}`, and the page
         decides that means "insert into the cart stream and flash 'Added'". It meets both halves and
         is closest to Spray, but it costs a page clause per fact, and every feature needs its own fact
         vocabulary. V35's typed facts plus manifest `reactions` are the closest variant.
      2. *Results on the feature's own ports, bound once by the page.* The feature returns
         `{:rows, {:added, item}}`, a collection change on a port it names itself. The page binds that
         port once (`rows: {:stream, :cart}`), and a generic interpreter applies the binding. It meets
         both halves with one line per port rather than per fact (V39's `Binder`, with thirteen kinds of
         binding; an unbound port is silently dropped, so test that every declared port is bound).
      3. *Pass the target in as configuration.* `Wishlist.new(stream: :wishlist)`, with the feature
         returning `stream_insert(config.stream, item)`. It's the smallest change to a V10/V36-style
         app, and it meets the "must". It meets the "should" only weakly, because the page can change
         which stream, but not what kind of receiver.
      ```elixir
      {:item_added, item}                   # 1: a fact; the page maps it
      {:rows, {:added, item}}               # 2: a result on the feature's own port; the page binds :rows
      stream_insert(config.stream, item)    # 3: an operation on a target the page configured
      Outcome.stream_insert(:cart, item)    # fails the "must": the feature names the page's stream
      ```
      Where the instruction is built matters. In V10 the page composition (`CartPage`) builds
      `stream_insert(:cart_items, item)`, so the page names its own streams, which is fine. In V36 the
      features do: `Wishlist.remove/2` returns `Outcome.stream_delete(:wishlist, product)` and the flash
      text "Removed from wishlist". That fails the "must"; technique 3 is the cheapest fix.
    - *Test against ports.* Pass a fake into the port (a stub struct implementing the protocol, a
      function, or a test pid as the output). Mox is fine for a paradigm-layer behaviour, since that
      *is* a port (see "Tests replace only ports" in the verify procedure).
  - **What doesn't meet it.** A `@callback` a feature defines for a peer (`Checkout.PaymentGateway`);
    `Inventory.Behaviour` written so peers can mock `Inventory`; a struct one peer defines and another
    matches on, even if moved to a shared module; an output that names a stream, topic, or message text;
    a module that looks up its own collaborator.
  - *Spray:* §2.3.4 (interfaces; owned interfaces "critically important"); §2.3.6; §4.4.2; §4.6
    (request/response); §4.8.1 (no DTOs; T passed in by the application); §6.5.5 (dependency
    inversion); §7.2.6 (outputs announce); §7.8; §7.24; §3.9 (configure once; the "should");
    §6.17.4 (a lambda at the wiring as an adapter).
- **R10 — no shared entity: no two features know the meaning of the same data.** Spray's rule is
  "no data coupling": two modules shouldn't have to agree on what a piece of data means (§3.8,
  §7.8).
  - *Must:* no domain struct is read or destructured by two features. *Reviewer's test:* could one
    feature change the shape of its data without editing another feature?
  - *Techniques that satisfy it* (the field guide compares them):
    - features share only an *identity key*, each keeping its own private data against it;
    - the composition resolves a value and passes it in (composed inputs);
    - a feature exposes a projection shaped for reading, which others read instead of its struct;
    - the two ends become two instances of one abstraction that owns the format (§7.8, R5).
  - *Visible shape:* a `*pN` (shared value) whose type is a named domain entity appears inside two
    sibling feature subtrees. *Verify:* is any domain struct referenced or destructured by two or
    more feature units? *Mechanizable:* yes: a struct read by two units of its own layer.
  - *The aggregate case (a judgement, not a hard defect):* Spray's own version is the ground symbol
    (§3.6.1). State that most instances interact with, like a game score, can legitimately become a
    domain abstraction one layer down that everything depends on as knowledge. So a struct in a
    *lower* layer that two features read may be a legitimate domain abstraction (a knowledge drop,
    fine), or Clean Architecture's shared-Entity coupling (business objects that every use case
    reads, bad). A tool can't tell them apart, so a
    shared *feature-tier* struct is a defect (above), and a shared lower-layer aggregate is a prompt
    for a human. `ala_lint` reflects this split: the feature-tier check is scored; the aggregate check
    is reported under `--strict` and above and scored by no tier (`--enforce` scores it).
  - **Elixir techniques, in more detail.**
    - *Give features private structs and share only a key.* V24's isolated capsules, V26's dual
      capsules, and V36's per-feature structs mean two features never hold the same data struct. They
      share an identity (a product id) and keep their own data.
      ```elixir
      %Wishlist{product_ids: [id]}         # share the id...
      %Cart{items: [%{product_id: id}]}    # ...not each other's struct
      ```
    - *Mediate cross-feature reads through a projection or the composition.* V25 hands feature A a
      projection of B, not B's struct, and V35 has the composition resolve the value and pass it in.
      ```elixir
      def projection(%Inventory{count: n}), do: %{in_stock?: n > 0}
      ```
    - *Make both ends one abstraction.* See R5's codec sketch. *Not tried in a variant yet.*
  - **What doesn't meet it.** `Checkout` pattern-matching on `%Cart{}`; one `Order` struct every
    feature reads and writes (Clean Architecture's shared entity), unless it is deliberately a
    lower-layer domain abstraction (the aggregate case above, a judgement).
  - *Spray:* §3.6.1 (the ground symbol); §3.8 (no data coupling); §4.8.1 (no shared DTOs); §7.8.
    The identity-key technique is an Elixir and database-backed way to meet it, not Spray's wording.
- **R11 — the application (top) layer is composition only.** The application instantiates,
  configures, and connects. It holds *all* app-specific knowledge and *no* app-specific logic: "no
  normal programming language code such as assignments and if statements" (§3.5), and about 3–10%
  of the code (§2.4). Spray's own examples show what that sentence covers and what it doesn't.
  - **Not banned:**
    - *Naming an instance so it can be wired twice.* `temperature = new FloatField()` (§1.6.6), or
      locals for cross-connections (§3.6.2). In Elixir, binding a configured struct or a pid to a
      variable so two wirings can use it.
    - *A predicate or small function passed in to configure a generic abstraction.* For example
      `new Filter(x => x>=0)` (§3.11.3), or `.Bind(x => x==0 ? -1 : 1000/x)` (§6.1.3). In Elixir,
      something like `Filter.new(keep: &(&1 >= 0))`. That states a requirement ("ignore negative
      readings") once, as configuration. The abstraction runs it.
  - **Banned:**
    - *Control flow that decides what runs:* a guard around a call, a loop over data.
    - *Assignments that hold or compute data between calls.*

    Spray flags his own §1.6.3 thermometer for this: the application "is still doing some logic
    work - the 'for loop' and 'if statement', which we will address soon".
  - **Where each kind of `if` goes.** These are Spray's moves, in order of how often they come up:
    1. *Propagation guards* ("only if there's a value", "stop on error") move into the connection
       mechanism. He factors the `if`s "into the Compose function" (Bind, the function that chains
       one step to the next in a monad, §6.1.3), and his
       thermometer's `if` disappears once `SampleEvery` simply emits nothing (§1.6.4). In Elixir:
       a step returning `{:quiet, step}`, `with`, `Stream` filtering.
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
  - *Mechanizable:* partly. The share of the code is mechanical, and so is counting branches. Telling
    the kinds apart is judgement, though some forms are recognizable (see the linter notes).
  - *In LiveView:* the framework hands a page events to route, URLs, and a lifecycle to follow, so
    some forms remain: clauses that match an event name and forward it, decoding string params, and
    a lookup from URL to step. None of these is logic. Everything else a page tends to contain (guards,
    rules, arithmetic, handled data, history, template loops, even the `connected?/1` guard, which an
    `on_mount` hook takes out of the page) has one of the moves above, and a page can reach zero
    logic this way. "The application layer, and where LiveView pieces sit" has a table of the forms a
    page typically contains, and says which are wiring, which are logic to move down, and which are
    framework-imposed departures to keep small. "How close a LiveView page can get" records what the
    variants reached and what it cost.
  - **Elixir techniques that meet it.**
    - *Split the shell: generic interpreter below, page dispatch in the page.* V36's shell reduces a
      list of outcomes into view state. That interpreter is generic, so it's an execution model in the
      Programming Paradigms layer, below the page. The part that names the page's composition is page
      code. (V35's `EffectInterpreter` is the same idea.)
      ```elixir
      # paradigm layer, shared by every page:
      def apply_all(socket, outcomes), do: Enum.reduce(outcomes, socket, &apply_outcome/2)
      ```
    - *Guards go into the connection mechanism.* `with`, a runner that stops when a stage returns
      `:quiet`, or `Stream` filtering (§6.1.3). *The `:quiet` runner is run in the thermometer steps,
      not tried in a variant.*
      ```elixir
      with {:ok, order} <- Checkout.submit(page.checkout),
           {:ok, receipt} <- Payments.charge(order), do: ...
      ```
    - *Requirement conditions become configured instances; history becomes a state machine.* A rule
      becomes a predicate passed to a generic abstraction (R3), or a small logic abstraction like the
      coffee maker's AND gate (§2.9.3). A rule that depends on what happened before becomes a state
      machine (§4.16). V33's flows channel, a pure transitions table, is the closest variant, and V39–V41
      pass the table in as configuration so the feature decides.
    - *Keep an abstraction's own rules inside it.* The coffee maker's `Boiler` enforces "the heater is
      off whenever I am empty or my valve is open" itself. The application asks it to heat; it refuses
      when that would be unsafe. The rule never reaches the top.
    - *Route outcomes; don't decide on them.* A `case` whose arms only send the ok and error results to
      different places is routing two output ports (§4.9.1). A `case` that computes in its arms is
      logic, and belongs in a feature. V39 routes a `handle_async` result to `Checkout.succeeded/2` and
      `Checkout.failed/2` with three clause heads.
    - *Move iteration and compound conditions out of the page template.* A generic list or table
      component iterates (§4.12). A feature computes a compound boolean and the template wires it.
      ```heex
      <.table rows={@streams.items} />   <%!-- the component iterates, not the page --%>
      ```
    - *Move lifecycle guards out of `mount/3`.* An `on_mount` hook is LiveView's own extension point.
      V39 and V41 configure a generic hook on the page, `on_mount {Subscribed, {Broadcast, :subscribe}}`,
      so their `mount/3` has no branch. Auth redirects fit the same hook; untried here.
    - *Generate the glue.* V30's committed codegen makes the page's wiring a generated file, so the
      hand-written page stays thin (§2.5.1). `mix zc.gen --check` fails the build if it drifts.
    - *Stop handling data at the page.* Three designs reach zero R11 findings, each by a different
      route: bind every feature output once in one map per page (V39's `Binder`), run the page as a
      circuit of instances and wires (V40), or make each feature a LiveComponent instance that lands
      its own outputs and announces the rest (V41). Their costs are under "How close a LiveView page
      can get".
  - **What doesn't meet it.**
    - Arithmetic and computed assigns in the page (`assign(socket, :total, Enum.sum(...))`).
    - A business rule in a handler ("can't check out an empty cart" as an `if`).
    - A `case` on the URL in `handle_params` that encodes which steps may follow which.
    - A handler that takes one feature's result and passes it to another.
    - Loading rows in `mount/3` and handing them to a feature (`Cart.new(items: Store.all(...))`).
    - A `for` or a compound `:if` in the page's template, including a comparison like
      `disabled={@item_count == 0}`.
  - **Limits.** Some forms stay in any LiveView page because the framework or the language puts them
    there: routing by event name and URL, decoding browser params, storing the program value back, a
    lookup from URL to step, and a JavaScript hook or two. None is logic. "How close a LiveView page
    can get" lists them with reasons, and records what the three zero-finding designs cost.
  - *Spray:* §2.9.3 (the coffee maker's application is a diagram of instances; its conditions are
    AND-gate instances, not `if`s); §3.5 ("no normal programming language code such as assignments
    and if statements"); §1.6.3 (the thermometer's `if` flagged as logic); §1.6.4 and §6.1.3 (guards
    move into the connection mechanism); §1.6.6 and §3.6.2 (instance variables for wiring);
    §3.11.3 (configuring with lambdas); §4.4.4, §4.9.1, §4.16. Reading LiveView's multi-clause
    handlers as routing, and the framework-imposed departures, are this checklist's Elixir
    adaptation.

Rules-of-thumb for using R1–R11: R1–R2 and R5 are the coupling core; R3 is requirements-locus; R4 is
state-with-its-owner; R9 is the port/interface discipline and R10 the data-sharing discipline (both
coupling-core in spirit); R6–R7 are design minimality/nameability; R11 is the composition-only top
layer; R8 is the human-judgement remainder. A mechanical tool (`ala_lint`) can check R1–R7 and R10–R11
(and R9 in part) at varying precision; treat R8 as the reason a green run is necessary but not
sufficient.

One topic is *in development* and not yet a rule: how wires differ in meaning and in how they run
(paradigms, push or pull, sync or async, fan-out, glitches, loops). See "Kinds of connection and
how they run" under the worked examples.

## Functional programming: what Spray says, and what it means in Elixir

Spray writes in C#, but he returns to functional programming many times, and it matters more for
Elixir than any other part of his site. His own chapters on it (§6.25 "Functional programming",
§6.26 "Functional programming with monads", §6.27 "Functional Reactive Programming") say only "TBD".
What he does say is spread across §2.3.6, §3.9, §3.10.2, §3.11, §6.1 to §6.3, §7.2 and §7.5. This
section gathers it, says what each point means in Elixir, and names the rule it supports. Where the
Elixir reading is this checklist's and not Spray's, it says so.

1. **The application layer is pure functional code.** "ALA is, essentially, functional programming.
   All the top layer code that implements the application itself by wiring up instances of domain
   abstractions in some combination is pure functional code" (§3.11). The impure work lives below it:
   "The fundamental idea of monads, that of separating execution details, state, and I/O into
   pretested units that are then composed using pure functional code is the same for ALA" (§3.11.1).
   "Domain abstractions completely hide their contained state in well tested code and are composed by
   pure functional code" (§3.10.2).
   - *In Elixir:* the composition (a page, a wiring function) builds values and routes them. It
     doesn't call the repo, send messages, or compute. Two placements of the side effects both meet
     this. Spray's is to hide them *inside* domain abstractions (his `Display` writes to the screen
     itself, his `CSVFileReaderWriter` reads the file). The common Elixir one (functional core,
     imperative shell) has features return effects as data and a generic interpreter in the
     Programming Paradigms layer perform them (V35's `EffectInterpreter`, V36's shell, V39's
     `Binder`). Either keeps the top layer pure. The interpreter form keeps the features pure too,
     which makes them testable without Phoenix. *Rules:* R11, R4.
2. **Plain functions can be ALA.** "ALA can be applied to functional programming too. Abstractions are
   then obviously functions, and the same ALA relationship restriction applies - a function may only
   call a significantly more abstract function. The functions then form layers" (§2.3.6). "Parameters
   and return values are effectively port", and when a function used to call a peer part-way through,
   "the second function will now need to be passed into it. The function parameter is also a port"
   (§2.3.6). And the wiring pattern isn't always needed: "if your whole problem is just an algorithm,
   and therefore suits a functional programming style, then you can still compose abstractions with
   function abstractions, provided all function calls are knowledge dependencies, and not say, just
   passing data or events" (§7.5).
   - *In Elixir:* a pricing calculation that calls `Money`, `Decimal` and a tax-rate function is
     fine as plain calls down the layers. It needs no ports, runner, or circuit. Ports and wiring are
     for where instances *communicate* (one feature's result feeding another, events, UI). Don't
     build a circuit for arithmetic. *Rules:* R1, R7, R9.
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
   - *In Elixir:* the first problem at the top is R11's "handling the data" (§1.6.3). The second is
     R4's reason for keeping state with its owner. The third, a parameter carried through a function
     that never reads it, is R6's tramp-parameter "should". Elixir invites it, because threading an
     `opts`, a `context`, or a `session` map down several calls is the easy default.
4. **Abstraction before referential transparency** (§3.11.2, Summary). "ALA prioritizes abstraction
   over referential transparency." Referential transparency "attempts to improve analysability by
   always removing time from the analysis, even when time is a fundamental aspect of what is being
   described. It will expose implementation details if necessary to do it." His example: "Passing the
   running average to the function every time there is new input data breaks an otherwise good
   abstraction." A computation is `input + state --> state + output`, which you can group two ways,
   the functional `(input + state) --> (state + output)` or the object form `input ( + state -->
   state +) output`, and "In ALA we choose between these two philosophies on a case by case basis."
   - *In Elixir:* immutability forces the functional grouping, so a filter's memory comes back as a
     new struct. What keeps the abstraction whole is that the caller stores the struct *without
     looking inside it*: the state is still the filter's. A runner or a program value held in assigns
     removes even the storing ("Past the pipe"). A process is the object form. Use it where the
     concept is concurrent, not to imitate objects. *Rule:* R4.
5. **From procedural code to ALA, without object-oriented design** (§3.9, §3.10). Spray derives objects
   from procedures in five steps, and in §3.10 calls ALA's use of them "objects as a language feature,
   not a design philosophy". Configuration goes in a struct, and that struct "is immutable". Configuring once
   matters: "If the abstraction consisted only of a single function, then that configuration data
   would need to be passed in every time the function is used. That would be awkward. It would also
   mix the data parameters of the function with the configuration parameters, breaking the Interface
   Segregation Principle." Wiring is two more fields, and "The fields are immutable after they are
   set." State that belongs to one abstraction joins its struct. State that belongs to none becomes a
   state abstraction with dataflow ports, wired in. The result: "both the configuration data, and the
   wiring data stored in an instance can be immutable. Only instances of abstractions that contain
   state data are mutable, and this is clear from very nature of the abstraction."
   - *In Elixir:* a struct holding configuration (and state, when the concept has any) is Spray's
     instance. Configuration set once, apart from run-time data, is R9's "should". One way is a
     configuration map or struct as the *first argument*, with run-time inputs after it. *Rules:* R9,
     R4.
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
   - *In Elixir:* `with` is a Bind for `{:ok, _}`/`{:error, _}`. A `Stream` pipeline is a deferred
     monad. A protocol such as `push(step, data) :: {:emit, out, step} | {:quiet, step}` plus a runner
     is a push-style Maybe bind: a step with nothing to say returns `:quiet`, and the runner, not the
     application, stops. The runner and protocol belong in the Programming Paradigms layer. *Rule:*
     R11 (guards move into the connection mechanism).
7. **Where monads fall short** (§3.11.3, §6.2). "ALA and monads are both about composition... The
   difference between ALA and monads is that ALA composes objects whereas monads compose functions."
   Monad functions "just have two 'ports', an input and an output"; "ALA's domain abstractions, on the
   other hand, can have many ports", and those ports "can use different programming paradigms, not
   just a specific data flow". "Monad libraries are more like discrete electronic components such as
   resistors and capacitors"; domain abstractions are like integrated circuits. "Composing functions
   creates mostly a chain structure whereas composing objects with ports creates an arbitrary network
   structure" (§6.2).
   - *In Elixir:* a `|>` or `Stream` chain suits a requirement that is a line. When one output feeds
     two places, or UI, events, and dataflow meet, the composition is a graph: a value holding named
     instances and a wire list, run by a runner, or HEEx for the UI part. See "Kinds of connection".
     *Rules:* R8, R11.
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
   - *In Elixir:* a `Stream` is pull-driven and is consumed by one run, so it suits a batch you
     iterate, not a LiveView that receives one reading at a time. The run-forever form is a
     composition value built once (in `mount/3`, or `init/1`), held by its owner, and fed each input
     as it arrives. A push protocol keeps the sync-or-async choice open: the same `Step` structs run
     synchronously in a pure `Chain` and asynchronously inside one process per instance, unchanged.
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
   - *In Elixir:* a composition held as data (structs plus a wire list) can be printed, checked for
     unwired ports, diffed, and drawn. A composition held as nested anonymous functions or a `Stream`
     can't be inspected. Committed generated code beats a macro for the same reason (V35): the wiring
     is a file you can read. `Circuit.new(...) |> Circuit.wire(:a, :b)` keeps the two steps apart; a
     piped DSL that creates and connects in one call is the fluent form, fine for a line. *Rules:*
     R8, "Past the pipe".
10. **Use a monad library you already have** (§6.3, Summary). "There would be no sense in reinventing
    that functionality as 2-port classes." He gives two ways: give some ports the library's own
    chaining type and put the library's operators between two instances (§6.3.1), or "write a general
    purpose domain abstraction that can be configured with a monad expression" (§6.3.2), which the
    Summary calls a domain abstraction called *query*.
    - *In Elixir:* don't write `Filter`, `Map`, or `Sort` domain abstractions when `Enum` and
      `Stream` exist. Either type a port as an `Enumerable` and put `Stream` functions between two
      instances, or write one generic `Transform` step configured with a function the composition
      supplies (`%Transform{fun: &Stream.filter(&1, fn r -> r.qty > 0 end)}`). *Not tried in a
      variant yet.* *Rules:* R7, R9.
11. **Lambdas: anonymous for one-offs, passed in for configuration and for calling up.** A function
    used once "is not an abstraction. The name itself becomes just a symbolic connection between two
    points in the code... It's indirection without abstraction. ... Lambda expressions solve this
    problem because they are anonymous" (§6.1; the `func1` point, §1.6.3). Lambdas also configure
    generic abstractions (`new Filter(x => x>=0)`, §3.11.3; the bowling `Frame` completion lambdas,
    §4.20). They are one of the legal ways to call up the layers (§2.1.3, §4.4.1). And they adapt
    mismatched ports: "This can be as simple as a lambda expression passed to the WireTo operator, in
    the same way that you would pass a lambda expression to a .Select clause in LINQ" (§6.17.4).
    - *In Elixir:* `fn` and `&` captures, written at the composition. *Rules:* R6, R9, R11.
12. **"No side effects" is not the goal** (§7.2). "Another contender to be king is 'no side effects'
    used by the functional mathematical purity guys... this lord is only effective because he usually
    serves the abstraction king. But, again, there are times when he doesn't, and 'no side effects' is
    not enough to make a good abstraction."
    - *In Elixir:* immutability rules out shared mutable state (R2) almost for free, and that inflates
      how clean an Elixir codebase looks. Pure code can still be fully coupled: a pure function that
      calls a peer (R1), reads a sibling's struct (R10), or matches a tuple shape another module
      produces (R5). Judge the design by the rules, not by purity.
13. **Immutability doesn't settle concurrency** (§3.10.2). "In functional programming, the issues of
    multithreading are handled by using immutable data. I suspect that systems that only use immutable
    state also have problems when using multiple threads. For example one thread could be using and
    inconsistent or outdated copy of the current actual state." His own rule is one thread per
    stateful instance, and for asynchronous calls, chaining with a task or future's Bind ("effectively
    the monad pattern for asynchronous calling"), or failing that, writing the abstraction as a state
    machine.
    - *In Elixir (this checklist's reading):* every LiveView process holds its own copy of shared data,
      such as stock levels. Decide at wiring time which instance is authoritative (a process or the
      database), treat each page's copy as a cache, and wire updates to it (PubSub set up by the
      page). Async work goes to `Task` or `start_async/3`, and a long-running activity becomes a state
      machine, not a blocked process.
14. **Recursion needs a forward-declared abstraction** (§8.1). "Circular knowledge dependencies happen
    all the time in functional programming where recursion replaces iteration." Recursion inside one
    abstraction is its own business. Across abstractions, put the abstract concept one layer down as
    an interface (a protocol in Elixir) that the concrete parts both provide and accept. A chain of
    calls that loops back is usually run-time communication and should be wired instead.
    - *In Elixir:* `Chain` implementing `Step`, so a chain can be a step inside a bigger chain, is this
      pattern. So is a tree of components that render children through a shared slot or protocol.

## Where this checklist departs from Spray

The rules follow Spray, and each one cites where. These are the places where this checklist adapts
him to Elixir or steps away from him, stated so a reader doesn't mistake them for his positions.

| Departure | Spray | This checklist | Why |
|---|---|---|---|
| Branches in a LiveView page | "no ... if statements" in the application (§3.5) | multi-clause handlers and forwarding `case`s read as routing; `connected?/1` and auth redirects tolerated (R11, the LiveView table), and `ala_lint` counts a `connected?/1` guard as routing | the framework delivers events and a lifecycle to the page; the departures are recorded, kept small, and never used for logic; both can move to an `on_mount` hook |
| Size | a fundamental constraint (Summary) | module size is an advisory check (R7; `ala_lint` scores a module over 500 lines only under `--strict`, and reports an average under 100 lines without scoring it) | line counts are a weak proxy for "readable alone", so a reader decides |
| Graded checks | ALA's constraints aren't graded | the linter's tiers: R7 and height advisory, R11 and public surface aspirational, the app-layer share, average size and shared aggregate reported only | lets a team adopt the checklist step by step; the tier is about scoring, not about whether a finding is real |
| Sharing data between features | no data coupling (§3.8), no shared DTOs (§4.8.1) | R10 states the same property, applied to features: no domain struct read by two features | not a departure in substance; the identity-key technique that meets it is the Elixir and database-backed part |
| The notation | diagrams and wiring code (§3.6) | a text encoding with `[tag]`, `$`, `q`, and tool-stamped marks | a whiteboard- and linter-friendly way to see the shapes; it isn't Spray's |
| Framework residue in a LiveView page | the application is wiring and configuration only (§3.5) | param decoding, a URL-to-step lookup, a JavaScript hook or two, and component instances kept mounted but hidden are recorded as departures ("How close a LiveView page can get") | the browser sends strings, the framework hands the page the URL, some things only the client can do, and `send_update` to an unmounted component raises |
| Holding the program value | objects change in place | an immutable program value held by its owner (a LiveView's assigns, a GenServer) and stored back after each run ("Past the pipe", R4) | Elixir has no mutation; holding the value is not handling the data |
| Execution model | prefers single-threaded solutions (§4.4.9) and treats the execution model as a wiring-time choice | the same property: execution-model choices are made where the instances are wired, and state belongs to its concept (R4, "Kinds of connection"). Whether to use processes is a technique, compared in the field guide | not a departure in substance; listed because Elixir makes processes tempting |
| Comments | "critically important" abstraction comments (§2.1.1) | a short `@moduledoc` naming concept, ports, configuration, and an example when needed (R6) | covers what Spray asks for without narrating bodies |

Earlier versions of this checklist split the LiveView application into shell → page → view
sub-layers. That was a departure without a reason, and it's gone: see "The application layer, and
where LiveView pieces sit".

## Enforcement tiers (what a linter scores, and when)

Not every rule is the same kind of obligation. The coupling and locus rules are hard requirements; the
minimality and shape rules are prompts a reader weighs; a couple are aspirational ideals a real app
cannot fully reach. `ala_lint` encodes this as three tiers, and the classification below is current.

| tier | rules and sub-checks | when scored |
|---|---|---|
| **Required** (a violation is a defect) | R1, R2, R3, R4, R5, R6, R9 (owned interfaces), R10, layer-validity | always (default) |
| **Advisory** (a prompt for a reader) | R7, module-size (over 500 lines), abstraction-height, pass-through, tramp parameters (R6's "should"), reference-level R1, self-subscription (the bottom layer may own its topic) | reported by default; scored under `--strict` |
| **Aspirational** (a purity ideal, not always obtainable) | R11 (no logic at the top; a `connected?/1` guard counts as routing), public-surface (encapsulate the little ball of mud; counted per function) | reported by default; scored only under `--super-strict` |
| **Reported only** (a ratio or a design choice, not a defect) | the application's share of all functions, files averaging under 100 lines, R10-aggregate (shared domain aggregate) | reported at every tier; scored by no tier (`--enforce` scores one) |
| **Not machine-scored** | R8 (judgement); the rest of R9 (outputs that name a destination or command, peer DTOs) | a human reads for these |

Two placements are deliberate and follow from the "little ball of mud" reasoning under R7. **R11** is
aspirational, not required. A LiveView page can reach zero findings, but only by adopting a design
built for it (results on ports bound by the page, a circuit of instances, or feature instances that
announce what they did), and a CI gate shouldn't choose the design for a team. Its findings are still
real, and each one is either moved or recorded as a departure (see R11).
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
  must know its parent, an upward dependency), **no global event names, no receiver subscribing itself** to a
  sender or a public topic (those are peer edges in disguise; the composition wires them). A
  callback the composition passes *down* is fine: that is how ALA calls up the layers. (Composition
  vs decomposition, §2.6, §3.7, §7.12; Lego, §2.8.2; by feature, §7.14; layers replace containment,
  Summary, §2.6.1, §7.18; inheritance, §7.16; global event names, §4.7.4; registration, §4.4.2.)
- **Ports go sideways into technical domains (behind R9).** Database, hardware, and network are
  run-time dependencies off to one side, reached through paradigm ports (like the spokes of
  Hexagonal Architecture, also called ports and adapters), not
  bottom layers and not downward adapters. Keep a *domain abstraction of* the DB or UI (configurable,
  composable — a persistent `Table` wires to a grid), not a raw port to an adapter. An OSI-style stack
  becomes a sideways chain of domain abstractions. (§6.19, §7.17.) In Elixir, an Ecto repo, an HTTP
  client, or a PubSub server is such a technical domain: reach it through an abstraction of it,
  wired in, not by calling it from every feature.
- **Two phases, and the diagram is the source (behind R11).** Wire the network once, then run it (an
  OTP supervision tree is exactly this). The topology/"diagram" is the single source of truth for
  requirements, architecture, and code, and should be flat, declarative, and greppable. (V30's
  committed codegen is one way to make that mechanical.) Spray: "the entire application is wired up
  once at the beginning when the application starts executing, and then all ports are considered
  infinite steams that work as long as the application is running" (§3.11.3; also §1.6.4, §2.4,
  §3.6).
- **Abstraction is not the same as instance (behind R6/R9).** The abstraction is the zero-coupled
  design artefact; the instance is the run-time thing that communicates. Blurring them is what tempts
  people to put dependencies *between abstractions* to move data, destroying them as abstractions.
  (§3.2.2.) In Elixir, a module is the abstraction; a struct value or a process is an instance.
- **Execution-model choices are design-time, made on performance/expressiveness grounds.** Push by
  default, pull for performance; decide sync vs async at wiring time (`cast` vs `call`, PubSub vs
  `send`), never inside a domain module; model time-spanning activities as state machines, not
  threads (§4.4, §4.19). Spray calls the usual rule of thumb, GALS (synchronous locally,
  asynchronous across processors), "too simplistic": anything that takes real time, such as I/O or
  a delay, should be asynchronous even on one processor (§4.6.1). On the BEAM that's natural, since
  message passing between processes is asynchronous.

### Procedure — build a new ALA program

Two roles, even when one person plays both, "wearing only one hat at a time" (§5.7). The
*architect* works from the requirements, expressing each as a wiring of instances and inventing the
domain abstractions it needs. The *developer* implements those abstractions, which "know nothing of
the requirements". Most of the hard thinking is the architect's, including designing paradigm
interfaces that work "between any two domain abstractions for which it may be meaningful".

1. **Iteration zero (a short up-front design pass, ≤ one sprint, whatever the project size).** Go through requirements one by one,
   fast (aim ~one feature/hour), *describing* each as a wiring of instances and **inventing domain
   abstractions and paradigms as you go** (each gets config params for the specifics). Decide
   app-layer vs domain-layer by *scope of knowledge* (specific to this app → app; reusable → domain).
   Output: a topology plus a named list of abstractions, each with a one-line insight, **no
   implementation yet**. This is not waterfall — the rest is zero-coupled abstractions, so
   implementation cannot feed back and force redesign. (§5.2.1, §7.26.1.)
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
   constructor, optional settings defaulted); give every instance a **name** for debuggability (in
   Elixir, an id in its configuration for logs and telemetry, not a registered process name, which
   would let senders find it themselves); put
   a **root readme** naming the knowledge prerequisites (ALA, the paradigms, the domain abstractions,
   "the diagram is the source"). Velocity climbs as the domain matures. (Convention over
   configuration, §5.5; the readme and knowledge prerequisites, §2.3.7, §5.6.)

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
  can't see the abstractions): get it working messily, then (1) move requirement-detail up to the app layer (R11/R3), leaving generalized units
  with params/config; (2) merge similar generalized units, their differences becoming config; (3) cut
  the peer calls — pass in functions/ports (a function passed in mid-computation *is* a port), and
  move any observer/subscribe *registration* up to the composition.
- **Legacy, per user story:** reverse-engineer the call tree for one story (an all-files
  search is legitimate here), pin it with acceptance tests at its input/output boundaries, factor the
  call-tree into a new domain abstraction (copy-pasting useful snippets) (§8.3), mark the old modules for
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
8. **Tests replace only ports:** "you always test with dependencies in place, but you mock the
   ports" (Summary).
   - *Must:* a test never replaces a knowledge dependency (a module the subject relies on for its
     meaning, in a lower layer), just as you wouldn't mock a square root. *Reviewer's test:* does any
     test swap out a module the subject calls by name, rather than an instance wired to one of its
     ports?
   - *A domain abstraction's unit test* uses its real lower-layer dependencies, and wires fake
     instances to its ports.
   - *Testing the application* with its real domain abstractions "is exactly acceptance testing".
   - *In Elixir, for example:* pass a fake into the port (a stub struct implementing the paradigm
     protocol, a function passed in, a test process as the output pid). A Mox mock of a
     paradigm-layer behaviour is a fake port, which is fine. A Mox mock of a lower-layer module is
     the failing case. A LiveView test of a page with its real features is the acceptance test. A test
     that must replace a *peer* by name is a sign the peer was never behind a port (R1, R9).

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
fallback R1 check (cycles, with no layer map) and the reference-level R1 advisory run on the
module→module graph; (2) R7's dead/single-use check is scoped to a module's *private* functions
(privates are the module-local, unambiguously-inlinable case); (3) `[tag]`s and the layer map
attach to module names.
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
2. else a layer whose **`uses:`** pattern matches a module the code `use`s (a schema is
   persistence even in a domain namespace);
3. else the first layer in the ordered map whose **module-name pattern** or **filesystem path glob**
   (`paths:`) matches — so a project declares its own convention (e.g. `~r{/features/}` → `:feature`)
   rather than inheriting magic directory names;
4. else **unassigned**.

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

### The application layer, and where LiveView pieces sit

Spray treats the application as **one abstraction** whose content is wiring and configuration. The
knowledge of what flows between its parts is "contained together inside the Thermometer
abstraction". It has no internal structure of its own: "there are no subfolders under the
application" (§5.4). A bigger app becomes a composition of **Features**, a layer of their own that
the application wires. So calls between the application's own functions are internal to one
abstraction, not peer edges (R1). The one caution is R1's working-chain clause: product functions
doing work and calling each other.

**Where each LiveView piece belongs.** A LiveView page is not a stack of application sub-layers
(shell → page → view). Only the page-specific parts are application:

| Piece | Layer | Why |
|---|---|---|
| The page's LiveView module (`mount/3`, `handle_event/3`, `handle_info/2`, `render/1`) | Application | it knows this page's requirements and wires everything else |
| The page's HEEx template | Application | the UI half of the diagram: nesting is the "display inside" wiring |
| The router | Application | it composes pages |
| Page-specific components used only here, which know the product | inside the page (Application), or a Feature if it has its own state and ports | part of the page's wiring, or a feature the page wires |
| Feature modules, feature LiveComponents | Features | product-knowing, and wired by the page |
| Generic function components (`core_components.ex`, `<.button>`, `<.table>`) | Domain Abstractions (UI) | product-free, reusable, configured by attrs and slots |
| A generic shell, effect interpreter, or runner (V35/V36's `EffectInterpreter`, a `use PageShell` dispatcher) | Programming Paradigms | an execution model: it knows how a kind of connection runs, not what this page does (Spray keeps "Execution models.doc" in this layer) |
| `Phoenix.LiveView`, `Phoenix.Component` | Programming Paradigms / Foundation, library-provided | LiveView's event → state → render loop is an execution model; a page implementing its callbacks is configuring a more general module (R9) |

Two consequences:
- **A generic shell sits *below* the page, not above it.** When it calls back into the page's
  callbacks, that's the passed-in callback case (R1, R9), not an upward edge. A page that uses the
  shell is dropping.
- **Generic view components aren't application code** even when only one page uses them so far.
  If a component would read the same in another product, it's a domain UI abstraction. A component
  that knows this page's data or events is part of the page.

**Getting the application right in a LiveView app.** This is where most of an Elixir app's ALA
quality is won or lost, because the page is the one place that is allowed to know everything.
Each point states a property first; what follows it is one way to get there.

- **Nothing below the page names a page.** Generic execution machinery is product-free and sits
  below the page (§2.9.2, §4.2). For example, a shell that interprets outcomes or effects (flash,
  stream insert, timer, patch) is a generic execution model, so split it: the interpreter goes into
  the Programming Paradigms layer, shared by every page, and the part that names this page's
  composition (dispatching an event to `CartPage.handle/3`, say) stays in the page.
- **Keep the page to wiring and configuration.** The page's callbacks connect events and data to
  features and domain abstractions and carry the application literals (R3). They don't compute,
  store, or decide (R11; §3.5). A `handle_event/3` that runs a feature and hands its result to an
  interpreter is wiring. One that works out a total is not.
- **The template is the UI diagram.** Nesting in the page's HEEx is the "display inside" wiring
  (§1.6.6, §4.12). A page-level `:if` or `for` is application logic. Sometimes it's unavoidable, and
  sometimes it's a missing component or a missing feature. Ask which (R11).
- **Push product-free markup down.** A component that would read the same in another product is a
  domain UI abstraction, even with one caller today (R7 asks whether it earns its existence).
  Keep only the markup that knows this page in the page.
- **Keep symbolic connections inside the page.** An assign name that the wiring writes and the
  template reads is Spray's `temperature` connection (§1.6.6). It's fine inside the page, where both
  ends live. A name that features or domain modules must also know goes through R5.
- **Features are wired by the page, not by each other.** Feature modules and feature
  LiveComponents are Features-layer instances. The page gives each one its inputs and routes its
  outputs (§7.15; R1, R5). A feature LiveComponent that reaches for a sibling's state
  or a named topic breaks R1 or R10.
- **Subscriptions and timers are set up by the page.** `mount/3` subscribes and routes messages to
  features, or passes a topic down as configuration (R1, §4.4.2).
- **Generated glue is application code.** If a manifest or diagram generates the page's wiring, the
  generated code belongs to the application, like Spray's generated wiring code, which "lives in a
  subfolder from where the diagram is, because it is not source code" (§2.5.1). The generator is a
  tool, outside the app's layers.

**`if`, `case`, and assignments in a LiveView page.** R11 gives Spray's reading, and the moves for
each kind of `if`. Here is how that applies to the forms a page actually contains. The
"wiring" verdicts on multi-clause handlers and `case` routing are this checklist's Elixir reading,
not Spray's words.

| Form in the page | What it is | Ways to fix, for example |
|---|---|---|
| Multi-clause `handle_event/3` or `handle_info/2`, one head per event | routing: each clause is a wire from an event to an input | keep each clause to forwarding |
| Decoding params (`%{"id" => id} = params`, `String.to_integer/1`) | adapting the browser's event format, a technical domain wired sideways (§7.17) | fine when small; if it repeats, a generic param-casting abstraction configured per event |
| `assign(socket, :x, value)` where `value` is an abstraction's output | the point where dataflow lands (§1.6.6) | nothing |
| `assign(socket, :total, Enum.sum(...))`, or any arithmetic in the page | data handling | move it into the feature or domain abstraction, and assign its output |
| `if` or `case` on `nil`, or on `{:ok, _}`, just to decide whether to go on | a propagation guard | `with`, or a runner that stops on `:quiet` or an error (§6.1.3) |
| `case` whose arms only send the ok and error results to different places | routing two output ports (read from §4.9.1) | acceptable as wiring; better still, the feature returns outcomes that an interpreter routes |
| `if`/`case` with computation in its arms, or a business rule (`if total > 100, do: free_shipping`) | application logic | a domain abstraction, or a predicate configured at the page (§3.11.3); a state machine if it depends on history (§4.16) |
| Several feature calls in one clause (remove the item, then record the undo) | fan-out wiring, written in order | fine when each call only passes outputs to inputs; say so when the order matters (§4.4.4) |
| HEEx `:if={@flag}` on one boolean that a feature produces | a "display when" wire | fine |
| HEEx `:if={length(@items) > 0 and @user.admin}` | application logic in the template | compute it in a feature and wire the boolean |
| HEEx `for` over rows in the page | iteration in the application | a generic list or table component that does the iterating (§4.12; §4.8.2 for tables) |
| `connected?/1` in `mount/3`, auth redirects | the framework's lifecycle | a departure: keep it to one line, or move auth to an `on_mount` hook, which is LiveView's own extension point |

The test for the page as a whole is Spray's: a reader should find every requirement as an
instance, a literal, a predicate passed in, or a wire, and no step where the page computes or
decides for itself (§3.5.2). Framework-imposed branches are recorded as departures, not argued
away.

**For the linter:** declare the application layer `peer_ok: true` (its internal calls aren't
flagged), and keep feature and domain layers `peer_ok: false`. A human applies the same rule: a
same-layer call between two different abstractions is a violation *below* the application, and
normal *inside* it.

- **Application literals live in the composition/config tier.** R3 exempts *config layers* from the
  literal check — by default the top layer, or any tier you mark `config: true` (so a manifest
  module standing in for the diagram can hold the application's configuration). A human mirrors
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
- **Calls *within* the application don't add height.** The application is one abstraction, so
  when computing abstraction height, a reader (and the linter) counts it as **1**, and height
  accrues only once a chain drops *below* it into features, domain, or paradigms. Proliferation
  *below* the application still counts fully. (The linter's `app: true` marks which layer that is;
  the top layer is the default. Marking several layers `app: true` to model shell → page → view
  sub-layers is the old model this section replaces.)
- **The view/template layer hides contracts and calls.** `~H` markup is opaque to the AST, so
  cross-feature calls and server↔DOM contracts embedded there escape function-level R1 and R5. The
  linter's reference-level R1 advisory, and template checks such as the variants' V32
  `ComponentPurity`, partially cover this; a human
  running the checklist must **read the templates** — this is the single biggest thing the linter
  cannot see and the human necessarily can.

### How close a LiveView page can get

The table of page forms above sorts them. This part records what the variants found when they tried to
remove them: which departures the framework really forces, which only look forced, and what the
designs that reach zero cost. It is the checklist's Elixir reading throughout.

**What LiveView hands the page.** `mount/3`, twice (once for the static render, once when the
websocket connects); `handle_event/3` for every browser event, named by a string with string
params; `handle_params/3` whenever the URL changes, including browser back; `handle_info/2` for every
message to the process; `handle_async/3` when a `start_async` job finishes; and `render/1`. The
page's state is its socket's assigns, which are immutable, and LiveView tracks changes per assign
key. Some of this is Spray's model under another name. Assigns are where the dataflow lands (his
`temperature` connection, §1.6.6). The template's nesting is the "display inside" wiring. One
process per page is the single-threaded execution he prefers anyway (§4.4.9). Other parts are
foreign: string-named events, a URL that can change under the page, and two mounts.

**Forced by LiveView or Elixir, and valid.** Keep these small and never use them for logic.
- *Event routing clauses.* The framework calls `handle_event("remove_item", params, socket)`, so
  something has to match on the string. One clause per event, each only forwarding, is one wire
  from an event to an input: pattern matching doing the job of Spray's port names. V41 shows the
  routing can move into feature components (`phx-target={@myself}`), but some module still matches
  the string.
- *Decoding params.* `String.to_integer(id)` adapts the browser's format, a technical domain reached
  sideways (§7.17). If it repeats everywhere, a generic param caster configured per event takes it
  out.
- *Holding the program value.* Elixir's instances don't change in place, so the page stores each
  feature's new state back after every step. That's Elixir's departure, not LiveView's, and it isn't
  "handling the data" while the page never looks inside what it stores (R4).
- *HEEx instead of wiring code for the UI.* HEEx is a better notation for the same "display inside"
  wiring Spray writes with `WireTo` nesting. Moving the UI tree into a wiring table would be
  reinventing a worse HEEx.
- *Assigns shaped for change tracking.* A page does better with `@summary` and `@saved_count` as
  separate assigns than one struct holding both. Those are named landing points, Spray's symbolic
  connections kept inside one abstraction; R5 accepts them while no feature needs the names.
- *URL steps belong to the page.* `handle_params/3` delivers the URL, and LiveView raises if a
  component calls `push_patch` inside `update/2`. So the page owns the table from flow step to path
  (V41's checkout panel announces `{:checkout, :step, step}` and the page patches). That agrees
  with R3: paths are application literals.
- *A few lines of client JavaScript.* Focusing a field on mount can't be done from the server. Most
  variants from V28 on ship an `AutoFocus` hook in its own file, attached from the template.

**Forced by LiveView, but it moves out of the page.**
- *`if connected?(socket)` in `mount/3`.* A generic `on_mount` hook in the Programming Paradigms
  layer, configured on the page, holds the guard once, below the page (V39, V41):
  `on_mount {ZeroCoupledWeb.Paradigms.Subscribed, {ZeroCoupled.Foundation.Broadcast, :subscribe}}`.
  That's Spray's first move: the guard goes into the connection mechanism.
- *Auth redirects.* Same shape, same fix: `on_mount` is where Phoenix already puts auth. Untried
  here, since no variant has auth.
- *`handle_async/3` results.* A payment job returns `{:ok, {:ok, url}}`, `{:ok, {:error, reason}}` or
  `{:exit, reason}`. V39 routes them to `Checkout.succeeded/2` and `Checkout.failed/2` with three
  clause heads, Spray's two output ports (§4.9.1). V40's `ToAsync` sink and V41's checkout panel own
  the job and its outcome, so the page doesn't see it.

**Not forced: habits the variants removed.** Arithmetic and computed assigns (the zero-coupled
variants from V28 on compute totals in a feature). Business rules in handlers (V39's
`Checkout.start/2` returns `blocked: :empty`, which the page binds to a flash). History in
`handle_params` (V33's transitions table; V39–V41 pass it in as configuration). Handing values from
one feature to another (V35 passes them in; V40 makes it a wire). Loading rows in `mount/3` (V40
wires a store source into the feature's `:load` input; V41's component loads itself). `for` loops
and compound conditions in the template (a generic list component; a feature computes the boolean).

**What remains in a page that reaches zero.**
1. routing by event name and URL, which Elixir does with pattern matching instead of port names;
2. decoding browser params, a technical-domain adapter at the edge;
3. storing the program value back, because Elixir is immutable;
4. a lookup table from URL to step (`Map.get(@steps_by_url, params["step"], :address)`), because the
   framework hands the page the URL;
5. a JavaScript hook or two for things only the client can do.

None of these is logic, and none has a Spray move that removes it without replacing it with
something equivalent. Two smaller residues from V41: a template comparison
(`disabled={@item_count == 0}`, which should be a boolean the cart announces; the linter doesn't
read templates, so only a reader finds it), and hidden subtrees (`send_update` to an unmounted
component raises, so the cart's instances stay mounted and hidden behind the checkout screen).

**R11 findings by variant** (`ala_lint --super-strict`, which doesn't count routing clauses, `with`,
ok/error routing, or the `connected?` guard):

| Variant | R11 findings | What's left in the top layer |
|---|---:|---|
| thermometer §1.6.3 | 3 | two nil guards and handled data, the things Spray flags in his own §1.6.3 |
| thermometer §1.6.4 onward, including §1.6.6 in LiveView | 0 | a runner moves the data; the page holds one program value |
| coffee maker | 4 | its user stories still branch in the composition |
| V36 | 3 | a plain composition that branches in a few places |
| V38 | 8 | branching `handle_event` clauses and handled data, kept on purpose (its charter was to evolve a conventional app without adding indirection) |
| V35 | 4 | URL and sequencing branches from V33's flows |
| V39, V40, V41 | 0 | routing clauses, param decoding, a URL lookup, and `with` |

**What reaching zero costs.** Each design is something a team has to learn and keep, which is why
R11 stays in the linter's strictest tier.
- *V39, bound ports:* a binder with thirteen kinds of binding and one map per page. An unbound port
  is silently dropped, so it wants a test that every port a feature declares is bound. Familiarity
  4 of 5.
- *V40, circuit instances:* Spray's diagram run as data. A click takes two message hops (component to
  page to circuit to component), and the cart's circuit has 38 instances and 41 wires. Familiarity 2.
- *V41, feature components:* the shape `phx.gen.live` already gives you, so nothing new to learn.
  Cross-feature effects take two message hops, and the design lives in about twenty `handle_info`
  clauses instead of one table. Familiarity 5.

The linter scores all three within a point of each other (97 under `--strict`). What separates them
is what it can't see: readability, hops, and vocabulary.

**One placement held loosely.** V41's feature panels are LiveComponents placed in the Features layer:
the page configures and wires each one, and none names the page. But their markup is shaped for this
page's layout, and a reviewer could fairly call that part application. If that reading wins, split
each panel into a feature instance that handles events and outputs, and page-owned markup passed in
through slots. Untried.

**Does stopping short pay?** Mostly, yes. Spray's own history says so: "The turning point was when I
noticed two (accidental) successes in parts of two projects" (§1.5.1). Those parts had "undergone
considerable maintenance", and "their simplicity had never degraded", like "two pieces of metal that
had never rusted" at a rubbish dump, inside projects that weren't ALA. His state-machine diagram that
replaced 5000 lines of C "was easy to maintain for years to come" (§4.16), and his advice for legacy
code is "Conversion of user stories takes place iteratively" (§8.3). Most benefits are local to one
abstraction's boundary: an abstraction whose own dependencies all drop can be read and changed
alone, whatever its neighbours do. A hoisted literal, a feature that stops reading a sibling's
state, or a guard moved into a runner each pays on its own (V38 got real value this way). Some
benefits need the whole page, though:
- *The page reads as the requirements* (R8) only when all of it is wiring. One handler that computes,
  and a reader has to check every handler.
- *The diagram is the source* only when all the wiring is in it (V35's drift check, V39's bindings
  map).
- *"No abstraction knows a peer"* is a guarantee only with no exceptions. One peer call and you're
  back to reading callers.
- *An abstraction's own isolation* is all-or-nothing at its boundary.

A page with a few forwarding clauses and a URL lookup loses nothing, because those forms aren't
logic. A page that keeps some real logic still gets the local benefits of every abstraction below
it. What it gives up are the whole-page properties above.

### Running the checklist by hand to compare with the linter

The linter's method is a mechanical shadow of the manual checklist, so a reviewer can reproduce and
extend it. The correspondence: (1) give every function its `[tag]` — the linter approximates this
with the layer map's convention + `@ala_layer` tags (its **coverage** = how many you'd have tagged);
(2) walk every call edge and confirm it **drops** — the linter scores this on the function graph and
flags reference/template edges advisorily; (3) check application literals sit at the composition (R3),
state is owned, not hidden (R4), no silent contracts (R5), names/earns-existence hold (R6/R7).
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
- **R4 (state owned, not hidden).** *Automatable:* the process dictionary; some `Agent`/`:ets`
  stashing. *Human must judge:* whether GenServer/process state is legitimate instance state or a
  hidden cross-call channel whose state belongs to some concept's abstraction (or to a wired
  `State<T>`) — a semantic distinction.
- **R5 (no silent contracts).** *Automatable:* duplicated identifier strings across modules;
  (extendable to) tagged-tuple message shapes. *Human must judge:* contracts that are **invisible
  to the AST** — anything inside `~H`/templates/EEx, cross-language constants (server string ↔ JS),
  or two ends that agree via *different* literals (a produced format parsed elsewhere). The tool
  sees textual duplication, not semantic agreement.
- **R6 (nameability).** *Automatable (weakly):* single-letter/`f\d` names, functions that wrap one
  primitive. *Human must judge:* the actual rule — "does this name a **learnable concept**?" A
  meaningful predicate (`empty?`) trips the primitive-wrapper heuristic (false positive); a
  meaningless-but-plausible name (`process2`, `handle_stuff`) passes it (false negative). Naming
  quality is irreducibly judgement. *Also automatable (advisory):* the "should" about tramp
  parameters, as `ala_lint`'s `tramp` check (a public function carries a parameter unread through
  another module's function that doesn't read it either). *Human must judge:* whether the carrying
  function is really a connection mechanism whose job is carrying, and whether the lower abstraction
  could be configured or wired from the composition instead.
- **R7 (earns its existence).** *Automatable (advisory):* dead code; trivial single-use one-liners;
  abstraction height past a ceiling. These are reported but **not scored by default** — reuse is
  evidence, not a requirement (see "Helper proliferation" above), so the tool never fails code on
  them unless the team opts in (`--enforce r7`/`--enforce height`). *Human must judge:* the core —
  "does this hide a decision worth naming?" A single-use named step can be excellent decomposition
  or premature extraction; only a reader decides, or a second consumer does (does the abstraction
  wire into a second, unrelated caller unchanged?).
- **R8 (reads as the requirements; names/config/shape serve the reader).** *Not automatable at
  all* — pure judgement. "Does the composition read as the spec?" and "are these good names?" have
  no shape-based decision procedure. R8 is the honest boundary: a linter can *prompt* it, never
  score it.

Two whole-design properties **no single-snapshot linter can check** (they need more than the code):
**reuse** (does an abstraction wire into two or more consumers unchanged? That needs a second
consumer to try it with) and **requirements coverage / correctness** (does the wiring
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
the tramp-parameter check (the encoding doesn't carry parameters),
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
sampler and display stages), and the application layer is 33% of functions (2 of 6). The linter
scores both only under `--super-strict`, so the default score stays 100/A. That's a statement about
the linter's tiers, not a clean bill: by Spray's reading these `if`s are application logic. He
flags the same `if` in his own §1.6.3 version, and it disappears at §1.6.4 when `SampleEvery`
simply emits nothing. So the finding is real. The fix is to move the guards into the connection
mechanism ("Past the pipe"), not to clear them as harmless. The `{app-literal}` on `new/1` is
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
  mechanics, tolerated at the boundary; mark it and say why, don't hide it. This is C-specific: an
  Elixir port of the thermometer reads readings as values, so it has no `*p1`.
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
handles the data at runtime" (§4.17). Rewriting the loop as a pipe that threads the leaves' state
(`p |> f4(4, 8.3) |> f5(s5, 10) |> ...`) doesn't get there. The app still handles every value, and
now handles the leaves' state too. The pipe is a restyled §1.6.3, not ALA's endpoint.

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
   language feature, not a design philosophy" (§3.10). Whatever form it takes, writing a new domain abstraction must
   stay easy: "It is necessary for developers to be able to write new domain abstractions, so this
   needs to be easy" (§1.6.5).
3. **Several kinds of connection in one app (§1.6.6).** "Monads usually only support dataflow."
   A real app also composes UI, events, and state-machine transitions, and "different lines in our
   diagram have different meanings". For UI the lines "mean 'display inside'". When the composition
   becomes a graph, the diagram is the source.

What this means in Elixir. These are options, not a required form; how `ala_lint` should check
these goals is still open.

| Goal | Some ways to meet it in Elixir | Where state lives |
|---|---|---|
| build, then run; no data in the app | a paradigm protocol (`push(step, data) :: {:emit, out, step} \| {:quiet, step}`) with a composition value and a runner; or `Stream` stages, each a configured `Stream -> Stream` function, then `Stream.run/1` | inside each stage (a struct, or a stream accumulator) |
| instances with paradigm ports | a pure graph value (named instances plus wires) and a runner; processes with a wired output port where the concept is concurrent | inside each instance |
| several kinds of connection | in LiveView: HEEx nesting is the "display inside" wiring; an assign is where dataflow lands; a generic runner or effect interpreter (a Programming Paradigms layer execution model) pushes results into assigns, so the page never computes them | the program value, held as one opaque assign |

Two Elixir-specific notes:

- **Holding the program isn't handling the data.** Immutable data means the top-level owner (a
  LiveView's assigns, a GenServer's state) keeps the program value and stores the updated one after
  each run. That's Spray's `program` variable. The app never looks inside it.
- **Don't organise by process.** A process per domain abstraction is the most literal translation of
  Spray's objects, but Elixir guidance treats "code organization by process" as an anti-pattern.
  Prefer pure composition values and runners, and use processes where the concept really is
  concurrent.
- **A `Stream` is a monad chain, with a monad's costs.** Each domain abstraction as a configured
  `Stream -> Stream` function reads almost like Spray's §1.6.4, and it's standard library. But it is
  one paradigm and one line, it's pull-driven, and one `Stream.run/1` consumes it, so it suits a batch
  or a source you can iterate, not a LiveView that receives one reading at a time. Spray's ALA builds
  the program once and runs it forever (§3.11.3); in Elixir that's a composition value built in
  `mount/3` or `init/1`, held by its owner, and fed each input ("Functional programming", point 8).
- **A graph value is a text form of the diagram.** Named instances plus a wire list allow fan-out
  without processes, and the wiring is data you can print, check for unwired ports, or draw
  ("Functional programming", point 9).

In the notation: the §1.6.4+ app shows no `pN` wires at all (the runner carries them), no branch, and
no leaf state. A composition line lists instances, their literals, and how they connect.

### Kinds of connection and how they run *(in development, not yet a rule)*

> **Status:** this section gathers Spray's thinking on composition techniques (his chapter 4, and
> §2.3–2.4, §3.6, §3.11) so that rules can be drawn from it later. Nothing here is scored, and the
> Elixir notes are tentative. It replaces an earlier claim that a compliant tree is one you can
> rewrite as a pipe, which was too narrow.

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
`Stream` pipeline or a `with` chain in Elixir) compose functions with one input and one output,
"like discrete electronic components such as resistors". Domain abstractions are more like
integrated circuits, with many pins of different kinds (§3.11.3). A monad chain can still live
*inside* a domain abstraction, as an adapter with ALA ports.

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
  steps. Elixir equivalent: a GenServer or LiveView callback should return promptly, and slow work
  goes to a `Task`, `start_async/3`, or a message to self.
- *Where instances run is a late decision.* Because asynchronous events don't care where the
  receiver is, "the physical view can be changed independently of the logical view" (§4.7.3). On
  the BEAM, a process can live on another node without its senders changing.
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
coffee maker and in FRP (functional reactive programming). Clocked means every instance latches its inputs on a tick. The data type is
never a DTO shared between two abstractions. It comes from a lower layer or is passed in by the app,
and ideally is inferred along the wire.

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
like a global, which hurts parallel tests and per-instance overrides. (Elixir equivalent: a theme or
settings module every component reads, or application config.)

#### Elixir notes (tentative)

- **Port types.** A port's type is owned by neither side and sits below both (R9). In Elixir that is
  often a protocol or behaviour in the Programming Paradigms layer (`Step` for push dataflow, a
  `Stream` stage for pull). It can also be a function passed in, a pid the composition supplies, or
  a message shape the paradigm defines.
- **LiveView already contains two of Spray's paradigms.** HEEx nesting is UI layout, with native
  fan-out where the order of the children is the layout order: Spray's `IUI` (his UI-layout port
  interface). An assign behaves like
  *live* dataflow: the template always sees the current value.
- **Sync or async is `call`, `cast`, or plain function call.** It should be chosen by whoever wires
  the instances, not buried inside a domain module that calls a named process.
- **PubSub topics.** A topic hardcoded in a domain or feature module is a global event name. If the
  composition owns the topic and hands it down, it's a wire.
- **Hot push and demand-driven pull.** Spray notes that "If you are using monads, especially I/O
  monads, or RX (reactive extensions), especially with hot observables, you are already using the
  wiring pattern" (§7.5). Phoenix PubSub is hot push: messages flow whether or not anyone asked.
  GenStage and Broadway are pull with back-pressure: consumers ask for demand. That fits Spray's own
  reasons for pull (lazy or expensive sources, §4.4.3), so it's a choice made at wiring time, not a
  departure from his push default.
- **Writing the kind of wire (a suggestion).** The notation has one kind of wire, `pN`. Where it
  matters, a suffix can say which paradigm a wire carries, such as `p2:event`, `p3:inside` (UI
  containment), or `p4:transition`. Spray draws different line meanings in one diagram (§1.6.6) but
  doesn't prescribe a text mark, so this is the checklist's suggestion. The encoding linter doesn't
  read it yet.
- **Intermediaries are instances too.** A fan-out, a buffer, or an ordering step belongs in the
  composition value (a `Chain`, a graph) like any other domain abstraction.
- **State machines.** A pure transitions table (like V33's flows channel) is one candidate for the
  state machine paradigm without `:gen_statem`.
- **Several inputs of the same kind.** Spray spends a section (§4.4.5) working around C#'s rule that
  a class can implement an interface only once, so an AND gate can't have four `IDataFlow<bool>`
  inputs. Less relevant in Elixir: ports can simply be named, such as a map of port name to wire, or
  a message tagged with the port (`{:push, :input2, value}`).

#### Candidate checks (not yet rules)

- A domain abstraction that decides sync or async, or push or pull, for its caller (for example,
  `GenServer.call` on a named process inside a domain module).
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

**Starting from an existing app.** Don't adopt a whole variant. Adopt the checklist, run `ala_lint`
against what you have, and let the findings send you to the technique lists under each rule. Most
well-structured Elixir apps are already close on R1, R2, and R4, because immutability and ordinary
module boundaries give you a lot. The work that remains is usually R3 (get the constants to the
top), R5 (name the contracts, and keep the names in the page), and R11 (sort the page's `if`s and move
the ones that aren't routing). Reach for the heavier techniques (typed outcomes, a manifest,
committed codegen, a composition value with a runner) only when a rule you care about isn't holding
by discipline alone. The lightest technique that meets the rule is the right one.

## Where even the good shape can lie (keep these in the guidance)

1. **Tags rot.** A `[]` leaf acquires a product constant in maintenance and nothing in the call
   tree changes. R3 is the tripwire — re-scan leaves for literals, not just edges. (Mechanized
   at scale, this is exactly a `CorePurity`-style check.)
2. **Contract coupling is invisible to call trees.** Two `[]` leaves agreeing on a format/name/
   topic (`q`) are peers in disguise; only R5 catches it. (Mechanized: `ContractPurity`.)
3. **Callbacks and pub/sub hide who set up the wire,** and indentation shows neither case. A leaf
   invoking a handler is fine when the composition passed that handler in: it's a port. It's a
   defect when the leaf found its partner itself, by registering with a sender, subscribing to a
   topic it names, or looking up a process by name. Write `^f` for a lookup of something higher, and
   treat a self-registration between peers as a peer edge. The fix is always "the composition wires
   it," never "the leaf knows whom to call."
4. **A perfect shape can encode the wrong program.** The encoding audits *structure*
   (coupling/knowledge placement), not correctness — same caveat the requirements-coverage
   analysis makes for manifests.

---

*Origin note: this file started as a numbering sketch of the two trees; the worked forms above
fix the f-numbering against Spray's actual §1.6 code and add the `[tag]`/`$`/`*`/`q` marks —
without the tag column, the bad and good trees are nearly the same shape, which is what the first
sketch ran into. The "Past the pipe" section follows Spray's §1.6.3–1.6.6 ladder, and its Elixir
options come from working the same four domain abstractions under a `Thermometer` composition
each way. The `q` rule and the mechanized-check parallels
come from the V32 design work.*

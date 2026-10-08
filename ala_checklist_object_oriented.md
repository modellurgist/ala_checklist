# The ALA Checklist (R1–R11): object-oriented edition

*(A concise encoding for spotting non-ALA code, plus the procedures to build it and verify it — refer
to it as "the ALA Checklist".)*

> **This edition** is for object-oriented languages, statically typed (C#, Java, C++, Kotlin) or
> dynamically typed (Ruby, Python, Smalltalk). It has the same rules (R1–R11), notation and
> citations as the functional edition, read for classes and objects, plus what Spray says about
> object-oriented programs in particular. Spray writes nearly all his examples in C# with classes, so
> much of his site *is* the object-oriented reading: "ALA is therefore highly object oriented. It is
> object oriented programming as it should have been" (Summary). Most of the OO material is gathered
> in "Object-oriented programming: what Spray says, and what it means in an OO language" below, and
> each rule carries an OO reading of its own. Where Spray is specific to C#, this edition says so and
> gives the general OO form.

> **If you care only about zero-coupling (not full ALA):** this checklist bundles three commitments:
> zero-coupling, the abstraction hierarchy, and requirements-on-the-diagram (the top layer reads as
> the requirements). The coupling part alone compresses to a smaller instrument. Two peers can be
> coupled in only three ways: one names or calls the other, they share mutable state, or they agree
> on a name or format that neither signature shows (a silent contract). One rule covers all three:
> no such channel connects two peers except through a lower layer or the composition above them.
> It can be used on its own, but it isn't sufficient: a program can be perfectly zero-coupled and
> still be *ugly*, which is what the nameability (R6) and earns-its-existence (R7) rules add on top.

A text notation for classes, methods and their relationships, small enough to type on a
whiteboard, that makes ALA violations *visible as shapes* instead of judgment calls. Worked against
Spray's thermometer example (site §1.6: the "bad code", the refactor toward ALA, and the version
"composing with plain objects", §1.6.5), with the object-oriented reading he develops throughout
(gathered under "Object-oriented programming" below)
([composing with plain objects](https://www.abstractionlayeredarchitecture.com/#truecomposing-with-plain-objects),
[ALA vs OOP](https://www.abstractionlayeredarchitecture.com/#trueala-compared-with-object-oriented-programming)).

> **Citations.** Section numbers such as §3.8 or §1.6.4 refer to John Spray's site,
> [abstractionlayeredarchitecture.com](https://www.abstractionlayeredarchitecture.com/), which is
> one long page with numbered sections. "Summary" is its unnumbered opening section. Each rule ends
> with a *Spray:* line naming the sections that support it, so you can check the rule against the
> source. Where a rule adapts Spray's C# to object-oriented languages in general, or departs from
> him, the line says so.

## The notation

A handful of marks; everything else is indentation.

```
name [tag] [@lvl]  a class or a method. `name` is its real name — fully
             qualified Namespace::Class#method (or Class.method) at scale, so it
             stays unique and grep-able — or a bare fN on a whiteboard. A class
             line holds its methods indented under it. [tag] = the most specific requirement noun
             it KNOWS (its name, constants, field names). [] = fully generic:
             nothing in it would change if the product changed. [@name-Lidx] is
             OPTIONAL: the ALA tier it is assigned to (top = L0), tool-assigned
             from a layer map (a declared list that assigns modules to layers; see
             "Assigning functions to layers"). The tag is WHAT it knows; the level is WHICH tier;
             they should agree, and a mismatch is itself a smell. R1 is checked
             on levels; on a whiteboard the tag stands in for them.
pN           a data value (a wire). p2 <- Filter(p1) : derived by a call.
*pN          shared/by-reference data (caller and callee both hold it) — an R2 channel.
             In an OO language this is the common case, not the exception: a mutable
             object two peers both hold a reference to (passed to both, or returned
             by one and kept by the other), a static field or class variable, a
             singleton, or a session or request object two features read and write.
$            the class or method keeps state between calls. On an instance whose
             concept is stateful (a filter's history) that's normal object state;
             the mark matters where the state is hidden or shared: a static field,
             class variable or singleton, or a field on a class whose concept isn't
             that state. Write it on the tag: [x]$.
qN           a silent contract: a format/name/shape two functions must agree on
             that is NOT visible in any signature.
{app-literal?}    an application-literal candidate lives in this function (value
             opaque; the linter has its file:line). A human/LLM resolves it to:
{app-literal}       an application literal — belongs at the composition (R3 hoist if
               it sits lower; at the top it is correct).
{intrinsic-literal}    intrinsic to the abstraction (an identity, a physical or
               mathematical constant) — correctly local, never hoisted.
(p) ->       an anonymous wiring lambda (a lambda, delegate, block or proc). No
             name: it is composition, not an abstraction. (Spray's "func1" point: in
             §1.6.3 he names a helper func1 because it has no meaning of its own; it
             only connects two points in the code, so it shouldn't be a named function.)
indent       "uses" — a dependency edge from the line above it. In OO code that is
             a `new` of a class, a call to a class or static method, a field or
             parameter typed by a class or interface, implementing an interface, or
             inheriting from a class (R1 and R9 judge each kind).
ports:       OPTIONAL, on a class line: its ports, `provides IX` for an interface
             it implements and `accepts IY name` for a field of an interface type
             that the wiring fills (Spray's word is "accepts", not "requires",
             §6.14.2). Each IX/IY should be a programming-paradigm type (R9). A
             suggestion of this edition; Spray draws ports on boxes, not in text.

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

**In an OO language, an abstraction is usually one file holding one class.** "In the absence of
direct computer language support for abstractions, ALA generally implements abstractions as source
files (like modules in C). The file usually contains one class, but may contain a small number of
classes, interfaces, enums, delegates, typedefs, functions, etc." (Summary; §2.3.2). A class is not
automatically an abstraction: "Classes, Modules, functions and encapsulation are artefacts of the
language… They are not necessarily abstractions" (§7.2.1). The class is the abstraction only when it
names a learnable concept (R6); the object is its instance (§2.3.3).

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
    `^f` when it's indirect but still found by the lower class itself: a hardcoded higher class, a
    singleton or service locator it asks, a registered name, a topic it knows. By contrast, a lambda,
    object, or handler that the composition passes *down* is not an upward edge: it's a port, and it's
    Spray's legitimate way to call up the layers ("executing a lambda expression that has previously
    been passed in", a callback, the observer or strategy pattern, §4.4.1). In OO terms: a lambda,
    delegate or block passed in as configuration, a strategy object the composition supplies, or an
    observer subscription the composition sets up.
  - **Not virtual methods.** "We don't use virtual functions in ALA for up calling because we don't
    need or want to use inheritance" (§4.4.1). A base class in a lower layer that calls an abstract
    or overridable method a subclass fills in is the inheritance form of calling up; ALA uses the
    passed-in forms above instead (§2.1.3, "Something inheritance provides is a mechanism for 'calling
    up the layers' at run-time through its virtual functions. In ALA, we do this with ordinary observer
    (publish/subscribe) pattern (events in C#), or by passing in a method as a configuration (usually
    anonymously or as a lambda expression), or by using the strategy pattern"). See R9 for the one
    interface-shaped exception, configuring a far more abstract framework.
  - **Who sets up the callback decides.** Set up by the layer above: legal. Set up by the receiver
    itself is a peer edge in disguise, and forbidden. That covers a receiver registering with a
    sender, or subscribing to a public event: "receivers never register themselves to a sender, or
    to a public event" (§4.4.2). Between peers the observer pattern only reverses a dependency that
    ALA doesn't have, so ALA wires instead. The one place Spray keeps an observer is *inside* a
    paradigm interface, for traffic running against the wire's direction, where "the subscriber
    does not know the publisher". In OO terms:
    - A class that subscribes its own handler to a peer's event (`button.Click += this.OnClick` written
      inside the receiver) is the defect. C# events are "usually registered by the receiver itself
      (observer or publish/subscribe pattern). In ALA of course, events must be wired by a layer
      above" (§4.7).
    - A feature or domain class subscribing to a publish/subscribe topic or notification name it
      hardcodes is the same defect. The composition subscribes and routes, or passes the topic down as
      configuration.
    - A handler in the composition that receives the event and hands it on is fine.
    - Asking a service locator, a dependency-injection container, a singleton, or a registry for a
      collaborator is the class naming its destination (R9). The composition should give it a port
      instead.
  - **A field or parameter typed by a peer is an edge, even when injected.** "Simple dependency
    injection or otherwise passing an object into another object doesn't remove the association
    between their respective classes… in ALA you are not allowed to know about a class in the same
    layer, not even its interface. Not even a base class" (§2.1.3). Injection changes who *chooses*
    the instance, not what the class *knows*. The field must be typed by a programming-paradigm
    interface from a lower layer (R9).
  - **A peer created with `new` is an edge.** A domain class that constructs a peer itself (Spray's
    `Oxygen` that does `new Hydrogen()`) has the composition relationship that destroys both
    abstractions: "By implementing the Oxygen-Hydrogen relationship needed to make water in oxygen, we
    have destroyed the oxygen abstraction" (§7.5). The `new` belongs in the class above (`Water`).
    Creating instances of *lower*, more abstract classes (a `List`, a `Regex`, a paradigm's
    intermediary) is an ordinary knowledge dependency.
  - **Inheritance across a boundary is an edge, and usually the wrong one.** A subclass knows its
    parent: when the parent is a peer, it's a peer edge; when the parent is lower, the subclass still
    "would break the abstraction of the (more abstract) base class in the lower layer" (§2.1.3). ALA
    composes instead (see "Object-oriented programming", point 7).
  - **Not edges:** calls *inside* one abstraction (a class and its private methods, or the small
    group of classes, enums and delegates in one abstraction's file, §2.3.2: "Internal to an
    abstraction, they interconnect with each other unconstrained"). They are the abstraction's
    inside, a little ball of mud it's allowed to have. A helper class that another abstraction could
    use is not inside, though: "Abstractions are never private" (§2.1.3), so it is a separate
    abstraction in a lower layer, or it's a nested sub-abstraction (R11).
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
    - *Give each class ports, and wire its objects from above.* Spray's own technique, and the one
      most of his site uses. A class sends on an output port, a private field typed by a paradigm
      interface, and receives on an input port, an implemented paradigm interface. The class above
      creates the instances and connects port to port (`new Switch().WireTo(new Light())`, Summary).
      Neither class names the other. "What OOP should have done is represent relationships between
      objects completely inside another class" (§6.14.5).
    - *Send a fact out of a port instead of calling a peer.* A class announces what happened on its
      output port rather than calling the sibling that should act on it. The sideways call never
      exists, which removes the temptation rather than policing it. What the output may say, and what
      it may not name, is R9's business.
    - *Return an outcome instead of calling a peer.* Where methods are composed directly, a method
      returns a description of what happened (a typed result object), and the composition decides
      what to do with it.
    - *Wire cross-feature work in one place.* Every join between features lives in the composition,
      in a function whose job is wiring. Features stay ignorant of each other, and the one module
      allowed to know both does the connecting. Spray's version: "features can also have ports and be
      wired together" by the application (§2.2). A wiring function that binds one feature's result to
      a name and hands it to another still handles data, which R11 counts; declarative wiring and a
      wired graph of instances remove that too.
    - *Make the wiring declarative.* A diagram that generates the wiring code, a manifest, or a
      routing table connects one feature's output to another's input. The routing lives in one place.
    - *Pass a lambda or object in where a peer used to be called.* When one method called a peer
      part-way through, "the second function will now need to be passed into it. The function
      parameter is also a port" (§2.3.6). It's also Spray's legitimate way to call up the layers
      (§4.4.1). The composition chooses the lambda or the object; the receiver never names the class
      that provides it.
    - *Let the composition subscribe.* The composition subscribes to an event source and routes the
      messages on, directly or through a small generic subscription helper in the Programming
      Paradigms layer that the composition configures.
    - *Move the `new` up.* A domain class that constructs a collaborator gets a port instead, and the
      class above constructs both and wires them (§7.5, `Water` wires `Oxygen` to two `Hydrogen`s).
    - *Replace a base class with composed parts.* Spray's vehicle example: invent the common parts as
      abstractions of their own and compose car, truck and tank from them; a behaviour such as `Drive`
      becomes a domain abstraction with ports (engine control, brake control) that the application
      wires, and "Most variations would be handled by a configuration interface (the public
      interface) on each domain abstraction" (§2.1.3).
    - *Enforce it in CI.* A static check given a layer map can fail the build on an upward or
      cross-peer reference, including domain → domain references, a field or parameter typed by a
      peer class, and a superclass in the same layer. It doesn't prevent the coupling, but it catches
      it the moment it appears.
    - *Count only bad dependencies.* Spray's suggested metric: coupling between objects (CBO) "does
      not distinguish between good and bad dependencies… If CBO counted only bad dependencies, it would
      be an extremely useful metric. Perfect software would require the metric to be zero" (Summary).
      A layer map is what lets a tool tell them apart.
  - **What doesn't meet it.**
    - A feature calling a sibling feature's method, including through a shared "service" class in
      the same layer.
    - A field, constructor parameter or setter typed by a peer class, or by an interface a peer owns,
      even when a container or the composition injects it (§2.1.3, §3.11.3).
    - A domain class that `new`s a peer, or inherits from one.
    - A lower class that finds its collaborator itself: a singleton, a service locator or
      dependency-injection container lookup, reading global configuration to pick a collaborator,
      sending to a name it knows, or subscribing to a topic or event it hardcodes.
    - An interface one feature defines for another to implement (see R9).
    - Product-knowing methods that do work and call each other (a working chain), even inside the
      application.
    - Choosing pull over push, or adding a callback, virtual method or observer, only to make a
      dependency point the "allowed" way. "Some communications that would naturally be a push have to
      be changed to a pull… we end up creating an indirection, such as a callback, virtual function
      call, or observer pattern (publish-subscribe). This indirection further obscures the already
      obscure communication flows through the system" (§3.4.5). In ALA push or pull is chosen per port
      on performance grounds, because wiring isn't a dependency.
  - *Composability test.* "If any two potential abstractions must work together, then they are not
    abstractions, they are just modules" (§3.2.3). Two classes that only make sense wired to each
    other have a fixed arrangement, whatever their dependency graph says.
  - *Spray:* Summary ("All dependencies are knowledge dependencies", "Good and bad dependencies",
    "Emerging layers", CBO); §2.1.3 (and "RIP the UML class diagram": associations, inheritance,
    injected peers); §2.2; §3.4 (and §3.4.6, knowledge dependencies reach all layers below);
    §4.4.1–4.4.2 (callbacks, no virtual functions for up-calls, registration); §4.7 (C# events);
    §6.14.5 (relationships between objects belong inside another class); §7.5 (`Oxygen`, `Hydrogen`,
    `Water`); §7.15 (features).
- **R2 — wires meet only at the top.** A `pN` may appear in two functions' bodies only if one of
  them is the composition that passed it. The same `pN` (especially `*pN`) inside two *sibling*
  subtrees means two peers share its meaning — the meaning belongs in the wiring.
  - *A wire joining many ports* is a smell of its own: "a new abstraction may be waiting to be
    discovered", like the ground symbol on a schematic. Spray's example is a game score that most
    instances interact with: make it a domain abstraction in the layer below instead of wiring
    everything to it (§3.6.1). See R10's aggregate case.
  - **In an OO language the rule bites hard.** Objects are mutable references by default, so passing
    one object to two peers, or keeping a returned object that its producer still holds, creates a
    `*pN` without anyone meaning to. Spray's target after his five steps from procedures to objects:
    "No class knows of the existence of state in any other class" (§3.9), and "both the configuration
    data, and the wiring data stored in an instance can be immutable. Only instances of abstractions
    that contain state data are mutable, and this is clear from very nature of the abstraction" (§3.9).
    His diagnosis of the conventional alternative: state "that does not belong to a specific
    abstraction… is the type of data we would stuff into a class anyway, and then have almost
    pointless accessor methods. The other classes then have harmful dependencies on these data
    classes. Furthermore, the dataflows through such a network of objects is completely obfuscated"
    (§3.9).
  - **Techniques that meet it.**
    - *Give each feature a private data structure.* Per-feature objects leave no shared mutable thing
      for two peers to fight over.
    - *Make state that belongs to no one a wired state object.* "In ALA, what we do is create a domain
      abstraction for such state. This abstraction has dataflow input/output ports… Such state objects
      are not globals, nor do they need to be passed around. Other domain abstractions do not even know
      about them. Instead they are wired to them by the user story abstractions using dataflow ports"
      (§3.9). This is `State<T>` (§3.10).
    - *Send values, not live references.* A port that carries a value object, a copy, or a frozen
      object can't become a back channel; one that carries a mutable object the sender keeps
      changing can.
    - *Make configuration and wiring fields immutable after they are set* (§3.9: "The fields are
      immutable after they are set"). A read-only field, `final`, `readonly`, or a frozen object.
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
    - A static field, class variable, singleton, or global registry that two features both read and
      write.
    - A feature reading another feature's slot of a shared session, request or composition object
      (the wishlist reading `session.cart`).
    - One mutable object handed to two peers, each of which changes it (a shared `Order` that the
      cart fills in and checkout updates).
    - A getter that returns a peer a live reference to internal state, which the peer then changes
      ("objects effectively know about the existence of another object's state and collaborate with
      that state. They reach into each other's data indirectly", §3.10).
    - A data class with accessors that several features hold and change (§3.9).
  - *OO-language note:* where a functional language gets this rule almost for free, an OO language
    makes it the default failure. A clean R2 says more about an OO design than about a functional
    one.
  - *Spray:* Summary ("Communication between instances of peer abstractions"); §3.8 (no data
    coupling); §3.9 (no class knows another's state; immutable configuration and wiring; the state
    abstraction); §3.10 (reaching into each other's data); §7.8.
- **R3 — application literals live at the composition line.** Application literals (product-specific
  constants: thresholds, prices, labels, formats) live in the composition, or in a manifest or
  config module the composition owns, never inside a `[]` leaf. Passing them as arguments on the
  composition line (`f4(p1, 4, 8.3)`) is the simplest form; a manifest, or a constant on the
  composition, are others. A literal buried in a leaf silently converts `[]` to
  `[thermo]` — the tag was lying. The contrast is an *intrinsic literal* (an identity or a physical/
  mathematical constant that is part of the abstraction's own definition), which correctly stays put.
  - *In an OO language* the composition line is the constructor call, its setters, or an object
    initializer: `new OffsetAndScale(offset=4, slope=8.3)`, `new Filter(strength=10)` (§1.6.5),
    `new Display(label: "Temperature")` (§2.1.3), `new StringConcat() { Separator = "," }` (§5.8.3).
  - **Defaults: convention over configuration, but not product decisions.** Spray wants defaults:
    "most of the configuration of that abstraction should have defaults… setters allow optional
    configuration… Any configuration that we wish to enforce goes into the constructor", because
    "optional configuration allows abstractions to default to their simplest form, making them easier
    to learn" (§5.5). That doesn't conflict with R3. A default that gives the abstraction its simplest
    form (a separator of `""`, an instance name of `"Default"`, no filtering) is intrinsic. A default
    that is a product decision (a free-shipping threshold of 100, a currency, a label) is an
    application literal hidden in the class: the tag was lying. *Reviewer's test:* would another
    product using this class want the same default? *Checklist reading.*
  - **Techniques that meet it.**
    - *Required configuration in the constructor, optional configuration in setters.* §5.5. The
      constructor's parameters are the configuration the composition must state; everything else has
      a simplest-form default and a setter, a named argument or a property.
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
    - **UI text is configuration: Spray's practice in every example, and this checklist's rule.**
      Every word a user reads in his code arrives from the application: `new Display(label:
      "Temperature")` (§2.1.3), `new TextBox(Label="Filter by name")`, `new Menu("File")`, a wizard's
      titles and items, even a status line wired in as a `LiteralString("Connected to device")`
      (§5.8, the device application); "Using one in a specific application only requires a label and
      a binding to an action" (§7.14). His UI abstractions carry no words of their own. He never
      states it as a rule, so the rule is this checklist's, drawn from §1.6.3 (literals at the
      composition) and §2.4 (the application holds the knowledge of the requirements): a word that
      belongs to one product is requirement knowledge, and a domain abstraction that held it would
      not be reusable in the sense of §2.2.
      - *The domain-vocabulary exception.* A word the abstraction's **domain** owns stays with it; a
        word the **product** owns moves to the composition. A date picker's "Today", a pager's
        "Next", a retail stock badge's "Out of stock" are their abstractions' vocabulary (Spray's
        domain is "the domain of applications", §2.2, so a domain UI abstraction may know its domain's
        words). "Your cart is empty." and "Standard (5–7 days)" are this store's. *Reviewer's
        test:* would another product in the same domain keep this word? Keep it if yes, hoist it if
        no. A linter can't tell the two apart; it reports every word below the composition and the
        reviewer decides, recording the decision in the code where the tool supports it.
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
    - A number or message string in a feature or domain class (a `FREE_OVER = 10000` constant inside
      `Shipping`, `"Saved to wishlist"` inside `Wishlist`).
    - A constructor or setter default that is a product decision (above).
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
  - *Spray:* §1.6.3 (literals at the composition); §1.6.5, §2.1.3 and §5.8.3 (constructor and
    initializer configuration in C#); §2.4; §3.5 and §3.5.2 (the application specifies the rounding,
    filter bandwidth, and resampling rate when it instantiates the abstractions); §5.5 (convention
    over configuration). Telling a simplest-form default from a product decision is this checklist's
    reading.
- **R4 — `$` is legitimate only inside a `[]` leaf** (state that *is* the abstraction's concept:
  a filter's memory, a sampler's counter). `$` on a tagged class or method is invisible coupling
  through time between application steps. State belongs with the abstraction whose concept it is:
  Spray "prioritizes abstraction over referential transparency", because passing a filter's running
  value in on every call "breaks an otherwise good abstraction". In an OO language the natural form
  is an instance field of the object whose concept the state is, and that is Spray's own form: "if
  state together with some methods makes a good abstraction, then we don't break it. ALA is therefore
  highly object oriented" (Summary). What R4 forbids is state whose meaning leaks out of its owner:
  the app or a peer keeping, reading, or computing the raw state itself (`f5(s5, p2)`, where the app
  holds `s5`), or a getter that hands out the raw state for others to compute on. State that no
  abstraction owns becomes its own abstraction (Spray's `State<T>`, a generic state-holder with input
  and output ports), wired in like any other.
  - *Two ways to hold state* (§3.11.2). Spray defines a computation as `input + state --> state +
    output`, which can be grouped as `(input + state) --> (state + output)` (the functional form: the
    caller passes the state in and gets it back) or `input ( + state --> state +) output` (the object
    form: the abstraction keeps its state). "In ALA we choose between these two philosophies on a
    case by case basis." Either is fine *inside* an abstraction. What R4 forbids is the caller
    managing state that belongs to another concept. In an OO language the object form is the default;
    an immutable value object returned from each call is the functional form.
  - *Which state, and which fields are mutable.* Spray's four reasons for objects (§3.10) sort an
    instance's fields: references to the objects it is wired to (ports), configuration, the
    abstraction's own state, and, in a `State<T>`, state that belongs to no other code. The first two
    are set once and are "immutable after they are set" (§3.9). Only the third and fourth change, and
    which classes have them "is clear from very nature of the abstraction" (§3.9).
  - *One thread per stateful instance.* "In a multithreaded environments, it would be prudent for
    only one thread to be using each instance of such objects" (§3.9). Locks don't go inside the
    class, because "Successful locking requires detailed knowledge of the specific threading
    allocations of all objects. So if locks are implemented in the classes themselves it introduces
    coupling, destroying the abstractions" (§3.10.2). Which instances share a thread is decided where
    they are wired (see "Object-oriented programming", point 12).
  - **Techniques that meet it.**
    - *Keep the state in the object whose concept it is.* A `LowPassFilter` keeps its running value,
      a `StandardDeviation` its `SumX`, `SumXX` and `N` (Summary). Nothing outside the object reads or
      writes those fields; the object answers questions about them or sends results out of its ports.
    - *Partition state by owner.* Split one large session or composition object into one object per
      concept (cart state, UI state, checkout state), so each piece of state visibly belongs to one
      concept.
    - *Give unowned state its own abstraction.* State that belongs to no concept becomes Spray's
      `State<T>`: an abstraction with input and output ports, wired in like any other (§3.9). "If the
      type system is dynamic, a state abstraction could hold any complex data structure… Only the
      application layer would know the actual structure of the data at design-time"; with a static
      type system the application passes the struct type in as a generic parameter, or the language
      infers it (§3.9).
    - *Make configuration and wiring fields read-only.* `readonly`, `final`, a frozen object, or no
      setter after construction. A field that is set once can't carry anything through time.
    - *Use a functional core inside a class where it reads better.* Pure methods over a value object
      the class owns are fine; the object is still the only one that reads or changes its state.
  - **What doesn't meet it.**
    - A static field, class variable, singleton or global registry used as a hidden channel between
      calls.
    - The composition or a peer reading or computing another concept's raw fields (summing the cart's
      items in the composition, or in a controller, from the cart's `items` getter).
    - A "manager" or "context" object holding several concepts' state that features reach into.
    - Locks inside domain classes to make them "thread-safe" for unknown callers (§3.10.2).
    - A thread per object for its own sake: threads are for throughput and latency, not for
      organizing code (§3.10.2).
  - *Spray:* Summary ("Classes with ports"; the standard deviation's state); §3.9 (state joins the
    struct of the concept it belongs to; immutable configuration and wiring; `State<T>`; one thread per
    instance); §3.10 (the four reasons for objects); §3.10.2 (no locks in classes); §3.11.2
    (abstraction before referential transparency).
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
    - *Type your messages.* A paradigm interface with named methods (`Push(T)`, `Execute()`), an enum,
      or a small value class turns sending a string and hoping the other side matches into a typed call,
      where a typo is a compile error (or, in a dynamically typed language, a `NoMethodError` at the
      first call rather than a silent mismatch).
    - *Send self-oriented events.* "In ALA, sending an event should be self-oriented", written as
      `Send(Sender, SenderPort)` rather than `Send(Event, Receiver, Priority)`: "The sender just
      sends the event out, not knowing where it goes, and the port identifies the event… The event
      framework gets information from the application in the top layer to know what to do with the
      event" (§7.2.6). The event has
      no name for two classes to agree on; the wire is the meaning.
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
    reader's job; a linter only sees duplicated text. A port name the composition passes to a wiring
    operator (`stringConcat.WireTo(connector, "output")`, §5.8.3) names a port the class documents in
    its ports list, so it is the composition using the class's published ports, a knowledge
    dependency downward, not two peers agreeing. *Checklist reading.*
  - **What doesn't meet it.**
    - A label that restates a configured number ("Gift wrap ($2.99)" beside a fee constant of 299):
      change the fee and the label lies. Build the label from the same constant.
    - A hash or dictionary shape one class builds and another reads by key, with no shared
      definition (`{ "sku" => …, "qty" => … }` built in the cart and read in checkout).
    - A session key written as one type by one class and read as another by a second (a symbol here,
      a string there).
    - The same topic, event or notification name written as a literal in two classes.
    - Features reading names from a shared constants class to agree with each other.
    - Two classes that both call a method by a name built from a string (`send("handle_#{event}")`,
      reflection on a method name) that only the two of them know: dynamic dispatch hides the contract
      from every signature.
  - *Spray:* §3.8 (no data coupling); §4.7.4 (global event names are symbolic wirings); §5.8.3 (port
    names given to WireTo); §7.2.6 (self-oriented sending); §7.8 (two instances of one abstraction);
    §7.25 ("no other elements should know about this one thing").

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
  - *One thing, possibly several responsibilities.* Spray reads the Single Responsibility Principle
    through this question: "It is always one thing it knows about, but it may have multiple
    responsibilites for that thing." An ADC driver initializes the chip and reads it; a protocol
    abstraction sends and receives; "A file format abstraction, such as CSVFileReaderWriter knows about
    a file format. It has the responsibility to both read it and write it" (§6.5.1). So don't split a
    reader from a writer of the same format to satisfy SRP: that makes two classes agree on the
    format (R5). "SRP is made redundant by thinking in terms of abstractions" (§6.5.1).
  - *A class is not an abstraction by being a class.* "Classes, Modules, functions and encapsulation
    are artefacts of the language… They are not necessarily abstractions" (§7.2.1). Nor is an object
    model of the domain's nouns: "trying to associate all data with code and all code with data
    causes inappropriate fragmentation of the code, encourages a model of highly coupled,
    collaborating agents, and creates dependency hell" (§3.10). ALA "uses objects as a language
    feature, not a design philosophy" (§3.10).
  - *Document the concept.* Spray calls it "critically important to add comments that make it
    learnable by explaining the concept it provides, its ports, its configurations and an example of
    its use" (§2.1.1). The property: a reader at a call site can find the concept, ports,
    configuration, and an example without reading the body. A short class comment (one or two
    lines, plus an example when it isn't obvious) does this and fits a sparse-comments style.
    Spray's C# convention is a doc comment that lists the ports, and a summary repeated on the
    constructor, because "The class name after a new keyword is actually the constructor name, so
    you must duplicate the summary section there", written "with the intention of it being a
    reminder to someone who has already met the abstraction" (§5.6.1; the `StringConcat` header,
    §5.8.2). In an OO language the IDE's hover text over a `new` (or a `.new`) is where the reader
    meets the concept again.
  - *Name the instance, too.* Spray gives every domain abstraction an instance name (`InstanceName`,
    §5.8.2; a `Name` "immutable and set by the constructor", §8.5), "because abstractions get reused.
    So you are likely to end up with multiple instances of an abstraction all over your application.
    If you name the instances, it makes debugging a lot easier" (§5.8.2). It's for logs and the
    debugger, never for lookup (R9's "No endpoints").
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
    - *The OO form is a tramp constructor parameter:* a class takes configuration it never uses
      itself, only to construct a collaborator with it (`new Checkout(rates, gatewayKey)`, where
      `Checkout` does `new PaymentGateway(gatewayKey)`). It's two defects at once: the collaborator is
      `new`ed inside (R1), and the constructor carries the collaborator's configuration. The
      composition constructs both and wires them.
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
    - *Keep one format's reading and writing in one class* (§6.5.1, above).
  - **What doesn't meet it.** `Utils`, `Helpers`, `Manager`, `Service` or `Handler` classes named for
    their role in a decomposition rather than for a concept; a method that renames a primitive
    (`add(a, b) = a + b`); names that teach nothing (`process2`, `handle_stuff`); a parameter carried
    down unread (the "should" above); a class per domain noun holding data and accessors, whose
    behaviour lives in other classes. A UI component that does I/O: a stateful UI
    component that loads, saves or charges through a store, gateway or placement instance bundles a
    data source or sink into a UI abstraction (Spray wires data sources to UI elements, §5.2.2).
  - *Spray:* §1.6.3 ("func1 is not an abstraction"); §2.1.1 (an abstraction must be "learnable as a
    concept"; document concept, ports, configuration, example); §7.2.5 ("a good abstraction
    separates the knowledge of different worlds"); Summary and §3.4 (a good abstraction is one where
    you "don't need to follow the indirection"); §7.23 (symbolic indirection without abstraction);
    §7.25 ("What do you know about?"); §3.11.1 and §3.9 (parameters carried through for lower
    functions, the "should"); §6.1 (a one-off named function is "indirection without abstraction");
    §3.10 and §7.2.1 (classes are not abstractions by default); §5.6.1, §5.8.2 and §8.5 (doc comments,
    ports lists, instance names); §6.5.1 (SRP as "what do you know about?"). The tramp constructor
    parameter is this checklist's OO reading of §3.11.1.
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
      abstractions when the language's collection and stream libraries exist (LINQ, Java streams,
      Ruby's `Enumerable`); Spray: "There would be no sense in reinventing that functionality as 2-port
      classes" (§6.3). Write one domain abstraction as an adapter with ALA ports and configure it with
      a library chain instead ("Functional techniques in an OO program", point 3).
    - *Don't build wiring machinery for an algorithm.* Plain method calls down the layers are enough
      when the problem is a calculation (§7.5; "Functional techniques in an OO program", point 1).
    - *Accept explicit delegation as the price of no inheritance.* When B, C and D each compose A
      instead of inheriting from it, "B must handle the call first, and then pass the call through to
      A. This is extra code in B, C and D… In ALA we add this extra code in B, C and D, so no virtual
      methods (indirections) are involved and everything is explicit" (§7.16). Such a delegating
      method can trip a pass-through advisory; it is the expected shape, not proliferation.
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
  - **What doesn't meet it.** A class that only forwards to another (other than the delegation that
    replaces inheritance, above); an interface written for every class, implemented once, so that
    each class can be mocked or swapped (a "header interface": a proliferation of names that also
    fails R9 as an owned interface); an abstract base class that exists to share code among a few
    subclasses (R1's inheritance edge); a class per noun holding only fields and accessors; a manifest
    or codegen step for a screen with two features and one reaction (declare in a manifest once the
    wiring is numerous or regular, inline in the composition while it's rare).
  - *Spray:* Summary, "All abstractions must be small" (a rule of thumb of 100 to 500 lines;
    averaging under 100 means "more abstractions than we need"); §2.3.2 ("of the order of 200-500
    lines of code"); §7.10.2 ("probably in the range of 50 to 500"); §7.2.3 ("abstractness decreases
    with more ports"); §6.3 (don't reinvent library functions as abstractions); §7.5 (plain function
    composition suffices for an algorithm); §7.16 (explicit delegation instead of inheritance). Spray
    makes size a fundamental constraint; this checklist scores it as advisory (see "Where this
    checklist departs from Spray"). Calling a single-implementation interface a "header interface"
    is general usage, not Spray's term.
- **R8 — the composition reads as the requirements; names and shapes serve the reader.** The top
  layer should read like the spec; config keys should name what they configure; state wires should
  be shaped for a reader. **R8 is judgement, not shape** — "reads as the requirement" cannot be
  mechanically decided the way "edges drop" can. It is a review prompt and the honest boundary of
  the checklist: R1–R7 and R10–R11 are checkable (mechanically or semi-mechanically), R9 in part; R8
  marks where automated checking stops and human review begins. A tool should *report* the R1–R7
  findings and *flag* R8 concerns, never claim to score R8.
  - **In an OO language the composition is a graph of objects, built once.** Spray's application
    creates every instance with `new`, configures it, wires it port to port, and then sets it
    running: "the entire application is wired up once at the beginning when the application starts
    executing" (§3.11.3). The structure it builds is a static UML object diagram, not a class diagram:
    "The application class specifies a number of objects that must be instantiated, configured, and
    wired together to execute at run-time. Since the structure is always static, ideally this would be
    done by the compiler… So, in the meantime, it is done at initialization time" (§6.14.5). Whatever
    changes at run time is "captured in a static way by inventing a new abstraction to describe that
    dynamic behaviour" (§6.14.5), such as his `Multiple`, which adds calculator rows when a button is
    pressed, so "The wiring diagram now statically describes how it dynamically changes itself"
    (§5.8.6).
  - **Techniques that meet it.**
    - *Draw the diagram, and treat it as the source.* "When you have a graph, your composition is best
      described by a diagram" (§1.6.6). Change the diagram first, then the code: "every time you make
      a change to the requirements, you need to do it on the diagram first, then update the
      hand-generated code" (§5.8.3).
    - *Generate the wiring code from the diagram.* Spray's tools write the instantiations and the
      wiring between marker comments in the application class (`// BEGIN AUTO-GENERATED WIRING FOR
      Calculator2Rows.xmind`), one diagram per marked region (§5.8.4). Generated code "lives in a
      subfolder from where the diagram is, because it is not source code" (§2.5.1); for the coffee
      maker, "This is not source code, it is code hand compiled from the CoffeeMaker application
      diagram" (§2.9.4).
    - *Hand-wire in fluent style when the graph is mostly a tree.* `WireTo` returns its first operand
      and `WireIn` its second, so nested `.WireTo(...)` calls follow the tree of the diagram with
      anonymous instances, "just as they were on the diagram" (§5.8.3). Name an instance in a local
      variable only where the diagram has a cross-connection: "Where the diagram has cross wires that
      formed a cycle, we need to give instances names so that we can complete all the wiring to them"
      (§5.8.3; also §3.6.2). Give a port name only where two ports have the same type (§5.8.3).
    - *Keep "create an instance" apart from "wire it".* Spray keeps `new` and `WireTo`/`WireIn`
      separate rather than writing combined operators, so that the application can choose chaining
      or fan-out, name the port, and "directly translate a diagram to code" (§6.2.2). Combined
      extension methods are fine "for very common cases" when hand-writing (§3.11.3).
    - *Repeated wiring becomes an abstraction, not a helper method.* A helper method in the
      application that wires one row (Spray's `WireRow`, §5.8.3) is the first sign; once it repeats,
      the repeated wiring becomes a feature-level abstraction with its own ports (`CalculatorRow`,
      "less abstract than the Domain abstractions it uses, but more abstract than the application",
      §5.8.5).
    - *Keep the composition in code, not a data language.* "In ALA, the explicit wiring should not be
      XML or JSON. I do not consider these readable programming languages" (§6.6); "No XML as code"
      (§3.6.2). A declarative subset of the programming language is the text form.
    - *Keep wiring order irrelevant.* An instance should work whether its ports are wired before or
      after it is configured, and whatever order the application wires things in. Spray found
      "temporal coupling to creep in between abstractions. It mattered whether the application wiring
      was done first. And if you let coupling creep in bugs will happen", and fixed it "by making
      CalculatorRow not care whether or not the external wiring is done when it is instantiated"
      (§5.8.5). Use a post-wiring hook or an `initialize` event (§4.7.4) for set-up that needs the
      wiring in place.
    - *Make every port's wiring status explicit.* A port that must be wired fails loudly when it
      isn't (`output.Data = result` "will throw an exception if the application has not wired it");
      an optional port is null-checked (`output?.Data = result`) (§5.8.2). Then a test that builds the
      composition and pushes one input through each path catches an unwired required port. Spray
      counts this as unsolved: "all abstractions need to check if there is something wired to a port
      before making a call on it. Enforcing this is a problem I have not yet addressed", and suggests
      that wiring generators "automatically put stubs on all unwired ports. These stubs either throw
      an exception at run-time, or just behave inertly" (§6.15). A null object of the paradigm
      interface as each port's default is the hand-written form of the inert stub. *Checklist
      reading* of §6.15.
    - *Fail at wiring time, not later.* Spray's `WireTo` throws when either object is null, when a
      named port is already wired, or when no matching port is found, so mistakes surface "at wiring
      time when the application first starts. Since in ALA all wiring is generally done at this
      time, at least you wont have potential exceptions later during normal run-time" (§2.2,
      Foundation layer).
    - *When hand-writing wiring, mirror the diagram mechanically.* Spray's conventions: "indent the
      code to exactly mirror the tree structures in the diagram", and "whenever a new instance of an
      abstraction instantiated, all its ports would be wired immediately, and they would be wired in
      the order they were declared in the abstraction… If the destination instance did not already
      exist it would be pre-instantiated" (§3.12.3). Generated code is simpler still: "a list of
      instantiations followed by a list of wirings between them" (§2.5.1). Keep a readme that says
      what the generated code should look like (§2.5.1).
    - *Test that every port is wired.* Each wiring form needs a coverage test, because an unwired
      output is lost or crashes only at run time. It's one small test per composition, generated over
      the declared ports, not one test per port: walk the composed object graph by reflection and list
      every port field still null, minus a list of ports deliberately left unbound.
    - *Draw the diagram from the code that runs.* A composed object graph can be walked by reflection
      (instances, their port fields, and what each is wired to) and drawn, so the picture can't drift
      from the code.
    - *Hold cross-feature wiring in one route table* when features are separate UI components that
      report to the composition: one table says where each output goes. Spray: connections "are
      typically cohesive, and belong in one place" (§3.3.1).
    - *Check port types before the program runs.* With a statically typed language, typed port fields
      make the compiler refuse a wire between two different paradigms. With reflection-based wiring
      or a dynamically typed language, the wiring operator checks the match when it wires, at
      start-up, and raises on a mismatch rather than at the first message.
  - **Wiring forms, side by side.** Hops are message hops per cross-feature effect.

    | Form | Where the wiring lives | How it runs | Hops | Coverage check |
    |---|---|---|---:|---|
    | Generated from a diagram | the diagram; generated code between markers in the application class | objects call each other through their port fields | 0 | the generator, plus a null-port walk |
    | Hand-written fluent `WireTo`/`WireIn` | one wiring method per application or feature | objects call each other through their port fields | 0 | a null-port walk of the built graph |
    | Setter or property injection per port | one wiring method; a setter per port on each class | the same | 0 | the same |
    | Intermediaries wired in | the wiring method, plus a connector, fan-out, buffer or queue instance per wire that needs one | the intermediary's execution model | 0 to 1 | the same |
    | Asynchronous wiring through an event loop | the wiring method; the paradigm queues each send | the paradigm's main loop | 1 | the same |
    | UI components plus controller or composition handlers | the components' wires plus the handlers | the UI framework's event delivery | 1 or more | a test fires every port output |
    | UI components plus a route table | the components' wires plus one route map | a generic router | 1 | a test fires every port output |

  - **What doesn't meet it.** Wiring spread across many classes' constructors or a DI container's
    registrations, so no one place shows it; a composition whose methods compute; configuration keys
    that don't name what they configure; an application class whose wiring can't be followed without
    a debugger.
  - *Spray:* §1.6.6 (a graph is best described by a diagram); §2.4 (executable expression of
    requirements; 3–10% of the code); §2.5.1 and §2.9.4 (generated code is not source); §3.3.1
    (connections belong in one place); §3.5; §3.6 and §3.6.2 (diagrams vs text; no XML as code);
    §3.11.3, §6.2.1, §6.2.2 (explicit wiring over hidden closures; keep instantiation and wiring
    apart); §4.7.4 (`initialize`); §5.8.2–5.8.6 (the calculator: required and optional ports, fluent
    hand wiring, generated wiring, `CalculatorRow`, `Multiple`, temporal coupling); §6.6 (not XML or
    JSON); §6.14.5 (the static object diagram); §7.7. The null-port walk and the reflection-drawn
    diagram are this checklist's techniques, not tried in a variant yet.

The next three rules each turn one of Spray's constraints into a check of its own, rather than
leaving it as background for an earlier rule.
- **R9 — ports are typed by a programming paradigm; an abstraction owns no interface except its
  own configuration.** Spray calls the second half "critically important" (§2.3.4).
  - **Main interface and ports.** "Classes have a 'main interface', the constructors, and any public
    methods and properties… a class's 'main interface'… are only used to instantiate and configure
    the class from a higher layer. It is never used to actually use the class to do its work"
    (§2.3.4). Every other input and output at run time goes through a *port*, and a port's type is a
    programming paradigm from a lower layer. "No other interface implemented or required by the class
    can be 'owned' by the class" (§2.3.4). Spray calls the split "effectively the ISP (interface
    segregation principle). The client who instantiates a class object is different from the classes
    whose objects will interact with it, so different interfaces are used" (§2.3.4; also §7.24,
    "Everything through interfaces").
  - **What a port is in a class.** "Syntactically, a port is either a field in a class of the type of
    an interface, or just implementing an interface, which we already do a lot in conventional code.
    What makes it a port is the abstraction level of the interface" (Summary). The field form is an
    *accepted* port; the implemented form is a *provided* one ("Instead of 'required' interfaces, in
    ALA they are called 'accepts' interfaces… as with Lego blocks, there isn't necessarily a connection
    to another instance", §6.14.2). Details from Spray's C# practice, each with its reason:
    - *The port's name is the field's name* (§7.2.6). "A port can have multiple interfaces. In this
      case I make the names of the multiple fields contain the port name" (§7.2.6), and a port can be
      "a pair of interfaces that allow methods in both directions" (§6.14.5), wired in one operation.
    - *Port fields are private,* so that the public interface of a class is "just… the
      'configuration interface' used by the system abstraction when it instantiates a Switch. The
      WireTo method is designed to be able to wire the private port" (Summary).
    - *Provided ports are implemented explicitly,* so the port's methods aren't on the class's main
      interface: "We only want the interface's method/event to be visible through a reference to the
      interface, not the public interface of the class… If there were two interfaces using the same
      method name or same event delegate, we will want to implement them separately" (§4.4.4).
    - *One port per peer that can be wired.* "There should be one port for each peer that can be
      wired. This is the Interface Segregation Principle" (§7.2.6).
    - *Ports are listed in the class's header comment* (§5.8.2), so a reader of the wiring knows
      what can be wired without opening the body.
  - **No associations, not even through an interface.** "Classes in ALA do not have association
    relationships. Instead they just have fields of the type of these more abstract interfaces or they
    implement these more abstract interfaces. We call both of these ports" (§2.3.4). "You are not even
    allowed to make an association with an interface belonging to a peer class" (§3.11.3).
  - **Owned interfaces fail, both directions.** A *required* interface is one the consumer defines
    for others to implement (Clean Architecture's ports, which the business layer defines for
    adapters to implement; the dependency inversion pattern). A *provided* interface is one a class
    defines, specific to itself, for its peers to call. Both carry one class's design inside the
    other: "There will be a fixed arrangement between the two classes. Over time, this fixed
    arrangement will cause a blurring of their respective responsibilities" (§2.3.4). Moving the
    interface to a package of its own doesn't help if it still describes what one class needs:
    "Simply moving IB doesn't make it more abstract" (§6.5.5; IB is the interface that class B
    requires).
  - **Interfaces in an OO language** (interfaces, abstract classes, protocols, traits, mixins, duck
    types). What matters is who defines the interface relative to who implements and calls it.
    - *Allowed: a paradigm port.* An interface in the Programming Paradigms layer that no domain
      abstraction owns (`IDataFlow<T>`, `IEvent`, `IUI`), implemented or accepted by domain classes
      (`class Light : IDataFlow<bool>`). This is how a domain abstraction gets a port. Such interfaces
      are usually tiny: "Programming paradigm interfaces are often this simple" (§2.2), and a CRUD
      interface shows that general ones exist: "The very existence of this acronym suggests an
      abstract interface" (§2.3.4).
    - *Allowed: configuring a more abstract class.* A framework far below its users defines an
      interface or a base class that higher classes implement to configure it: a web framework's
      controller or request-handler base class, a test framework's test case class, a UI framework's
      window or component class, a job runner's job interface. The implementer depends *down* on it,
      which is legal, and Spray's own UI abstractions build on WPF's classes (§4.12). It's the
      interface-shaped form of Spray's "lambda passed in" for calling up the layers. *Checklist
      reading:* it is still inheritance, so keep the subclass a thin configuration of the framework
      (R11 for a composition, R6 for a domain abstraction), and never build an inheritance hierarchy
      of your own on top of it.
    - *Not allowed: a required interface between peers.* `Checkout` defines an `IPaymentGateway`
      interface with `Charge`, and `StripeClient` in the same layer implements it. The interface
      describes what `Checkout` needs, so `StripeClient` is written to `Checkout`'s design. Moving it
      to a shared contracts package doesn't change that. The ALA route is a paradigm-typed port on
      `Checkout` (request/response, say) that the composition wires to a payment abstraction.
    - *Not allowed: a provided interface made for peers.* An `IInventory` interface describing
      `Inventory`'s own methods, so peers can call it through the interface or mock it. Its callers
      are still written to `Inventory`'s design. Spray's testing rule is to "mock the ports": wire a
      fake instance to a paradigm port in the test, not replace a peer by name.
    - *Not allowed: an abstract base class as a port.* A port's interface "cannot be thought of as an
      abstract base class. It is even more polymorphic than that" (§6.2.1). A base class that a few
      peers extend is "a finite set" of receivers, which "potentially allows the sender to have some
      implicit knowledge of those receivers" (§3.10.1).
    - *In a dynamically typed language,* a port has no declared type, so the paradigm is the set of
      messages a port may send or receive (`push(value)`, `execute`), defined and documented once in
      the Programming Paradigms layer, as a module, a mixin, or a documented protocol that a wiring
      check can verify with `respond_to?`-style tests. The owned-interface failure looks the same as
      in a typed language: a class that calls `cart.line_items` or `gateway.charge(order)` on whatever
      it was given is written to its peer's design even though nothing declares it. Spray's own
      reasons cover it. A dependency exists "if there is a reference to something named inside the
      class or interface", "Even if using an object reference" (§3.4); injecting the object doesn't
      help, because "Even if Light is injected into Switch by a higher entity called System, Switch
      still knows the specific interface of a light (LightOn(), LightOff())" (§3.3.1); and coupling
      can exist with no compile-time dependency at all: "a function of a decomposed system will tend
      to be written to do what its caller requires even if there is no explicit compile-time
      dependency on its caller" (§2.6.2). Duck typing removes the declaration, not the knowledge.
      *Checklist reading* of how those sections apply to dynamic languages; Spray's dynamic examples
      keep static paradigm interfaces and make the *data* dynamic (`ExpandoObject`, §6.3.1; §3.9).
  - **Composing functions.** When the composition calls methods directly, "Parameters and return
    values are effectively port", and so is a function passed in (§2.3.6). The same rule applies to
    their types.
  - **Designing the paradigm interface is the architect's hard job.** "I have sometimes got stuck for
    a day or so trying to figure out how the interfaces should work, while still keeping them more
    abstract than the domain abstractions. The ALA constraints are that these interfaces should work
    between any two domain abstractions for which it may be meaningful if they are composed together
    in an application" (§5.7). Of his student: "It was his job to invent the methods he needed in the paradigm
    interfaces to make the system work, but at the same time keep them abstract by not writing methods
    or protocols for particular class pairs to communicate" (§7.26.3). *Reviewer's test:* does any
    method on a paradigm interface exist for one pair of classes?
  - **Data on a port, and data-transfer types.** Two domain abstractions "may not… share a DTO" (a
    data-transfer object: a class made only to carry data between two classes). The type "must be
    more abstract and come from a lower layer, so is often a primitive type from the programming
    language", and "T may be passed in by the application, which always knows types of data moving
    through the system" (§4.8.1). That's why Spray's paradigm interfaces are generic (`IDataFlow<T>`):
    "This makes sense because only the application, the Thermometer, knows the actual types it needs
    to use" (§7.6).
    - *Allowed: standard types* (numbers, strings, lists, dictionaries), and a generic parameter the
      application fills in.
    - *Allowed: a type from a lower layer that is itself a real abstraction,* such as a decimal, a
      date, or a domain `Money`. Both ends depend down on it, a knowledge dependency. A DTO can earn
      this place by use: "If many abstractions want to know about the same DTO… then maybe it is
      sufficiently abstract to be in a lower layer and shared" (§6.17.4).
    - *Allowed: an application-defined type passed in.* The app defines it, and the abstraction
      either carries it without reading its fields or reaches its fields only through what the app
      configures it with (field names, accessor lambdas, a paradigm interface). This is Spray's "T
      passed in by the application". With type inference the app may name the type only once, at the
      source: "Ideally the form, select and join abstractions do not also have to be configured by
      the application to know the types of their ports. Instead they are able to infer the type as an
      anonymous class as it goes from port to port at compile-time" (§4.8.1).
    - *Allowed: run-time types for dynamic data.* When the fields aren't known at compile time (a CSV
      file, a user-configured table), the paradigm carries the type information with the data, as
      Spray's `ITable` does (§4.8.2), or the data travels as a dynamic object (`ExpandoObject`,
      §6.3.1). In a dynamically typed language this is the ordinary case: a hash or a plain object
      whose keys only the application knows.
    - *Allowed: a type used only inside one abstraction* (a class and the small types in its own
      file).
    - *Not allowed: a data-transfer type between peers.* `Cart` builds an `OrderRequest` defined by
      `Checkout`, which reads its fields. Or both read a class neither owns but which was designed
      for exactly their exchange. Either way one abstraction's design now lives in the other. Moving
      the class to a shared package in the same layer doesn't fix it, for the same reason as a moved
      interface.
    - *Not allowed: a DTO for transport.* "In ALA you wouldn't use DTO for transport purposes.
      Instead, invent an abstraction say called multiplexer_demultiplexer for packing/unpacking (or
      serializing/deserializing) multiple input or output ports" (§6.17.4), which the wiring inserts
      when two instances are deployed apart.
    - *Fixes:* the composition converts between the two shapes with a `(p) ->` lambda, "as simple
      as a lambda expression passed to the WireTo operator, in the same way that you would pass a
      lambda expression to a .Select clause in LINQ" (§6.17.4); the application supplies the type;
      two primitive-typed ports instead of one DTO port ("this will increase the abstraction level,
      reusability and composability of your abstractions", §6.17.4); or the two ends become two
      instances of one abstraction that owns the format (§7.8, see R5). Spray prefers the paradigm
      interface to the adapter: "Although this is ALA compliant, in ALA we generally prefer not to use
      adapters" (§6.17.4).
    - *R9 vs R10:* R9 is about the message type on a port between two abstractions. R10 is about a
      domain entity that two features both read.
  - **Outputs announce; they don't command.** "An output port from an abstraction may say 'This has
    happened' or 'Here is my result', not 'do this next', or 'here is your input'" (§7.2.6). What
    happens next is the wiring's business. The one exception is request/response. Wired point to
    point, "a request is implicitly a command" (§4.6), and that's fine because the application set
    up the wire. In OO terms this is the message view of objects Spray takes from Alan Kay: "A
    procedure's name describes what it does. A method's name is a message name describing something
    that has happened elsewhere" (§3.10). An output port's method is named for the paradigm
    (`Push`, `Send`, `Execute`), never for what one receiver will do with it (`SaveOrder`,
    `RefreshGrid`). "The choice of the word Send rather than Execute is to indicate it's only sending
    the event not executing it" (§4.7.3).
  - **Paradigm instructions as outputs.** Some designs have features return or send instructions
    such as "insert into the cart list" or "notify 'Added'", which a generic interpreter carries
    out. Such an instruction bundles a verb ("insert"), a target ("the cart list"), and a payload. R9
    treats the target and the verb differently:
    - *Must: an output never names its destination.* Whatever decides where an output goes, and how
      it is presented, comes from the composition: list names, topics, message text, a named
      receiver. *Reviewer's test:* could the composition send this output somewhere else without
      editing the feature? If not, it fails. This follows from "No endpoints" below (§4.4.2) and from
      R3, since message text is an application literal.
    - *Should: an output reads as a result, not an operation.* Prefer outputs that say what happened
      or what the result is (`itemAdded.Push(item)` on a port the class names), over an operation the
      class has decided on (§7.2.6). *Reviewer's test:* could the composition wire this output to a
      different kind of receiver (a counter, a log, nothing), not just a different list? A "no" is a
      prompt to reconsider, not a defect.

    The "should" is weaker because the evidence is. One reading of Spray treats a paradigm
    instruction as request/response, where "a request is implicitly a command" (§4.6). But his
    request/response is two-way, used when "the requester needs to know" something back, and a
    list insert expects nothing back. The checklist doesn't prescribe a technique. Classes sending
    facts on ports that the composition wires, and targets passed in as configuration, both meet the
    "must".
  - **No endpoints, and no container.** An abstraction never names where its input comes from or
    where its output goes. Receivers never register themselves with a sender or subscribe to a public
    event (§4.4.2). The composition sets every wire. "The dependency injection wiring must be
    explicit. It must be specified in cohesive user story abstraction in a higher layer. The wiring
    cannot be done by using a dependency injection container or relying on matching interfaces"
    (§3.11.3). Why: "Container based dependency injection works by matching interface types… The
    matching of this interface type is the implicit wiring of the two classes. There is no place where
    you can see the wiring explicitly. This is really bad" (§3.10.1), and "This type of implicit wiring
    is indirect and obfuscated and illegal in ALA" (§6.6). Ports are "conventional dependency
    injection, but with two additional constraints": the interface "must be significantly more
    abstract", and the wiring "must be explicit" (§3.11.3). Spray doesn't prescribe how the
    indirection is built: "They can be callbacks, signals & slots, dependency injection, or calls to a
    framework send function" (§7.2.6).
    - *Allowed:* the application constructing instances and wiring them by hand, by generated code,
      or with a reflection-based wiring operator it calls explicitly (`a.WireTo(b)`), which matches
      types *within the one pair the application named*. That's different from a container, which
      chooses the pair.
    - *Not allowed:* a container that resolves a class's constructor parameters by type, a service
      locator, auto-wiring by name or annotation, wiring declared in XML or JSON (§6.6), and a class
      reading global configuration to pick its own collaborator.
  - *Visible shape:* a `[]` leaf takes only `pN` wires, `(p) ->` lambdas, and configuration, and
    **no edge names a peer**. The smells:
    - a port field or parameter typed by a peer class, or by an interface a peer owns;
    - an interface or abstract base class defined in a feature or domain layer and used across a
      boundary;
    - public methods on a class other than its constructors and configuration (its ports leaking onto
      its main interface);
    - a class that names its own source, destination, or topic, or asks a container or locator for
      a collaborator.

    *Verify:* for each abstraction, ask three questions:
    - Does it name the source or destination of any input or output, including a list name, topic,
      or message text in an output it sends?
    - Does any port's type belong to a peer rather than a paradigm, the standard library, a lower
      layer, or the application?
    - Does it define an interface that its own peers implement or call (rather than a paradigm port,
      or a framework's extension point that higher classes implement to configure it)?

    Any yes is a defect.
  - *Mechanizable:* partly. A `[]` class that references a peer class is already an R1 edge.
    Interfaces and abstract classes declared outside the paradigm layer and used across a boundary
    can be found from the syntax tree, as can a field typed by one, a container registration, or a
    public method that isn't configuration. Literal list names, topics, or message text inside
    feature-layer outputs are findable, which covers much of the "must" for outputs. Whether a type
    is "more abstract", and whether an output reads as a result, are judgements. In a dynamically
    typed language the types aren't in the source, so most of this needs a human or a run-time check
    at wiring time.
  - **Should: configuration is set once, apart from run-time data.** Spray's §3.9: "If the abstraction
    consisted only of a single function, then that configuration data would need to be passed in
    every time the function is used. That would be awkward. It would also mix the data parameters of
    the function with the configuration parameters, breaking the Interface Segregation Principle." It
    is a "should" because his own §1.6.3 thermometer passes its settings on every call
    (`OffsetAndScale(adc, offset=4, slope=8.3)` inside the loop), a rung he then climbs past. In an
    OO language the object *is* the remedy: configuration goes into the constructor and setters
    once, and the port methods take only run-time data (§1.6.5, `new Filter(strength=10)`).
    - *Reviewer's test:* does any method take configuration and run-time data mixed in one
      parameter list, so every caller repeats the same settings on every call?
    - *What doesn't meet it:* settings mixed into the data arguments (`Cost(method, subtotal,
      rates)`), each caller repeating them; defaults baked into the domain that are product
      decisions (R3); global configuration read on every call inside a domain class.
  - **Techniques that meet it.**
    - *Put port interfaces in the Programming Paradigms layer,* one small interface per paradigm
      (`IDataFlow<T>`, `IDataFlowPull<T>`, `IEvent`, `IUI`, `IRequestResponse<TReq, TResp>`), in a
      namespace and folder of that name (§2.3.1). A domain class gets a port by implementing one or by
      holding a field of one.
    - *Wire with one operator from the Foundation layer.* Spray's `WireTo`/`WireIn` extension methods
      find the port by type (or by an optional port name) and assign it, so classes need no setter
      per port (§2.2 "Foundation layer"; Summary). Reflection is optional: "ALA does not require the
      use of reflection… You could use dependency injection setters in every domain abstraction
      instead. You would need one setter per port… You wouldn't use constructor dependency injection
      because sometimes wiring a port is optional" (§2.2; Summary). Generated code can also assign
      public port fields directly: `new Switch().output = (IDataFlow<bool>) new Light();` (Summary).
    - *Run set-up that needs the wiring from a post-wiring hook.* Spray's `WireTo` calls a private
      `<port>PostWiringInitialize` method right after wiring a port, used to subscribe to events in
      that port's interface (§5.8.2; §4.4.5). The subscription is inside the paradigm interface, set up
      by the wiring, so it isn't a receiver registering itself.
    - *Let the application supply the type* through a generic parameter or type inference (§4.8.1).
    - *Collapse I/O into one paradigm value.* A sensor-reading value (all inputs) and a
      hardware-command value (all outputs) give the application ports shaped by their kind, not by
      any particular device, and make the boundary plain data a test can build.
    - *Adapt mismatched ports with a lambda at the wiring* (§6.17.2, §6.17.4), or wire in an
      intermediary instance (a buffer, a poller, a fan-out) that the paradigm provides (§4.4.3,
      §4.4.6).
    - *Wrap a library that wasn't written for ALA.* "What happens if one of the abstractions to be
      used is written without knowledge of ALA? It has a conventional API that includes both
      configuration and data input/output methods. In this case the team responsible for the user
      story itself will write a wrapper that will make the abstraction into an ALA compatible domain
      abstraction that has a separate configuration interface and the relevant ports. The wrapper and
      the abstraction that it wraps become a single abstraction together" (§3.8.2). A large legacy
      component can hide behind such a facade without its complexity spreading (§7.3.2).
    - *Use a factory port when instances must be made at run time.* Spray's `Multiple` makes N
      instances of whatever is wired to its `factory` port, typed by an `IFactory` paradigm interface;
      each abstraction that can be multiplied carries a small factory class implementing it, and the
      application configures `Multiple` with two lambdas that wire each new instance in (§5.8.5;
      §6.11.1).
    - *Test against ports.* Wire a fake into the port (a stub class implementing the paradigm
      interface, a lambda, a test double that records what it receives). A mocking library is fine
      for a paradigm-layer interface, since that *is* a port (see "Tests replace only ports" in the
      verify procedure).
    - *Configure domain instances once, in the constructor and setters.* Every rule the composition
      used per call becomes configuration of an instance: rates, promo codes, the gift-wrap fee, the
      low-stock level, the currency.
    - *Name output ports as facts, and let the wiring turn them into actions.* `persist` becomes
      `changed`, an undo offer's `timer` becomes `captured`, `payment` becomes `readyToPay`, and the
      cart's "pay" becomes `checkoutRequested`. The composition's wiring then starts and stops the
      undo clock on those facts.
  - **What doesn't meet it.** An interface a feature defines for a peer (`Checkout`'s
    `IPaymentGateway`); an `IInventory` interface written so peers can mock `Inventory`; a class one
    peer defines and another reads, even if moved to a shared package; an output that names a list,
    topic, or message text; a class that looks up its own collaborator; a DI container that wires by
    matching types; public port methods on a class's main interface.
  - *Spray:* Summary ("Classes with ports"; private ports and `WireTo`); §2.2 (paradigm interfaces;
    the Foundation layer's `WireTo`); §2.3.4 (main interface; owned interfaces "critically
    important"; no associations); §2.3.6; §3.10 (messages); §3.10.1 and §6.6 (no containers);
    §3.11.3 (ports as constrained dependency injection); §4.4.2; §4.4.4 (explicit interface
    implementation); §4.6 (request/response); §4.7.3 (Send, not Execute); §4.8.1 and §4.8.2 (no DTOs;
    T passed in; type inference; run-time types); §5.7 and §7.26.3 (designing paradigm interfaces);
    §5.8.2 and §5.8.5 (post-wiring initialize; `Multiple` and `IFactory`); §6.2.1 (not an abstract
    base class); §6.5.5 (dependency inversion); §6.14.2 ("accepts"); §6.14.5 (a port is a pair of
    interfaces); §6.17.2 and §6.17.4 (adapters, interface and transport DTOs); §7.2.6 (ports, one per
    peer; outputs announce); §7.8; §7.24; §3.9 (configure once; the "should"). The duck-typing reading
    for dynamically typed languages and the framework-base-class reading are this checklist's.
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
    sibling feature subtrees. *Verify:* is any domain type referenced or read by two or more feature
    units? *Mechanizable:* yes: a class read by two units of its own layer.
  - **Entity and model classes in an OO language.** An object-oriented codebase usually has a class
    per business entity (`Customer`, `Order`, `Product`), often persistent (an active-record or ORM
    model), that every feature reads and writes. That is exactly Clean Architecture's entity, which
    Spray rejects: entities "are an easy place to just add all fields to do with an identity. They
    will tend to hold some fields that, although they associate with an identify, really belong to
    separate use cases" (§6.17.2). His alternative keeps one shared abstraction, the identity, and
    gives each use case private data stored against it: "a user story should be able to have private
    data that is associated with an identity and still ultimately stored with all other data for that
    identity in the database… Adding this field should cause a database migration, but not changes to
    other use cases" (§6.17.2). Two OO-specific points:
    - *Subclassing an entity per use case isn't the fix.* "Subclassing, so that every use case has its
      own subclass may solve the problem in one way, but I expect would cause other problems"
      (§6.17.2). Each subclass still knows the base entity (R1's inheritance edge).
    - *A persistent model class is a data source as well as a type.* When features call its query and
      save methods, it is both a shared entity (R10) and a store each feature reaches by name (R1,
      R9). The ALA shape is a domain abstraction of the persistent table, configured by the
      application with its schema and wired to the features that use it (§6.19: "We don't just have a
      port to the persistence adapter, we have an abstraction of persistence"). *Checklist reading* of
      how the active-record pattern meets §6.17.2 and §6.19.
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
  - **What doesn't meet it.** Checkout reading the cart's class; one `Order` class every feature
    reads and writes (Clean Architecture's shared entity); a persistent model class every feature
    queries, reads and saves; a subclass of a shared entity per use case; a domain aggregate that
    carries one use case's data and is shared with another (the strict reading above).
  - *Spray:* §3.6.1 (the ground symbol); §3.8 (no data coupling); §4.8.1 (no shared DTOs); §6.17.2
    (use cases depending on entities "is incompatible with ALA"; private use-case data stored against
    an identity; subclassing per use case; dataflows carry "only the data that is needed by the use
    case"); §6.19 (an abstraction of persistence); §7.8. The identity-key technique is Spray's
    (§6.17.2); its name is this checklist's.
- **R11 — the composition layers are composition only: the application, and Features when an app
  has them.** The application instantiates, configures, and connects. It holds *all* app-specific
  knowledge and *no* app-specific logic: "no normal programming language code such as assignments
  and if statements" (§3.5), and about 3–10% of the code (§2.4). Spray's own examples show what that
  sentence covers and what it doesn't.
  - **Not banned:**
    - *Naming an instance so it can be wired twice.* `temperature = new FloatField()` (§1.6.6), or
      locals for cross-connections (§3.6.2; `Formula[] formulas = { new Formula(), new Formula() }`
      "because we need to cross wire them", §5.8.3). In an OO language, a local variable, or a field
      of the application class, holding an instance.
    - *A predicate or small function passed in to configure a generic abstraction.* For example
      `new Filter(x => x>=0)` (§3.11.3), or `.Bind(x => x==0 ? -1 : 1000/x)` (§6.1.3). That states a
      requirement ("ignore negative readings") once, as configuration. The abstraction runs it.
    - *Setting the built program running.* `program.Run()` or `mainWindow.Run()` after the wiring
      (§1.6.5, §1.6.6), and an `Initialize()` call or event that tells instances the wiring is done
      (§7.6, §4.7.4).
    - *An application-specific function passed in as configuration, even one with branches.* In
      Spray's game scoreboard, the application configures `ScoreBinding` instances with query lambdas
      over the scoring engine, and keeps a private `TranslateFrameScores` function (loops and `if`s
      that turn pins into "X", "/" and "-") in the application layer, commented "This function is an
      abstraction (does not refer to local variables or have side effects)" (§6.28.1). His reading:
      "not all dataflows have to go directly between wired up instances of domain abstractions. The
      data can come up into the application layer code, and then back down… The important thing is
      that all the code in the application is specific to the application requirements" (§6.28.3).
      *Checklist reading:* this is the predicate case scaled up. A pure, application-specific
      function handed to an instance as configuration is allowed; the wiring code itself still
      doesn't branch or carry run-time data.
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
    - *In an OO language* the same finding covers a nested, inner or private class used as a part of
      its enclosing abstraction, and an application class that builds its own private helper
      classes. "Abstractions are never private. The reason they are never private is simple. An
      abstraction that is depended on should be more abstract than the abstraction using it. A more
      abstract abstraction needs to be public so it can be reused" (§2.1.3). A nested type that is
      just part of one abstraction's little ball of mud (an enum, a delegate, a small value type in
      the same file, §2.3.2) is fine; a nested class that is a concept of its own is a sub-abstraction.
    - *Packages aren't abstractions either.* "In ALA, packages would only be used as a distribution
      mechanism, not as part of the architecture for information hiding" (§2.1.3). A package whose
      internal classes are hidden behind a facade still has those classes as abstractions; make them
      public, in a domain-qualified namespace if they belong to another domain
      (`CompilerDomainAbstractions`, §2.1.3).
    - *Repeated wiring is a feature abstraction, not a nested part.* Spray's `CalculatorRow` holds the
      wiring for one row behind its own ports and sits in a layer of its own between the application
      and the domain abstractions, used several times (§5.8.5). It isn't contained by the application;
      it's a less abstract abstraction the application uses.
  - **Where each kind of `if` goes.** These are Spray's moves, in order of how often they come up:
    1. *Propagation guards* ("only if there's a value", "stop on error") move into the connection
       mechanism. He factors the `if`s "into the Compose function" (Bind, the function that chains
       one step to the next in a monad, §6.1.3), and his
       thermometer's `if` disappears once `SampleEvery` simply emits nothing (§1.6.4). In an OO
       language: an instance that doesn't push when it has nothing to say (Spray's `LowPassFilter`
       pushes only every `cutoff` inputs, §7.6), an intermediary that drops or buffers, a null check
       on an optional output port inside the sender (`next?.Push(…)`), or a library chain (LINQ,
       Rx, a stream) inside an adapter.
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
    instance's configuration isn't). A fluent chain of `new`, setters and `WireTo`/`WireIn` isn't
    handled data: it builds the graph and passes no run-time values. Telling the kinds of branch apart
    is judgement, though some forms are recognizable.
  - *Generated code is judged by its diagram.* Spray's coffee maker compiles to six lines with `if`s
    in them (`if (userInterface.Button && warmerPlate.PotOnPlate && !boiler.Empty) { state =
    Brewing; }`), but "This is not source code, it is code hand compiled from the CoffeeMaker
    application diagram" (§2.9.4), and the diagram has an AND gate and a state machine, not `if`s.
    When the wiring is generated, R11 reads the diagram, and a linter should skip the generated file
    (§2.5.1).
  - *In a UI framework:* the framework hands a screen events to route, URLs, and a lifecycle to
    follow, so some forms remain: handler methods that receive an event and forward it, decoding
    string params, and a lookup from URL to step. None of these is logic. Everything else a screen
    tends to contain (guards, rules, arithmetic, handled data, history, template loops, lifecycle
    checks) has one of the moves above, and a screen can reach zero logic this way.
  - **Techniques that meet it.**
    - *Build, then run.* The application's constructor or set-up method creates the instances, wires
      them, and calls `Run()`; after that it does nothing (§1.6.5, §5.8.3). Data moves between
      instances through their ports, never through the application.
    - *Split the shell: generic dispatcher below, wiring in the composition.* A shell that applies a
      list of outcomes to view state is generic, so it's an execution model in the Programming
      Paradigms layer, below the composition. The part that names the composition's parts is
      composition code.
    - *Guards go into the connection mechanism.* An instance that doesn't push, an intermediary that
      filters, a null-checked optional port, or a library chain inside an adapter (§1.6.4, §6.1.3,
      §6.3).
    - *Requirement conditions become configured instances; history becomes a state machine.* A rule
      becomes a predicate passed to a generic abstraction (R3), or a small logic abstraction like the
      coffee maker's AND gate (§2.9.3). A rule that depends on what happened before becomes a state
      machine (§4.16): a state-machine abstraction whose states and transitions are instances the
      composition wires, or a transitions table passed in as configuration so the feature decides.
    - *Keep an abstraction's own rules inside it.* The coffee maker's `Boiler` enforces "the heater is
      off whenever I am empty or my valve is open" itself. The application asks it to heat; it refuses
      when that would be unsafe. The rule never reaches the top.
    - *Route outcomes; don't decide on them.* Give the abstraction two output ports, one for success
      and one for failure, and wire each (§4.9.1). Where a handler must still receive one result, a
      branch whose arms only send the ok and error results to different places is routing two output
      ports; one that computes in its arms is logic, and belongs in a feature.
    - *Move iteration and compound conditions out of the screen's template.* A generic list or table
      component iterates (§4.12). A feature computes a compound boolean and the template wires it.
    - *Move lifecycle guards out of the screen's start-up.* Use the framework's own extension point
      for lifecycle hooks, configured on the screen, so start-up has no branch.
    - *Generate the glue.* Committed code generation makes the composition's wiring a generated
      file, so the hand-written composition stays thin (§2.5.1). A check fails the build if it drifts.
    - *Stop handling data at the composition.* Several designs get the composition's handlers to pure
      routing: wire instances port to port so their outputs never return to the composition (Spray's
      own form), bind every feature output once in one route table per screen, or make each feature a
      component instance that wires its own outputs and sends the rest.
    - *Make start-up an event on the diagram.* Loading at start-up hands a store's result to a
      feature. Instead, deliver a `started` event through the wiring, bound to "read the store, feed
      the result to the cart's `load` input", so the composition never holds the result.
    - *Replace a composed input with an output.* A handler that reads composition state to build
      another feature's input (fetching stock for payment, finding a line for the wishlist) becomes one
      input on the first feature whose output carries the value (the cart sends `checkout_requested`
      with its lines, wired to checkout's `pay`).
    - *Route every message explicitly, and test that you do.* Where a composition receives messages
      in handler methods, those handlers are its wiring. With a catch-all handler, a message nobody
      routes disappears silently. Without one, it fails loudly, and a test can send every port each
      instance declares it sends and fail on a missing handler. The trade: the composition must also ignore, explicitly, every
      broadcast on a shared topic it doesn't use. Explicit is safer where the test suite runs on every
      change; the catch-all is the more tolerant default where it doesn't.
    - *Move store work out of composition helpers.* A private composition method that the wiring
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
    - *Add the store method that removes a hand-off.* Fetching a record and then deleting it in a
      handler is the composition passing one store result into another call. A store abstraction's
      method that does both keeps the handler to one call.
  - **What doesn't meet it.**
    - Arithmetic and computed values in the composition (summing items into a display value).
    - A business rule in a handler ("can't check out an empty cart" as an `if`).
    - A match on the URL that encodes which steps may follow which.
    - A handler that takes one feature's result and passes it to another.
    - Loading rows at start-up and handing them to a feature.
    - A loop or a compound condition in the screen's template, including a comparison like
      `disabled = item_count == 0`.
    - A screen-specific stateful UI component: a contained sub-component, whatever it calls.
    - A class in a Features layer that holds logic, or a coded abstraction in a namespace called
      Features (R8).
    - A nested or private class that is a concept of its own (§2.1.3).
  - **Limits.** Some forms stay in any UI composition because the framework puts them there:
    routing by event name and URL, decoding browser params, a lookup from URL to step, and a
    client-side hook or two. None is logic. Unlike a functional language, an OO language doesn't
    force the composition to store a program value back after each step: objects change in place.
  - *Spray:* §2.9.3 (the coffee maker's application is a diagram of instances; its conditions are
    AND-gate instances, not `if`s); §3.5 ("no normal programming language code such as assignments
    and if statements"); §1.6.3 (the thermometer's `if` flagged as logic); §1.6.4 and §6.1.3 (guards
    move into the connection mechanism); §1.6.6 and §3.6.2 (instance variables for wiring);
    §3.11.3 (configuring with lambdas); §1.6.5 and §5.8.3 (build, then `Run()`); §2.1.3 (abstractions
    are never private; packages); §2.9.4 and §2.5.1 (generated code); §4.4.4, §4.9.1, §4.16; §5.8.5
    (`CalculatorRow`). Reading forwarding handlers as routing, and the framework-imposed departures,
    are this checklist's adaptation.

Rules-of-thumb for using R1–R11: R1–R2 and R5 are the coupling core; R3 is requirements-locus; R4 is
state-with-its-owner; R9 is the port/interface discipline and R10 the data-sharing discipline (both
coupling-core in spirit); R6–R7 are design minimality/nameability; R11 is the composition-only top
layer; R8 is the human-judgement remainder. A mechanical tool can check R1–R7 and R10–R11 (and R9 in
part) at varying precision; treat R8 as the reason a green run is necessary but not sufficient.

One topic is *in development* and not yet a rule: how wires differ in meaning and in how they run
(paradigms, push or pull, sync or async, fan-out, glitches, loops). See "Kinds of connection and
how they run" under the worked examples.

## Object-oriented programming: what Spray says, and what it means in an OO language

Spray writes almost every example in C#, with classes, and he says plainly that ALA is object
oriented: "ALA is object oriented, but its object oriented in a much more disciplined way than
conventional object orientation. That's because peer classes may NOT have associations" (Summary).
His comparisons with OOP are spread across the Summary, §2.1.3, §2.3, §3.2.2, §3.9–3.10, §3.11.3,
§4.4, §5.8, §6.2.1, §6.4–6.6, §6.11, §6.14.5, §6.17, §6.19, §7.5–7.6, §7.16 and §8.6. This section
gathers them, says what each means in an OO language, and names the rule it supports. Where the
reading is this checklist's and not Spray's, it says so. Where Spray's practice is specific to C#,
the general OO form is given beside it.

1. **ALA is OOP done right, using objects as a language feature, not a design philosophy.** "ALA is
   really just OOP done right" (§4.4.5); "It is object oriented programming as it should have been"
   (Summary). But "you don't try to model everything with objects. It uses objects as a language
   feature, not a design philosophy. Secondly, you can't create associations between classes"
   (§3.10). He agrees with Brian Will's critique of conventional OOP "in so far that trying to
   associate all data with code and all code with data causes inappropriate fragmentation of the
   code, encourages a model of highly coupled, collaborating agents, and creates dependency hell"
   (§3.10).
   - *In an OO language:* don't start from a domain model of nouns, each a class with its data and
     methods, collaborating. Start from the requirements, invent abstractions, and use a class for
     each one that needs configuration, ports or state. *Rules:* R6, R1.
2. **The thing OOP got right is classes and objects, kept apart.** "I think the most fundamental and
   important characterising feature of OOP is under-rated. That is the separation of the concepts of
   classes and objects… What OOP should have done is represent relationships between objects
   completely inside another class. The problem is that OOP doesn't take advantage of this
   opportunity. Instead, it puts these relationships between objects inside those objects' classes,
   as associations or inheritance, thereby turning them into design-time dependencies, and destroying
   the abstract qualities of the classes" (§6.14.5). "Classes are the design artefacts that know
   nothing about one another. Objects are the run-time artefacts that communicate with one another at
   run-time" (§2.3.3). In ALA's words: "ALA addresses the problem by calling classes abstractions and
   objects instances" (§6.14.5; §3.2.2).
   - *In an OO language:* every relationship between two objects is written in a third class, the
     one above that creates both. *Rules:* R1, R8, R11.
3. **Why objects at all: four reasons, derived from procedures** (§3.9, §3.10). Spray gets to
   objects from procedural code in five steps, introducing "objects but not object oriented
   programming per se" (§3.9). The four reasons an ALA class has fields: "Objects store references to
   other objects to which they are wired"; "Domain abstractions, being reusable entities, often need
   configuring. The object stores its own configuration data"; "Some abstractions naturally have
   state"; and "There is usually some state data that doesn't belong with any code. In ALA we often
   create a special domain abstraction called 'State<T>'" (§3.10). He also defends the syntax:
   "Classes have a fundamental advantage over structs and procedures because with structs and
   procedures, the caller must specify the struct and the procedure both from among all visible in the
   scope. With classes, the caller specifies primarily the object and then a method from among the
   methods of that class only" (§3.10).
   - *In an OO language:* a field that is none of the four (a cached peer, a "context" handed around,
     a reference to the application) is a smell. Wiring and configuration fields are set once;
     only state fields change (§3.9). *Rules:* R4, R2, R9.
4. **Messages, sent fully polymorphically.** "The thing that OOP actually brought to the table has
   nothing to do with encapsulation, inheritance, or polymophism. It was to think about programming as
   passing messages… A procedure's name describes what it does. A method's name is a message name
   describing something that has happened elsewhere… Full polymorphism is not even knowing what the
   receiver is" (§3.10). "ALA is always fully polymorphic. All messages are sent polymorphically"
   (§3.10), and in a way "extremely polymorphic… from the point of view inside an abstraction sending
   a message out of it port, there is potentially an infinite number of different abstraction types
   that it could go to. In conventional OOP, it's typically a finite set" (§3.10.1). The usual cost of
   indirection, a flow that's hard to trace, doesn't arise, because the wiring "to do with a given
   user story is in one place" (§3.10.1; §4.4.2).
   - *In an OO language:* every call to a peer goes through a port, and the sender can't tell which
     class receives it. A call whose receiver's class the sender could name is not yet a port.
     *Rules:* R9, R1.
5. **A class has two interfaces: its main interface, for the layer above, and its ports, for
   peers** (§2.3.4, §7.24; Summary). The constructor, setters and properties configure; everything
   at run time goes through paradigm interfaces. Ports are private fields of an interface type
   (accepted) or explicitly implemented interfaces (provided), named, listed in the header comment,
   and wired from outside. See R9 for the details and their reasons. *Rules:* R9, R3.
6. **No associations: the UML class diagram of an ALA program has no lines.** "If a UML class
   diagram were drawn of an ALA application, there would be no lines at all, just boxes in space
   arranged in layers" (§2.1.3). "Class diagrams are evil. I think they have done more damage to
   software architecture than any other meme in our industry" (§2.1.3). The one legal relationship
   is using a more abstract class or interface by name, "inside the class. It's part of the class's
   internal implementation", so it isn't drawn either (§2.1.3). Composition, aggregation and
   realization are allowed only downward: "instantiating a class, using objects of a class or
   interface, or implementing an interface in a lower, more abstract, layer… never up or across
   within a layer" (§2.1.3). The whole-part pattern is "only used with knowledge dependencies" (§7.4.5):
   water is made of oxygen and hydrogen, but oxygen doesn't contain hydrogen. The diagram that
   matters is the application's: boxes are instances, "close to a UML object diagram", but static
   (§6.14.5).
   - *In an OO language:* a reviewer can draw the class diagram of the domain layer and expect no
     lines between its classes. Any line is a defect: an association, an injected peer, a peer
     created with `new`, a shared base class. *Rules:* R1, R9.
7. **No inheritance; compose instead.** "ALA doesn't need or use inheritance. It would break the
   abstraction of the (more abstract) base class in the lower layer. Instead we always use
   composition" (§2.1.3). "Inheritance is now seen as a big mistake. That means that polymorphism in
   the form of virtual methods are also a mistake. That leaves interface polymorphism" (§3.10). "I get
   the impression that most inheritance is lazy coding of what is really composition" (§7.16).
   - *Shared parts become abstractions:* "given the concepts of vehicle, car and truck… If we restrict
     ourselves to composition, common parts of the vehicle domain would be invented as their own
     abstractions in the lower layer… the word, 'vehicle', would mean a category or domain, not a
     part in itself" (§2.1.3).
   - *Shared behaviour becomes a domain abstraction with ports:* "In the domain layer we may
     implement a behaviour class that implements general driving behaviour. Sych a Drive domain
     abstraction may have ports that are connectible to other domain components' ports such as engine
     control port, and brake control port… Most variations would be handled by a configuration
     interface" (§2.1.3).
   - *Up-calls use passed-in lambdas, observers set up by the wiring, or strategies, not virtual
     methods* (§2.1.3, §4.4.1).
   - *Delegation is explicit:* a composing class passes calls through itself, so "no virtual methods
     (indirections) are involved and everything is explicit" (§7.16).
   - *The one use Spray leaves open is versioning,* representing a new version of an abstraction as
     old form plus differences, and he calls even that "a highly dubious concept… not sustainable" and
     "a topic for future research", noting that "even this scenario doesn't require inheritance. The
     difference abstraction may just be composed of the older unchanged abstraction" (§7.16).
   - *In an OO language:* a superclass in your own code is a finding; extending a framework's base
     class to configure the framework is the R9 exception, kept thin. *Rules:* R1, R7, R9.
8. **Encapsulation, polymorphism and inheritance are replaced, not kept.** "ALA replaces
   encapsulation with abstraction. ALA removes associations and inheritance and instead uses
   composition (provided the composition uses a more abstract abstraction). ALA replaces polymorphism
   with zero coupling" (§6.4). Polymorphism, information hiding, protected variations, the dependency
   inversion principle and the open-closed principle are, in his reading, five names for one
   pattern: client B talks to C1 or C2 through an interface. "It's important that we realize that in
   this pattern the interface is owned by B. It describes what B requires… Therefore C1, C2 etc have
   a dependency on B… So this is illegal in ALA" (§6.4). ALA keeps an interface, but at the level of a
   programming paradigm, and adds the abstraction A that does the wiring (§6.4).
   - *In an OO language:* "program to interfaces" (a phrase Spray adds to the five) is not
     enough; ask whose interface. *Rule:* R9.
9. **SOLID, read through ALA** (§6.5).
   - *Single responsibility* becomes "what do you know about?": "It is always one thing it knows
     about, but it may have multiple responsibilites for that thing" (§6.5.1). See R6.
   - *Open-closed:* "None of them are principles - they would need to be used in the right conext at
     best" (§6.5.2). In ALA, new behaviour usually comes from new wiring of unchanged abstractions
     (the coffee maker's coin device, §2.9.3; the calculator's rows, §5.8.5).
   - *Liskov substitution* and *interface segregation* are "TBD" in §6.5. Interface segregation
     appears elsewhere as the split between the main interface and ports (§2.3.4), configuration
     apart from data (§3.9), and one port per peer (§7.2.6). Spray says nothing about Liskov; with no
     inheritance it has little to apply to, except that every implementation of a paradigm interface
     must honour the paradigm's meaning. *Checklist reading.*
   - *Dependency inversion* "goes some way toward ALA in one respect and far too far in another"
     (§6.5.5). Where DIP turns `B → C` into `B ← C` with an interface B owns, ALA makes it `B → I`
     and `C → I`, with I a programming paradigm, plus `A → B` and `A → C` for the wiring (§6.5.5). And
     ALA is looser where DIP over-reaches: "If a low-level module is much more abstract, ALA allows to
     keep the dependency… I want to commit to only one implementation of the framework. It would be
     silly to have to use ports on every single domain abstraction so I can wire in a framework of my
     choice… I don't need to allow for swapping out the math library implementation" (§6.5.5).
   - *In an OO language:* don't put an interface in front of every framework or library a class
     uses. Ports are for peers and for technical domains you wire sideways (a database, a device), not
     for the language's or framework's stable abstractions. *Rules:* R6, R9, R1.
10. **Dependency injection, but explicit, by hand, and to paradigm interfaces** (§3.10.1, §3.11.3,
    §6.2.1, §6.6). "Ports can also be thought of as just conventional dependency injection, but with
    two additional constraints": the interface "must be significantly more abstract", and the
    wiring "must be explicit… The wiring cannot be done by using a dependency injection container or
    relying on matching interfaces" (§3.11.3). It also shouldn't be data: "the explicit wiring should
    not be XML or JSON" (§6.6). Spray doesn't even like the name: "it doesn't really make sense to
    call what we are injecting 'dependencies'. We just think of it as wiring things up. You wouldn't
    describe what an electronics engineer does as dependency injecting components into each other"
    (§6.6). And he notes the DI pattern "came too late to make the famous GOF patterns book. The
    authors wish they had included it instead of singleton" (§3.10.1).
    - *How to inject:* a wiring operator from the Foundation layer (`WireTo`/`WireIn`), which in C#
      uses reflection to fill private port fields; or a setter per port, which the C++ project used
      ("wired them together using dependency injection setters", §7.26.2); but not constructor
      injection, "because sometimes wiring a port is optional" (§2.2). Reflection "is just a
      syntactic shortcut that allows domain abstractions to have many ports without also having many
      setters. It also allowed us to keep the ports private from direct access by the application
      layer" (§6.6).
    - *In an OO language:* a DI framework is fine as a way to *run* code (a framework that constructs
      controllers, say), but not as the thing that decides which instance meets which. The
      application decides each wire by name. *Rules:* R9, R8, R1.
11. **Fluent wiring is how the text form follows the diagram** (§1.6.5, §5.8.3, §7.6, §7.26.2). `new`
    returns the object, `WireTo` returns its first operand, `WireIn` its second, and configuration
    setters return the object too, so a tree of instances reads as nested calls with anonymous
    instances. Cross-connections need named locals. "This is called fluent syntax" (§1.6.5). *Rule:*
    R8.
12. **Threads: one by default, and never locks inside a class** (§3.10.2, §3.10.3, §4.4.9, §4.4.10).
    "The first strategy in ALA is to use a single thread by default" (§3.10.2). Long-running work
    becomes a state machine, async/await, callbacks, or chained tasks, not a blocking thread; "it is
    still better to implement a state machine manually than to resort to multiple threads to solve the
    problem" (§3.10.2). When threads are needed for performance, "synchronous message passing may only
    be used between instances of abstractions allocated to the same thread. Instances that are
    allocated to different threads must be wired using asynchronous message passing" (§3.10.3), and it
    is the user story that allocates instances to threads, because "performance requirements are
    requirements too" (§3.10.3). Locks in classes couple them (§3.10.2); "when you use multiple
    threads, you use the messages and processes execution model" (§3.10.3).
    - *In an OO language:* a domain class needs no thread-safety code if the wiring obeys the
      convention; a class that locks, or spawns threads to make itself responsive, has taken an
      execution-model decision that belongs to the wiring. Private fields don't make an object safe:
      "Shared state occurs all the time in object oriented programs. Any objects accessed from
      different threads are shared state even if all state in an object is private" (§4.18).
    - *Transactions are a user-story concern, wired.* Even on one thread, asynchronous calls
      interleave, so a resource that must not be interrupted (a database transaction, a robot arm)
      needs locking. Spray's answer is an arbitration paradigm: "All instances using a given resource
      are wired to a single instance of an arbitrator abstraction. Effectively this wiring specifies
      the collaboration that must occur between the instances. This collaboration is done at the
      abstraction level of the system, where it belongs, not inside the abstractions" (§4.15; also
      §4.4.10, "This needs to happen at the user story level becasue it is the user story that
      understands transactions"). His `IArbitrator` has an awaitable lock method and a release. A
      shared resource that is busy gets a queueing intermediary in front of it (§4.4.12).
    - *Priorities belong to the application too:* "Priorities are generally a system wide concern,
      so the application abstraction (or feature or user story abstractions) are the only ones that
      have the knowledge to know how to set priorities" (§4.4.11), for example as a parameter of the
      asynchronous `WireTo`. *Rules:* R4, "Kinds of connection".
13. **Push by default, and execution models chosen at wiring time** (§3.10.2, §4.4.3, §4.4.6).
    "In ALA, for the dataflow programming paradigms we usually default to using a push execution
    model. This makes it easier to wire for either synchronous or asynchronous messages" (§3.10.2).
    Incompatible ports (push into pull, synchronous into asynchronous, different locations) are
    joined by intermediary objects that the wiring inserts, which a wiring operator can do itself
    through registered overrides (§4.4.6, which Spray marks TBD). *Rule:* "Kinds of connection".
14. **The classic design patterns, placed in layers.**
    - *Factory method:* "The IProduct and ICreator interfaces are in the ProgrammingParadigms layer…
      The Client and all the different ConcreteProducts are in the DomainAbstractions layer… The
      ConcreteCreator is in the Application layer" — "But in ALA we typically accomplish that in a far
      simpler way": the application `new`s the right concrete class and wires it (§6.11). Where
      instances must be created at run time, a generic `Multiple` takes an `IFactory` port (§6.11.1,
      §5.8.5). Spray's second case puts a persistence object in a paradigm-layer variable the
      application sets once, so every `Table` can reach it without being wired to it (§6.11.2). That
      behaves like the global Style he warns about elsewhere (§2.4.1): "it effectively makes the
      style object a global… if you want to say test a UI domain abstraction with styles, and do
      these tests in parallel, the global wont work". His alternative there is to wire one instance
      to every compatible port with a `WireMany` operator (§2.4.1). Keep the paradigm-layer variable
      for concepts nearly every instance needs. *Checklist reading* of the trade-off.
    - *Observer:* only inside a paradigm interface, for calls against the wire's direction, where
      "the subscriber does not know the publisher" (§4.4.2); never between peers. Spray rejects
      `IObservable` as a port because "the destination wires itself to the source" (§6.3.3).
    - *Strategy* and passed-in lambdas: the legal up-call (§2.1.3, §4.4.1).
    - *Decorator:* a dataflow instance wired between two others is one; Spray's `LowPassFilter` "is a
      DataFlow paradigm decorator" (§7.6), and paradigm interfaces give the wiring "compositionality.
      For example, two domain abstractions currently wired together can have another domain
      abstraction, which is a decorator such as a filter, wired between them" (§6.17.5).
    - *Adapter:* a lambda at the wire, or an adapter instance in a layer above the things it adapts
      (§6.17.1, §6.17.4, §6.17.5); preferred less than a shared paradigm interface (§6.17.4).
    - *Bridge:* Spray's version of hexagonal architecture uses "the Bridge Pattern to keep cohesive
      knowledge belonging to the application from being split" (§6.19): the application configures a
      domain abstraction of the database or UI, which passes the work on through a port.
    - *Facade and packages:* a facade over a hidden package doesn't make an abstraction of its parts;
      they stay public (§2.1.3).
    - *Singleton:* not named as a pattern by Spray beyond the GoF remark above. A singleton that peers
      reach by name is a global, and fails R1, R2 and R9. *Checklist reading.*
    - *Whole-part:* only with knowledge dependencies (§7.4.5).
    - *Composite, decorator and prototype in one paradigm:* Spray's game-scoring engines are built
      from one `IConsistsOf` paradigm. `Frame` forms a composite tree at run time, `Bonus` and
      `WinnerTakesPoint` are decorators that both implement and accept the interface, and
      `GetCopy` is the prototype pattern, used to start each new child frame (§4.20, §4.21). The
      rules of bowling are about 8 lines of wiring and lambdas, and tennis about 15 (§4.21.2).
    - *Strategy class from the application layer:* configuration can be "a whole object of a class
      that you write in the application layer (which is the strategy pattern)" (§2.2).
    *Rules:* R1, R9, R8.
15. **Layered OO architectures, compared** (§6.14.4, §6.17, §6.19).
    - *Presentation / application / domain / infrastructure:* "The middle two layers appear to be the
      same as ALA's", but presentation and infrastructure "are not present in ALA" as layers: UI is
      composed from domain UI abstractions like everything else, and persistence is a domain
      abstraction such as a persistent `Table` (§6.14.4).
    - *Clean architecture:* the business logic's own view is close to ALA, except for its entities
      (R10); its inverted dependencies aren't, unless removed with adapters in a layer above (§6.17.1).
      "A system built from a wiring layer at the top, then an adapters layer below that, and then a
      layer below that for independent features, use cases, databases, UIs etc is ALA compliant"
      (§6.17.5), and it can be mixed with the paradigm-interface style (§6.17.5).
    - *Hexagonal:* ALA keeps the sideways ports, but puts a domain abstraction of each external system
      in front of them, because "The Database and the UI will have a lot of application specific
      knowledge given them as configuration" (§6.19).
    - *Separate by feature first:* "In ALA, the primary separation is by features first. The UI and the
      business logic for a particular feature is considered to be cohesive" (§6.17.3).
    - *Swapping technology* is substituting one domain abstraction for another with the same
      paradigm ports, a wrapper rather than an adapter (§6.17.6).
    *Rules:* R10, R9, R11.
16. **Static or dynamic typing.** A port's data type comes from the application, as a generic
    parameter (`IDataFlow<T>`) or by inference (§4.8.1, §7.6). With a static type system, a state
    abstraction "will be a generic, and the struct type is passed to it at compile-time"; with a
    dynamic one, "a state abstraction could hold any complex data structure… Only the application
    layer would know the actual structure of the data at design-time. Or it may be completely dynamic
    until run-time" (§3.9). His dynamic CSV example carries `ExpandoObject`s and its header lines
    define the types at run time, so mistakes surface as "run-time exceptions… rather than compiler
    errors" (§6.3.1).
    - *In a dynamically typed language (this checklist's reading):* paradigm interfaces still exist,
      as named modules or documented message sets in the Programming Paradigms layer, and the wiring
      operator can check at start-up that each end responds to the paradigm's messages. Without that
      check, the first wrong wire shows up at the first message. Duck typing makes owned interfaces
      invisible, not absent (R9). *Rules:* R9, R4.
17. **Files, namespaces and folders are the layers.** One abstraction per file (§2.3.2), three to five
    folders named for the layers, and namespaces that match them: "Knowledge dependencies only go down
    these layers… There are no dependencies between files in any folder" (§2.3.1). Namespaces aren't
    encapsulations: "namespaces are not encapsulations. Namespaces only make names unique", and a tool
    that drew dependencies on namespaces "gave a completely misleading view" (§2.1.3). "Unfortunately,
    there is no convenient way of telling the compiler or the IDE to not 'see' classes, interfaces etc
    in other files in the same namespace or folder" (§2.3.1), which is what a layer check in CI is
    for. A readme at the root points to ALA as a knowledge prerequisite (§2.3.7). *Rules:* R1, layer
    assignment.
18. **Writing a domain class: Spray's conventions** (§5.8.2, §4.4.4, §5.6.1, §8.5). "Abstractions are
    generally trivial to implement because they are zero coupled with anything. They are like tiny
    stand-alone programs" (§5.8.2). His C# class layout:
    - a doc comment with the concept and a numbered list of ports, repeated on the constructor;
    - an `InstanceName` property (or an immutable `Name` set by the constructor);
    - a "Properties" section of optional configuration with simplest-form defaults; required
      configuration in the constructor (§5.5);
    - a "Ports" section of private interface-typed fields;
    - provided ports implemented explicitly;
    - required ports used directly (an unwired one throws), optional ones null-checked;
    - a private `<port>PostWiringInitialize` method where a port's interface has events to subscribe
      to;
    - in the file, "the implementation of the public configuration interface first, then the ports
      that are implemented as fields, then any other internal state, then the implementations of the
      ports" (§2.1.1);
    - a comment naming the knowledge the reader needs, such as "You need to understand the
      programming paradigm abstraction, IDataFlow, to understand this code" (§2.1.1, §2.2), or a
      device's datasheet (§2.1.3).
    - *Write it as if you didn't know the application:* "After the need for an abstraction is
      decided, pretend you don't know anything about the application, and are writing something to
      be useful, reusable and learnable as a new concept" (§2.2). Spray's counter-example is a
      `LowPassFilter` written as a module of one application: "knowing which application it is part of
      may cause it to take on ancilliary functions, such as offset and scale to output the units needed by the application. Or it might
      grab its input knowing where it comes from. Or it might send it's output to a specific place…
      It might connect to the application's settings menu for its settings" (§2.1.1).
    - *Generalize by adding configuration with a default.* "Often you can generalize an abstraction
      to make it more reusable by adding a configuration. The configuration has a default, so it
      doesn't affect existing uses of the abstraction" (§3.3.2); Spray's `Frame` was generalized from
      one player's score to two for tennis (§4.21).
    - *In an OO language:* the same sections work in any class-based language; what replaces the
      reflection-based private ports depends on the language (setters, public port fields assigned by
      generated code, or a small wiring operator in the Foundation layer). *Rules:* R6, R9, R3.
19. **Testing objects.** "You always test with dependencies in place, but you mock the ports. Just as
    you would not mock out a dependency such as squareroot, you do not mock any dependencies in ALA"
    (Summary). A domain class's unit test wires fakes to its ports; testing the application "is
    exactly acceptance testing" (Summary). On TDD: making modules testable improves the design, "but
    in my experience, TDD doesn't create good abstractions nearly as well as pursuing that goal
    directly" (§6.8).
    - *In an OO language:* a mocking library that replaces a class by name, or a header interface
      created so a peer can be mocked, is the failing case; a fake implementing a paradigm interface
      is the passing one. *Rule:* "Tests replace only ports" in the verify procedure.
20. **Legacy OO code.** Reverse-engineer one user story at a time: "I build a UML class diagram from the
    searches (their one useful application) as the background… and a tree of method calls for the
    specific user story on top of it… especially when inheritance is involved. The tree of method calls
    will come into the base class but leave from a subclass method" (§8.3). Then pin it with acceptance
    tests and factor it into new abstractions (§8.3). A coupling metric helps find where to start only
    if it counts bad dependencies alone (Summary, CBO). *Rule:* the refactor procedure.
21. **What an ALA language would add, and what stands in for it now** (§2.3.2, §7.2.6, §8.6). "A port
    is not an artefact of programming languages (yet) so they must be implemented logically somehow
    as normal code" (§7.2.6). His ALA language would have abstractions and instances as first-class
    elements, layers the compiler checks, ports "defined in a lower layer", "multiple ports of the same
    interface", and ports configurable as push, pull, synchronous or asynchronous "without changing
    the Abstraction" (§8.6). He also wishes for an `Abstraction{}` construct whose public members are
    "only visible to code in higher layer abstractions… If we had this, we would have compiler checking
    for illegal dependencies" (§2.3.2).
    - *In an OO language:* conventions stand in for each missing feature: a ports section and naming
      convention for ports, a wiring operator for connecting them, connector objects for repeated
      interfaces (§4.4.5, §5.8.3), and a static check with a layer map for the visibility the compiler
      can't enforce. *Rules:* R1, R9, R8.

## Functional techniques in an OO program

Spray's functional material (gathered in the functional edition of this checklist) mostly concerns
using functional tools inside an object-oriented ALA program. These points carry over.

1. **Plain function or method composition is fine for an algorithm.** "If your whole problem is just
   an algorithm, and therefore suits a functional programming style, then you can still compose
   abstractions with function abstractions, provided all function calls are knowledge dependencies,
   and not say, just passing data or events" (§7.5). Ports and wiring are for instances that
   communicate. A pricing calculation that calls a money type and a tax-rate method needs no ports.
   *Rules:* R1, R7.
2. **Abstraction before referential transparency, both ways** (§3.11.2, Summary). The object form
   (state kept in the instance) and the functional form (state passed in and out) are chosen "on a
   case by case basis". An immutable value object returned from each step is fine inside an
   abstraction; making the *caller* hold another concept's state is not. *Rule:* R4.
3. **Use the library's chaining instead of reinventing it** (§6.3, §3.11.3). "We wouldn't normally
   create ALA domain abstractions to do the same jobs as monads if we already have a monad library.
   Instead, we would create a domain abstraction as an adapter with ALA ports. Into this adapter we can
   simply plug in a monad chain" (§3.11.3). The two ways: give some ports the library's chaining type
   and put library operators between two instances (§6.3.1), or write a general domain abstraction
   configured with a chain (§6.3.2). The Summary calls such an abstraction a *query*. In C# that's
   LINQ or Rx; elsewhere, the language's streams or `Enumerable`. *Rules:* R7, R9.
4. **Reactive-extension types are poor ports** (§6.3.3, §3.11.4). `IObservable` has "the destination
   wires itself to the source", stops for good after `OnCompleted` or `OnError`, and mixes push and
   pull, so Spray defines a pure-push paradigm, `IObserverPush<T>` (`IObserver<T>` plus `OnStart()`),
   that the layer above wires once and that can carry batches and errors (§6.3.3). *Rules:* R9, R1.
5. **Lambdas: anonymous for one-offs, passed in for configuration, for calling up, and for adapting**
   (§6.1, §1.6.3, §3.11.3, §4.4.1, §6.17.4). A named method used once "is not an abstraction… It's
   indirection without abstraction" (§6.1). *Rules:* R6, R9, R11.
6. **Objects are easier to compose than monads, for most programmers** (§6.2.1, §3.11.3). "ALA's
   domain abstraction objects are easier to understand than monads because they are plain objects…
   ALA is like composing ICs (integrated circuits with many pins with many functions) and monads is
   more like composing two-port components such as resistors" (§6.2.1). "Because ALA uses plain
   objects, and plain interfaces as their ports, ALA developers can add new domain abstractions and
   programming paradigms themselves… In ALA, the set of domain abstractions and programming paradigms
   that you write is a DSL" (§6.2.1). And unlike a monad chain used once, an ALA program is "wired up
   once at the beginning… and then all ports are considered infinite steams" (§3.11.3). *Rules:* R8,
   "Past the pipe".
7. **Immutability is a help, not the answer, for threads** (§3.10.2). Immutable configuration and
   wiring fields (§3.9) remove most sharing; for the state that remains, the single-thread and
   asynchronous-between-threads conventions do the work (OO point 12). *Rules:* R2, R4.

## Where this checklist departs from Spray

The rules follow Spray, and each one cites where. These are the places where this checklist adapts
his C# practice to object-oriented languages in general, or steps away from him, stated so a reader
doesn't mistake them for his positions.

| Departure | Spray | This checklist | Why |
|---|---|---|---|
| Branches in a UI composition | "no ... if statements" in the application (§3.5) | handler methods that only forward, and branches that only route an ok and an error result, read as routing; a lifecycle check tolerated where the framework requires it (R11) | the framework delivers events and a lifecycle to the screen; the departures are recorded, kept small, and never used for logic, and most can move to a framework hook |
| Size | a fundamental constraint (Summary) | class or file size is an advisory check (R7): one over 500 lines is scored only in a strict mode, and an average under 100 lines is reported without scoring | line counts are a weak proxy for "readable alone", so a reader decides |
| Graded checks | ALA's constraints aren't graded | enforcement tiers: R7 and height advisory, R11 and public surface aspirational, the app-layer share, average size and shared aggregate reported only | lets a team adopt the checklist step by step; the tier is about scoring, not about whether a finding is real |
| Sharing data between features | no data coupling (§3.8), no shared DTOs (§4.8.1), no shared entities (§6.17.2) | R10 states the same property, applied to features: no domain type read by two features | not a departure in substance |
| The notation | diagrams and wiring code (§3.6) | a text encoding with `[tag]`, `$`, `q`, optional `ports:`, and tool-stamped marks | a whiteboard- and linter-friendly way to see the shapes; it isn't Spray's |
| Framework residue in a UI composition | the application is wiring and configuration only (§3.5) | param decoding, a URL-to-step lookup, a client-side hook or two, and component instances kept alive but hidden are recorded as departures | the browser sends strings, the framework hands the screen the URL, some things only the client can do, and some frameworks can't deliver to an unmounted component |
| Wiring mechanism | reflection-based `WireTo`/`WireIn` extension methods with private port fields, his preference, not a requirement (Summary, §2.2, §6.6) | any explicit mechanism meets R8 and R9: a wiring operator, a setter per port, public port fields assigned by generated code, or a framework's own explicit wiring | Spray says reflection is optional; languages differ in what a wiring operator can reach |
| Framework base classes | "ALA doesn't need or use inheritance" (§2.1.3); his own UI abstractions wrap WPF (§4.12) | extending a framework's base class to configure it (a controller, a component, a test case) is allowed as configuring a far more abstract class, kept thin; your own hierarchies are not (R9, R1) | most OO frameworks are extended by subclassing; refusing them would rule out the frameworks, which isn't Spray's point |
| Dynamically typed languages | paradigm interfaces are C# interfaces; dynamic *data* uses `ExpandoObject` (§6.3.1) and dynamic state abstractions (§3.9) | a paradigm interface may be a module or a documented message set, checked at wiring time; duck-typed calls to a peer's own methods are an owned interface (R9) | the language has no interfaces to declare, but the coupling Spray describes is the same |
| Execution model | prefers single-threaded solutions (§3.10.2, §4.4.9) and treats the execution model as a wiring-time choice | the same property: execution-model choices are made where the instances are wired, and state belongs to its concept (R4, "Kinds of connection") | not a departure in substance; listed because thread pools and async frameworks make it tempting to decide inside a class |
| Comments | "critically important" abstraction comments (§2.1.1); triple-slash summaries on class and constructor (§5.6.1) | a short class comment naming concept, ports, configuration, and an example when needed (R6) | covers what Spray asks for without narrating bodies |

## Enforcement tiers (what a linter scores, and when)

Not every rule is the same kind of obligation. The coupling and locus rules are hard requirements; the
minimality and shape rules are prompts a reader weighs; a couple are aspirational ideals a real app
cannot fully reach. A linter can encode this as tiers:

| tier | rules and sub-checks | when scored |
|---|---|---|
| **Required** (a violation is a defect) | R1 (including fields and parameters typed by a peer, a peer created with `new`, and a superclass in your own layers), R2, R3, R4, R5, R6, R9 (owned interfaces, abstract base classes used as ports, container or locator wiring), R10, layer-validity | always (default) |
| **Advisory** (a prompt for a reader) | R7, module-size (over 500 lines), abstraction-height, pass-through, tramp parameters (R6's "should"), reference-level R1, self-subscription (the bottom layer may own its topic) | reported by default; scored in a strict mode |
| **Aspirational** (a purity ideal, not always obtainable) | R11 (no logic at the top), public-surface (encapsulate the little ball of mud: public methods beyond the constructors, configuration and explicitly implemented ports; counted per method) | reported by default; scored only in the strictest mode |
| **Reported only** (a ratio or a design choice, not a defect) | the application's share of all functions, files averaging under 100 lines, the shared domain aggregate | reported at every tier; scored only when a team opts in |
| **Not machine-scored** | R8 (judgement); the rest of R9 (outputs that name a destination or command, peer DTOs) | a human reads for these |

Two placements are deliberate and follow from the "little ball of mud" reasoning under R7. **R11** is
aspirational, not required. A UI composition can reach zero findings, but only by adopting a design
built for it (results on ports bound by the composition, a circuit of instances, or feature instances
that announce what they did), and a CI gate shouldn't choose the design for a team. Its findings are
still real, and each one is either moved or recorded as a departure (see R11).
**Abstraction-height** and **pass-through** are measured on the graph of *abstractions*, not the raw
call graph: a call inside one class is internal decomposition, so it adds no height and is no
pass-through; only real hops and public cross-class renames between abstractions count. A
delegating method that replaces inheritance (§7.16) will show as a pass-through; that's expected. And
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
  must know its parent, and "It would break the abstraction of the (more abstract) base class", §2.1.3),
  **no associations between peer classes**, not even through a peer's interface or a base class
  (§2.1.3), **no global event names, no receiver subscribing
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
- **Two phases, and the diagram is the source (behind R11).** Wire the network once, then run it (an
  application class that `new`s and wires every instance, then calls `Run()`, is exactly this). The topology/"diagram" is the
  single source of truth for requirements, architecture, and code, and should be flat, declarative,
  and greppable. (Committed generated code is one way to make that mechanical.) Spray: "the entire
  application is wired up once at the beginning when the application starts executing, and then all
  ports are considered infinite steams that work as long as the application is running" (§3.11.3;
  also §1.6.4, §2.4, §3.6).
- **Abstraction is not the same as instance (behind R6/R9).** The abstraction is the zero-coupled
  design artefact; the instance is the run-time thing that communicates. Blurring them is what tempts
  people to put dependencies *between abstractions* to move data, destroying them as abstractions.
  (§3.2.2.) In an OO language, a class is the abstraction and an object is an instance: "Object
  oriented languages of course already have these two concepts as classes and objects" (§3.2.2).
  The UML class diagram blurs them anyway, because it "encourages you to create relationships between
  classes, destroying them as abstractions" (§3.2.2).
- **Execution-model choices are design-time, made on performance/expressiveness grounds.** Push by
  default, pull for performance; decide sync vs async at wiring time (an asynchronous message vs a
  synchronous call, a broadcast vs a direct send), never inside a domain module; model time-spanning
  activities as state machines, not threads (§4.4, §4.19). Spray calls the usual rule of thumb, GALS
  (synchronous locally, asynchronous across processors), "too simplistic": anything that takes real
  time, such as I/O or a delay, should be asynchronous even on one processor (§4.6.1). In an OO
  language that means async/await, tasks, callbacks or a state machine inside the abstraction, and a
  queue or event loop chosen by the wiring, rather than a thread per object (§3.10.2).

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
   abstractions, §8.1.) Spray's specific prompts (§5.2.2): data on its way between end points "may
   need to be transformed, aggregated, filtered, sorted, validated or transacted… All of these are
   great candidates for domain abstractions"; a protocol is one abstraction, and "protocols on top of
   protocols" are two; a file format is one, and a header-row convention on top of it is a second; a
   hardware device is one ("everything that is in the datasheet"), and its SPI bus another. Each "will
   usually handle the data going in both directions". Expect about fifty domain abstractions in a
   typical application (§7.3.2).
3. **Choose each paradigm's execution model** on performance/expressiveness grounds (push vs pull,
   sync vs async), keeping that choice out of the abstraction and in the wiring. (§4.4.)
4. **Implement, then wire.** Each abstraction is an independent little program, fast to write because
   there is no coupling to reason about. Use **convention over configuration** (enforced config in the
   constructor, optional settings defaulted); give every instance a **name** for debuggability (an id
   in its configuration for logs and telemetry, not a globally registered name, which would let
   senders find it themselves); put a **root readme** naming the knowledge prerequisites (ALA, the
   paradigms, the domain abstractions, "the diagram is the source"). Velocity climbs as the domain
   matures. (Convention over configuration, §5.5; the readme and knowledge prerequisites, §2.3.7,
   §5.6.) In an OO language:
   - *Write the wiring first, then the classes.* "You write the application code (diagram) first (or
     part of it), just focusing on expressing the requirements. This causes you to invent domain
     abstractions and programming paradigms. Then you come up with an execution model that will make
     the programming paradigms execute" (Summary).
   - *Write the paradigm interfaces before the classes that use them,* each in its own file in the
     Programming Paradigms folder, and the wiring operator in Foundation (§2.2, §2.3.1).
   - *Write each domain class to Spray's layout:* header comment with ports, instance name,
     properties with simplest-form defaults, private port fields, explicit implementations, required
     and optional ports, post-wiring hook ("Object-oriented programming", point 18).
   - *Organize teams by abstraction and by user story.* Domain abstractions can be written by
     people who don't talk to each other; "In fact abstractions will be better quality if the teams do
     not collaborate with each other so that the abstractions themselves do not collaborate" (Summary).
     Teams split by deployed location (frontend and backend, rover and lab) will couple their modules
     through the APIs they agree; Spray wants "a team responsible for each user story" that composes
     the abstractions the other teams provide (§3.8.3).
   - *Keep deployment out of the logical design.* A user story spanning machines is still one
     composition; it annotates instances with where they run, and "Another abstraction sits in a
     lower layer that knows about the concept of a physical view", deploying the instances and
     connecting them to the middleware (§3.8.1).
   - *Expect classes to be quick to write.* The student on Spray's device project wrote 12 of 50
     classes and "as the student completed certain abstractions that allowed parts of other features
     to be done, he would quickly go and write the wiring code and have the other features working as
     well" (§7.26.3).

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
  composition. Spray's own version for classes: "adding dependency injection setters… This step
  removes all uses of the "new" keyword from the class (except ones that are instantiating classes in
  a lower abstraction layer such as your framework)", and "If you have any uses of the observer
  pattern… move the code that does the actual registering or subscribing up to the application.
  Provide a dependency injection setter for it to use" (§5.2.2). In OO code, step (3) has four common
  moves:
  - an association (a field typed by a peer, injected or not) becomes a port typed by a paradigm
    interface you invent or reuse, wired by the class above (§2.1.3, §7.5);
  - a peer created with `new` inside a class moves up to the class above, which creates both and
    wires them (§7.5);
  - a superclass in your own code becomes composed parts and a behaviour abstraction with ports, with
    explicit delegation where the subclass added nothing (§2.1.3, §7.16);
  - an interface one class owns (required or provided) is replaced by a paradigm interface, or, where
    the shapes truly differ, by a lambda or adapter at the wiring (§2.3.4, §6.17.4).
- **Legacy, per user story:** reverse-engineer the call tree for one story (an all-files
  search is legitimate here; in OO code Spray draws a UML class diagram as a faint background with
  the story's method-call tree over it, because inheritance scatters the story: "The tree of method
  calls will come into the base class but leave from a subclass method", §8.3), pin it with acceptance tests at its input/output boundaries, factor the
  call-tree into a new domain abstraction (copy-pasting useful snippets) (§8.3), mark the old modules
  for deprecation, and repeat per story.

### Procedure — verify a program is ALA

Run the mechanical checks first (a tool can do R1–R7 and R10–R11, and R9 in part), then apply the
human tests it cannot. Fail any and it is not ALA.

**First, look for Spray's smells of decomposition** (§7.12.5): hierarchical diagrams (boxes inside
boxes, package diagrams); a dependency graph with many levels; "Encapsulation without abstraction",
whose interfaces "tend to be specific to pairs of modules, and will tend to get increasingly wide";
modules responsible for who they communicate with (a sender that names its receiver, or a receiver
that subscribes to its sender); "all files" searches to trace a flow; and run-time indirection
(the observer pattern or automatic dependency injection) that needs a debugger to follow.

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
8. **Draw the class diagram of the layers below the application.** It should have no lines between
   classes in the same layer, and none going up ("If a UML class diagram were drawn of an ALA
   application, there would be no lines at all, just boxes in space arranged in layers", §2.1.3).
   Each line that appears is an association, an injected peer, a peer created with `new`, or a shared
   base class (R1, R9).
9. **Tests replace only ports:** "you always test with dependencies in place, but you mock the
   ports" (Summary).
   - *Must:* a test never replaces a knowledge dependency (a module the subject relies on for its
     meaning, in a lower layer), just as you wouldn't mock a square root. *Reviewer's test:* does any
     test swap out a module the subject calls by name, rather than an instance wired to one of its
     ports?
   - *A domain abstraction's unit test* uses its real lower-layer dependencies, and wires fake
     instances to its ports.
   - *Testing the application* with its real domain abstractions "is exactly acceptance testing".
   - *For example:* wire a fake into the port (a stub class implementing the paradigm interface, a
     lambda passed in, a recording test double as the output). A mock of a paradigm-layer interface is
     a fake port, which is fine. A mock of a lower-layer class, or a class-level stub that replaces a
     class's methods for every caller, is the failing case. A test of a screen with its
     real features is the acceptance test. A test that must replace a *peer* by name is a sign the
     peer was never behind a port (R1, R9).
10. **Wait for async work in UI tests.** When a screen starts an asynchronous task, a test that reads
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

**In an OO language the class is the natural unit, and calls are only one kind of edge.** A linter
for an OO language should collect every way one class can know another, because R1 and R9 judge
them differently:

| Edge kind | Example | Judged by |
|---|---|---|
| method call on a class, or a static call | `Shipping.Cost(…)` | R1 |
| instantiation | `new Hydrogen()`, `Hydrogen.new` | R1 (a peer `new` is the §7.5 defect) |
| field, property, parameter or return type | `private Inventory inventory;` | R1, and R9 when it's a port typed by a peer |
| superclass | `class Truck : Vehicle` | R1 (inheritance edge) |
| implemented interface or included mixin | `class Light : IDataFlow<bool>` | R9 (a paradigm interface is a port; an owned one is a defect) |
| constant or nested-type reference | `Order::STATUSES` | R1, R5 |
| reopening or patching another class from outside it | adding methods to a lower class from application code | R1: the lower class now behaves as a higher one decided; *checklist reading* |
| container registration or locator lookup | `services.AddScoped<IGateway, Stripe>()` | R9 ("No endpoints") |

In a dynamically typed language, field and parameter types aren't in the source, so the linter sees
fewer of these edges statically; a run-time check at wiring time, or a type-annotation layer, can
recover some of them.

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
   inheriting magic directory names. In an OO language the namespace is the natural pattern, since
   Spray names namespaces after the layers (`Application`, `DomainAbstractions`,
   `ProgrammingParadigms`) and keeps folders of the same names (§2.3.1);
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

**UI-framework pieces as abstractions.** A screen in a server-rendered, component-based or MVC UI
framework is not a stack of application sub-layers (shell → screen → view). Each piece is either the application,
a domain abstraction of one of the kinds above (UI, feature, data source or sink), or a paradigm.
Spray's UI abstraction (see "What a domain abstraction is") sets what the UI pieces may do: render
what's wired in, emit events, contain other UI. *This mapping is the checklist's reading.*

| Piece | What it is | What it may do |
|---|---|---|
| The screen's class or controller (start-up, callbacks, actions, render) | Application | instantiate and configure, wire, hold application literals; no computing, deciding, fetching or persisting of its own (R11) |
| The screen's template | Application: the UI-layout wiring | contain UI abstractions in each other (the UI-layout paradigm), wire data into them through attributes and slot values, name the events they emit |
| Event, message, action and async-result handler methods | Application: wires | route an emitted event, a request, a message or an async result to one input, decoding params at the edge; handlers in one class are one place (R8) |
| The router | Application | it composes screens |
| A stateless UI component (a row, tabs, a button) | UI domain abstraction | render data passed in as attributes; emit events whose names it's given; contain children through slots (its UI-layout port); no state, no I/O |
| A stateful UI widget (a menu, a date picker, an autocomplete) | UI domain abstraction with internal state | keep the state of its own interaction (open or closed, the text typed so far); emit events; never fetch, persist or apply the product's rules |
| A stateful component that holds a feature's value | a host | fine when it's generic (a paradigm that runs any feature, renders only what the screen passes in through slots, and sends outputs); a bundle when it also renders its own markup and does I/O, which is an R6 finding |
| A screen-specific stateful component (a generated form component, a screen's own panels) | a contained sub-component | nothing: it's an R11 finding (§2.2). Use a generic domain UI abstraction the screen configures and wires, or move a reusable one down a layer |
| State-and-rules classes (a cart, an undo offer) | in Spray's terms, stateful domain abstractions; his features are compositions (see "When the wiring outgrows one composition") | objects with state and declared ports; no markup, no events by name, no I/O unless I/O is the concept |
| Store, gateway and placement instances (add a line, place an order, charge a payment, a store object passed as configuration) | data source and sink abstractions | do the I/O they're configured for; wired by the screen into a feature's inputs or from its outputs |
| A domain UI component configured with a rule (a stock indicator) | UI domain abstraction | render passed-in data using a configured rule instance |
| A generic shell, effect interpreter, runner or host | Programming Paradigms | an execution model: it knows how a kind of connection runs, not what this screen does (Spray keeps "Execution models.doc" in this layer) |
| The UI framework itself | Programming Paradigms / Foundation, library-provided | its event → state → render loop, or request → controller → view cycle, is an execution model; a screen or controller subclassing its base class or implementing its callbacks is configuring a more general class (R9) |
| Persistent model classes (active-record or ORM entities) | usually a shared entity plus a store, two concepts in one class | as a domain abstraction of a persistent table, configured by the application and wired to the features that use it (§6.19); not read and saved by every feature (R10, R9) |

What follows from the table:
- **Data is wired into UI, never fetched by it.** A component that calls a store to get its rows, or
  to save a change, is a UI abstraction with a data source or sink inside it. Pass the rows in as
  attributes or slot values, and wire its change events to a store instance at the screen (§5.2.2,
  R6).
- **UI elements have input ports too.** "It is common these days for GUI elements such as buttons,
  menu items, etc to have event-driven output ports. But then we often just wire them to imperative
  methods with a dependency. In ALA you create input ports as well. For example all popup window
  abstractions such as file browsers, wizards, settings, navigable pages, etc have input ports. The
  main window has a close input port" (§3.5.1). A button whose handler calls into a named business
  class is the dependency Spray means.
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
| Event, message, action and async-result handlers | each handler forwards one event, request, message or result to one input, decoding params at the edge; all handlers sit in one class | a handler computes, decides, or builds one feature's input from another's state | R11, R8 |
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
| Publish/subscribe and model callbacks | a framework hook the screen configures subscribes; a handler routes the fact to an input | a class subscribes itself to a topic it names, or a model's save callback reaches into another feature | R1, R5 |
| Navigation | the screen navigates on a feature's `step` through its path table, or to a URL an output carries | a component navigates, or a feature names a path | R3, R9 |
| Forms and validation | a feature or domain class validates; the screen supplies the messages; the form posts events to the screen or host | validation messages live in the feature; the screen validates | R3, R11 |
| Client-side hooks | named in the template for things only the client can do | a hook carries product rules | R3 |
| Screen-specific stateful components | never: a screen composes its UI from domain UI abstractions it configures (a generic record form), and its stateless components hold only markup | a stateful component with its own state and handlers used only inside one screen: a contained sub-component (§2.2) | R11 |

Four settled readings behind the table:
- **Handlers in one class meet R8.** Handler methods on one screen or controller are one place; a
  route table or bindings map is a technique, not a requirement.
- **A UI component that does I/O is an R6 bundle.** A stateful component that loads, saves or charges
  through a store, gateway or placement instance is a UI abstraction with a data source or sink
  inside it. The fix keeps the component: data comes in through a pull port or attributes, changes go
  out as port outputs, and the screen wires the store.
- **A pull port is wiring; a required store class is not.** A read function the screen passes is
  Spray's pull dataflow (his Grid "is able to pull rows of data as needed", Summary). Passing a store
  *object* and calling its own API makes the component define what it needs from a store: an
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

**MVC frameworks.** Spray's section on MVC is "TBD" (§6.10.1), and he discusses the presentation /
application / domain / infrastructure layering only briefly (§6.14.4), so this mapping is the
checklist's reading of his general rules for a request-based MVC framework:
- *The router and each controller are the application* (or, in a big app, each controller is a
  feature-sized composition). An action instantiates or looks up its configured instances, forwards
  the decoded params to one input, and hands the result to a view. An action that computes, decides,
  or queries several models and combines them is R11 logic.
- *Views and templates are UI-layout wiring,* with partials and components as UI domain abstractions
  when they'd read the same in another app (R11's template rules apply).
- *Model classes are where most OO web apps break ALA.* A model that every feature reads, writes and
  queries is a shared entity (R10) and a store each feature names (R1, R9); its callbacks and
  validations often carry several features' rules (R6). The ALA reading keeps an identity abstraction
  shared, gives each feature its own data stored against the identity, and treats persistence as a
  domain abstraction the application configures (§6.17.2, §6.19). This is a large departure from the
  framework's conventions, and a team may reasonably keep models as a persistence layer while moving
  rules and per-feature data out of them; record that as a departure.
- *Business logic can be decorators between a grid and a table.* "A simple Application might wire a
  grid directly to a table. When Business logic is needed, any number of decorators (that do
  validation, constraints, calculations, filtering, sorting, etc.) can be inserted in between the
  grid and the table by changing the wiring of the application" (§7.11.2). This is the ALA
  alternative to rules in model callbacks.
- *A persistent table can be shared without being a shared entity.* Spray suggests a plug-in layer
  for large applications in which "Plug-in abstractions may actually be instances of domain
  abstractions, such as a settings Menu, or a customer Table. A feature can then add settings to the
  menu, or columns to the table that remain unknown to any other features" (§7.11.2). The `Table`
  itself is composed from abstractions ("we compose it from abstractions") in a database technical domain through a polymorphic
  interface one layer down, not decomposed into helpers (§7.17), and it can learn to migrate itself
  when a feature adds columns (§7.26.1).
- *"Service objects" are domain abstractions only if they pass R6, R1 and R9.* A service named for
  one controller's action, which calls three models and a mailer by name, is a working chain moved
  out of the controller (R1), not an abstraction.
- *Framework-wide configuration and conventions* (autoloading, naming conventions that find classes
  by name) are Foundation-layer knowledge, like Spray's convention dependencies (§3.10.3, "a knowledge
  dependency on an underlying convention"). They become a problem only when a class uses them to find
  its own peer at run time (R9's "No endpoints").

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
  ends live. A name that features or domain classes must also know goes through R5.
- **Features are wired by the screen, not by each other.** State-and-rules classes and feature
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
verdicts on forwarding handlers and routing branches are this checklist's reading, not Spray's words.

| Form in the screen | What it is | Ways to fix, for example |
|---|---|---|
| One handler method per event, action or message | routing: each handler is a wire from an event to an input | keep each handler to forwarding |
| Decoding params (extracting fields, parsing integers) | adapting the browser's event format, a technical domain wired sideways (§7.17) | fine when small; if it repeats, a generic param-casting abstraction configured per event |
| Setting screen state to an abstraction's output | the point where dataflow lands (§1.6.6) | nothing |
| Setting screen state to a computed value, or any arithmetic in the screen | data handling | move it into the feature or domain abstraction, and set its output |
| A condition on "nothing", or on success, just to decide whether to go on | a propagation guard | an instance that doesn't push when it has nothing to say, an intermediary that filters, or a library chain inside an adapter (§1.6.4, §6.1.3) |
| A match whose arms only send the ok and error results to different places | routing two output ports (read from §4.9.1) | acceptable as wiring; better still, the feature returns outcomes that an interpreter routes |
| A condition with computation in its arms, or a business rule (`if total > 100 then free_shipping`) | application logic | a domain abstraction, or a predicate configured at the screen (§3.11.3); a state machine if it depends on history (§4.16) |
| Several feature calls in one handler (remove the item, then record the undo) | fan-out wiring, written in order | fine when each call only passes outputs to inputs; say so when the order matters (§4.4.4) |
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

What measuring a real split showed (in a functional-language corpus; not yet repeated in an OO one):
- **Codebases' "features" are often not Spray's features.** Modules or classes called features are
  often coded state abstractions with ports and logic inside. Spray's features contain only
  instances, configuration and wiring. In his terms those classes are stateful domain abstractions:
  an undo offer would serve any app in the domain, and a cart is a storefront's cart. The edges are
  the same either way, so R1 doesn't change, but the name misleads a reader who knows Spray's
  vocabulary: an R8 "should" (−0.5). *Checklist reading.*
- **Few links cross between features, and they meet at a hub.** In one storefront cart screen, 9 of
  29 wires connected one feature to another, and every one touched the cart. The rest wired a feature
  to the screen's own landing points: display values, lists, notifications, timers and store
  instances. Cut by user story (edit the cart, undo a removal, save for later, the wishlist,
  checkout), the screen wired a handful of links between stories.
- **A split surfaces hidden links.** That cut gave 12 links between stories, not 9: three had been
  hidden in shared screen state and in an event one feature's view sent straight to another. A story
  owns its view, so each became an explicit wire.

Techniques for a composition that outgrows the limit:
- **A story class.** A Features-layer class for one user story: it instantiates and configures that
  story's domain abstractions, holds its wires, builds its layout from domain UI abstractions, and
  declares ports for the few links that leave it. The composition composes stories and wires their
  ports. One story can serve several screens, each configuring it differently. Spray's `CalculatorRow`
  is one, implemented by a diagram of its own: "When you implement an abstraction by an internal
  diagram, there needs to be some extra code to wire from the ports… to the internal wiring"
  (§5.8.5).
- **Nested wiring, not merged wiring.** Each story holds its own wiring and the composition holds only
  the links between stories, so nothing has to be merged or delegated.
- **Wire a story's border ports to its inside.** A story class's own ports are paradigm interfaces
  like any other class's, and its constructor builds its internal wiring. Spray's `Temperature` and
  `LoadCell` features show both directions (§2.2, "Features layer"). An input port is implemented by
  forwarding to an internal instance. An output port can't simply be wired to an internal
  instance's output in the constructor, because the composition hasn't wired the story yet, so the
  field is still null; Spray wires an intermediary instead, `ToLambda((d) => output?.Push(d))`, that
  reads the port when data arrives. An input that may be left unwired gets a default through a
  `DataFlowInitializer`. The story also exposes `Run()` for whatever must start. Spray found this
  "quite tricky the first time… But there is a pattern to it" (§5.8.5), and expects a tool to
  generate it.
- **Settle who owns each name after the split.** One class that both fired an event in its template
  and handled it agreed with itself. Split it, and the name becomes a contract between two classes
  (R5). The class whose view fires an event should handle it, with the composition dispatching to it
  from the event list it declares; and a timer or task a story starts should be named by the
  composition and passed to the story as configuration.
- **Split state along with the composition.** A state class whose public surface keeps growing is
  often several concepts. Split it into smaller abstractions wired to each other inside one story, so
  the extra wires live in the story, not on the composition.
- **Don't let the order of wiring matter.** A story that must be wired before it is configured, or
  configured before its border ports are wired, has temporal coupling with the composition (§5.8.5).

Many paradigms and specialized ports are the norm in Spray's projects, not an extra: "we will for the
first time use multiple programming paradigms, a usual thing in real ALA projects" (§7.14). Each one
is also something a team has to learn, which is a real familiarity cost.

### How close a UI composition can get

**What a UI framework hands the screen.** Depending on the framework: a start-up callback or
constructor (in some frameworks twice: once for a static render, once when a live connection opens);
a handler or controller action for every client event or request, named by a string with string
params; a callback whenever the URL changes, including browser back; a callback when an async job
finishes; and a render function or a template. Some of this is Spray's model under another name: the
screen's fields are where dataflow lands (his `temperature` connection, §1.6.6), the template's
nesting is the "display inside" wiring, and a UI thread or a request handled on one thread is the
single-threaded execution he prefers anyway (§3.10.2, §4.4.9). Other parts are foreign: string-named
events, a URL that can change under the screen, repeated start-ups, and, in request-based frameworks,
a screen object that lives for one request, so instances that must outlive it are rebuilt or held
elsewhere.

**Forced by the framework or the language, and valid.** Keep these small and never use them for
logic.
- *Event and action routing.* The framework calls a handler or action named by a string, so something
  has to map the string to a method. One handler per event, each only forwarding, is one wire from an
  event to an input: the framework's dispatch doing the job of Spray's port names.
- *Decoding params.* Parsing a string id adapts the browser's format, a technical domain reached
  sideways (§7.17). If it repeats everywhere, a generic param caster configured per event takes it
  out.
- *Rebuilding the composition per request.* Where the framework creates the screen per request, the
  composition is rebuilt each time and long-lived state lives in a store or session abstraction
  wired in. That's the framework's execution model, not handling the data, while the screen never
  looks inside what it stores (R4).
- *A template instead of wiring code for the UI.* A template language is a better notation for the
  same "display inside" wiring Spray writes with `WireTo` nesting. Moving the UI tree into wiring
  code would be reinventing a worse template. Spray's own view: XAML-like declarative UI is fine, but
  the subset of the programming language used declaratively is as good (§6.6).
- *State shaped for change tracking.* A screen often does better with a summary and a count as
  separate fields than one object holding both. Those are named landing points, Spray's symbolic
  connections kept inside one abstraction; R5 accepts them while no feature needs the names.
- *URL steps belong to the screen.* The framework delivers the URL to the screen, so the screen owns
  the table from flow step to path. That agrees with R3: paths are application literals.
- *A few lines of client code.* Focusing a field on load can't be done from the server.

**Forced by the framework, but it moves out of the screen.**
- *A connection or lifecycle check at start-up.* A generic lifecycle hook in the Programming
  Paradigms layer, configured on the screen, holds the guard once, below the screen. That's Spray's
  first move: the guard goes into the connection mechanism.
- *Auth redirects.* Same shape, same fix: the framework's lifecycle hook.
- *Async results.* A payment job returns success, failure, or a crash. Route them to the feature's
  `succeeded` and `failed` inputs with one handler each, Spray's two output ports (§4.9.1), or let a
  component or sink own the job and its outcome so the screen doesn't see it.

**Not forced: habits a screen can drop.** Arithmetic and computed state (compute totals in a
feature). Business rules in handlers (the feature sends `blocked` on a port, which the screen wires
to a notification). History in URL handling (a transitions table passed in as configuration). Handing
values from one feature to another (pass them in, or make it a wire). Loading rows at start-up (wire
a store source into the feature's `load` input). Store work in screen helpers (a domain abstraction
that does its own I/O). Loops and compound conditions in the template (a generic list component; a
feature computes the boolean).

**What remains in a screen that reaches zero.**
1. routing by event name and URL, which the framework's dispatch does instead of port names;
2. decoding browser params, a technical-domain adapter at the edge;
3. in a request-based framework, rebuilding the composition per request;
4. a lookup table from URL to step, because the framework hands the screen the URL;
5. a client-side hook or two for things only the client can do.

None of these is logic, and none has a Spray move that removes it without replacing it with
something equivalent.

**What reaching zero costs.** Each design is something a team has to learn and keep, which is why
R11 belongs in a linter's strictest tier.
- *Objects wired port to port (Spray's form):* paradigm interfaces, a wiring operator, and classes
  written to his layout. Zero hops; outputs never return to the screen. The cost is the paradigm
  layer itself, the workarounds for repeated port types (§4.4.5), and a diagram or a null-port walk
  to see and check the wiring.
- *Feature components:* the shape many UI frameworks already give you, so little new to learn.
  Cross-feature effects take message hops through the screen, and the design lives in many handler
  methods instead of one table.
- *A route table:* one map per screen from each component's output to its destination, applied by a
  generic router. An unrouted output is silently dropped, so it wants a test that every port a
  feature declares is routed.
- *Stories:* any of these split along user stories once a screen passes the size limit, with each
  story wiring its own parts.

These costs were measured for functional designs; the OO forms are Spray's, and haven't been built
as variants yet.

**Limits of measuring it.**
- *Hand walks miss things.* Later walks of the same code find findings an earlier one missed (words
  built in a feature, validation messages in feature code, a currency code in a domain class, a
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
state, or a guard moved into the connection mechanism each pays on its own. Some benefits need the whole screen,
though:
- *The screen reads as the requirements* (R8) only when all of it is wiring. One handler that
  computes, and a reader has to check every handler.
- *The diagram is the source* only when all the wiring is in it.
- *"No abstraction knows a peer"* is a guarantee only with no exceptions. One peer call and you're
  back to reading callers.
- *An abstraction's own isolation* is all-or-nothing at its boundary.

A screen with a few forwarding handlers and a URL lookup loses nothing, because those forms aren't
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

- **R1 (edges drop).** *Automatable, given a layer map:* a linter checks altitude on the graph of
  **every edge kind** (calls, instantiations, field and parameter types, superclasses, included
  mixins; see "A note on modules") against each class's or method's assigned layer — upward edges
  and cross-peer edges are scored. In a dynamically typed language, field and parameter types and
  many call targets are invisible statically, so R1 there is weaker unless the tool also runs a
  wiring-time check. Because it sees only real calls in code, a
  cross-feature call **inside a template is invisible to it**; it can recover that with an
  *advisory* **reference-level** check over the module reference graph, reported for a human to
  confirm. *Human must judge:* whether the declared boundaries are the *right* ones; whether a
  same-layer call is genuine cohesion or disguised peer coupling (a tool can take a hint about which
  modules form one unit, but cannot tell a legitimate collaboration from a smell); and every
  reference-level advisory (is that referenced peer actually called in a template?). Without a layer
  map a tool degrades to module **cycle detection** — a small subset of R1.
- **R2 (no shared mutable state between peers).** *Automatable:* static fields, class variables,
  singletons, global registries, and session or request objects read and written by two features.
  *Human must judge:* whether a given shared store is actually a *back-channel between peers* vs a
  legitimate single-owner cache, and whether one mutable object reaches two peers through the
  wiring — aliasing is a run-time fact a linter mostly can't see. (Where a functional language gets
  R2 almost for free, an OO language does not, so a clean automated R2 means less here: it misses the
  aliased object.)
- **R3 (application literals on the diagram).** *Automatable, given a layer map:* magic literals
  outside the composition layer; words below the composition in markup (text and label-like
  attributes of lower layers' templates); validation message strings; sentences built by
  interpolation; currency and unit codes. *Human must judge:* whether a literal is an *application
  literal* (should hoist) or an *intrinsic literal* (a validation pattern, a physical constant that
  belongs in its abstraction), whether a word is the product's (hoist it) or the abstraction's own
  (an error name a developer reads), and whether the "composition" is really where requirements
  should read.
- **R4 (state owned, not hidden).** *Automatable:* static and class-level mutable state; fields
  reassigned after construction on classes in the application or feature layers; getters returning
  mutable internal collections; locks inside domain classes. *Human must judge:* whether an instance
  field is the abstraction's own concept or a hidden cross-call channel whose state belongs to some
  other concept's abstraction (or to a wired `State<T>`) — a semantic distinction.
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
- **R9 (ports by paradigm).** *Automatable:* owned interfaces and abstract base classes across a
  boundary; fields typed by a peer; public methods beyond the constructors and configuration;
  container registrations and locator lookups; drift between a class's documented ports and its port
  fields. *Human must judge:* whether an output reads as a fact or names its destination, whether a
  paradigm interface has grown methods for one pair of classes, and, in a dynamically typed language,
  whether the messages a class sends to whatever it was given are the paradigm's or a peer's.
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
tier). Everything else re-lints identically (measured with the Elixir linter; an OO linter would
also stamp the extra edge kinds of "A note on modules").

### Worked check — the ALA thermometer as objects with ports, encoded

Spray's §1.6.5 thermometer composes plain objects: `ADC`, `Foreach`, `OffsetAndScale`, `Filter`,
`SampleEvery`, `NumberToString` and `Display` as `[]` domain classes, wired by `WireIn`, with
`IDataFlow<T>` as the paradigm and `WireIn` from the Foundation layer. Encoded with `[?]` tags resolved
by hand, and the optional `ports:` mark. Spray's listing shows only the wiring, so the port types
below are this checklist's filling-in:

```
class ADC              [analog input]         @domain-L1   ports: accepts IDataFlow<int[]> output
class Foreach          [batch to items]       @domain-L1   ports: provides IDataFlow<T[]>; accepts IDataFlow<T> output
class OffsetAndScale   [linear calibration]   @domain-L1   ports: provides IDataFlow<double>; accepts IDataFlow<double> output
class Filter           [smoothing algorithm]$ @domain-L1   ports: provides IDataFlow<double>; accepts IDataFlow<double> output
class SampleEvery      [decimation]$          @domain-L1   ports: provides IDataFlow<T>; accepts IDataFlow<T> output
class NumberToString   [number formatting]    @domain-L1   ports: provides IDataFlow<double>; accepts IDataFlow<string> output
class Display          [text display]         @domain-L1   ports: provides IDataFlow<string>
interface IDataFlow<T> []                     @paradigms-L2
WireIn                 []                     @foundation-L3

class Thermometer      [this product's wiring] @app-L0
  Thermometer.main   {app-literal}   -- channel 2, batch 100, offset 4, slope 8.3, strength 10, every 15, "#.#" ✓
  depends on:
    → ADC, Foreach, OffsetAndScale, Filter, SampleEvery, NumberToString, Display  @domain-L1  drops ✓
    → WireIn  @foundation-L3  drops ✓
  each domain class depends on:
    → IDataFlow<T>  @paradigms-L2  drops ✓ (a port, provided or accepted)
```

Read by hand, every rule passes. Every edge drops, and the domain classes have no edges to each other
at all: draw their class diagram and it has no lines (§2.1.3). `$` sits only on `Filter` and
`SampleEvery`, whose concepts are stateful (R4). Every application literal is on the composition
line (R3). There is no branch and no handled data in `main`: the §1.6.3 `if` is gone because
`SampleEvery` simply doesn't push (R11), and `program.Run()` sets the built graph running. No class
names a peer type or owns an interface; the only interface is the paradigm's (R9). The latent `q1`
of worked example 2 (the string format agreed by the formatter and the display) is settled the §7.8
way: `NumberToString` takes the format as configuration, and `Display` shows whatever string arrives.

*Status:* this check was done by hand. The functional and Elixir editions record the same
thermometer re-linted by a tool; no OO linter has run this encoding yet, so treat the findings above
as a reviewer's, not a tool's.

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
   with ports that you wire together like electronic components". It doesn't require OOP, though OOP
   is its natural form. Spray
   gets there from procedural code (§3.9): a struct holds the configuration, two more fields hold the
   wiring, and state joins the struct when it belongs to that concept. He sums it up as "objects as a
   language feature, not a design philosophy" (§3.10). Whatever form it takes, writing a new domain
   abstraction must stay easy: "It is necessary for developers to be able to write new domain
   abstractions, so this needs to be easy" (§1.6.5).
3. **Several kinds of connection in one app (§1.6.6).** "Monads usually only support dataflow."
   A real app also composes UI, events, and state-machine transitions, and "different lines in our
   diagram have different meanings". For UI the lines "mean 'display inside'". When the composition
   becomes a graph, the diagram is the source.

What this means in an OO language. Rung 2 is Spray's home ground, so the OO forms are mostly his own
code; how a linter should check these goals is still open.

| Goal | Some ways to meet it in an OO language | Where state lives |
|---|---|---|
| build, then run; no data in the app | instantiate and wire every object, then `program.Run()` (§1.6.5); or a library chain (LINQ, Rx, streams) inside an adapter where a stretch of the requirement is a line (§6.3) | inside each object |
| instances with paradigm ports | classes with private port fields and explicitly implemented paradigm interfaces, wired by `WireTo`/`WireIn`, setters, or generated code (§1.6.5, §5.8) | inside each object; configuration and wiring fields set once (§3.9) |
| several kinds of connection | several paradigm interfaces in one wiring: `IUI` for "display inside", `IDataFlow<T>` for data, `IEvent` for events (§1.6.6, §5.8.6); or a UI framework's template for the layout part, with dataflow ports bound to it | inside the objects; the window or screen holds only references |

Notes for OO languages:

- **The program is a graph of objects.** It changes in place, so nothing has to store it back. The
  application keeps a reference to the root (`program`, `mainWindow`) so the graph stays alive, and
  never reads the objects' state.
- **Don't give every object a thread.** One thread runs the whole graph by default; instances on
  other threads are wired asynchronously (§3.10.2–3.10.3).
- **A library chain is a monad chain, with a monad's costs.** A LINQ or stream pipeline reads almost
  like Spray's §1.6.4, and it's standard library, but it is one paradigm and one line, and usually
  one run, so keep it inside an adapter with ALA ports ("Functional techniques in an OO program",
  point 3).
- **The object graph can be the diagram.** Walking the wired objects by reflection gives instances
  and wires to print, check for unwired ports, or draw. Generated wiring code goes the other way,
  from the drawn diagram to the code (§5.8.4).

In the notation: the §1.6.5 app shows no `pN` wires at all (the objects carry them), no branch, and
no state outside the leaves. A composition line lists instances, their literals, and how they
connect. See the worked check above.

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
  Calling back up is legal only indirectly, through a lambda, strategy object or observer that was
  passed in, and "We don't use virtual functions in ALA for up calling".
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
  steps. In an OO language, an event handler or UI callback should return promptly, and slow work
  goes to async/await, a task or future, or a state machine inside the abstraction; "The C# 'yield
  return' keyword will tell the compiler to do this for you" for a long loop (§4.7.3).
- *Where instances run is a late decision.* Because asynchronous events don't care where the
  receiver is, "the physical view can be changed independently of the logical view" (§4.7.3). The
  wiring operator can insert the middleware when two instances are deployed apart, and a
  multiplexer/demultiplexer abstraction packs several ports onto one transport (§6.17.4).
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

#### OO-language notes (tentative)

- **Port types.** A port's type is owned by neither side and sits below both (R9). In an OO language
  it is a small interface in the Programming Paradigms layer: `IDataFlow<T>` (push), `IDataFlowPull<T>`
  (pull), `IEvent`, `IUI`, a request/response interface (§4.4.3, §5.8.6, §4.12). It can also be a
  delegate or lambda field, or, in a dynamically typed language, a documented message set.
- **One port, both directions.** A port can carry calls both ways: "A port is a pair of interfaces
  that allow methods in both directions" (§6.14.5). The method along the wire is a plain call; the
  one against it is an event in the paradigm interface that the receiving end subscribes to through
  the interface it was wired to, never by naming the other class (§4.4.2). Spray's `IDataFlowB` /
  `IDataFlow_R` interfaces carry a `DataChanged`-style event for exactly this (§5.8.2, §4.4.5).
- **Several inputs of the same kind.** "C# and other languages don't allow an interface to be
  implemented more than once", so an AND gate can't implement `IDataFlow<bool>` four times (§4.4.5).
  Spray's workarounds, in order of generality:
  - a distinct wrapper type for the second input (`Double2`, "not a general solution", §4.4.5);
  - reverse the input: hold fields of a reversed interface (`IDataFlow_R<bool> Input1…`), subscribe to
    their events after wiring, and wire through an intermediary object that implements both the
    forward and the reversed interface (§4.4.5);
  - a list field of a reversed interface for an indefinite number of inputs, wired through a connector
    (`List<IDataFlowB<string>> inputs` in `StringConcat`, §5.8.2), with `WireFrom` to write the wire in
    the data's direction (§5.8.3).
  The cost is wiring that runs against the data, which Spray calls "unintuitive at the wiring level",
  and proposes a `WireTo` override to hide (§4.4.5, TBD). In a language with method references or
  named ports, give each input its own named object instead.
- **Fan-out and its order.** An output field usually wires to one place; fan-out goes through a
  fan-out intermediary (`DataFlowFanout<T>`, §4.4.4) or a connector (§5.8.3), except UI layout, whose
  children list is the fan-out. Where the order of the fan-out matters, chain fan-outs through their
  `Last` port, or use activity flow, rather than rely on the order things were wired (§4.4.4).
- **Sync or async is chosen at wiring.** The same push call in the sender can reach the receiver
  directly or through a queue the wiring inserts (§3.10.2, §4.4.9). A sender that awaits its
  receiver, or assumes the receiver has finished when its call returns, has made the choice itself:
  "If a certain domain abstraction needs to make an assumption that the next line of code executes
  after the call must execute after the effects of the call, then that abstraction knows something
  about the outside world. It isno longer an abstraction" (§4.4.10).
- **C# events and observer interfaces are not wires by themselves.** A language's event mechanism is
  synchronous, supports fan-out, and is "usually registered by the receiver itself" (§4.7). Used
  inside a paradigm interface and subscribed by the wiring, it's fine; subscribed by the receiver to a
  peer it names, it's R1's defect.
- **Hot push and demand-driven pull.** Spray notes that "If you are using monads, especially I/O
  monads, or RX (reactive extensions), especially with hot observables, you are already using the
  wiring pattern" (§7.5). A demand-driven stream with back-pressure is pull, which fits Spray's own
  reasons for pull (lazy or expensive sources, §4.4.3): a choice made at wiring time.
- **Writing the kind of wire (a suggestion).** The notation has one kind of wire, `pN`. Where it
  matters, a suffix can say which paradigm a wire carries, such as `p2:event`, `p3:inside` (UI
  containment), or `p4:transition`. Spray draws different line meanings in one diagram (§1.6.6) but
  doesn't prescribe a text mark, so this is the checklist's suggestion.
- **Intermediaries are instances too.** A fan-out, a buffer, a poller, a queue or an ordering step is
  an object the composition (or the wiring operator) creates and wires like any other. Spray's
  dataflow paradigm ships several: `ChangeType<int, double>` between ports of different types,
  `DataFlowConvert` as a Select/Map, `DataFlowFanout`, `DataFlowInitializer` for a default on an
  input that may stay unwired, `ToLambda` to reach a port from inside a feature (§2.2), `Order` and
  `EventBlock` to control a diamond (§4.8.3), and a debug decorator that prints a stream (§2.2). The
  wiring makes it "Easy to insert a new instance into the wiring e.g. a debugging, logging,
  monitoring, playback, caching, or buffering instance" (§3.4.1).
- **Events are never public.** In Spray's event paradigm, events "are never public - the layer above
  always wires them up from point to point explicitly" (§4.3.2). An asynchronous version is wired
  with the same call plus the execution model, `new A().WireTo(new B(), eventLoop)`, which inserts a
  queueing intermediary; the main loop can live in Foundation with each execution model registering
  a poll method (§4.3.2).
- **Two-way ports and timeouts.** A request/response port can't be both synchronous and
  asynchronous, so "to have domain abstractions with two-way ports zero-coupled with respect to
  synchronous/asynchronous communications, the senders need to be asynchronous by nature" (§4.4.10):
  async/await, tasks, callbacks or a state machine, with an intermediary when the responder is
  synchronous. Often the better move is to split the request/response port into an event input
  (start) and a dataflow output, which in Spray's ADC example "has actually lead to better
  abstractions and a better solution overall" (§4.4.10). An abstraction that can be wired across a
  network should expect to need timeouts, "because we don't know to whom the ports will be wired"
  (§4.4.10).
- **Wire as a variable.** Components-and-connectors is one more paradigm: a connector object holds
  the "data on the wire", the sender sets it, the receiver reads it, and a `Wire(ref a.output, ref
  b.input)` helper creates the connector between two port fields (§2.7.1).
- **State machines.** A state machine whose states and transitions are instances, wired by the
  composition, is Spray's form (§4.16); a transitions table passed in as configuration is the
  lighter one.

#### Candidate checks (not yet rules)

- A domain class that decides sync or async, or push or pull, for its caller (for example, awaiting
  a result it then assumes is applied, or starting its own thread or timer to deliver an output).
- A domain class whose correctness depends on the order its outputs are handled.
- One object or topic joining many instances: the ground-symbol smell.
- A subscriber that names what it subscribes to.
- A composition that is an inherent graph but is spread across several classes' constructors or a
  container's registrations instead of kept in one place.
- A paradigm interface that grows a method used by only one pair of classes (§5.7, §7.26.3).

## How to use it on your own code

1. **Pick one entry point** (a request handler or controller action, an event, main). List the
   classes and methods it transitively reaches — that's your `f` set. Keep real names in trailing comments; the ids
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
| field or parameter typed by a peer class or its interface | replace it with a port typed by a paradigm interface; the class above wires it |
| `new Peer()` inside a domain class | move the `new` up to the class above, which creates both and wires them |
| superclass in your own layers | compose: shared parts become abstractions, shared behaviour a class with ports, and the subclass delegates explicitly |
| DI container or service-locator wiring | write the wiring by hand (or generate it from a diagram) in the application class |

The end state is recognizable at a glance: **the tagged layers (the application, and features
under it in a bigger app) carry all the literals and only wire; everything they use is a `[]`
abstraction in a lower layer; and no edge goes sideways or up.** Then take the next step:
make that top *describe* the connections and let the objects move the data through their ports (see
"Past the pipe").

**Starting from an existing app.** Adopt the checklist, run a linter that implements it against what
you have, and let the findings send you to the technique lists under each rule. A conventional OO
app usually starts furthest from ALA on R1 and R9 (associations, injected peers, owned interfaces,
inheritance), R2 and R4 (shared mutable objects, singletons), and R10 (entity classes every feature
reads). Work one user story at a time (§8.3): invent the paradigm interfaces it needs, turn each
association on its path into a port, and move the wiring into one application or feature class. R3
and R11 then follow, as constants and `if`s surface in the new wiring class. Reach for the heavier
techniques (a reflection-based wiring operator, a diagram with generated wiring, intermediaries for
asynchronous wiring) only when a rule you care about isn't holding by discipline alone. The lightest
technique that meets the rule is the right one.

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
   topic it names, or asking a container, locator or singleton for it. Write `^f` for a lookup of
   something higher, and treat a self-registration between peers as a peer edge. The fix is always "the composition wires
   it," never "the leaf knows whom to call."
4. **A wire can carry a shared object.** In an OO language a `pN` the composition passes to two
   instances looks like a clean wire, but if it's a mutable object that both keep and change, it's a
   `*pN` (R2). The encoding shows what the code passes, not what the objects later do with it; read
   the receivers for stored references.
5. **A perfect shape can encode the wrong program.** The encoding audits *structure*
   (coupling/knowledge placement), not correctness — same caveat the requirements-coverage
   analysis makes for manifests.

## Glossary

The checklist's vocabulary, in alphabetical order. Section numbers point to Spray's site. Where a term is this project's and not Spray's, the entry says so.
**Abstraction.** The only unit of code in ALA: "a 'generalized conceptual idea'", "learnable as a concept" (§2.1.1). In an OO language it is usually one class in its own file, sometimes a small group of types in one file (§2.3.2). What makes it an abstraction is that a reader can use it without reading its body; being a class doesn't (§7.2.1).

**Abstraction height.** The longest chain of knowledge dependencies between abstractions. Calls inside one module don't add height. A tall stack of thin layers is a sign of helper proliferation. A linter metric, not Spray's term.

**Accepts / provides.** The two sides of a port. A class *accepts* an interface by holding a field of its type that the wiring fills, and *provides* one by implementing it. Spray uses "accepts" rather than "requires" because a port may be left unwired (§6.14.2, §7.6).

**Application layer.** The top layer. It instantiates abstractions, configures them, and wires them together, and holds all of the app's specific knowledge and none of its logic (§3.5). In a UI app, the screen, its template, and the router.

**Application literal.** A constant that belongs to this product (a price, a threshold, a label, message text). It lives at the composition (R3). Contrast *intrinsic literal*.

**Association.** A UML relationship from one class to another that it uses at run time, including through an injected field or a peer's interface. Illegal between peers in ALA; the wiring in the layer above replaces it (§2.1.3).

**Communication dependency.** One class calling another to move data or events between them as peers. ALA eliminates these; the layer above wires the peers instead. In UML terms, an association (§2.1.3).

**Composition.** Building something from instances of abstractions by connecting them. Spray's opposite is decomposition, splitting a system into specific parts that collaborate (§3.7). "The composition" also names the code that does the composing: the application, or a feature's wiring.

**Configuration.** The settings an instance gets once, when it is created: its main interface, used only by the layer above (R9). R9's "should" keeps configuration apart from run-time data.

**Connection mechanism.** Whatever carries data along a wire at run time: a port field's direct call, an intermediary, an event queue, a monad's Bind. Guards like "only if there's a value" belong here, not in the application (§1.6.4, §6.1.3).

**Connector / intermediary.** An object the wiring puts between two ports: a fan-out, a buffer, a poller, a queue, or Spray's workaround for several inputs of the same type (§4.4.4–4.4.6, §5.8.3).

**Departure.** A place where this checklist knowingly differs from Spray, with a reason, such as the routing handlers a UI screen must keep. Recorded, kept small, never used for logic.

**Dependency injection container.** A framework that wires objects by matching the interface types they require and provide. Illegal in ALA, because the wiring is implicit (§3.10.1, §3.11.3, §6.6).

**Diagram.** Spray's source of truth: boxes are instances, lines are wires, and the whole reads as the requirements (§2.5, §3.6). In code it may be a manifest, a graph value, or a readable composition.

**Domain abstraction.** A reusable abstraction in the layer below the application (or below the features). It knows nothing about this product: `LowPassFilter`, `OffsetAndScale`, a generic table component.

**DTO (data-transfer object).** A type made only to carry data between two modules. Two peers may not share one (§4.8.1).

**Execution model.** The code that makes a kind of connection actually run, such as a runner, an interpreter, or a UI framework's event loop. It lives in the Programming Paradigms layer.

**Explicit interface implementation.** Implementing an interface so its methods are reachable only through a reference of the interface type, not on the class's main interface. Spray's way to keep provided ports off the configuration interface (§4.4.4).

**Feature.** A product-knowing abstraction in its own layer under the application, wired by it, used once an application is too big to be one abstraction (§2.2, §7.15). Like the application, it holds instances, configuration and wiring, including its UI layout from domain UI abstractions (§2.2, §7.14). Modules of state and rules that a codebase calls features are stateful domain abstractions under that name.

**Fluent wiring.** Wiring code where each `new`, setter and `WireTo`/`WireIn` returns an object, so a tree of instances reads as nested calls with anonymous instances (§1.6.5, §5.8.3).

**Ground symbol.** Spray's name for a wire that joins many ports, like ground on a schematic. It suggests a missing abstraction one layer down (§3.6.1).

**Handling the data.** The application catching one abstraction's result only to pass it to another. Spray names it at his §1.6.3 step and removes it at §1.6.4. An R11 finding.

**Identity key.** An id two features share while each keeps its own data. One way to meet R10. The technique is Spray's: "use cases should all know about the abstraction, customer identity. A particular use case should only know about it's own data, and only store it against a customer identity" (§6.17.2); the name is this checklist's.

**Inheritance.** Not used in ALA: a subclass knows its parent and breaks the base class as an abstraction; composition and passed-in behaviour replace it (§2.1.3, §7.16). Extending a framework's base class to configure it is this checklist's allowed exception (R9).

**Instance.** The run-time use of an abstraction: in an OO language, an object (§2.3.3, §3.2.2). Instances are wired by their ports; classes never know each other.

**Instance name.** A name given to each instance at construction (`InstanceName`, `Name`) for debugging and logs, never for lookup (§5.8.2, §8.5).

**Intrinsic literal.** A constant that is part of an abstraction's own definition, such as an identity or a physical constant. It stays in the abstraction. Contrast *application literal*.

**Knowledge dependency.** One abstraction using another, more abstract one by name, the way code uses a square root. The only kind of dependency ALA allows, and it must point to something significantly more abstract (§2.1.3, §3.4).

**Layer.** One of a few levels ordered from concrete to abstract. Spray's usual stack is Application, Features, Domain Abstractions, Programming Paradigms, and Foundation. Knowledge only flows down.

**Little ball of mud.** An abstraction's inside, which may be procedural and messy as long as the abstraction is small, names one concept, and is clean at its boundary. ALA governs the relationships between abstractions, not their insides.

**Main interface.** A class's constructors and public configuration methods and properties, used only by the layer above to instantiate and configure it, never by peers at run time (§2.3.4, §7.24).

**Monad.** A functional pattern that composes functions through a Bind function, which hides execution details. Spray treats monads as the nearest functional analogue of ALA, and as more limited: two ports and one paradigm (§3.11, §6.1–§6.2).

**Must / should.** A "must" is a defect when it fails. A "should" is a prompt a reviewer weighs, used where Spray's evidence is weaker.

**Object graph.** The wired instances an application builds once at start-up and sets running; the run-time form of the diagram, and a static UML object diagram (§3.11.3, §6.14.5).

**Owned interface.** An interface specific to one module, either defined by a consumer for others to implement (required) or by a module for its peers to call (provided). Both fail R9.

**Paradigm interface.** An interface in the Programming Paradigms layer, owned by no domain abstraction, that ports are typed by: `IDataFlow<T>`, `IEvent`, `IUI` (§2.2, §2.3.5).

**Pass-through.** A public function with one caller whose body is a single call into another module. It renames a call without hiding a decision. A linter check, advisory.

**Peer.** Another abstraction in the same layer. Peers never know each other. Only a layer above connects them.

**Port.** An input or output of an instance whose type is a programming paradigm from a lower layer, set up by the layer above. In an OO language, a private field of a paradigm interface type (accepted) or an explicitly implemented paradigm interface (provided) (Summary, §2.3.4). It can be a pair of interfaces, one for each direction (§6.14.5).

**Programming paradigm.** What a kind of connection means: dataflow, events, UI layout, state-machine transitions, request/response (§4.1). Each one is an abstraction in the Programming Paradigms layer, and ports are typed by them.

**Projection.** A read-only view a feature offers of its data, shaped for its readers, so they never read its own data type. A technique for R10.

**Push / pull.** Whether the sender calls the receiver (push) or the receiver asks the sender (pull). Spray defaults to push because it works synchronously or asynchronously (§3.11.4).

**Runner.** A generic execution model that moves data between instances so the application never touches it. In Spray's OO form the objects call each other through their ports and need no runner; an event loop or queue the wiring inserts is the asynchronous one.

**Silent contract.** An agreement between two modules that appears in neither's signature, such as a matching string, a tuple shape, or a session key (R5).

**State abstraction.** Spray's `State<T>`: an abstraction with input and output ports that holds state belonging to no other concept, wired in like anything else (§3.9).

**Symbolic connection.** A name used only to connect two points in the code, like a local variable that carries a value between two calls. Fine inside one abstraction; across modules it becomes a registry of global names (§4.7.4, §7.23).

**Tag.** In the checklist's notation, `[thermo]` marks a function that knows a product requirement, and `[]` marks a generic one. The one judgement the notation asks of a human.

**Tramp parameter.** A parameter a function never reads and only carries down to something further below. Spray: middle layers end up with "extra parameters that don't have anything to do with them" (§3.11.1). R6's "should". The term itself is general programming usage, not Spray's.

**Wire.** A run-time connection between two instances' ports, set up by the layer above. Circular wiring is fine; circular knowledge dependencies are not.

**WireTo / WireIn.** Spray's wiring operators, extension methods in the Foundation layer. `WireTo` connects a port of its first operand to its second and returns the first; `WireIn` returns the second. His versions use reflection to find private port fields, which ALA doesn't require (Summary, §2.2).

**Working chain.** Product-knowing functions that do work and call each other, the shape of the bad thermometer. Each part should be either a real abstraction in a lower layer or wiring in the composition.

**Zero coupling.** No design-time knowledge between peers, not merely less of it. ALA removes bad dependencies rather than loosening them (§7.4).

---

*Origin note: this file started as a numbering sketch of the two trees; the worked forms above
fix the f-numbering against Spray's actual §1.6 code and add the `[tag]`/`$`/`*`/`q` marks —
without the tag column, the bad and good trees are nearly the same shape, which is what the first
sketch ran into. The "Past the pipe" section follows Spray's §1.6.3–1.6.6 ladder. This
object-oriented edition was made from the functional edition by re-reading each rule for classes
and objects and adding what Spray says about object-oriented programs, nearly all of it from his own
C# practice; its OO techniques haven't yet been built as variants or checked by an OO linter.*

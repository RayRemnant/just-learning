OOP & Encapsulation

Verbosity vs. maintainability

- More code ≠ more maintainable. Maintainability comes from good decomposition, clear naming, low coupling — not from OOP ceremony (getters/setters, DTOs, unnecessary abstraction).
- A lot of "OOP boilerplate" is a Java-specific tax (no native properties, verbose generics, checked exceptions), not something inherent to object-orientation. Modern Java features (records, var, sealed classes, pattern matching) exist precisely because that boilerplate was accidental complexity, not essential.
- Boilerplate actively hurts when it adds indirection (interfaces/DI for everything), premature abstraction (flexibility nobody needed), or mutable state via setters (which undermines the "minimizes side effects" claim).

Invariants

- An invariant = a condition that must hold true about an object's state at all times, not just after one method call.
- Example: "balance must never go negative." A public field can't enforce this — anyone can set it directly.
- A private field + a gated method (withdraw() that checks before mutating) creates a single enforcement point. This is the real value of "single place of coding" — not saving keystrokes, but making the rule structurally impossible to violate.
- Note: a plain getter/setter pair does NOT give you this. setBalance(-500) is just as broken as direct field access. Encapsulation only helps when the method actually enforces a rule.

"Hiding implementation" — what it actually buys you

- Types describe shape (String, int), not behavior (when/how a value is produced, whether it's validated, whether it triggers a side effect like a DB fetch).
- A field is just a memory slot — it cannot run code on read/write. If a value needs to come from a database, network call, or needs validation, you need a method/property, not a typed field. Type alone can't express this.
- "Renaming a variable breaks everything" (the slide's dramatized example) is a Java-specific problem (no native properties: dog.name is direct field access). In C#, Kotlin, Python, Swift, a property can be backed by a field or by computed logic with no syntax change for callers. Not a universal OOP law — a Java limitation.
- Conclusion: encapsulation matters when there's actual behavior to gate (validation, laziness, side effects, invariants). It's dead weight when it's just a field wrapped in a pass-through getter/setter with no rule attached — which is most real-world usage.

The real limit of encapsulation: representation churn vs. semantic churn

- Encapsulation only promises: if the meaning of an operation stays the same but the mechanism changes, callers don't need to change. (e.g., getName() backed by a field vs. a DB call — same contract, different mechanism.)
- It does NOT promise: if the meaning/behavior of the object itself changes, callers are insulated.
- Example: dog goes from 4 legs to 2 legs. This isn't a representation change, it's a semantic change — walk() now behaves differently. Any code that depended on the old behavior (speed, stability, gait) breaks, regardless of how well-encapsulated the class is.
- Why: an object's correctness is relational — defined by how it interacts with other objects. Encapsulation protects the inside of that relationship (how the contract is fulfilled) but has no power over whether the contract/relationship itself still makes sense after a semantic change.
- Bottom line: encapsulation is a shield against implementation churn, not semantic churn. Most real "requirements changed" events in practice are semantic churn — the category encapsulation can't protect against.

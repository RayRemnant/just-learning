# Learn Java: Conceptual Map

A roadmap of core Java, JVM, and object-oriented design concepts, organized by mental models and trade-offs.

---

## 01. Platform & Foundations
- [01 - Platform & Execution Model](file:///c:/Users/rayre/Learning%20with%20AI/ide-md-context/Learn%20Java/01%20-%20Platform%20&%20Foundations/01%20-%20Platform%20&%20Execution%20Model.md) — Bytecode, JVM & Tiered JIT compilation, primitive vs reference dichotomy, wrapper overhead, `String` immutability, `var`, and switch expressions.
- [02 - Packages & Namespaces](file:///c:/Users/rayre/Learning%20with%20AI/ide-md-context/Learn%20Java/01%20-%20Platform%20&%20Foundations/02%20-%20Packages%20&%20Namespaces.md) — Namespace hierarchy, filesystem 1:1 coupling, package-private encapsulation boundaries, compile-time import mechanics, and JPMS modules.
- [03 - Access Modifiers & Visibility](file:///c:/Users/rayre/Learning%20with%20AI/ide-md-context/Learn%20Java/01%20-%20Platform%20&%20Foundations/03%20-%20Access%20Modifiers%20&%20Visibility.md) — The 4 visibility levels, target matrix (classes vs members vs parameters/locals), the "Two-Gate" rule, constructor gating, and protected inheritance subtleties.
- [04 - CodeCache, Hardware Realities & The Rust Comparison](file:///c:/Users/rayre/Learning%20with%20AI/ide-md-context/Learn%20Java/01%20-%20Platform%20&%20Foundations/04%20-%20CodeCache,%20Hardware%20Realities%20&%20The%20Rust%20Comparison.md) — CPU L1i/iTLB caches, 32-bit RIP-relative call limits, 1995 historical JVM premises vs 2020s cloud realities, and a deep architectural comparison with Rust.

## 02. Object Mechanics & Design
- [01 - Class Mechanics & Object Lifecycle](file:///c:/Users/rayre/Learning%20with%20AI/ide-md-context/Learn%20Java/02%20-%20Object%20Mechanics%20&%20Design/01%20-%20Class%20Mechanics%20&%20Object%20Lifecycle.md) — Stack definite assignment vs heap zero-initialization, constructor delegation `this(...)`, overloading vs overriding (`@Override`), static context in Metaspace, and the universal `java.lang.Object` root.
- [02 - OOP & Encapsulation](file:///c:/Users/rayre/Learning%20with%20AI/ide-md-context/Learn%20Java/02%20-%20Object%20Mechanics%20&%20Design/02%20-%20OOP%20&%20Encapsulation.md) — Invariants, gating state mutation, accidental Java ceremony, representation vs. semantic churn.
- *Polymorphism & Subtyping (Planned)* — Ad-hoc vs. parametric vs. subtype polymorphism, fragile base class problem.
- *Modern Java Modeling (Planned)* — Records, sealed interfaces, algebraic data types (ADTs), pattern matching vs. classical visitor/polymorphic dispatch.

## 03. Persistence & Transactions
- [01 - Hibernate & Transactions](file:///c:/Users/rayre/Learning%20with%20AI/ide-md-context/Learn%20Java/03%20-%20Persistence%20&%20Transactions/01%20-%20Hibernate%20&%20Transactions.md) — Hibernate sessions as identity maps, Managed vs Detached states, transaction propagation (`REQUIRES_NEW`), and race condition recovery patterns.

---

## Future Tracks
- **Memory, Mutability & Invariants (Planned)**: Immutability, unmodifiable collections vs true snapshots, JVM memory layout (compressed OOPs, object headers).
- **Concurrency & Type System (Planned)**: Java Memory Model, happens-before guarantees, generics & type erasure, PECS.

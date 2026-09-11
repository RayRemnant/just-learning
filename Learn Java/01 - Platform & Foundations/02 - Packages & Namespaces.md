# Packages & Access Control

## Jargon & Keywords in Plain English

- **Package**: A namespace mechanism that groups related classes, interfaces, and sub-packages together to prevent naming collisions and organize codebases.
- **Reverse Domain Name**: The universal Java convention for package names (e.g., `com.company.project`), leveraging internet domain ownership to guarantee global uniqueness.
- **Classpath**: The runtime or compile-time search path used by the JVM and `javac` to locate `.class` bytecode files and bundled JAR files on the disk.
- **Compilation Unit**: A single `.java` source file, typically containing a single public top-level class matching the filename.
- **Import Statement**: A compile-time directive telling the Java compiler where to look up shorthand type names used in a file. It has **zero** runtime performance or memory overhead.
- **Fully Qualified Name (FQN)**: The complete, unambiguous name of a class that includes its full package prefix (e.g. `java.util.ArrayList`, `org.financial.insurance.Health`).
- **Access Modifier**: Keywords (`public`, `protected`, `private`, or omitted/default) that declare which other classes and packages are permitted to see and invoke a class, field, constructor, or method.
- **Package-Private (Default Access)**: The access level applied when no modifier keyword is written. Restricts access strictly to other classes residing inside the exact same package.
- **JPMS (Java Platform Module System)**: Introduced in Java 9, a higher-level modular boundary (`module-info.java`) that groups packages and enforces explicit `requires` and `exports` rules across modules.

---

## 1. Code Aggregation: Class $\to$ Package $\to$ Module

Java organizes code in three hierarchical tiers:

```
[ Module ]  (Java 9+: groups packages, enforces explicit external contracts)
   └── [ Package ]  (groups related classes, manages namespace & folder mapping)
          └── [ Class ]  (fundamental unit of state & behavior)
```

### Physical Directory Coupling
Unlike many modern languages where namespace declarations are decoupled from disk layout, Java strictly couples package declarations to the physical directory structure:

```
src/
└── org/
    └── financial/
        └── insurance/
            ├── Health.java
            └── Life.java
```

Inside `Health.java`:
```java
package org.financial.insurance;

public class Health {
    // ...
}
```

- **Invariants & Collision Rules**:
  - The package statement must be the absolute first non-comment line in the source file.
  - The folder hierarchy on disk must mirror the package path identically relative to the source root or classpath.
  - No two classes in the same package may share the same class name.
  - The combination of **Package Name + Class Name (FQN)** is globally unique.

### The Reverse Domain Name Convention
By convention, package paths begin with the organization's reversed internet domain (e.g., `org.apache.commons`, `com.google.common`, `com.soldo.product`). Because domain names are globally registered and owned, this convention mathematically eliminates library namespace collisions when disparate third-party JARs are combined on the classpath.

---

## 2. Packages as Encapsulation Boundaries

A package is not merely a directory folder; it establishes Java's default access boundary: **package-private**.

- **Package-Private as the Original Component Boundary**: Package-private (the default when no modifier is specified) allows classes within the same package to collaborate as tightly coupled collaborators without exposing internal plumbing to outside callers. Only outward-facing APIs need to be `public`.
- **The Scope of Modifiers**: Access modifiers apply to **classes** and **class members** (methods, fields, constructors), but **never to method parameters or local variables** (which are strictly bounded by their stack frame).
- **The Two-Gate Rule**: To access any member, a caller must first have access to the enclosing class before member visibility even takes effect.

> [!TIP]
> For an in-depth breakdown of the 4 access levels, the target matrix, the "Two-Gate" rule, constructor gating, and protected inheritance subtleties, see the dedicated chapter: [Access Modifiers & Visibility](file:///c:/Users/rayre/Learning%20with%20AI/ide-md-context/Learn%20Java/01%20-%20Platform%20&%20Foundations/03%20-%20Access%20Modifiers%20&%20Visibility.md).

---

## 3. Import Mechanics & Symbol Resolution

The `import` statement in Java is frequently misunderstood. It is **purely a compile-time syntactic tool**; it does not load code, pull in dependencies, or increase bytecode size.

```java
// Option 1: Explicit Class Import (Best practice for clarity)
import org.financial.insurance.Health;

public class Insurance {
    Health h = new Health();
}
```

```java
// Option 2: Wildcard Import (Imports all classes in that specific package)
import org.financial.insurance.*;

public class Insurance {
    Health h = new Health();
}
```

```java
// Option 3: Fully Qualified Name (FQN) Inline
public class Insurance {
    org.financial.insurance.Health h = new org.financial.insurance.Health();
}
```

### Import Semantics & Invariants
- **Zero Runtime Overhead**: `import org.financial.insurance.Health;` does not cause the JVM to load the class at startup. In compiled bytecode, all type references are replaced with their Fully Qualified Names (`org/financial/insurance/Health`).
- **Wildcard (`*`) Limitations**:
  - `*` is not recursive. `import org.financial.*` does **not** import `org.financial.insurance.Health`.
  - Wildcards can cause ambiguous symbol collisions if two packages export classes with identical names (e.g. `java.util.List` vs `java.awt.List`).
- **Implicit Import (`java.lang.*`)**:
  - The Java compiler automatically and invisibly prepends `import java.lang.*;` to every single Java file.
  - This is why core types like `String`, `System`, `Object`, `Math`, and `RuntimeException` are always available without explicit import declarations.
- **When Inline FQNs Are Required**: When a file must use two classes with identical simple names from different packages:
  ```java
  import java.util.Date; // Primary import

  public class AuditLog {
      private Date eventDate;              // java.util.Date
      private java.sql.Date sqlTimestamp;  // Disambiguated inline FQN
  }
  ```

---

## 4. Architectural Evolution: Java 9 Modules (JPMS)

While packages solved namespace collision and package-private scoped code within a package, they had two major architectural flaws:
1. **Public meant public to the world**: Any `public` class in a JAR was visible to every other JAR on the classpath.
2. **Classpath hell**: Missing dependencies or duplicate JAR versions were only detected at runtime when a `NoClassDefFoundError` or `ClassNotFoundException` was thrown.

To solve this, Java 9 introduced the **Java Platform Module System (JPMS)**:

```
module com.soldo.product.insurance {
    requires java.base;               // Explicit dependency
    requires org.apache.commons.lang3;

    exports org.financial.insurance;   // Public classes here are accessible outside
    // Any package not explicitly exported remains strictly private to this module!
}
```

- **Strong Encapsulation**: Even if a class is declared `public`, if its enclosing package is not exported in `module-info.java`, outside modules cannot access or instantiate it.
- **Reliable Configuration**: The JVM verifies module dependency graphs at application startup, failing immediately if a required module is missing or cyclic.

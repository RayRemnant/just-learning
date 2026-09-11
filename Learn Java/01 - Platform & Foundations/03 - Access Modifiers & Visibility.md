# Access Modifiers & Visibility

## Jargon & Keywords in Plain English

- **Access Modifier**: A Java keyword (`public`, `protected`, `private`, or omitted/default) that defines the accessibility boundary for types and members.
- **Top-Level Class**: A class defined directly in a `.java` file, outside of any other class.
- **Member**: A component defined inside a class: a field (variable), method, constructor, or nested class.
- **Local Variable & Parameter**: Temporary variables existing exclusively on the execution call stack of a method or block. They **never** accept access modifiers because their visibility is physically trapped inside their stack frame.
- **`public`**: The widest access level. Accessible to any other class across any package on the classpath or module graph.
- **`private`**: The narrowest access level. Accessible strictly within the curly braces `{ ... }` of the enclosing class.
- **`protected`**: Accessible to any class in the same package, plus any subclass anywhere—even if that subclass lives in a completely different package.
- **Package-Private (Default Access)**: The access level when no keyword is written. Accessible to all classes residing in the exact same package, but completely invisible outside it.
- **The "Two-Gate" Rule**: The foundational rule that a caller must have visibility to the outer class (Gate 1) before it can evaluate or access any member inside that class (Gate 2).
- **Effective Visibility**: The true, practical accessibility of a member after factoring in the visibility of its enclosing class (e.g. a `public` method inside a package-private class has an effective visibility of package-private).
- **Static Factory Method**: A static method used to create and return object instances, often paired with a `private` or package-private constructor to control instantiation.

---

## 1. The Target Matrix: Where Modifiers Can and Cannot Go

Java strictly limits which modifiers can be applied based on the syntactic target:

| Target | Allowed Access Modifiers | Why & Compiler Constraints |
| :--- | :--- | :--- |
| **Top-Level Class / Interface / Record / Enum** | `public` or *(package-private)* | Cannot be `private` or `protected`. A file-level class with no outer class cannot be "private" or "subclass-only" to anything. |
| **Class Members**<br>*(Methods, Fields, Constructors)* | `public`, `protected`, *(package-private)*, `private` | Controls which callers can invoke the method, mutate/read the field, or construct the object. |
| **Nested / Inner Classes** | `public`, `protected`, *(package-private)*, `private` | Because they are members of an outer enclosing class, they support all 4 access levels. |
| **Method Parameters & Local Variables** | ❌ **NONE** *(Compile Error)* | Bounded strictly to the method's stack frame. Outside code cannot reach into a running method's stack, making access modifiers nonsensical. |

```java
public void calculate(public int amount) { // COMPILE ERROR: illegal modifier for parameter
    private int total = 0;                 // COMPILE ERROR: illegal modifier for local variable
}
```

> [!NOTE]
> While parameters and local variables reject access modifiers, they **do** accept the `final` modifier (making the variable reference immutable once assigned) and annotations.

---

## 2. The Four Visibility Levels

```
                     Accessibility Spectrum
┌──────────────────────────────────────────────────────────────┐
│  private  ◄───  package-private  ◄───  protected  ◄───  public │
│ (narrowest)                                        (widest)  │
└──────────────────────────────────────────────────────────────┘
```

| Modifier | Same Class | Same Package | Subclass (different package) | World (any package) |
| :--- | :---: | :---: | :---: | :---: |
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(default / package-private)* | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ *(via inheritance)* | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

### The Subclass Subtlety of `protected`
`protected` allows cross-package access to subclasses, but only **through inheritance**, not through arbitrary object reference traversal:

```java
package com.framework.base;

public class CoreService {
    protected void internalHook() {}
}
```

```java
package com.app.extension;
import com.framework.base.CoreService;

public class CustomService extends CoreService {
    public void execute() {
        // ALLOWED: Invoking inherited protected member on 'this'
        this.internalHook();
        internalHook();

        // FORBIDDEN: Invoking protected member on a foreign parent instance
        CoreService base = new CoreService();
        // base.internalHook(); // COMPILE ERROR: internalHook() has protected access in CoreService
    }
}
```

*Why*: `protected` was designed to permit subclasses to specialize and extend inherited behavior—not to grant subclasses back-door access into arbitrary instances of the base class.

---

## 3. The "Two-Gate" Rule: Class Visibility vs. Member Visibility

Declaring a method `public` does **not** guarantee it can be called by anyone. Access evaluation is hierarchical:

```
Caller Attempting Access
          │
          ▼
┌───────────────────────────────────┐
│ Gate 1: Can caller see the CLASS? │ ──NO──► 🛑 Access Denied (Cannot name/import type)
└───────────────────────────────────┘
          │ YES
          ▼
┌────────────────────────────────────┐
│ Gate 2: Can caller see the MEMBER? │ ──NO──► 🛑 Access Denied (Cannot invoke/read member)
└────────────────────────────────────┘
          │ YES
          ▼
       ✅ ACCESS GRANTED
```

### Concrete Demonstration
Consider a `public` method inside a package-private class:

```java
package org.financial.engine;

// Gate 1: Package-private (visible ONLY inside org.financial.engine)
class SettlementProcessor {
    // Gate 2: Public
    public void processSettlement(long accountId) {
        // ...
    }
}
```

Now consider an external caller in `com.app.billing`:

```java
package com.app.billing;
import org.financial.engine.SettlementProcessor; // COMPILE ERROR: SettlementProcessor is not public

public class BillingService {
    void run() {
        // Caller cannot pass Gate 1. 
        // Gate 2 (the public method) is completely unreachable!
    }
}
```

- **Declared Visibility**: What you write on the member (`public void processSettlement(...)`).
- **Effective Visibility**: The intersection of Gate 1 and Gate 2. Here, the effective visibility of `processSettlement` is strictly **package-private**.

---

## 4. Architectural Patterns Enabled by Visibility

Disciplined access control enables powerful architectural boundaries without third-party frameworks.

### Pattern A: Package-Private Implementation with Public Interface
This pattern completely conceals concrete implementation classes from external packages:

```java
package org.financial.payment;

// PUBLIC Interface: What external packages see and program against
public interface PaymentGateway {
    void process();
}
```

```java
package org.financial.payment;

// PACKAGE-PRIVATE Implementation: Completely hidden from the outside world
class StripePaymentGateway implements PaymentGateway {
    @Override
    public void process() { /* ... */ }
}

// PUBLIC Factory: The controlled single entry point
public class PaymentGatewayFactory {
    public static PaymentGateway createDefaultGateway() {
        return new StripePaymentGateway(); // Safe: same package can instantiate
    }
}
```

- **Architectural Value**: External consumers in `com.client` can call `PaymentGatewayFactory.createDefaultGateway()`. They receive a `PaymentGateway` reference, but cannot even import or name `StripePaymentGateway`. The implementation can be rewritten, renamed, or replaced without breaking a single external caller.

### Pattern B: Constructor Gating
Constructors can use access modifiers to strictly govern who can instantiate an object:

```java
public class SystemConfig {
    // PRIVATE Constructor: Prevents direct 'new SystemConfig()' anywhere outside
    private SystemConfig() {}

    // Static Factory Method: Single point of controlled creation
    public static SystemConfig loadFromEnvironment() {
        return new SystemConfig();
    }
}
```

- **`private` Constructor**: Used for utility classes (e.g., `java.lang.Math`, `java.util.Collections`) to prevent instantiation entirely, or for Singletons and static factories.
- **Package-Private Constructor on a `public` Class**: Allows any code in other packages to reference the class type, but only classes *within* the package can create instances.

---

## 5. Interface & Record Visibility Invariants

Java imposes special default access rules on certain language constructs:

### Interfaces
- All methods in an interface are implicitly **`public`** (unless explicitly marked `private` for internal default-method helpers).
- All fields in an interface are implicitly **`public static final`** (constants).
- You cannot declare a method in an interface as `protected` or package-private.

### Records (Java 16+)
- A record's canonical constructor and auto-generated accessor methods cannot have more restrictive access than the record class itself.
- If a record is `public`, its accessor methods (e.g. `user.id()`) are automatically `public`.

---

## 6. Summary: Key Takeaways

1. **Parameters & Local Variables Have No Access Modifiers**: Their scope is strictly physical (the thread stack frame).
2. **Access Evaluation Requires Passing Two Gates**: You cannot call a `public` method if you cannot see its enclosing class.
3. **Default (Package-Private) Is Not an Accident**: It is the natural component boundary for cohesive collaboration within a package.
4. **Visibility Dictates Coupling**: Restricting class visibility to package-private and exposing only `public` interfaces makes refactoring and architecture maintenance trivial.

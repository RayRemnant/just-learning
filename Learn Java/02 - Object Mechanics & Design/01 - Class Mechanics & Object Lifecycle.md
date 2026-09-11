# Class Mechanics & Object Lifecycle

## Jargon & Keywords in Plain English

- **Class**: The compile-time blueprint defining the state (fields) and behavior (methods) of a category of things.
- **Instance / Object**: An individual, concrete chunk of memory allocated on the heap at runtime according to the class blueprint.
- **Field (Instance Variable)**: A state variable bound to an individual object instance, living on the heap for the entire lifetime of that object.
- **Local Variable**: A temporary variable declared inside a method or code block, living briefly on the CPU call stack and destroyed when execution leaves that block.
- **Definite Assignment**: The Java compiler's mandatory rule requiring that a local variable must be explicitly assigned a value on every possible code branch before its value can be read.
- **Default Value / Zero-Initialization**: The automatic initialization performed by the JVM on all heap-allocated fields (numbers to `0`/`0.0`, booleans to `false`, object references to `null`).
- **Constructor**: A specialized initialization routine executed when an object is created with `new`. Its sole purpose is to establish initial state and invariants.
- **Constructor Chaining (`this(...)`)**: Delegating work from one overloaded constructor to another within the same class to centralize initialization logic.
- **Method Signature**: The unique identity of a method consisting strictly of its **name** and its **ordered list of parameter types**. Return types and parameter names are *not* part of the signature.
- **Method Overloading**: Providing multiple methods in the same class sharing the same name but with distinct parameter signatures. Resolved statically at compile-time.
- **Method Overriding**: Redefining an inherited parent class method in a child class with the identical signature. Resolved dynamically at runtime based on the actual object instance.
- **`super`**: A reference to the immediate superclass (parent) implementation or constructor.
- **`@Override`**: A compiler check annotation. It instructs `javac` to verify that a method actually overrides a superclass method, catching typos and signature drift at compile time.
- **`static`**: A keyword declaring that a variable or method belongs to the **class itself** (stored in Metaspace) rather than to any individual instance on the heap.
- **`java.lang.Object`**: The single root ancestor of the entire Java class hierarchy. Every class automatically inherits from it.

---

## 1. Variable Scopes, Lifetimes & Initialization Invariants

Java treats variables on the stack fundamentally differently from variables on the heap.
↳ *For physical allocation mechanics, hardware caching, and pointer tracing: [01 - Platform & Execution Model: Stack vs. Heap Allocation Mechanics](file:///c:/Users/rayre/Learning%20with%20AI/ide-md-context/Learn%20Java/01%20-%20Platform%20&%20Foundations/01%20-%20Platform%20&%20Execution%20Model.md#physical-reality-stack-vs-heap-allocation-mechanics).*

```
       Stack (Thread execution)                 Heap (Shared Memory)
┌──────────────────────────────────────┐     ┌────────────────────────┐
│ Method Frame: doWork()               │     │ Instance: Employee     │
│  ├── int localCounter (no default!)  │     │  ├── firstName: null   │
│  └── Employee ref ───────────────────┼────►│  ├── lastName:  null   │
│                                      │     │  └── salary:    0.0    │
└──────────────────────────────────────┘     └────────────────────────┘
```

### Local Variables (Call Stack)
- Declared inside methods, constructors, or loops.
- Allocated directly on the thread's stack frame.
- **Definite Assignment Rule**: The JVM does *not* zero-initialize stack memory. Reusing stack memory from previous method executions means uninitialized stack slots contain stale memory bits. To prevent reading garbage data, Java strictly refuses to compile code that reads an unassigned local variable:
  ```java
  void calculate() {
      int total; // declared but unassigned
      // System.out.println(total); // COMPILE ERROR: variable total might not have been initialized
  }
  ```

### Instance Fields (Heap Allocation)
- Declared directly in a class outside any method.
- Stored inside the object's allocated heap memory block.
- **Deterministic Zero-Initialization**: When `new` is called, the JVM zeroes the entire heap block before any constructor logic runs. Fields therefore automatically receive guaranteed zero-values:
  - Numerical primitives (`byte`, `short`, `int`, `long`, `float`, `double`) $\to$ `0` or `0.0`
  - `char` $\to$ `'\u0000'` (null character)
  - `boolean` $\to$ `false`
  - Any reference type (e.g. `String`, custom objects) $\to$ `null`

> [!WARNING]
> While default zero-initialization prevents memory corruption, relying on default `null` references is the leading cause of `NullPointerException`. Robust classes explicitly initialize fields or enforce invariants inside constructors.

---

## 2. Constructors & Constructor Delegation (`this(...)`)

A constructor's job is not just to allocate memory (the JVM does that before the constructor runs); its job is to **establish class invariants**.

### Constructor Overloading & Centralized Initialization
When a class offers multiple ways to initialize an object (e.g. defaults for omitted parameters), chaining constructors via `this(...)` ensures a single point of validation:

```java
public class Stuff {
    private final String unit;
    private final double value;

    // Canonical / Master Constructor: single place where invariants are enforced
    public Stuff(String unit, double value) {
        if (unit == null || unit.isBlank()) {
            throw new IllegalArgumentException("Unit must not be blank");
        }
        this.unit = unit;
        this.value = value;
    }

    // Convenience Constructor 1: defaults value to 0
    public Stuff(String unit) {
        this(unit, 0.0); // Delegates to master constructor
    }

    // Convenience Constructor 2: defaults both
    public Stuff() {
        this("default_unit", 0.0); // Consistent state, no duplicated validation
    }
}
```

### The Invariants of Constructor Chaining
1. **Must be the first statement**: In any constructor, a call to `this(...)` or `super(...)` must be the absolute first line of code.
   - *Why*: Java guarantees that the superclass and master constructor establish base state before any child or auxiliary code executes. You cannot inspect or mutate state before delegating.
2. **Cycle Prevention**: Circular delegation (`A calls B, and B calls A`) is detected and rejected at compile time as a recursive constructor invocation error.

---

## 3. Dispatch Mechanics: Overloading vs. Overriding

Understanding method dispatch is the difference between static compile-time binding and dynamic runtime polymorphism.

| Feature | Method Overloading | Method Overriding |
| :--- | :--- | :--- |
| **Where** | Same class (or subclass) | Subclass redefining a superclass method |
| **Signature** | Same name, **different** parameters | Same name, **identical** parameters and compatible return type |
| **Resolution** | **Compile-time (Static binding)** based on reference type | **Runtime (Dynamic dispatch)** based on actual heap object |
| **Annotations** | None | `@Override` (strongly recommended) |

### Method Signatures & Overloading Rules
In Java, a method's signature consists exclusively of:
1. The method name.
2. The ordered list of parameter types.

```java
class Employee {
    void paySalary(int value) {}
    
    // INVALID: Parameter name differences DO NOT alter the signature
    // void paySalary(int amount) {} // COMPILE ERROR: duplicate method

    // INVALID: Return type DOES NOT alter the signature
    // double paySalary(int value) {} // COMPILE ERROR: duplicate method

    // VALID: Different parameter count/types creates a distinct signature
    void paySalary(int value, int bonus) {}
}
```

*Why return types cannot differentiate overloaded methods*: When a caller writes `employee.paySalary(100);` and ignores the return value, the compiler has no way to determine which overload was intended if they share identical parameter lists.

### Method Overriding & `@Override`
Overriding allows a derived class to replace the behavior of an inherited method:

```java
public class Experiment {
    public String getSummary() {
        return "Experiment";
    }
}

public class BetterExperiment extends Experiment {
    @Override // Compiler verifies that getSummary() actually exists in Experiment
    public String getSummary() {
        // super invokes the parent class implementation
        return super.getSummary() + " but BETTER";
    }
}
```

- **Dynamic Dispatch**: If a variable is typed as `Experiment exp = new BetterExperiment();`, calling `exp.getSummary()` will invoke `BetterExperiment`'s version at runtime via the JVM's virtual method table (vtable).
- **The Purpose of `@Override`**: The annotation is technically optional, but omission is dangerous dogma. If you misspell `getSumary()` without `@Override`, the compiler treats it as a brand-new method rather than an override. The parent method remains active, silently breaking polymorphic behavior.

---

## 4. Static Context vs. Instance Context

The `static` modifier shifts a member from individual heap instances to the class level in Metaspace.

```java
public class Counter {
    private static int globalCount = 0; // Exactly 1 copy exists in Metaspace
    private int instanceCount = 0;       // Exists inside every Counter instance on Heap

    public static int incrementGlobal(int delta) {
        globalCount += delta;
        // instanceCount += delta; // COMPILE ERROR: Cannot make static reference to non-static field
        return globalCount;
    }

    public void incrementInstance(int delta) {
        this.instanceCount += delta; // Valid: instance method has access to 'this'
        globalCount += delta;        // Valid: instance methods can access class-level state
    }
}
```

### The Receiver Invariant: Why Static Cannot Access Instance Members
- An instance method receives an implicit hidden first parameter: `this`, representing the address of the specific object on which the method was called.
- A `static` method has **no receiver object**. When `Counter.incrementGlobal(5)` runs, no instance was involved in the call. Because there is no `this`, the compiler cannot determine *which* instance's `instanceCount` you would want to modify.

### The Calling-Static-From-Instance Anti-Pattern
Java permits invoking a static method via an instance reference, but doing so is deceptive and dangerous:

```java
Counter c = new Counter();
c.incrementGlobal(1); // Compiles, but is misleading!
```

- **The Illusion of Polymorphism**: Calling `c.incrementGlobal(1)` looks like virtual dispatch, but the compiler rewrites the call to `Counter.incrementGlobal(1)` using the *compile-time declared type* of `c`, not its runtime instance.
- **Best Practice**: Always invoke static members directly using the Class identifier (`Counter.incrementGlobal(1)`).

---

## 5. The Root of Everything: `java.lang.Object`

Every class in Java implicitly extends `java.lang.Object`. If you declare:

```java
public class Experiment {}
```

The compiler transforms it into:

```java
public class Experiment extends Object {}
```

Because of this universal root, every Java object inherits four critical identity and representation methods:

| Method | Default `Object` Implementation | What it Represents |
| :--- | :--- | :--- |
| `getClass()` | Native JVM lookup | Returns the runtime `Class<?>` metadata token of the instance. |
| `equals(Object o)` | `return (this == o);` | Reference identity: returns `true` only if both point to the exact same heap memory address. |
| `hashCode()` | Native memory address / random state | Numerical hash used in hash-based collections (`HashMap`, `HashSet`). Invariant: if `a.equals(b)`, then `a.hashCode() == b.hashCode()`. |
| `toString()` | `getClass().getName() + "@" + Integer.toHexString(hashCode())` | Default identity string. |

### Demystifying the Default `toString()` Output
When printing an un-customized object, you see strings like `Experiment@5e2de80c`.
- **What this actually is**:
  ```java
  public String toString() {
      return getClass().getName() + "@" + Integer.toHexString(hashCode());
  }
  ```
  - `getClass().getName()` produces `"Experiment"`.
  - `@` is a literal separator.
  - `Integer.toHexString(hashCode())` converts the default identity hash code (derived by the JVM from object memory identity) into a hexadecimal string (`5e2de80c`).
- **Takeaway**: The default `toString()` exposes object identity, not object *state*. Real domain classes override `toString()` to display meaningful fields for debugging and logging.

---

## 6. Encapsulation & Gating Invariants

Classes use private fields and gated methods to enforce state invariants rather than simply wrapping fields in boilerplate getters and setters.

> [!TIP]
> For a deep dive into why mechanical getters and setters fail to provide true encapsulation, how invariants protect domain state, and the difference between implementation churn and semantic churn, see [OOP & Encapsulation](file:///c:/Users/rayre/Learning%20with%20AI/ide-md-context/Learn%20Java/02%20-%20Object%20Mechanics%20&%20Design/02%20-%20OOP%20&%20Encapsulation.md).

# Hibernate Sessions & Transactions

## What is Hibernate?

A translator between Java objects and database rows. Instead of writing SQL manually, you work with Java objects and Hibernate generates the SQL for you.

```java
// Without Hibernate
"INSERT INTO config (name, policy_id) VALUES ('Default', 42)"

// With Hibernate
repository.save(configuration);
```

---

## The Session (identity map / "cart")

Hibernate doesn't talk to the DB on every line. It keeps a **session** — a local cache of everything it has loaded or saved.

- You load an entity → Hibernate puts it in the cart
- You modify it → Hibernate notes the change
- Transaction ends → Hibernate flushes → writes SQL to DB

The session tracks objects by reference. When you call `save(obj)`, Hibernate checks: **"is this object in my cart?"**

| Answer | What Hibernate does |
|---|---|
| Yes (managed) | `UPDATE` — it knows the row already exists |
| No (detached) | `INSERT` — it assumes the object is brand new |

---

## Managed vs Detached

```java
// MANAGED — session A loaded this itself
TransactionRuleConfiguration config = repository.findById(id);
repository.save(config); // → UPDATE ✅

// DETACHED — came from a different session
TransactionRuleConfiguration config = sessionB.save(something);
repository.save(config); // → INSERT 💥 (duplicate row if it already exists)
```

**Detached does not mean the DB row is gone.** The row is real and committed. The Java object just has no session tracking it anymore.

---

## @Transactional

Wraps the entire method in one DB transaction. Nothing is committed until the method returns. If an exception is thrown, everything rolls back.

```java
@Transactional
public void doWork() {
    // all SELECTs, INSERTs, UPDATEs here are in one session (session A)
    // committed atomically at the end
}
```

---

## PROPAGATION_REQUIRES_NEW

Pauses the outer transaction (session A), opens a brand new independent transaction (session B), commits it immediately, then resumes session A.

Used when you need an operation to commit independently — even if the outer transaction later rolls back.

```java
// session A is running...

TransactionTemplate requiresNew = new TransactionTemplate(txManager);
requiresNew.setPropagationBehavior(PROPAGATION_REQUIRES_NEW);

Long id = requiresNew.execute(status -> {
    // ── session B: independent, commits immediately on exit ──
    return repository.save(entity).getId(); // only the Long escapes
    // ── session B closes here ────────────────────────────────
});

// back in session A — entity from session B is now DETACHED
repository.findById(id); // session A loads it itself → MANAGED ✅
```

Why only pass the `Long` out? Because a `Long` is a plain number — it has no session attachment. The entity object would be detached the moment session B closes.

---

## The Race Condition Pattern (why REQUIRES_NEW is used here)

When two concurrent requests both try to create the same row:

```
Thread A: SELECT → not found → INSERT ✅
Thread B: SELECT → not found → INSERT 💥 (unique key violation)
```

The catch block handles the loser gracefully:

```java
try {
    persistedId = requiresNew.execute(status ->
        repository.save(configuration).getId());
} catch (DataIntegrityViolationException e) {
    // lost the race — row already exists, just return it
    return repository.findByPolicyIdAndSystemTrue(policy.getId())
                     .orElseThrow(() -> e);
}
```

Without `REQUIRES_NEW`, a failed INSERT poisons the outer session — no more queries allowed on it. The isolation means only session B rolls back; session A stays healthy and can still do the fallback SELECT.

---

## Summary

| Concept | One line |
|---|---|
| Session | Hibernate's local cart of tracked objects |
| Managed | Object is in the session's cart → `save()` = UPDATE |
| Detached | Object exists in memory, no session owns it → `save()` = INSERT |
| @Transactional | One session for the whole method, committed at the end |
| REQUIRES\_NEW | Opens a second independent session, commits immediately |
| Safe boundary crossing | Only pass primitives (e.g. `Long id`) between sessions, then re-fetch |

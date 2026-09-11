# Tailwind CSS Limitation: Dynamic Class Names

## The Problem

Constructing utility class names dynamically with string interpolation does not work in Tailwind CSS:

```ts
// ❌ Broken: Tailwind cannot detect these classes
const color = "green"
const className = `text-${color}-500`
```

The styles are not generated, and elements render unstyled.

---

## Why This Happens

Tailwind uses static regex extraction at build time, not runtime JS evaluation:

- **Static Scanner**: Only matches literal, unbroken class strings (e.g. `text-green-500`).
- **No Evaluation**: Expressions like `${color}` are never resolved.
- **Purging**: Any class not found as a complete literal token is omitted from the bundle.

---

## Solutions

### Approach 1: Complete Static Lookup Maps

Map states directly to complete class strings:

```ts
// ✅ Full static strings visible to the scanner
const STATUS_STYLES = {
  in_progress: "text-amber-500 bg-amber-400/10",
  watched: "text-emerald-500 bg-emerald-400/10",
}
```

*Tradeoff*: Straightforward for simple classes, but repetitive when managing complex interactive states (`:hover`, `[data-state="on"]`, focus rings) across many colors.

---

### Approach 2: Custom `@utility` + Scoped CSS Variables (Tailwind v4)

Decouple **theme tokens** from **interactive styling** by composing static utilities:

```css
/* 1. Theme modifiers: define the color token */
@utility status-progress {
  --status-color: var(--color-amber-400);
}
@utility status-watched {
  --status-color: var(--color-emerald-400);
}

/* 2. Component utility: consume token across states */
@utility status-badge {
  color: var(--status-color);
  background-color: color-mix(in srgb, var(--status-color) 10%, transparent);

  &:hover {
    background-color: color-mix(in srgb, var(--status-color) 20%, transparent);
  }
}
```

```ts
// ✅ Statically compose component + modifier utilities
const STATUS_CONFIG = {
  in_progress: { className: "status-badge status-progress" },
  watched: { className: "status-badge status-watched" },
}
```

#### Key Advantages
- **Static token extraction**: Both classes are static literals visible to the Tailwind compiler.
- **DRY state logic**: Interaction states (`:hover`, opacity via `color-mix`) are authored once in CSS.
- **Clean JSX**: Avoids inline `style` objects.

---

### Approach 3: Inline CSS Variables (Runtime Colors)

For arbitrary values only known at runtime (e.g., user-picked hex colors):

```tsx
// ✅ Bridge dynamic runtime values into static utilities
<button
  style={{ "--accent": user.colorHex } as React.CSSProperties}
  className="text-[var(--accent)] hover:bg-[var(--accent)]/10"
>
```

---

## Rule of Thumb

> **Always write complete, unbroken class names in source code.**
> - Fixed theme variants with interactive states → **Compose Tailwind v4 `@utility` + scoped CSS variables**.
> - Pure runtime values (user-defined hex codes) → **Bridge via inline CSS variables**.
> - Simple one-offs → **Static lookup map**.

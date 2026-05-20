# Custom Button Pattern
## Contents
- [When to Use Shadcn Button vs Native Button](#when-to-use-shadcn-button-vs-native-button)
- [Base Button Reset Class](#base-button-reset-class)
- [Property Breakdown](#property-breakdown)
- [Caveat — transition-all](#caveat--transition-all)
- [Usage Patterns](#usage-patterns)
---

## When to Use Shadcn Button vs Native Button

Not every `<button>` should use shadcn's `<Button>` component. A quick decision guide:

| Use shadcn `<Button>` | Use native `<button>` |
|---|---|
| CTA / form submit actions | Icon-only controls |
| Labeled actions ("Add to cart") | Custom-shaped elements (dots, pills) |
| Consistent variant system matters | One-off styled interactions |
| Needs `asChild`, loading state, etc. | Simple click handlers |

> Shadcn `<Button>` ships with padding, sizing, and variant styles that fight against heavily custom shapes like dot indicators or floating icon controls.

---

## Base Button Reset Class

Instead of repeating browser-reset boilerplate across every custom button, extract it into a shared CSS class:

```css
.custom-button {
  -webkit-tap-highlight-color: transparent;

  @apply appearance-none bg-transparent border-0 p-0
    cursor-pointer select-none
    flex justify-center items-center
    outline-none focus-visible:border-ring focus-visible:ring-ring/50
    focus-visible:ring-[3px]
    disabled:opacity-50 disabled:cursor-not-allowed
    transition-colors duration-150;
}
```

This sits between raw `<button>` and shadcn `<Button>` — full visual control, zero repeated boilerplate.

> **⚠️ Wrap in `@layer components`** — without it, `@apply` rules compile after Tailwind utilities and win on specificity. Any `bg-*` or `p-*` you add in `className` will be silently overridden.
> ```css
> @layer components {
>   .custom-button { ... }
> }
> ```
> Inside `@layer components`, utilities always win — no `!important` needed.

---

## Property Breakdown

| Property | Why |
|---|---|
| `-webkit-tap-highlight-color: transparent` | Removes the blue flash on mobile tap |
| `appearance-none` | Strips native browser button styles |
| `bg-transparent border-0 p-0` | Full visual reset, start from zero |
| `cursor-pointer select-none` | Expected button UX behaviour |
| `flex justify-center items-center` | Almost always needed for icon buttons |
| `outline-none focus-visible:ring-[3px]` | Accessible focus ring, hidden for mouse users |
| `disabled:opacity-50 disabled:cursor-not-allowed` | Standard disabled state |
| `transition-colors duration-150` | Smooth state changes |

---

## Caveat — transition-all

Avoid `transition-all` in the base class — it transitions every CSS property including `width`, `height`, and `transform`, which can cause unintended animations or minor performance issues on complex layouts.

```css
/* ❌ Too broad */
transition-all duration-150

/* ✅ Scoped */
transition-colors duration-150

/* ✅ Explicit if you need more */
transition-[opacity,background-color] duration-150
```

Only reach for `transition-all` on specific components where you know multiple properties animate together (e.g. a pill indicator that changes both width and color).

---

## Usage Patterns

**Basic icon button:**
```tsx
<button className="custom-button hover:bg-accent rounded-full p-2">
  <ChevronLeft className="size-4" />
</button>
```

**Carousel nav button (overlay style):**
```tsx
<button className="custom-button bg-white/80 hover:bg-white rounded-full p-1 shadow">
  <ChevronRight className="size-4" />
</button>
```

**Compose with `cva()` for variants:**
```tsx
const button = cva("custom-button", {
  variants: {
    variant: {
      ghost: "hover:bg-accent rounded-md px-2 py-1",
      pill:  "bg-white/80 hover:bg-white rounded-full p-1 shadow",
      dot:   "rounded-full bg-white/50 data-[active=true]:bg-white",
    },
  },
})

// Usage
<button className={button({ variant: "ghost" })}>
  <X className="size-4" />
</button>
```

> `cva()` is from the `class-variance-authority` package — the same utility shadcn uses internally.

> **💡 VS Code Preview:** `Ctrl+Shift+V` (or `Cmd+Shift+V`) — or click the split-pane icon top-right.
# Hero Project — Lessons Learned

A record of bugs I hit as a learner, what caused them, and how I fixed them. Written as a reference to come back to if I run into similar issues.

---

## 1. Element not centring despite `justify-content: center`

**Problem:** The "Some random information." heading looked off-centre even though `.second-title` had `display: flex; justify-content: center` on it.

**Root cause:** The parent container `.full-second-section` had asymmetric padding — `520px` on the left and `300px` on the right. The title was centred *within its own box*, but that box was already shifted right of the true page centre because the parent's padding was lopsided. The title's own padding (`30px 250px 30px 0px`) made things worse by being asymmetric in the opposite direction.

**Fix:** Removed all asymmetric padding from `.full-second-section`, set it to `align-items: center` as a flex column, and zeroed out the lopsided padding on `.second-title`. Every child now centres itself naturally based on its own width.

**Lesson:** When centering isn't working, check the *parent's* padding first. A perfectly centred child inside a shifted box still looks off. Asymmetric padding is a common culprit.

---

## 2. CSS not applying to an element (`font-weight: 900` ignored)

**Problem:** `.call-to-action { font-weight: 900 }` had no effect, and neither did targeting it as `h1` or `.blue-content h1`.

**Root cause:** A missing `>` in the HTML:

```html
<!-- broken -->
<div class="blue-content"
<div class="call-to-action">Call to action! It's time!</div>
```

The browser's HTML parser was still expecting more attributes on `.blue-content` when it hit the next `<div`, so it treated everything up to the closing `>` of `class="call-to-action"` as one mangled tag. The `.call-to-action` div never existed as a real element in the DOM. CSS can't target an element that doesn't exist, no matter how correct the rule is.

**Fix:** Close the opening tag properly:

```html
<!-- fixed -->
<div class="blue-content">
    <div class="call-to-action">Call to action! It's time!</div>
```

**Lesson:** If a CSS rule is doing absolutely nothing, open DevTools → Elements panel and check whether the element actually exists in the rendered DOM. If it's missing from the tree, the bug is in the HTML, not the CSS.

---

## 3. Spacing between flex children coming from browser defaults

**Problem:** The four `<figure>` images had gaps between them even though no `gap` was set on the flex container.

**Root cause:** Browsers ship with a built-in default style roughly equivalent to `figure { margin: 1em 40px; }`. The spacing was coming from that user-agent stylesheet, not from anything written explicitly. This is fragile because it's an invisible dependency that can vary slightly between browsers.

**Fix:** Zero out the default margin and set `gap` explicitly on the flex container so there is one clear source of truth for the spacing:

```css
.image-container figure {
    margin: 0;
}

.image-container {
    gap: 40px;
}
```

**Lesson:** Unexpected spacing is often browser default margins, especially on `figure`, `p`, `h1`–`h6`, `ul`, and `body`. Always zero out defaults you didn't write if you want full control. Use `gap` on flex/grid containers for spacing between children rather than relying on margins.

---

## 4. Unnecessary or no-op CSS rules

Redundant rules I found and cleaned up, with the reason each was unnecessary:

| Rule | Why it's unnecessary |
|---|---|
| `display: flex; justify-content: center` on `.second-title` alongside `text-align: center` | The div only contains plain text. `text-align: center` alone centres it. Flex properties had no additional effect. |
| `max-width: 100%` on block-level containers | Block elements don't exceed their parent's width by default. Capping at 100% changes nothing unless something is forcing them wider. |
| `display: flex; align-items: center` on `.full-third-section p` | `align-items` needs a container with a defined height to do anything. This `<p>` had no set height so its box just shrunk to fit the text — nothing to centre against. |
| `display: flex; justify-content: flex-end` on `.random-person` | `text-align: right` does the same job on a plain text div without the overhead of making it a flex container. |

---

## 5. Consistent max-width and centering across sections

**Problem:** `.header-container` used `width: 62%` while `.hero-container` used `width: 100%; max-width: 900px`. Different sizing logic means their edges land in different positions depending on screen width, so elements can visually drift out of alignment.

**Fix:** Gave `.header-container` the same sizing as `.hero-container`:

```css
.header-container {
    width: 100%;
    max-width: 900px;
    margin: 0 auto;
    padding: 4px 40px 16px;
}
```

**Lesson:** The reliable way to keep a multi-section page aligned is to use `width: 100%; max-width: Npx; margin: 0 auto` on every section's inner content wrapper, using the same `max-width` value throughout. This is the "centered container" pattern. The outer sections (the coloured bands) stay full width; only the inner wrappers get constrained.

Using fixed pixel padding (e.g. `padding: 50px 300px`) to fake centering is fragile — it only looks centred at one specific viewport width and breaks on anything wider or narrower.

---

## 6. Responsive design (to revisit)

The project currently has no responsive design — no media queries, no `flex-wrap`, and several hardcoded pixel widths (hero image at `391px`, `.blue-box` at `1000px`, padding values of `300px`). On a phone screen (~375–430px wide) this would cause horizontal scrolling and broken layouts.

**Topics to come back to after covering them in The Odin Project:**
- `@media` queries and breakpoints
- `flex-wrap: wrap` to restack rows into columns on narrow screens
- Replacing fixed `px` widths with `%`, `max-width`, or `min()` where appropriate
- Mobile-first vs desktop-first approach

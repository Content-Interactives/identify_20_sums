# Identify 20 Sums

React (JSX) drill: a **target sum** (random **6–20**) is shown with **four** expressions **`a + b`**. Exactly **one** evaluates to the target; the learner taps an expression. Correct choice runs a **fade-out**, **canvas-confetti**, success copy, then **`generateNumberAndSums`** after ~4s. Wrong choice **shakes** that button.

**Live site:** [https://content-interactives.github.io/identify_20_sums](https://content-interactives.github.io/identify_20_sums)

Standards and curriculum: [Standards.md](Standards.md).

---

## Stack

| Layer | Notes |
|--------|--------|
| Build | Vite 7, `@vitejs/plugin-react` |
| UI | React 19 (`.jsx` only—**not** TypeScript) |
| Styling | Tailwind 3, `Identify20Sums.css`, `fade.css` from `ui/reused-animations/` |
| Effects | `canvas-confetti` |
| Lint / deploy | ESLint 9; `gh-pages -d dist` with `predeploy` build |

---

## Layout

```
vite.config.js          # base: '/identify_20_sums/'
src/
  main.jsx → App.jsx → components/Identify20Sums.jsx
  components/
    Identify20Sums.jsx, Identify20Sums.css
    ui/reused-ui/       # Container (+ optional sound/reset)
    ui/reused-animations/
```

---

## Question generation (`generateNumberAndSums`)

1. **`newNumber`** = `Math.floor(Math.random() * 15) + 6` → **uniform 6…20** (inline comment says “7–20”; the implementation includes **6**).
2. **`availableNumbers`** = `[1, …, newNumber - 1]`.
3. **Correct pair:** enumerate **`x`** in `1…newNumber-1`, **`y = newNumber - x`**, require **`y` ∈ [1,15]**, **`x ≠ y`**, both still in **`availableNumbers`**. Pick a random valid pair, push `` `${x} + ${y}` ``, remove **`x`** and **`y`** from the pool.
4. **Three distractors:** loop until **four** strings total (up to 100 attempts per inner logic). Prefer drawing two distinct values from **`availableNumbers`** when at least two remain; require **`x + y ≠ newNumber`**. If the pool is exhausted, fall back to random **`x, y`** in **`1…newNumber-1`** with **`x ≠ y`** and wrong sum.
5. **Final while:** if still short of four, add non-duplicate wrong expressions (same random range).
6. **Shuffle** in place (reverse Fisher–Yates).

Parsing answers uses **`sum.split(' + ')`**—expressions must keep that exact spacing format.

---

## Answer handling (`handleAnswerClick`)

- Parses operands, compares **`x + y`** to **`number`**.
- **Correct:** sets **`isProcessingCorrectAnswer`** to block double-clicks; **`buttonFadeState`** to **`fade-out`** (all four buttons get the fade class); confetti at 200 ms; success UI at 200 ms; at **4000 ms** clears flags and calls **`generateNumberAndSums`**.
- **Incorrect:** **`shakeButton === index`** for 500 ms.

Success view shows **`{clickedExpression} = {number}`** and a random string from **`messages`**.

---

## `Container` props

- **`showSoundButton={true}`** with **no `onSound`** — icon is present but does nothing unless you pass a handler.
- **`showResetButton={false}`** — no in-app reset.

---

## `vite.config.js`

**`base: '/identify_20_sums/'`** must match the GitHub Pages path (underscores). Adjust if the repo slug changes.

---

## Scripts

| Command | Purpose |
|---------|---------|
| `npm run dev` | Vite dev server |
| `npm run build` | `dist/` |
| `npm run preview` | Preview production build |
| `npm run lint` | ESLint |
| `npm run deploy` | Build + gh-pages |

---

## Embedding

- **`Container`** uses a fixed **~500px** tall chrome (see `Container.jsx`); grid of four **80px**-tall buttons in two columns.
- No LMS / `postMessage` API.

---

## Maintenance

- Fix the **target range comment** in code (`7–20` vs actual **6–20**) or change the formula if **7** was intended.
- **`useRef`** could replace the long **`setTimeout`** chain for cleanup on unmount.
- Consider resetting **`buttonFadeState`** inside **`generateNumberAndSums`** explicitly before other resets (already **`'visible'`** there).

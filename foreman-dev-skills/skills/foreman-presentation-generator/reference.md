# Foreman Presentation Generator — Reference

Full slide-design rules and the HTML / speaker-notes templates. `SKILL.md`
links here; follow these exactly when generating a deck.

## Slide Design Rules

These rules were established through iteration and MUST be followed.

### Layout & Structure

1. **Every content slide** uses two divs: `<div class="slide-title">` then `<div class="slide-body">`. No exceptions.
2. **`center: false`** in Reveal config — all slides pin to the top. Never use Reveal's vertical centering.
3. **Title and thank-you slides** use `data-background-color="#1a5276"` (teal). All other slides use `data-background-color="#ffffff"`.
4. **Two-column layouts** use `<div class="two-col"><div class="col">...</div><div class="col">...</div></div>`.
5. **Architecture/flow diagrams** use `<div class="diagram">` with `<span class="bad">` for problems and `<span class="good">` for solutions.
6. **Before/after diagrams** — put each item on its own line with `<br>`. Do NOT separate with `|` on a single line.
7. **No inline margin-top overrides** — spacing is controlled by the CSS classes.
8. **Foreman logo** URL: `https://theforeman.org/static/images/foreman.png`
9. **PR references** use `<span class="pr">PR #608</span>`.
10. **Explanatory text** below code blocks should use `<p>` with the `.two-col p` styling (smaller, muted color).

### Content Fitting — CRITICAL

Every slide MUST fit within the visible viewport without scrolling. Apply these budgets uniformly to every slide:

11. **Never add `overflow: scroll` or `overflow-y: auto`** to slides. If content doesn't fit, split the slide.
12. **Vertical content budget** — A content slide has roughly 420px of usable vertical space (after the slide-title bar). Before writing a slide, mentally budget the height:
    - `<h3>` heading: ~30px
    - `<p>` paragraph: ~25px per line
    - `<ul>` bullet: ~22px per item
    - `<pre>` code block: ~20px per line of code + 20px padding
    - `<div class="diagram">`: ~20px per line + 24px padding
    - Two-column `<h3>` sub-headings: ~30px each
    If the total exceeds ~420px, split into two slides or cut content.
13. **Code blocks in two-column layouts**: Maximum **7 lines** of code per column. Use the default `0.48em` font size — never use `0.58em` inside two-column layouts.
14. **Code blocks in full-width layouts**: Maximum **10 lines** at `0.48em`, or **8 lines** at `0.58em`.
15. **Standalone full-width code blocks** may use `<pre style="font-size:0.58em;">`. Two-column code blocks must use the default `0.48em`.
16. **One code block per column** — never stack two `<pre>` blocks vertically inside a single `.col`.
17. **Trim code for slides** — show only the essential lines that illustrate the point. Strip comments, collapse boilerplate parameters, and omit obvious fields. Put the full context in speaker notes, not on the slide.
18. **Uniform density across slides** — every slide should feel similarly weighted. If one slide has 3 bullets and another has 2 code blocks + 6 bullets + a diagram, rebalance by splitting or trimming. The audience should never feel a jarring shift in density.

## HTML Template

Use this exact boilerplate. Replace the slide `<section>` blocks with your content.

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PRESENTATION_TITLE</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/dist/reveal.css">
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/dist/theme/white.css">
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/plugin/highlight/github.css">
<style>
  @import url('https://fonts.googleapis.com/css2?family=Red+Hat+Display:wght@400;500;700&family=Red+Hat+Mono:wght@400;500&display=swap');

  :root {
    --teal: #1a5276;
    --text: #333;
    --muted: #666;
    --bg: #f5f6f8;
    --border: #dde;
    --red: #c0392b;
    --green: #27ae60;
  }

  .reveal {
    font-family: 'Red Hat Display', 'Segoe UI', sans-serif;
  }

  .reveal .slides section {
    padding: 0 !important;
    text-align: left;
    box-sizing: border-box;
  }

  .reveal h1, .reveal h2, .reveal h3 {
    color: var(--teal); font-family: inherit;
    font-weight: 700; text-transform: none; letter-spacing: -0.02em;
  }
  .reveal h3 { font-size: 0.9em; margin: 8px 0 4px 0; color: var(--teal); }
  .reveal p, .reveal li { color: var(--text); font-size: 0.68em; line-height: 1.5; margin: 3px 0; }
  .reveal ul { margin: 4px 0 4px 20px; }
  .reveal code { font-family: 'Red Hat Mono', monospace; font-size: 0.92em; color: var(--teal); }
  .reveal pre { width: 100%; font-size: 0.48em; margin: 6px 0;
    box-shadow: 0 1px 4px rgba(0,0,0,0.08); border-radius: 4px; }
  .reveal pre code { padding: 10px 14px; border-radius: 4px; }

  .slide-title {
    display: flex; align-items: center; gap: 14px;
    padding: 14px 36px;
    background: var(--teal);
    height: 36px;
  }
  .slide-title img { height: 32px; }
  .slide-title h2 {
    margin: 0; font-weight: 600; font-size: 1.15em; color: #fff;
  }

  .slide-body {
    padding: 16px 36px 12px 36px;
  }

  .bad { color: var(--red); font-weight: 600; }
  .good { color: var(--green); font-weight: 600; }

  .two-col { display: flex; gap: 20px; text-align: left; margin-top: 8px; }
  .two-col .col { flex: 1; }
  .two-col li { font-size: 0.55em; }
  .two-col p { font-size: 0.48em; color: var(--muted); }

  .diagram {
    background: var(--bg); border: 1px solid var(--border); border-radius: 6px;
    padding: 12px; margin: 8px 0; font-family: 'Red Hat Mono', monospace;
    font-size: 0.48em; line-height: 1.8; text-align: center; color: var(--text);
  }
  .diagram .lbl { color: var(--muted); font-family: 'Red Hat Display', sans-serif; font-size: 0.9em; }

  .pr { display: inline-block; background: rgba(26,82,118,0.1); border: 1px solid var(--teal);
    border-radius: 3px; padding: 1px 8px; font-size: 0.65em; color: var(--teal); font-weight: 700; }
</style>
</head>
<body>
<div class="reveal">
<div class="slides">

<!-- Title Slide -->
<section data-background-color="#1a5276">
  <div style="text-align:center; padding:50px 36px;">
    <img src="https://theforeman.org/static/images/foreman.png" alt="Foreman" style="height:80px; margin-bottom:24px;">
    <h1 style="color:#fff; font-size:2em;">PRESENTATION_TITLE</h1>
    <p style="color:rgba(255,255,255,0.6); font-size:0.7em;">SUBTITLE</p>
    <br>
    <p style="font-size:0.6em; color:rgba(255,255,255,0.35);">PRESENTER_NAME</p>
  </div>
</section>

<!-- Content Slide Template -->
<section data-background-color="#ffffff">
  <div class="slide-title">
    <img src="https://theforeman.org/static/images/foreman.png" alt="">
    <h2>SLIDE_HEADING</h2>
  </div>
  <div class="slide-body">
    <!-- Content here -->
  </div>
</section>

<!-- Thank You Slide -->
<section data-background-color="#1a5276">
  <div style="text-align:center; padding:50px 36px;">
    <img src="https://theforeman.org/static/images/foreman.png" alt="Foreman" style="height:80px; margin-bottom:24px;">
    <h1 style="color:#fff; font-size:2.2em;">Thank You!</h1>
    <p style="color:rgba(255,255,255,0.6); font-size:0.8em;">Questions?</p>
  </div>
</section>

</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/dist/reveal.js"></script>
<script src="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/plugin/highlight/highlight.js"></script>
<script>
  Reveal.initialize({
    hash: true,
    transition: 'slide',
    margin: 0.04,
    center: false,
    plugins: [ RevealHighlight ]
  });
</script>
</body>
</html>
```

## Speaker Notes Template

```markdown
# Presentation Notes — TITLE

## Slide-by-Slide Speaker Notes

### Slide 1 — Title

Brief intro of what you'll cover.

### Slide 2 — HEADING

Talking points as natural prose...

### Slide N — Thank You

Open for questions.

---

## Anticipated Questions & Answers

### Topic Area

**Q: Question text?**

Answer paragraph.
```

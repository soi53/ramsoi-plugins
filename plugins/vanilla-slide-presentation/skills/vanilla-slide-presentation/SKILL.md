---
name: vanilla-slide-presentation
description: >
  Build a complete, single-file HTML/CSS/JS slide presentation without any external libraries.
  Use this skill whenever the user wants to create a presentation, slideshow, 슬라이드, 발표자료,
  프레젠테이션, or any kind of slide-based content that runs in a browser — even if they don't
  say "vanilla" or "HTML". Supports arrow key navigation, F for fullscreen, slide number display,
  smooth slide transitions, and speaker notes out of the box.
---

# Vanilla Slide Presentation Skill

Create a polished, single-file HTML presentation using only HTML, CSS, and JavaScript.
No Reveal.js, no external CDN, no dependencies — everything in one `.html` file.

## Architecture (always use this structure)

```
presentation-viewport (100vw x 100dvh, Flex center)
  └── slide-deck (aspect-ratio: 16/9, CSS Grid stack, container-type: size)
        ├── section.slide.active   ← visible slide (grid-area: 1/1)
        ├── section.slide          ← hidden slides (grid-area: 1/1, opacity:0)
        ├── .progress-bar          ← bottom strip
        ├── .slide-counter         ← "1 / N" bottom-right
        └── #notes-panel           ← speaker notes (S key toggle)
```

**Key CSS insight**: All slides share `grid-area: 1/1` so they stack in the same cell.
Only `.active` has `opacity:1; visibility:visible`.

## Required features (always include)

### Navigation
- `←` `→` `↑` `↓` `PageUp` `PageDown` — prev/next slide
- `Space` — next, `Shift+Space` — prev
- `Home` / `End` — first / last slide
- `F` — toggle fullscreen
- `S` — toggle speaker notes panel
- `Esc` — close notes panel

### UI elements
- **Progress bar** at bottom: `transform: scaleX(progress)` — GPU accelerated
- **Slide counter** bottom-right: "1 / 5" format
- **Speaker notes**: `<aside class="speaker-notes">` inside each slide

### Slide transition animation
Use CSS class-based state machine:

```
1. Add .entering-forward to next slide
2. Force reflow: void next.offsetHeight
3. Add .transition-ready to both slides
4. Double-rAF then add .leaving-forward to current, .animating to next
5. Await transitionend + 800ms safety timeout
6. Cleanup classes, add .active to next
```

## CSS skeleton

```css
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

.presentation-viewport {
  width: 100vw; height: 100vh; height: 100dvh;
  display: flex; justify-content: center; align-items: center;
  background: #0f0f1a; overflow: hidden;
}

.slide-deck {
  aspect-ratio: 16 / 9;
  max-width: 100%; max-height: 100%; width: 100%;
  display: grid; grid-template: 1fr / 1fr;
  container-type: size; container-name: deck;
  overflow: hidden; position: relative;
}

@media (min-aspect-ratio: 16/9) { .slide-deck { width: auto; height: 100%; } }

.slide {
  grid-area: 1 / 1;
  display: flex; flex-direction: column; justify-content: center;
  padding: 6% 8%;
  font-size: clamp(0.7rem, 2.2cqi, 1.5rem);
  opacity: 0; visibility: hidden; pointer-events: none;
}
.slide.active { opacity: 1; visibility: visible; pointer-events: auto; }

.slide.transition-ready {
  transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1), opacity 0.6s ease;
}
.slide.leaving-forward   { transform: translateX(-100%); opacity: 0; }
.slide.leaving-backward  { transform: translateX(100%);  opacity: 0; }
.slide.entering-forward  { transform: translateX(100%);  opacity: 0; visibility: visible; }
.slide.entering-backward { transform: translateX(-100%); opacity: 0; visibility: visible; }
.slide.entering-forward.animating,
.slide.entering-backward.animating { transform: translateX(0); opacity: 1; }

@media (prefers-reduced-motion: reduce) {
  .slide { transition-duration: 0.01ms !important; }
}
```

## JS engine skeleton

```javascript
class SlideEngine {
  constructor() {
    this.slides = Array.from(document.querySelectorAll('.slide'));
    this.total = this.slides.length;
    this.current = 0;
    this.isAnimating = false;
    this._init();
  }

  _init() {
    this._bindKeyboard();
    this._bindSwipe();
    this._loadFromHash();
    this._updateUI();
    window.addEventListener('popstate', () => this._loadFromHash());
  }

  next() { if (!this.isAnimating) this._goTo(this.current + 1, 'forward'); }
  prev() { if (!this.isAnimating) this._goTo(this.current - 1, 'backward'); }

  async _goTo(target, dir) {
    if (target < 0 || target >= this.total || this.isAnimating) return;
    this.isAnimating = true;
    const cur = this.slides[this.current];
    const nxt = this.slides[target];
    cur.style.willChange = nxt.style.willChange = 'transform, opacity';
    nxt.classList.add(`entering-${dir}`);
    void nxt.offsetHeight;
    cur.classList.add('transition-ready');
    nxt.classList.add('transition-ready');
    requestAnimationFrame(() => requestAnimationFrame(() => {
      cur.classList.add(`leaving-${dir}`);
      nxt.classList.add('animating');
    }));
    await new Promise(resolve => {
      const h = e => { if (e.target !== nxt || e.propertyName !== 'transform') return;
        nxt.removeEventListener('transitionend', h); clearTimeout(t); resolve(); };
      nxt.addEventListener('transitionend', h);
      const t = setTimeout(() => { nxt.removeEventListener('transitionend', h); resolve(); }, 800);
    });
    cur.classList.remove('active', 'transition-ready', `leaving-${dir}`);
    cur.style.willChange = 'auto';
    nxt.classList.remove('transition-ready', `entering-${dir}`, 'animating');
    nxt.classList.add('active');
    nxt.style.willChange = 'auto';
    this.current = target;
    this._updateUI();
    this._syncHash();
    this.isAnimating = false;
  }

  _bindKeyboard() {
    document.addEventListener('keydown', e => {
      if (e.repeat || e.ctrlKey || e.altKey || e.metaKey) return;
      if (['INPUT','TEXTAREA','SELECT'].includes(document.activeElement?.tagName)) return;
      switch (e.key) {
        case 'ArrowRight': case 'ArrowDown': case 'PageDown':
          e.preventDefault(); this.next(); break;
        case 'ArrowLeft': case 'ArrowUp': case 'PageUp':
          e.preventDefault(); this.prev(); break;
        case ' ': e.preventDefault(); e.shiftKey ? this.prev() : this.next(); break;
        case 'Home': e.preventDefault(); this._goTo(0, 'backward'); break;
        case 'End':  e.preventDefault(); this._goTo(this.total - 1, 'forward'); break;
        case 'f': case 'F':
          document.fullscreenElement
            ? document.exitFullscreen?.()
            : document.documentElement.requestFullscreen?.(); break;
        case 's': case 'S':
          document.getElementById('notes-panel').classList.toggle('open');
          this._updateNotes(); break;
        case 'Escape':
          document.getElementById('notes-panel').classList.remove('open'); break;
      }
    });
  }

  _bindSwipe() {
    let sx = 0, sy = 0, st = 0;
    document.addEventListener('pointerdown', e => {
      if (e.pointerType === 'mouse' && e.button !== 0) return;
      sx = e.clientX; sy = e.clientY; st = Date.now();
    });
    document.addEventListener('pointerup', e => {
      const dx = e.clientX - sx, dy = e.clientY - sy;
      if (Date.now() - st > 400 || Math.abs(dx) < Math.abs(dy) || Math.abs(dx) < 50) return;
      dx < 0 ? this.next() : this.prev();
    });
  }

  _syncHash() {
    const h = `#slide-${this.current + 1}`;
    if (location.hash !== h) history.pushState({ slide: this.current }, '', h);
  }

  _loadFromHash() {
    const m = location.hash.match(/^#slide-(\d+)$/);
    if (!m) return;
    const idx = parseInt(m[1], 10) - 1;
    if (idx >= 0 && idx < this.total && idx !== this.current) {
      this.slides[this.current].classList.remove('active');
      this.slides[idx].classList.add('active');
      this.current = idx;
      this._updateUI();
    }
  }

  _updateUI() {
    const p = this.total > 1 ? this.current / (this.total - 1) : 0;
    document.getElementById('progressFill').style.transform = `scaleX(${p})`;
    document.getElementById('counterCurrent').textContent = this.current + 1;
    document.getElementById('counterTotal').textContent = this.total;
    this._updateNotes();
  }

  _updateNotes() {
    const aside = this.slides[this.current].querySelector('.speaker-notes');
    document.getElementById('notes-content').textContent = aside?.textContent?.trim() || '(노트 없음)';
  }
}

document.addEventListener('DOMContentLoaded', () => { window.presentation = new SlideEngine(); });
```

## HTML template

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>프레젠테이션 제목</title>
  <style>/* CSS here */</style>
</head>
<body>
  <div class="presentation-viewport">
    <main class="slide-deck" role="region" aria-label="프레젠테이션" aria-live="polite">
      <section class="slide active" data-slide="0" aria-roledescription="slide">
        <div class="slide__header"><h1 class="slide__title">제목</h1></div>
        <div class="slide__body"><p>내용</p></div>
        <aside class="speaker-notes">화자 노트</aside>
      </section>
      <!-- more slides -->
      <div class="progress-bar"><div class="progress-bar__fill" id="progressFill"></div></div>
      <div class="slide-counter"><span id="counterCurrent">1</span> / <span id="counterTotal">1</span></div>
      <div id="notes-panel">
        <div class="notes-label">Speaker Notes (S)</div>
        <div id="notes-content"></div>
      </div>
    </main>
  </div>
  <script>/* JS engine here */</script>
</body>
</html>
```

## Slide content patterns

- **Title slide** — centered, large title + subtitle
- **Bullet list** — `<ul>` with `<li>` items
- **Two-column** — `display:flex; gap:3em` with `.slide__col` divs
- **Code block** — `<pre><code>` with dark background

## Customization checklist

1. Ask (or infer): topic, number of slides, color theme, language
2. Generate all slide content, then wrap in the template
3. Apply a color theme via gradient on `.slide__title`
4. Add `<aside class="speaker-notes">` to every slide
5. Output as a single `.html` file

## Reference files

- `references/research.md` — architectural decisions and browser compatibility notes
- `references/demo.html` — fully working 3-slide demo

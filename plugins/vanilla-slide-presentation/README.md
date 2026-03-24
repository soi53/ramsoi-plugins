# vanilla-slide-presentation

A Claude Code plugin for building complete, single-file HTML/CSS/JS slide presentations — no external libraries required.

## Features

- Arrow key navigation (← →)
- F key for fullscreen
- Slide number display
- Smooth slide transition animations (GPU accelerated)
- Speaker notes (S key toggle)
- Touch/swipe support
- URL hash sync (deep linking)
- Progress bar

## Installation

```
/plugin add-marketplace ramsoi-plugins soi53/ramsoi-plugins
/plugin install vanilla-slide-presentation@ramsoi-plugins
```

## Usage

Just ask Claude to create a presentation:

- "발표자료 만들어줘"
- "슬라이드 5장으로 React 소개 프레젠테이션 만들어줘"
- "Create a presentation about machine learning"

Claude will generate a single `.html` file you can open directly in any browser.

## Output

A single self-contained `.html` file with no dependencies.

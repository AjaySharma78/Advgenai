# Advgenai · Advanced Agentic AI

A six-week hands-on course. Each week has four days. Open `index.html` to start.

## Folder structure
```
Advgenai/
├── index.html            main page: the 6 week cards (only Week 1 is open)
├── .gitignore
├── assets/               shared by every page
│   ├── css/  site.css (home + week pages) · sketch.css + deck.css (the slide decks)
│   └── js/   site.js (Colab links, paper/chalkboard) · deck.js (the slide engine)
├── week-1/
│   ├── index.html        the 4 day cards (only Day 1 is open)
│   ├── class-1.html      Day 1 deck (the sketch deck)
│   └── class-1.ipynb     Day 1 notebook (opens in Google Colab)
└── week-2/ … week-6/     empty for now
```
`assets/` is the one folder added on top of the layout you showed, so the CSS and JS are not copied into every page.

## Run it on your computer
```
cd D:\Advgenai
python -m http.server 8000
```
Then open http://localhost:8000.

## The "Open in Colab" buttons
Colab opens notebooks straight from a **public GitHub repository**. The button links are built in one place, `assets/js/site.js`:
```js
var CONFIG = { github: "AjaySharma78/Advgenai", branch: "main" };
```
Change `github` if your repository has a different name. Students add their key in Colab: the 🔑 **Secrets** panel, a secret named `GROQ_API_KEY`, and **Notebook access** switched on.

## Open the next day
1. Add `class-2.html` and `class-2.ipynb` to `week-1/` (the same names, `class-N`).
2. In `week-1/index.html`, replace the locked Day 2 `<article class="card locked">…</article>` with a copy of the Day 1 `<article class="card ready">…</article>` and change its text and the two file names.

## Open the next week
1. Copy `week-1/index.html` to `week-2/index.html` and edit the four day cards (lock all of them again).
2. In the main `index.html`, replace Week 2's `<div class="card locked">` with a copy of Week 1's `<a class="card ready" href="week-2/index.html">`.

## Look
- Paper by default; the **☾ chalkboard** button (or **T** inside a deck) switches. The choice is remembered.
- Letter spacing is one number: `--ls` at the top of `assets/css/site.css` and `assets/css/sketch.css` (now `0.03em`). Raise it for more space.

## Deploy
Any static host works (Vercel, GitHub Pages). Use this folder as the root, with no build step.

# Advgenai · Advanced Generative AI

A six-week hands-on course. Each week has four days. Open `index.html` to start.

## Folder structure
```
Advgenai/
├── index.html            main page: the 6 week cards (Weeks 1 and 2 are open)
├── themes.html           the look of each of the six weeks side by side
├── .gitignore
├── assets/               shared by every page
│   ├── css/  site.css (home + week pages) · themes.css (the 6 week themes) · sketch.css + deck.css (the slide decks)
│   └── js/   site.js (Colab links, paper/chalkboard) · deck.js (the slide engine)
├── docs/
│   └── WEEK_RECIPE.md    how a week is made + the checklist for adding one
├── week-1/               Agent foundations (theme: the sketchbook)
│   ├── index.html        the 4 day rows
│   ├── class-1 … class-4 .html (the decks) and .ipynb (the notebooks, opened in Google Colab)
│   └── trip_planner_agent.ipynb · trip_planner_agent_modern.ipynb
├── week-2/               Design patterns (theme: the blueprint)
│   ├── index.html        the 4 day rows
│   ├── class-1 … class-4 .html (the decks, 7 to 10 slides each) and .ipynb (student notebooks with TODO hints and PASS / NOT YET checks)
│   ├── assignment.ipynb  the weekly homework: five short tasks
│   └── class-1 … class-4 _solution.ipynb and assignment_solution.ipynb   the finished, commented notebooks (publish them after class)
├── week-3/               Context & memory (theme: the index cards)
│   ├── index.html        the 4 day rows
│   ├── class-1 … class-4 .html (the decks) and .ipynb (student notebooks with TODO hints and PASS / NOT YET checks)
│   ├── assignment.ipynb  the weekly homework: five short tasks
│   ├── course_notes.md   the documents that Day 3 and Day 4 search (RAG); the notebooks download it from GitHub
│   └── class-1 … class-4 _solution.ipynb and assignment_solution.ipynb   the finished notebooks (publish them after class)
└── week-4/ … week-6/     empty for now
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
2. In `week-1/index.html`, replace the locked Day 2 `<article class="day locked">…</article>` with a copy of the Day 1 `<article class="day ready">…</article>` and change its text and the two file names.

## Open the next week
See the checklist in `docs/WEEK_RECIPE.md`. In short: copy `week-1/index.html` to `week-2/index.html`, set `data-week="2"` on `<html>`, edit the four day rows (lock them again), then in the main `index.html` turn Week 2's `<div class="card locked">` into a copy of Week 1's `<a class="card ready" href="week-2/index.html">` and make its header pill a link.

## Look
- Paper by default; the **☾ chalkboard** button (or **T** inside a deck) switches. The choice is remembered.
- **Each week has its own theme** (palette, paper, heading font, bottom edge, doodle). Open `themes.html` to see all six. To use one, put `data-week="N"` on `<html>` (whole page) or on any element, and load `assets/css/themes.css` after `site.css` / `sketch.css`. To change a colour, edit that week's block in `themes.css`.
- Page width is one number: `--page-max` at the top of `assets/css/site.css` (now `1600px`).
- Letter spacing is one number: `--ls` at the top of `assets/css/site.css` and `assets/css/sketch.css` (now `0.03em`). Raise it for more space.

## Deploy
Any static host works (Vercel, GitHub Pages). Use this folder as the root, with no build step.

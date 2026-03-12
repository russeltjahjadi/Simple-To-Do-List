# Tasky — Todo List App

A single-file, zero-dependency todo list web app. No build tools, no frameworks, no server required — just one `index.html` file.

---

## Stack

| Layer | Technology |
|---|---|
| Markup | HTML5 |
| Styling | CSS3 (custom properties, flexbox, animations) |
| Logic | Vanilla JavaScript (ES6+) |
| Fonts | Google Fonts (Fraunces + DM Sans) |
| Persistence | Browser `localStorage` |

---

## File Structure

Everything lives in a single `index.html` file, split into three sections:

```
index.html
├── <head>        → fonts, metadata
├── <style>       → all CSS
├── <body>        → HTML structure
└── <script>      → all JavaScript
```

---

## How the CSS Works

### Design Tokens
All colors are defined as CSS custom properties on `:root`, making the theme easy to change in one place:

```css
:root {
  --cream: #0F1117;   /* page background */
  --ink: #E8E3DA;     /* primary text */
  --accent: #7C6FCD;  /* purple highlight color */
  --muted: #4E5363;   /* secondary text */
  --line: #1E2130;    /* borders */
  --done-bg: #141720; /* completed task background */
}
```

### Visual Effects
- **Grain texture** — a `body::before` pseudo-element overlays an SVG `feTurbulence` noise filter at low opacity to add subtle texture to the background.
- **Blobs** — two fixed `<div>` elements with `border-radius: 50%` and `filter: blur(80px)` create the soft glowing background shapes.
- **Animations** — `@keyframes fadeUp` staggers the page sections on load using `animation-delay`. Task cards use `@keyframes slideIn` when added, and a `.removing` class triggers a slide-out before the element is removed from the DOM.

---

## How the JavaScript Works

### State
The app holds two main pieces of state in memory:

```js
let tasks = [];           // array of task objects
let selectedCat = 'none'; // category chosen in the input
let activeFilter = 'all'; // current filter tab
```

Each task is a plain object:
```js
{
  id: "1710000000000",  // Date.now() as string — unique ID
  text: "Buy groceries",
  done: false,
  cat: "personal",      // "work" | "personal" | "health" | "ideas" | "none"
  ts: 1710000000000     // timestamp for display
}
```

### Persistence
Tasks are saved to and loaded from `localStorage` as a JSON string:

```js
function save() {
  localStorage.setItem('tasky_tasks', JSON.stringify(tasks));
}

// On page load:
let tasks = JSON.parse(localStorage.getItem('tasky_tasks') || '[]');
```

This means tasks survive page refreshes and browser restarts, but are local to the device and browser.

### Rendering
The app uses a single `render()` function that rebuilds the task list DOM from scratch every time state changes. It:

1. Filters `tasks` based on `activeFilter` (all / active / done)
2. Clears the `#task-list` container
3. Loops over visible tasks and creates a card element for each using `innerHTML`
4. Updates the three stat counters (Total, Remaining, Completed)
5. Shows/hides the "Clear completed" button

### Adding a Task
```js
function addTask() {
  const text = input.value.trim();
  if (!text) return;
  tasks.unshift({ id: Date.now().toString(), text, done: false, cat: selectedCat, ts: Date.now() });
  save();
  render();
}
```
New tasks are added to the **front** of the array with `unshift()` so they appear at the top of the list.

### Completing & Deleting
- **Toggle** — flips `task.done` and calls `save()` + `render()`.
- **Delete** — adds a `.removing` CSS class to animate the card out, then removes it from the array after 280ms to let the animation finish before the DOM update.

### Category Picker
The category dropdown is a custom-built component (not a native `<select>`). It works by:

1. Toggling an `.open` class on the dropdown `<div>` to show/hide it via `display: flex`
2. Listening for clicks outside via `document.addEventListener('click', closeDropdown)`
3. Using `e.stopPropagation()` on the trigger and options to prevent the outside-click listener from immediately closing the dropdown when it opens
4. Calling `setCat(cat)` which updates the trigger button's label, dot color, and CSS variable to reflect the selection

### XSS Protection
User input is sanitized before being inserted into the DOM via `escHtml()`:

```js
function escHtml(s) {
  return s.replace(/&/g,'&amp;').replace(/</g,'&lt;')
          .replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}
```

This prevents any HTML or script tags typed into the input from being executed.

---

## Deployment

Since this is a static file, it can be deployed anywhere that serves HTML:

- **Netlify** — drag and drop the file at [netlify.com/drop](https://netlify.com/drop)
- **GitHub Pages** — push to a repo, enable Pages in settings
- **Vercel** — run `vercel` in the folder containing the file
- **Any web server** — place `index.html` in the root directory
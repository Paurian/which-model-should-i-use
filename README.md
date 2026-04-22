# Which Model Should I Use

Identify which AI or Statistics model is suitable for the data you have and the outcome you want to reach.

Claude Prompts:
I now want you to create a decision tree web app in javascript. Make it completely contained in one HTML page that includes the javascript in its &lt;script&gt; tags. This program takes the mermaid chart you've been working on and makes it ask the user the question in each diamond. They are given 4 options: "Yes", "No", "Go Back" and "Restart". At each leaf/terminal node (the rounded boxes), give the suggested model type (the text in the rounded boxes) with its example (in the notch boxes). Stylization is done in CSS &lt;style&gt; definitions within the same document. Each question should be prompted on the screen in a medium, bold font with answers/actions in buttons beneath. After an anwer/action is given, the question text is replaced with either the next question or the final text. The final text should show the model type in large, bold font with the example beneath in medium, normal style of the same font family.

Earlier in this conversation session you provided a table for "Real-world case for each leaf". I would like for you to edit the HTML page you provided so that there is a choice "Start Choose-Your-Own-Adventure Model Quest" and a choice "View The Model Index". If the user clicks "Start Choose-Your-Own-Adventure Model Quest", begin the questionaire as you have it. If the user clicks "View The Model Index", show a page with that table in it and a "Return To Main Screen" button beneath. If Someone clicks the "Restart" button, take them to this new initial screen. Keep all the code within the same HTML file. Do not change the existing code except where it is ncessary to incorporate the two new screens mentioned in this prompt. I want to be able to use Beyond Compare to do a code review and that can only happen if you don't shift code around.

Please add to this page some logic so that when the person hovers over one of the breadcrumbs (a crumb in the trail element in the HTML file) that it shows a stylized pop-up or tool-tip showing the question with their answer. Additionally, if the user clicks on the specific crumb, it jumps him to that question. Upon jumping to that question, only the trail up to that question is kept and the trail of and after that question are cleared (as if the person clicked the "Go Back" button until going to that question).

Check and fix the code. The crumb-tooltip node is not updating when the mouse hovers over the trail's crumb nodes. Furthermore, nothing happens when the user clicks on the crumbs.

---

```
---

Prompt:

---

Build a self-contained single-file HTML decision tree web app. All CSS goes in <style> tags and all JavaScript in <script> tags within the same file. The decision tree is driven by the Mermaid flowchart at the bottom of this prompt.

---

How to read the diagram

- diamond nodes — questions. Each has a -- Yes --> and a -- No --> edge leading to child nodes.
- rounded nodes — terminal results. Their label is "Use [Model Name]". Strip the "Use " prefix when displaying.
- notch-rect nodes — real-world example text. Each is connected to its parent rounded node with a ---- (long dashed) edge. Every rounded node has exactly one notch-rect companion.
- Edge labels (Yes / No) are part of the tree logic, not display text.

---

App structure — three screens

All three screen divs exist in the DOM at all times. Exactly one is visible at a time, controlled by display: none / block.
Initialize by showing the Home screen.

---

Screen 1 — Home (id="screen-home")

Two stacked full-width buttons, centered:
1. "Start Choose-Your-Own-Adventure Model Quest" — filled primary-color style
2. "View The Model Index" — ghost/outline style

---

Screen 2 — Quest (id="screen-quest")

Layout (top to bottom inside a card):
1. Breadcrumb trail (id="trail") — pill badges, one per answer given so far, separated by ›
2. Content area (id="display") — shows either a question or a result
3. Answer buttons row — "Yes" and "No" (filled, primary color, min-width 100px each)
4. Navigation buttons row — "Go Back" and "Restart" (ghost style)

Question display:
- Question text: 1.3rem, font-weight: 700, centered.
- If the node has a hint string, render it below the question in 0.9rem muted text.

Result display (terminal node reached):
- Hide the Yes/No row.
- Model name: 2.1rem, font-weight: 800, primary color.
- A short decorative horizontal rule (3rem wide, 3px tall, primary color, 50% opacity).
- Example text from the notch-rect companion: 1rem, font-weight: 400, #bbb.

Button behavior:
- "Yes" / "No" — push { node: current, choice } to history, advance current to node[choice], transition.
- "Go Back" — pop history, restore previous node, transition. Disabled (opacity: 0.3, cursor: default) when history is empty.
- "Restart" — reset history to [], reset current to the root node, call showScreen('home'). Does NOT restart the quest; it returns to the Home screen.

Transition animation:
Fade the content div (#stage) out (opacity 0, translateY -10px), swap content, fade back in (opacity 1, translateY 0). Use inline style manipulation + requestAnimationFrame double-tick for the fade-in so CSS transition applies correctly. Total duration ~180ms out + ~180ms in.

---

Screen 3 — Model Index (id="screen-index", max-width: 960px)

A heading "Model Index" and a full-width table with three columns:

| Model | When to Use | Real-World Example |

Derive one row per terminal node from the diagram:
- Model column: Model name (strip "Use " prefix). If the same model name is reachable by more than one distinct path (e.g. "Clustering Models" appears twice), include a row for each path and add a small secondary label (e.g. "time-based" or "cross-sectional") rendered as a smaller muted <span> below the model name to disambiguate.
- When to Use column: Write a concise phrase summarizing the Yes/No path that leads to this terminal. Derive it from the question labels along the path — e.g., a terminal reached via "predicting over time → only one observation → not time-to-event → causal" becomes "Single series, causal relationships." Keep it brief (5–10 words).
- Real-World Example column: Copy the exact text from the notch-rect companion node verbatim.

Row order: depth-first, left-branch-first (Yes before No at each fork), matching diagram traversal order.

Below the table: a "Return To Main Screen" ghost button that calls showScreen('home').

---

JavaScript architecture

Represent the tree as a flat object const nodes = { ... } keyed by node ID. Each entry:
// Question node:
{ type: 'question', text: '...', hint: '...', yes: 'childId', no: 'childId' }
// Result node:
{ type: 'result', model: 'Model Name', example: '...' }
hint is optional (omit if not needed). Node IDs can be short arbitrary strings.

State: let current = rootId and let history = [] (array of { node, choice }).

Functions required:
- showScreen(name) — hides all three screens, shows screen-${name}
- startQuest() — resets state, calls showScreen('quest'), calls render()
- render() — reads current and history, updates #trail, #display, answer-row visibility, back-button disabled state
- transition(fn) — animated swap: fade out, call fn, fade in
- answer(choice) — push to history, advance current, call transition(render)
- goBack() — pop history, restore node, call transition(render)
- restart() — reset state, call showScreen('home')
- jumpToQuestion(index) — set current = history[index].node, set history = history.slice(0, index), call transition(render)

---

Breadcrumb tooltip and click-to-jump

Each crumb <span> must have a data-index attribute set to its position in history.

Tooltip (id="crumb-tooltip"):
- position: fixed, width: 260px, display: none, initial top: -9999px; left: -9999px (prevents flash before first positioning).
- Contains two children: .tt-q (question text, bold) and .tt-a (answer, bold, color-coded).
- pointer-events: none so it never intercepts mouse events.
- z-index: 999, dark background #1e1e1e, 1px solid primary-color border, border-radius: 10px.
- CRITICAL: The <div id="crumb-tooltip"> HTML must appear in the document before the <script> tag. If it appears after, document.getElementById('crumb-tooltip') will return null when the script runs.

Use event delegation on #trail for all three events:
- mouseover — if e.target.closest('.crumb') finds a crumb: populate tooltip with the question text (strip HTML tags via a temp element) and the answer, call display: block, then call getBoundingClientRect() on both the tooltip and the crumb to position the tooltip centered above the crumb (or below if insufficient space above), clamped 8px from viewport edges. If target is not a crumb, return without hiding (do not call display: none here — that causes flicker when the mouse crosses separators).
- mouseleave (on #trail) — set tooltip display: none.
- click — if e.target.closest('.crumb') finds a crumb, hide tooltip, call jumpToQuestion(index).

---

Styling

Color tokens (define as CSS custom properties in :root):
--bg:           #0e0e0e   (page background)
--card-bg:      #1a1a1a   (card fill)
--border:       #2e2e2e   (card border)
--primary:      #830067   (magenta — buttons, accents, result model name)
--primary-h:    #9e007c   (primary hover)
--text:         #f0f0f0   (body text)
--muted:        #888      (hints, labels, ghost button text)
--ghost-border: #3a3a3a   (ghost button border)

Cards: border-radius: 16px, padding: 3rem 3.5rem, overflow: hidden. Quest and Home cards: max-width: 660px. Index card: max-width:
960px (override with inline style or ID rule).

Breadcrumb badges: background: #252525, border-radius: 4px. YES crumbs: color: #7ecfb3. NO crumbs: color: #e07777. .crumb must have
cursor: pointer; user-select: none.

Page title <h1> (outside card, above everything): 0.8rem, font-weight: 500, uppercase, letter-spacing: 0.15em, muted color,
margin-bottom: 2.5rem.

Index table: border-collapse: collapse. Headers: border-bottom: 2px solid var(--primary), primary color, 0.78rem, uppercase,
letter-spaced. Cells: padding: 0.75rem 1rem, border-bottom: 1px solid var(--border). Last row: no bottom border. First column:
font-weight: 700. Second column: muted, 0.85rem, white-space: nowrap. Third column: #bbb.

Font: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif.

---

Mermaid diagram

[PASTE YOUR MERMAID DIAGRAM HERE]

---
```
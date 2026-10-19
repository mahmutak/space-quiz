# Cosmic Quest

## Running and checking the app

Open `index.html` directly in a browser. On Windows PowerShell:

```powershell
Start-Process .\index.html
```

The app must work through `file://`, offline, without a server or dependencies. Keep CSS, JavaScript, SVG, and decorative artwork embedded in this single HTML file.

There are no configured build, test, or lint commands, or individual automated tests. Check changes in the browser: complete a quiz, inspect correct and incorrect feedback, and restart from results. For results changes, exercise scores in each reaction band (0-3, 4-6, 7-9, and 10), including restarting during the score animation. Check keyboard-only navigation, narrow layouts, dark mode, and reduced motion.

## Architecture and state

`index.html` contains theme/layout CSS, a persistent HTML shell, and a strict-mode classic script. The script renders answer buttons into the shell; `#quiz` and `#results` alternate visibility using `hidden`.

The `questions` array is the content source: each entry has `topic`, `question`, four `answers`, a zero-based `correct` index, and an `explanation`. Questions run in fixed order. The quiz has ten questions; several HTML labels, the progress maximum, results denominator, and reaction thresholds also encode that count. Keep them consistent if the count changes.

The `elements` object caches DOM nodes by ID. Keep its ID list aligned with markup changes. Runtime state consists of `current` (zero-based question index), `score`, `answered`, and `scoreAnimation` (the pending animation-frame handle).

- `renderQuestion()` rebuilds the answers and resets feedback and the answer lock. Its optional focus flag is used when advancing or restarting, not on initial load.
- `selectAnswer()` locks the question against duplicate scoring, disables all answers, reveals the correct answer and explanation, and enables Next. Progress measures answered questions, not the currently displayed question number.
- The Next handler prevents skipping unanswered questions and calls `showResults()` after the final answer.
- `showResults()` selects an emoji/message band and animates the displayed score from zero for one second. The underlying score and final-score announcement remain authoritative. Restart cancels the pending animation before resetting state.

## UI and accessibility conventions

Use the existing CSS custom properties for themed surfaces, text, borders, and feedback. Light values live in `:root`; dark overrides use `prefers-color-scheme`. The answer grid becomes a single column at 540px.

Correct answers use green styling and a check mark; incorrect selections use red styling and a shake. Preserve text explanations and accessible answer labels so feedback does not depend on color or motion.

Use native buttons and preserve the deliberate focus sequence: Next after an answer, the question heading after advancing/restarting, and the results heading on completion. The headings have `tabindex="-1"` for programmatic focus. `#announcement` is the polite, atomic live region for feedback and final scores; decorative symbols are hidden from assistive technology, and the reaction emoji has an accessible label.

Reduced-motion behavior has two independent paths: CSS disables animations/transitions, and JavaScript checks `prefers-reduced-motion` before starting the score count-up. Preserve both, and cancel any pending score animation on restart.

Dynamic content is built with `createElement`, `textContent`, `append`, and `replaceChildren`, rather than HTML-string injection.

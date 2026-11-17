# PROMPT FOR THE AI (copy everything below this line)

---

You are a senior front-end engineer. I am giving you two things:

1. **MY PROTOTYPE**: the phone prototype you built for me earlier (a foldable phone with a control panel: "Fold Phone", "Fold Angle", "Lock Phone", "Call", widgets, an app grid, a lock screen with a notification).
2. **THE PACK** (`PearPhone_Apps_Animations_Pack.zip`): extracted from a *working, interactive* reference phone UI. Unzip it and read it. It is real, tested code, not a description.

## The problem
In my prototype the apps are not interactive (they do not behave like real apps) and the animations are broken. In the reference, every app works and the animations (app open/close, gestures, app switcher, lock/unlock, and the book-style fold/unfold) are smooth and correct.

## Your goal
Upgrade **my prototype** so that:
- every app is a real, working app (not a static screen),
- every animation behaves like the reference,
- the foldable opens and closes with the reference's book animation, including drag,
- **my prototype's design, branding and extra controls stay** (do not turn it into a copy of the reference's look unless I ask).

## Step 0: Read the pack in this order (do not skip, do not skim)
1. `README.md`
2. `docs/01_ARCHITECTURE.md` (how apps, state and the shell fit together)
3. `docs/02_ANIMATIONS.md` (every animation, with exact numbers and file locations)
4. `docs/03_APPS_CATALOG.md` (the 46 apps and the file that defines each)
5. `docs/04_DIFFERENCES_AND_PITFALLS.md` (what usually goes wrong in prototypes like mine)
6. The code: `animations/excerpts/*.txt`, then `animations/css/motion.css`, `apps/js/calculator.js`, `apps/js/notes.js`, `reference/js/system.js`, `reference/js/ui.js`

Then **run the reference** so you see the target behaviour:
```
cd reference
python -m http.server 8000
# open http://localhost:8000/#frame=fold-star
```
(First launch shows a setup wizard; the PIN is 1225. See `docs/01_ARCHITECTURE.md` section 5 to skip it.) Open apps, drag the home indicator, open the app switcher, and open/close the fold with the left-edge drag or `System.setFoldOpen(true)` in the console.

## Step 1: Audit my prototype (output this before writing code)
Open my prototype and produce:
- **App audit table**: each app/icon in my prototype -> what it does now -> what it must do (use the matching app in `docs/03_APPS_CATALOG.md`).
- **Animation audit table**: each animation (app open, app close, home, lock/unlock, notification, fold/unfold, fold angle slider) -> why it is broken (name the real cause: for example animating `width/height`, class added in the same frame, `transition: all`, no transform-origin from the tapped icon, layout read during drag) -> the reference mechanism that fixes it (file and line).
- **Diff table**: my prototype vs the reference for: app model, state, navigation inside apps, lifecycle, animation engine, gestures, fold, lock screen. Confirm or correct each row of `docs/04_DIFFERENCES_AND_PITFALLS.md` against my real code.

## Step 2: Choose the strategy and say why in 3 lines
- **A. Adopt the reference runtime** (recommended): reuse its scripts and CSS unchanged, apply my look by editing tokens/wallpaper/icons, and add my extra controls (Fold Angle slider, Lock Phone, Call) as a small separate script that calls the reference API.
- **B. Port the concepts** into my own code, only if my prototype must stay a small single file. Port in the order given in `docs/04_...`, testing after each step.
Make the decision yourself. Do not ask me questions unless something is truly impossible to decide; write your assumptions instead.

## Step 3: Requirements (all of these must be true when you finish)
**Apps**
1. Each app is registered with `App.define(id, {...})` (or an equivalent registry in strategy B) and builds real DOM with real event handlers when opened.
2. Apps have real behaviour and persist their data (`Store`): Calculator computes (including the scientific keys when the fold is open), Notes create/edit/delete, Calendar adds events, Clock has alarm/stopwatch/timer, Messages send and receive and trigger a notification, Phone/Contacts dial and call (the call appears in the Dynamic Island), Safari navigates, Music plays and shows in the Island/widget, Photos/Camera work with mock data, Weather and Maps render their data, Settings toggles actually change the system (dark mode, accent, wallpaper, text size, Reduce Motion).
3. Apps adapt to the fold: after `fold:state` they re-layout (list | detail when open).
4. Opening an app from a notification or deep link while locked asks for the PIN first (`Lock.requestUnlock`).

**Animations**
5. Open: the window starts on the tapped icon (`rectOf`/`toTransform`), grows with `transform`, content fades in at the end, home scales to 1.14 and fades. Close: shrinks back into the icon. Same timings as `docs/02_ANIMATIONS.md`.
6. Start state is painted before the end state (double `requestAnimationFrame`).
7. Only `transform` and `opacity` animate; one owner per state class; no `transition: all`.
8. Gestures follow the finger/mouse: home indicator, left-edge back, top pull for Control Center and Notifications, App Switcher, drag-scroll with the mouse, rubber-band, spring on release.
9. Fold: one progress value `p` in [0..1] is the single source of truth. Book rotation `perspective(3800px) rotateY(-180*p)` around the hinge, the live outer screen is the front of the leaf, the inner screen is drawn behind, geometry swap at 90 degrees, 600 ms open / 500 ms close, drag on the left edge (680 px path, completes after 40% or a flick, otherwise springs back), tap-on-spine nudge, Reduce Motion = fade only, timeout fallback so it never stays half way.
10. **My "Fold Angle" slider** drives the same `p` (`p = angle / 180`, 180 = fully open) and stays in sync with drag, button and API changes in both directions. "Lock Phone" locks with the reference lock animation. "Call" triggers a real incoming-call experience in the Island/lock screen.

**Quality**
11. No console errors. No failed network requests (self-contained, relative paths). Works when served from a local static server.
12. Keep the reference's `credits.txt` files and note any license text found in the pack; do not remove attribution.

## Step 4: Rules
- **Use the code in the pack. Do not invent APIs.** If you need something that is not in the pack, say so and propose the smallest addition.
- **Do not rewrite working reference logic** "to simplify". Change structure only where my prototype requires it.
- Deliver **complete files**, ready to run. No "...rest of the code", no snippets that I must merge by hand.
- If the project is too large for one reply, deliver it in parts and tell me exactly which files come in which part; never stop silently.
- Keep the file/folder layout so that a single `index.html` opens everything.

## Step 5: Verify before you answer
Run it (headless browser or manual checklist) and report PASS/FAIL for each item:
- every app opens, accepts input, and persists after reload,
- open/close animation visibly runs (icon -> window -> icon),
- drag the home indicator, the switcher opens, swipe back works,
- fold opens and closes by slider, by drag and by `setFoldOpen`, in both directions, mid-drag release both ways,
- lock -> PIN -> unlock, notification tap while locked opens the app after the PIN,
- Reduce Motion on: no motion, only fades.

## Output format
1. Audit tables (Step 1).
2. Strategy (Step 2).
3. The full set of files (zip or file by file).
4. A short CHANGELOG: kept / replaced / removed.
5. The PASS/FAIL checklist (Step 5) with anything that still fails and why.

Start with Step 0 now.

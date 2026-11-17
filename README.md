# PearPhone Apps + Animations Pack

Extracted from the `CodeVexel_phone` resource (web UI: **PearOS 27** iOS-style phone and the **PearPhone Fold** foldable). Purpose: give another AI everything it needs to make a prototype's apps interactive and its animations correct.

## What is inside

```
AI_INTEGRATION_PROMPT.md      <- the prompt to give to your AI (copy the part under the line)
README.md                     <- this file
docs/
  01_ARCHITECTURE.md          how apps, state, navigation and the shell work; how to run
  02_ANIMATIONS.md            every animation with exact numbers, curves, file locations
  03_APPS_CATALOG.md          all 46 registered apps, where each is defined
  04_DIFFERENCES_AND_PITFALLS.md   typical prototype mistakes vs the reference; integration strategies
reference/                    THE RUNNABLE UI (open through a local web server)
  index.html  css/  js/  fonts/  img/  vendor/
apps/                         copies for quick reading
  js/   all 48 app scripts (js/apps/*.js)
  css/  apps.css + 31 app-*.css files
animations/                   copies for quick reading
  css/  motion.css, liquid27.css, lock.css, notify.css, control.css
  js/   gestures, switcher, island, lock, notify, home, control, dragscroll
  excerpts/   the two engines hidden inside system.js, with original line numbers:
              A_foldable_book_animation   (open/close, drag, rig, timings)
              B_app_launch_close_animation (icon <-> window)
              C_App_registry              (the App.define contract)
```

`apps/` and `animations/` are **copies** of files in `reference/` so they are easy to find. The source of truth that actually runs is `reference/`.

## Run it

```
cd reference
python -m http.server 8000
```
Open `http://localhost:8000/#frame=fold-star` (white foldable, the one in your screenshot) or `#frame=fold-night` (dark blue). Without a hash you get the normal phone. First launch shows a setup wizard; the PIN is `1225`. Do not open `index.html` by double click: browsers block some file loads without a server.

Open/close the fold: drag the left edge of the device, or in the console `System.setFoldOpen(true)` / `System.setFoldOpen(false)`.

## What was removed to keep the pack small

`reference/` is the original `html/` folder **without**:
- `models/` (19 MB of `.glb` 3D phone models used only by the in-game shop viewer),
- the 13 extra language dictionaries (`js/i18n-<code>.js`); `i18n-en.js` is kept. Other languages load lazily and simply fall back if absent, no errors.

Everything else is untouched. It was tested in a headless browser: no page errors, no failed requests, PIN unlock works, apps open, the calculator computes (7 x 6 = 42), fold opens and closes.

## What it is not

It is the **front-end** of a FiveM (GTA) phone resource. Inside the game, `Bridge` talks to Lua and a real database; in a browser it talks to `js/localdb.js`, a mock server that stores data in `localStorage`. The Lua side (`client/`, `config.lua`, SQL) is not part of this pack because the prototype does not need it.

## Credits and licence

The code belongs to its original author(s). The `credits.txt` files are kept inside `reference/`. Keep them, and use the code according to the licence or terms of the product you obtained it from.

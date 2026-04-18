# لعبة الجاسوس - PWA

Installable offline-ready Progressive Web App for the Spyfall family game.

## Files
- `index.html` — the game
- `manifest.json` — PWA metadata (name, icons, colors)
- `service-worker.js` — caches files for offline use
- `icons/` — app icons (192px, 512px, regular + maskable)

## ⚠️ Important: PWAs require HTTPS
You cannot install a PWA by just double-clicking `index.html`. It must be served over HTTPS (or `localhost` for testing). Here are your options:

---

## Option A: GitHub Pages (free, permanent, ~5 min)

1. Create a free account at github.com if you don't have one
2. Create a new repository (e.g., `spyfall`) — make it **public**
3. Upload all files from this folder (drag & drop works)
4. Go to **Settings → Pages**
5. Under "Source," select branch `main` and folder `/ (root)`, click Save
6. Wait ~1 minute — your app will be live at:
   `https://YOUR_USERNAME.github.io/spyfall/`
7. Open that URL on your phone → browser will offer "Install app" or "Add to Home Screen"

## Option B: Netlify Drop (free, fastest, ~1 min)

1. Go to https://app.netlify.com/drop
2. Drag the entire `spyfall-pwa` folder onto the page
3. You get an instant URL like `https://random-name.netlify.app`
4. Open on phone → install

## Option C: Local testing (no install, just for trying)

If you have Python installed:
```bash
cd spyfall-pwa
python3 -m http.server 8000
```
Then open `http://localhost:8000` on your computer. Note: install prompts only work on HTTPS or localhost, and install won't persist to other devices.

---

## Installing on Phone

**Android (Chrome):**
1. Open the URL
2. Tap the three-dot menu → "Install app" or "Add to Home Screen"
3. The game appears as a real app icon

**iPhone (Safari):**
1. Open the URL in Safari (not Chrome)
2. Tap the Share button
3. Scroll down → "Add to Home Screen"

Once installed, it works offline — no internet needed to play.

---

## Updating the App

If you change the HTML later:
1. Open `service-worker.js`
2. Change `const CACHE_VERSION = 'spyfall-v1'` to `'spyfall-v2'` (bump the number)
3. Re-upload. Users get the update on next launch.

Without bumping the version, browsers will keep serving the old cached version.

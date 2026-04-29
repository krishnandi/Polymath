# Polymath

A personal operating system for the decade from 20 to 30. Built as a Progressive Web App — runs on iPhone with its own home screen icon, offline support, and real-time Google Sheets sync. No backend required.

---

## What it is

A daily tracker, decade planner, and knowledge system built around one idea: the full life and the ambitious life are not in competition. You can build a company, speak 9 languages, travel 36 countries, and cook food from every culture you visit — not sequentially, but simultaneously.

The app has four sections:

- **Today** — auto-detects the day of the week and serves a different set of tasks. Monday is deep CS. Tuesday is language sprint. Wednesday is ship day. No configuration needed.
- **Decade** — the four phases from university through Stanford to the Rise, with milestones, track breakdowns, and the Forbes 30 under 30 target visible at all times.
- **Clusters** — six knowledge domains (cognitive engine, builder's stack, money mind, communicator, creative soul, Bourdain layer), each with an honest % initiated and individual interests to activate.
- **Laws** — ten principles. The ones that matter: never miss two days in a row, consistency over intensity, the Bourdain rule, LinkedIn is not optional, nine languages is a superpower.

---

## Stack

- Vanilla HTML / CSS / JavaScript — single file, zero dependencies, zero build step
- Progressive Web App (PWA) — installable on iPhone via Safari
- Service Worker — offline caching, push notification scaffolding
- Google Apps Script — optional real-time sync to a Google Sheet
- `localStorage` — state persists between sessions without a backend

---

## Files

```
polymath-app/
├── index.html          # The entire app
├── sw.js               # Service worker (offline + notifications)
├── manifest.json       # PWA manifest (icon, name, display mode)
├── icon-192.png        # App icon
└── icon-512.png        # App icon (large)
```

---

## Running it locally

```bash
git clone https://github.com/YOUR_USERNAME/polymath.git
cd polymath/polymath-app

# Any static server works. Simplest:
npx serve .

# Or Python:
python3 -m http.server 8080
```

Open `http://localhost:8080` in your browser.

> **Note:** The service worker requires HTTPS or `localhost` to register. It will not work if you just open `index.html` as a file.

---

## Installing on iPhone

The app is a PWA — it installs directly from Safari, no App Store required.

1. Host the `polymath-app/` folder anywhere that serves HTTPS. The fastest options:
   - **GitHub Pages** — push to a repo, enable Pages in Settings → Pages → Deploy from branch
   - **Vercel** — `vercel deploy` from the folder
   - **Netlify** — drag the folder onto netlify.com/drop

2. On your iPhone, open the URL in **Safari** (not Chrome — Chrome on iOS cannot install PWAs)

3. Tap the **Share button** (box with arrow at the bottom of Safari)

4. Tap **"Add to Home Screen"**

5. Rename it `Polymath` if it is not already → tap **Add**

The app now appears on your home screen with its own icon, opens full-screen with no browser chrome, and works offline.

---

## Google Sheets sync

Every task you complete can write to a Google Sheet in real time. Setup takes about five minutes and requires no server.

### 1. Create the sheet

Go to [sheets.google.com](https://sheets.google.com) and create a new spreadsheet. Name it **Polymath**.

### 2. Open Apps Script

In the spreadsheet: **Extensions → Apps Script**

### 3. Paste the script

Delete the default `myFunction()` code and paste the following:

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const ss = SpreadsheetApp.getActiveSpreadsheet();

    // ── Daily log ──────────────────────────────────────────────
    let logSheet = ss.getSheetByName('Daily Log');
    if (!logSheet) {
      logSheet = ss.insertSheet('Daily Log');
      logSheet.appendRow(['Date', 'Day', 'Focus', 'Task', 'Time', 'Done']);
      logSheet.getRange(1, 1, 1, 6).setFontWeight('bold');
    }

    if (data.tasks) {
      data.tasks.forEach(task => {
        logSheet.appendRow([
          data.date,
          data.day,
          data.focus,
          task.title,
          task.time,
          task.done ? '✓' : '—'
        ]);
      });
    }

    // ── Cluster tracker ────────────────────────────────────────
    let clusterSheet = ss.getSheetByName('Clusters');
    if (!clusterSheet) {
      clusterSheet = ss.insertSheet('Clusters');
      clusterSheet.appendRow(['Date', 'Cluster', '% Initiated', 'Active', 'Total']);
      clusterSheet.getRange(1, 1, 1, 5).setFontWeight('bold');
    }

    if (data.clusters) {
      data.clusters.forEach(c => {
        clusterSheet.appendRow([data.date, c.name, c.progress, c.active, c.total]);
      });
    }

    // ── Language tracker ───────────────────────────────────────
    let langSheet = ss.getSheetByName('Languages');
    if (!langSheet) {
      langSheet = ss.insertSheet('Languages');
      langSheet.appendRow(['Date', 'Language', 'Active']);
      langSheet.getRange(1, 1, 1, 3).setFontWeight('bold');
    }

    if (data.languages) {
      data.languages.forEach(l => {
        langSheet.appendRow([data.date, l.language, l.active ? 'Active' : 'Queued']);
      });
    }

    return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch(err) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'error', message: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok', message: 'Polymath sync active' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

### 4. Deploy

- Click **Deploy → New deployment**
- Type: **Web app**
- Execute as: **Me**
- Who has access: **Anyone**
- Click **Deploy** → authorise when prompted

### 5. Copy the URL

Copy the Web App URL. It looks like:
```
https://script.google.com/macros/s/AKfycb.../exec
```

### 6. Paste into the app

In the app: **Laws tab → Settings → Google Sheets Sync** → paste the URL → tap **Save**.

From now on, every task you tick syncs to the sheet automatically. Tap **⬆ Sync to Sheets** on the Today screen to push a full snapshot at any time.

---

## Notifications

Notification toggles are in **Laws → Settings**. Three reminders:

| Reminder | Time | Message |
|---|---|---|
| Morning | 07:00 | *Wake. No phone. Move. You decided to.* |
| Evening anchor | 21:00 | *Language + 10 pages. Phone in hall by 22:00.* |
| Midnight check | 23:30 | *Any tasks unfinished? Mark them and close the laptop.* |

**iPhone note:** Push notifications for PWAs require iOS 16.4 or later and the app must be installed to the Home Screen. When you tap a toggle for the first time, Safari will ask for permission. If you previously denied it, go to **Settings → Safari → Advanced → Website Data** or **Settings → Notifications** to re-enable.

---

## Deployment (GitHub Pages — fastest)

```bash
# From the repo root
git add .
git commit -m "init polymath"
git push origin main
```

Then in your GitHub repo: **Settings → Pages → Source → Deploy from branch → main → / (root)** or `/polymath-app` if that is your subfolder.

Your app will be live at `https://YOUR_USERNAME.github.io/polymath/` within a minute.

---

## Customising

Everything is in `index.html`. The data is at the top of the `<script>` block:

- `dayFocus` — the "Full Focus" line for each day of the week
- `dayTasks` — the task list for each day (indexed 0 = Sunday, 1 = Monday, etc.)
- `decades` — the four phases, their tracks, and milestones
- `clusters` — the six knowledge domains and their interests
- `laws` — the ten laws

No build step. Edit the file, push, done.

---

## Roadmap

- [ ] Notion API sync (alternative to Google Sheets)
- [ ] Countries visited tracker (tap to add, map view)
- [ ] Language level tracker with spaced repetition reminders
- [ ] Weekly review screen with auto-generated summary
- [ ] Streak leaderboard (optional, for accountability partners)

---

## Philosophy

Built on three sources:

- **Anthony Bourdain** — the full life and the ambitious one are not in competition
- **The Bhagavad Gita** — you are not entitled to the fruit, only the action
- **The person who built this** — who decided that 20 to 30 is not a rehearsal

---

*Built by Krish. Age 20. The decade starts now.*

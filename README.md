# Pod

An iPod-style music player that runs entirely in your browser. Drag in your own MP3s, spin the click wheel, and listen. Your library is stored on your device and nothing is uploaded.

**Live demo:** `https://<your-username>.github.io/<repo>/`

<!-- Add a screenshot: ![Screenshot](screenshot.png) -->

## Features

**Library**
- Drop audio files anywhere on the page, use **Add Music**, or **Add Folder** (MP3, M4A, AAC, WAV, OGG, Opus, FLAC; browser support varies by format)
- Reads ID3 tags: title, artist, album, track number, genre and embedded cover art
- **Add from Link** downloads an MP3 from a URL (the host must allow cross-origin downloads)
- Library lives in your browser's IndexedDB, so it works offline once songs are added

**Browsing**
- Music: Songs, Artists, Albums, Genres, Recently Added, Top Rated
- Search, Cover Flow, Playlists, Favorites, Recently Played, Most Played, Up Next, Shuffle Songs
- Sort songs by title, artist or newest; type a letter to jump in a list
- Star ratings (0 to 5) and favorites

**Playback**
- Shuffle, repeat (off / all / one), speed (0.75× to 2×), sleep timer (15 / 30 / 60 min)
- Equalizer presets: Flat, Bass Boost, Treble Boost, Vocal, Loudness
- Sound Check (volume levelling) and Track Fade
- Resumes where you left off, and remembers position inside long tracks
- Lock-screen and media-key controls through the Media Session API

**Lyrics and artwork**
- Time-synced lyrics from [lrclib.net](https://lrclib.net), cached on your device after the first fetch
- Paste your own lyrics, or refetch, from the long-press menu on the lyrics screen
- Missing cover art is looked up automatically from the iTunes Search API when you are online (can be turned off)

**Extras**
- Visualizer (bars, mirrored bars, waveform) and clock
- Seven colours: silver, black, blue, red, green, pink, gold
- Light or dark screen, optional accent colour taken from the album cover
- Backlight timeout, click-wheel sound and haptics, Hold switch
- Languages: English, Spanish, French, German, Portuguese, Japanese, Chinese

## Controls

| Action | How |
| --- | --- |
| Scroll a list or change volume | Drag in a circle on the wheel, mouse wheel, or ↑ ↓ |
| Select | Centre button or **Enter** |
| Back | **MENU**, **Esc** or **Backspace** |
| Play / pause | ⏯ button or **Space** |
| Next / previous | ▸▸ / ◂◂, or → / ← |
| Seek | Hold ◂◂ or ▸▸, or drag the progress bar |
| Song options (play next, playlist, rating, info, delete) | Hold the centre button on a song |
| Favorite the current song | Hold the centre button on Now Playing |
| Switch Now Playing wheel mode (Volume, Scrub, Rating) | Press the centre button on Now Playing |
| Jump to a letter in a list | Type the letter |
| Hold switch (lock controls) | The switch at the top of the pod, or **Shift+H** |
| Shuffle all | Shake your phone (iOS asks for motion permission on first tap) |
| Swipe | Swipe right to go back; on Now Playing swipe to change songs |

## Run locally

Service workers and installation need `http://localhost` or HTTPS, so serve the folder instead of opening the file directly:

```bash
python -m http.server 8000
# then visit http://localhost:8000
```

## Deploy to GitHub Pages

1. Save the player as `index.html` in the repo root.
2. Add the PWA files listed below (the page links to them).
3. In the repo, go to **Settings > Pages**, choose **Deploy from a branch**, select `main` and `/ (root)`, then save.
4. Open `https://<your-username>.github.io/<repo>/`.

### Files the page expects

`index.html` already references these. The player still works without them, but it cannot be installed or run offline until they exist.

| File | Purpose |
| --- | --- |
| `manifest.json` | App name, icons and colours for "Add to Home Screen" |
| `icon-192.png` | Favicon and app icon (192×192) |
| `apple-touch-icon.png` | iOS home-screen icon (180×180) |
| `sw.js` | Service worker that caches the page for offline use |

Example `manifest.json`:

```json
{
  "name": "Pod",
  "short_name": "Pod",
  "start_url": "./",
  "display": "standalone",
  "background_color": "#15161b",
  "theme_color": "#15161b",
  "icons": [
    { "src": "icon-192.png", "sizes": "192x192", "type": "image/png" }
  ]
}
```

Minimal `sw.js` (caches the page, serves it when offline):

```js
const CACHE = 'pod-v1';
const FILES = ['./', 'index.html', 'manifest.json', 'icon-192.png', 'apple-touch-icon.png'];

self.addEventListener('install', e => {
  e.waitUntil(caches.open(CACHE).then(c => c.addAll(FILES)));
  self.skipWaiting();
});
self.addEventListener('activate', e => {
  e.waitUntil(caches.keys().then(keys =>
    Promise.all(keys.filter(k => k !== CACHE).map(k => caches.delete(k)))));
  self.clients.claim();
});
self.addEventListener('fetch', e => {
  if (e.request.method !== 'GET') return;
  e.respondWith(
    caches.match(e.request).then(hit => hit || fetch(e.request).catch(() => caches.match('./')))
  );
});
```

Bump the `CACHE` name whenever you change the page so users get the update.

## Install as an app

- **iPhone / iPad (Safari):** Share, then **Add to Home Screen**
- **Android / desktop (Chrome, Edge):** use the browser's **Install** option

## Your data and privacy

- Songs, tags and cover art are stored in IndexedDB; settings and playlists in `localStorage`. Nothing is uploaded and there is no account, analytics or cookie.
- Network requests happen only for:
  - **Lyrics:** when you open the Lyrics screen, the song title, artist and duration go to lrclib.net
  - **Cover art:** when a song has no art, artist and title go to the iTunes Search API (turn off with **Settings > Auto Cover Art**)
  - **Add from Link:** the URL you enter
- **Backup Data** saves settings, playlists, favorites, play counts and ratings to a JSON file. It does not include the audio files; **Restore Data** matches songs by file name and size.
- Browsers can clear site data under storage pressure. Installing the app and the built-in persistent-storage request reduce that risk, but keep your original files.

## Browser notes

Works best in current Chrome, Edge and Safari. Cover Flow reflections use `-webkit-box-reflect` (Chromium and Safari). The battery indicator is hidden where the Battery API is unavailable (Safari, Firefox).

## Customising

Near the top of the script there is an empty `STARTER` array. Add links to CORS-enabled MP3s you have the rights to share and a **Get Starter Music** item appears in the menu.

## Legal

Only add music you own or have permission to use. This project is an independent tribute to a classic interface design and is not affiliated with or endorsed by Apple Inc. "iPod" is a trademark of Apple Inc.

## License

MIT. See [LICENSE](LICENSE).

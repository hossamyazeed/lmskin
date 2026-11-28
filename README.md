# LMSkin

A Roon-style skin for Lyrion Music Server 9.0+. Ships as a normal LMS plugin so it
can be enabled or disabled without touching the server install, and coexists with
Material Skin (choose the skin in Settings → Interface).

## Install

**From the plugin manager (recommended):** add the repository URL
`https://raw.githubusercontent.com/hossamyazeed/lmskin/main/repo.xml`
in Settings → Plugins, then install **LMSkin** from the list. This is the
managed install: it survives the Plugins page, LMS upgrades, and shows
updates when a new version is published. The same URL works on any LMS
server, Windows or Linux.

**Manual / development:** copy or symlink this folder to
`<LMS>/server/Plugins/LMSkin`
(on Windows: `C:\ProgramData\Squeezebox\Cache\InstalledPlugins\Plugins\LMSkin`),
restart LMS. (A manually copied folder under `InstalledPlugins` is deleted
by LMS whenever the Plugins page is applied — use the repository route for
anything you want to keep.)

Either way: Settings → Interface → select **lmskin**, then open
`http://<server>:9000/lmskin/`.

## Build the client

The skin's JavaScript and CSS are built into `HTML/lmskin/html/app/`, which is the
only build output directory. `index.html`, `skin.css` and `skinconfig.yml` are
hand-written and never overwritten.

```
cd client
npm install
npm run build        # writes ../HTML/lmskin/html/app/
npx svelte-check     # types
```

Dev loop against a live server (proxies JSON-RPC, CometD, artwork):

```
LMS_URL=http://192.168.1.10:9000 npm run dev
```

## Server commands

All are JSON-RPC commands under `lmskin` / `lmskin-client`.

| Command | Purpose |
| --- | --- |
| `lmskin bootstrap` | Version, prefs, plugin flags - one call at start-up |
| `lmskin browsemodes` | Library browse modes registered by any plugin |
| `lmskin apps` | Streaming-service apps (Tidal, Qobuz, Spotify…) |
| `lmskin extras` | Pages contributed by other plugins |
| `lmskin scan-progress` | Library scan state and last-scan time |
| `lmskin info` | Server information panel |
| `lmskin similar artist:<name>` | Last.fm similar artists, cached 90 days |
| `lmskin transferqueue from:<id> to:<id> mode:move\|copy\|swap` | Queue transfer / sync / group |
| `lmskin focus facet:<genre\|decade\|format\|rating> filters:<k=v;k=v>` | Facet counts for the Focus panel |
| `lmskin wrapped-stats range:<year\|quarter\|month\|30d\|90d\|all>` | Listening-time statistics from the play log |
| `lmskin wrapped-history range:<key> offset:<n> limit:<n>` | Paged play-log rows |
| `lmskin wrapped-clear confirm:1` | Empty the play log (destructive) |
| `lmskin ai-generate prompt:<text> [genre: decade: energy: language: count: mode:auto\|library\|free]` | Start an AI playlist job (async) |
| `lmskin ai-result job:<id>` | Poll a job: status + matched tracks |
| `lmskin ai-replace job:<id> index:<n>` | Swap a missing track for a same-artist library track |
| `lmskin ai-play job:<id> player:<id> mode:load\|add` | Play/queue the result |
| `lmskin ai-save job:<id> name:<name>` | Save the result as an LMS playlist |
| `lmskin ai-cancel job:<id>` | Drop a job |
| `lmskin-client sort-queue order:<n>` | Re-sort a player's queue |
| `lmskin-client remove-queue indexes:<i,i>` | Remove queue entries |
| `lmskin-client set-lib id:<libraryId>` | Per-player virtual library |

## AI playlist studio

Studio (rail → Studio) generates playlists from a natural-language request plus
filter chips, grounded in your real library: with a small enough candidate pool
the model picks actual `tracks.id`s; with a big pool it suggests titles that are
matched back with a normalized ladder (Found / Uncertain / Missing). Configure an
OpenAI-compatible endpoint (OpenAI, OpenRouter, Gemini, Ollama, LM Studio) in
Settings → Advanced → LMSkin. The API key is stored in LMS prefs and never sent
to the client or written to logs. Server: `lib/Plugins/LMSkin/Ai.pm`.

## Wrapped listening statistics

Wrapped (rail → Wrapped) counts real listening time per track in
`<cachedir>/lmskin/plays.db`: a row is written when a track has played for at
least 5 s, with skip detection (< 40% of the track) and a local/remote/radio
source split. Radio streams are under-reported (no `newsong` events). Rows older
than 24 months are pruned. Server: `lib/Plugins/LMSkin/PlayLog.pm`.

## Layout

```
install.xml          plugin manifest
Plugin.pm            command layer (trimmed fork of lms-material, MIT)
Settings.pm          settings page registration
lib/Plugins/LMSkin/  Ai.pm (studio), PlayLog.pm (wrapped)
HTML/lmskin/           the skin: index.html, skinconfig.yml, built app
  html/app/          build output (generated)
client/              Svelte 5 + TypeScript + Vite source
```

## Credits

The Perl command layer derives from [lms-material](https://github.com/CDrummond/lms-material)
by Craig Drummond (MIT).

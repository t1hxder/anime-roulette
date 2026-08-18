# Anime Roulette

An anime roulette. Tell it what you've already seen, set a few filters, and it pulls something you haven't watched yet.

**Live:** https://t1hxder.github.io/anime-roulette/

Built because picking what to watch takes longer than watching it.

## What it does

- **Random pull** from ~2000 top-rated titles, excluding anything on your watched list
- **Filters** for length, format, genre, and a rating floor
- **First seasons only** toggle, so it never tells you to start at season 3
- **"Recommend something like X"** seeds the roulette from any anime's community recommendations
- **AniList import** pulls your completed list in by username, no login required
- **Share links** in the form `?like=<id>` open the app already seeded on that title

## Stack

One HTML file. No build step, no dependencies, no backend.

- Vanilla JS, no framework
- [AniList GraphQL API](https://anilist.gitbook.io/anilist-apiv2-docs/) for all data, free and unauthenticated
- `localStorage` for the watched list
- GitHub Pages for hosting

Everything runs client side, so there's nothing to pay for and nothing to maintain.

## Running it locally

Opening the file directly gives the page a `null` origin, which browsers block from making cross-origin requests. Serve it instead:

```bash
git clone https://github.com/t1hxder/anime-roulette.git
cd anime-roulette
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

## Notes from building it

The interesting problems weren't in the UI.

**The first API didn't hold up.** Started on Jikan, the community MyAnimeList mirror. It returned 429s under normal use and eventually a 504. Reading the status code told me which was mine to fix and which wasn't: 429 meant back off and retry, 504 meant their server was down and no amount of retrying would help. Switched to AniList, which is properly funded and has been stable since.

**Loading 2000 titles naively took 40 requests.** GraphQL lets you alias several queries into one document, so five pages arrive per round trip and a shared fragment keeps the query small. Adding `format_in:[TV,TV_SHORT,MOVIE]` server-side meant no bandwidth spent on results that were going to be filtered out anyway. 40 requests became 8.

**Perceived speed beat actual speed.** The pool array is assigned once and pushed into, so the pull button unlocks after the first batch while the rest streams in behind it. Users can start using the app about two seconds in rather than waiting for all of it.

**A silent `catch` hid a real bug for three rounds.** The 2000-title cache pushed `localStorage` past its 5MB quota, so writes to the watched list started throwing `QuotaExceededError`, which an empty catch block swallowed. Imports appeared to work and vanished on refresh. The fix was to make the cache lose that fight: on a quota error, evict the pool and retry, because the pool re-downloads in ten seconds and the user's list doesn't. Failed writes now surface in the UI instead of disappearing.

**Search needed two guards, not one.** Debouncing stops a request per keystroke. It does not stop a slow response for `"att"` landing after a fast one for `"attack"` and overwriting good results. Every request carries a token and any response whose token is stale gets discarded.

**Mobile zoom came from CSS, not JS.** The reveal animation scaled a ring past the viewport edge, widening the scrollable area, and the browser zoomed out to compensate. Offset shadows and scale animations both do this quietly. `overflow-x: hidden` on the root fixed it.

## License

MIT. Anime data belongs to AniList.

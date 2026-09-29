# SyncLyrics (multi-artist fix)

Same as the upstream SyncLyrics add-on, built locally with one patch
(`patches/0001-lyrics-multi-artist-fallback.patch`) applied to the upstream code:

- **LRCLib:** when the normal searches find nothing, retry with each individual
  artist ("K.S. Chithra, Ilaiyaraaja & Arunmozhi" -> "Arunmozhi", ...) and with the
  title stripped of "(film name)" / " - From ..." suffixes, then a title-only search.
  A loose result is accepted only if its title matches **and** its duration is within
  6 s or it shares an artist, so a same-named different song is not shown.
  Among search results the one closest in duration is preferred.
- **Musixmatch:** ignores matches whose title has nothing in common with the query
  (e.g. the "NOKIA by Drake" hit returned for every query).

The Docker build fails loudly if upstream changes these files and the patch no
longer applies.
- **QQ Music:** its search endpoint answers HTTP 500 to every query. Server errors
  are no longer retried (was 4 attempts, ~15 s and 5 ERROR lines per song); instead
  QQ is skipped for 60 minutes with a single warning, then tried again.
- **Musixmatch login (fix3):** optional `musixmatch_user_token` / `musixmatch_token_guid`
  settings. When set, SyncLyrics uses your own Musixmatch login instead of the anonymous
  guest token (which now gets the placeholder "NOKIA by Drake" match for every song).
  If Musixmatch rejects the token, it logs one warning and skips Musixmatch until you
  paste a fresh token and restart - it does not fall back to the guest token.

### Getting a Musixmatch token

The token is the `usertoken` the Musixmatch **desktop app** sends to
`apic-desktop.musixmatch.com`. With the desktop app installed and signed in:

1. Open its developer tools (Ctrl+Shift+I on Windows; if the app blocks this, capture its
   traffic with a proxy tool such as Fiddler or mitmproxy instead).
2. In the Network tab, play or search a song and open any request to
   `apic-desktop.musixmatch.com/ws/1.1/...`.
3. Copy the `usertoken` query parameter into **Musixmatch login token**, and the value of the
   `x-mxm-token-guid` cookie (if present) into **Musixmatch token GUID**.

Tokens are personal; treat them like a password. They can expire - if the log shows
"Configured login token was rejected", repeat the steps.

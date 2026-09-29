# SyncLyrics (multi-artist fix)

Same as the upstream SyncLyrics add-on, with the patches in `patches/` applied on top.

**Fast build (fix4):** the Dockerfile starts from the author's published, already-compiled
image (`ghcr.io/anshulj999/synclyrics-ha-<arch>:2.4.0`) and only applies the patches, so
installs and updates take a minute or two. To follow a new upstream release, bump
`UPSTREAM_VERSION` in the Dockerfile and `version:` in config.yaml.

Patches:

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
- **Album covers (fix5):** the iTunes cover lookup used to accept any song by the same
  artist (or even the first search result), so tracks on compilations got the cover of an
  unrelated film. A cover is now used only if its album or its title (ignoring
  "[From 'Film']" style suffixes) matches; otherwise the player's own cover is kept.
- **Player choice (fix6):** with no player set, SyncLyrics followed the first player
  reporting "playing" - often a TV running YouTube, which has no Music Assistant queue,
  so the page stayed Idle while music played elsewhere. Players playing a Music Assistant
  queue are now preferred.
- **Next-up cover (fix7):** the "next song" card loaded its cover straight from the Music
  Assistant address, which the browser often can't reach (LAN-only address blocked by
  Chrome's local network access check), so it showed a broken image. SyncLyrics now serves
  that cover itself via `/api/ma-image` (only Music Assistant image URLs are accepted).

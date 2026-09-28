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

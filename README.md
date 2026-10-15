# analyzerx-assets

Market data **assets** for **AnalyzerX**, kept in a separate repository from the application code
(`saayad566-tech/analyzerx`). The app stays light; this repo holds the data, split into
size-bounded chunks and fetched at runtime.

## Layout

```
manifests/index.json     <- the "main file": lists every market pack + where its manifest lives
manifests/<MARKET>.json  <- per-market manifest: datasets, chunk list, sizes, SHA-256 digests
packs/<MARKET>/*.bin     <- chunked data (each file <= the manifest's chunkBytes)
source/<MARKET>/*.json   <- the human-readable datasets the chunks were cut from (for audit)
SHA256SUMS               <- flat list of every chunk + manifest digest
```

## How the app uses this

1. Fetch `manifests/index.json` (the main file).
2. Pick the entry for the **selected market only** (no cross-market fallback).
3. Fetch that market's manifest and verify its self-digest.
4. Download only the chunks whose SHA-256 does not already match locally (resumable).
5. Verify every chunk and every assembled dataset, then **atomically activate** the pack.

Market isolation is enforced end to end: a pack belongs to exactly one market and can never be
served for another.

## Rebuilding these assets

From the application repository:

```
node scripts/build-resource-pack.mjs --src resource-packs-src --out .assets-dist --version <YYYY.MM.DD>
```

Then publish the contents of `.assets-dist` (plus `source/`) to this repository. The build is
deterministic: identical inputs produce byte-identical manifests and chunks.

## Verified

Every manifest carries a SHA-256 over its canonical body; every chunk and dataset carries its own
SHA-256. `SHA256SUMS` lists them all. The application's pure verifier reproduces every digest.

## Music Artist Collaboration Network
Final project for SI 507: Intermediate Programming

A command-line tool that builds a graph of musical artists from Last.fm data and lets you explore how they connect.

*Python, Last.fm API, Claude Code, Codex*

<img width="1834" height="854" alt="Screen Recording 2026-08-23 at 12 01 12 PM" src="https://github.com/user-attachments/assets/00d79db9-b218-44d5-b2fc-31221b7f98fc" />

### What it does

Nodes are artists. Edges connect artists that Last.fm's community data marks as similar. Once that graph exists, you can ask questions a flat list can't easily answer:

- **Artist Profile**: look up an artist and see their direct neighbors in the graph
- **Shortest Path**: find the shortest chain of similarity connecting two artists
- **Influence Rankings**: rank artists by degree centrality (most connected)
- **Genre Filter**: list artists matching a genre or community tag

The graph starts from 11 seed artists across hip-hop, pop, K-pop, indie, and electronic, then grows breadth-first through each artist's similar-artist list. A full run produces 52 artists and 45 edges, cached locally so repeat runs don't hit the API again.

### Architecture

- `music_graph.py` — `Artist` dataclass + `MusicGraph`, a thin wrapper around `networkx.Graph`
- `clients.py` — `LastFmClient`, a Last.fm API wrapper with a JSON response cache
- `build_graph.py` — builds the graph breadth-first from seed artists
- `main.py` — CLI with the four interaction modes

Splitting these apart meant the graph logic could be unit-tested completely offline.

### Notes

- Two caching layers: raw API responses (`cache.json`) and the assembled graph (`graph_cache.json`), so changing how the graph is built doesn't require new API calls.
- Missing artist lookups return `None` instead of raising, so one bad name doesn't crash a multi-minute build.
- Rankings use degree centrality rather than a more complex measure — since edges represent similarity, not directed influence, that's the honest way to describe what's being measured.

## Testing

29 unit tests, all running against small hand-built graphs (no network calls). They cover things like:

- self-paths, disconnected nodes, and missing IDs returning `None` instead of erroring
- genre filtering across both structured genres and Last.fm tags, case-insensitively
- centrality correctly identifying the center of a star graph

## What's next

- Streamlit front end with a live graph visualization
- Expanding past 52 artists into the thousands
- An LLM layer to narrate a shortest path in plain language
- Spotify audio features (tempo, energy, valence) layered onto each hop

## Stack

Python, `networkx`, Last.fm API, `requests`, `unittest`

---

Built for SI 507 (University of Michigan School of Information).

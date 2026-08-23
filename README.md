## Music Artist Collaboration Network
Final project for SI 507: Intermediate Programming

*Python, Last.fm API, Claude Code, Codex*

<img width="1834" height="854" alt="Screen Recording 2026-08-23 at 12 01 12 PM" src="https://github.com/user-attachments/assets/00d79db9-b218-44d5-b2fc-31221b7f98fc" />

Streaming platforms are good at recommending songs. They're not very good at answering a more interesting question: *how is one artist connected to another, and what does that path reveal about the shape of a genre?*

I wanted to treat music as a graph: nodes are artists and edges are similarity relationships pulled from Last.fm's community-driven "similar artists" data. Once that structure exists, questions that are awkward to ask of a flat dataset — what's the shortest chain of influence between Amy Winehouse and TWICE? which artist sits at the center of this whole network? — become simple graph traversals.

## What It Does

The program is a command-line tool with four ways to explore the network:

| Mode | What it answers |
|---|---|
| **Artist Profile** | Who is this artist, and who are their direct neighbors in the graph? |
| **Shortest Path** | What's the shortest chain of similarity connecting two artists? |
| **Influence Rankings** | Which artists are most connected, by degree centrality? |
| **Genre Filter** | Which artists in the graph match a given genre or community tag? |

The graph is seeded from 11 artists spanning hip-hop, pop, K-pop, indie, and electronic (Kendrick Lamar, Bad Bunny, TWICE, Mitski, Amy Winehouse, and others), then grown through each artist's similar-artist list until it reaches a target size. A completed run produces a **52-artist, 45-edge network**.

## Architecture

- **`music_graph.py`**: `Artist` dataclass + `MusicGraph` 
- **`clients.py`**: `LastFmClient`, a Last.fm API wrapper with a JSON response cache
- **`build_graph.py`**: builds the graph breadth-first from seed artists
- **`main.py`** — the CLI shell that ties it together into the four interaction modes.

Keeping the graph logic, the API client, and the construction pipeline in separate files meant I could unit-test `MusicGraph` completely in isolation — the test suite never makes a network call.

## Engineering Decisions Worth Calling Out

**Caching at two layers.** Every raw API response is cached by `LastFmClient` (`cache.json`), and the fully assembled graph is cached separately (`graph_cache.json`). That split matters: if I ever want to change how the graph is *built* — a different BFS cutoff, a different similarity threshold — I can delete only `graph_cache.json` and rebuild from already-cached API responses, with zero new network calls.

**Fail-soft artist construction.** `build_artist_from_lastfm` returns `None` instead of raising when Last.fm has no data for a name — a very real scenario given how many spelling variants and lesser-known collaborators show up in similarity lists. The graph-building loop treats a missing artist as "skip," not "crash," so one bad lookup out of a hundred doesn't take down a multi-minute build.

**Degree centrality for "influence."** `top_by_centrality` uses `networkx.degree_centrality` — the simplest legitimate centrality measure — deliberately. Given that edges here represent listener-perceived similarity rather than a directed influence relationship (like "sampled" or "produced by"), degree centrality is the honest measure: it answers "who sits at the busiest crossroads of the network," not a claim about who actually shaped whom.

## Testing as Documentation

The test suite (`test_music_graph.py`) has **29 tests** and deliberately never touches the network — it constructs small, hand-built graphs (a four-node chain, a five-node star) to test graph behavior in isolation from API flakiness. A few examples of what it locks down:

- Shortest path between a node and itself returns a single-element path, not an error.
- Disconnected nodes return `None` rather than raising.
- `add_edge` silently no-ops if either endpoint isn't in the graph, instead of crashing on a bad ID.
- Genre filtering matches against *both* the structured genre list and the free-text Last.fm tags, case-insensitively.
- In a star graph, the center node — and only the center node — ranks first by centrality.

Reading the test file top to bottom tells you what `MusicGraph` is supposed to guarantee, which was the actual goal — not just "tests pass," but "tests explain the contract."

## What I'd Build Next

With more time, the natural extensions are the ones the proposal's "dream scope" already pointed at:

- A Streamlit front end with a live PyVis visualization of the network, so the graph is something you explore by clicking rather than typing artist names
- Expanding past the current 52-artist seed set into the thousands, across more genres and eras
- An LLM layer that turns a `shortest_path` result into a written narrative — *"Amy Winehouse's soul influence reaches TWICE through three hops of genre-blending pop"* — instead of a bare list of names
- Spotify audio-feature data (tempo, energy, valence) layered onto each hop of a path, so a traversal shows not just *who* connects two artists but *how the sound itself shifts* along the way

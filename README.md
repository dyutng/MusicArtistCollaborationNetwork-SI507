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

- `music_graph.py`: `Artist` dataclass + `MusicGraph`, a thin wrapper around `networkx.Graph`
- `clients.py`: `LastFmClient`, a Last.fm API wrapper with a JSON response cache
- `build_graph.py`: builds the graph breadth-first from seed artists
- `main.py`: CLI with the four interaction modes

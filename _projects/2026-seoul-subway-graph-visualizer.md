---
title: "Seoul Station Graph Routing Visualizer"
category: other # not "research" -> appears under "Other Projects"
year: 2026
summary: "I built a visualizer that shows BFS, DFS, Dijkstra, and A* searching a roughly 50-station Seoul subway graph. On Gangnam → Hapjeong, A* visited 7 nodes and Dijkstra visited 40, with the same route cost."
thumbnail: /assets/img/projects/seoul-graph-routing.gif
hide_hero: true
links:
    - name: Code
      url: https://github.com/sbkim-208/session-4-graph-playground
---

I could implement BFS, DFS, Dijkstra, and A*, but I wanted to see why they explored different parts of the same transportation network. I built a visualizer around roughly 50 Seoul subway stations so I could watch the search and compare the results.

## Choosing a heuristic
{: #study-process }

Before writing `a_star.py`, I considered precomputing costs with Dijkstra and reusing the table as A*'s heuristic. The part that needed checking was whether those values would still be valid lower bounds for the goal in a new query. A cost table for one set of pairs doesn't establish that on its own.

I used straight-line distance instead, which didn't require that precomputation. It still needs to lower-bound the routing cost in compatible units. Switching to distance doesn't remove the need to check the graph's edge-cost definition.

## Watching the search on the full graph
{: #study-scope }

Each algorithm uses the same interface and streams intermediate search states. The frontend renders those frames without needing a separate implementation for each algorithm. New algorithms are discovered at startup, so I could add A* without changing the frontend.

The backend uses FastAPI and WebSocket. React and TypeScript handle the frontend, with React Flow for the graph canvas. `/ws/run` streams a single run, while `/eval` compares all algorithms on the same start and goal. I also used the shared interface for the session's homework problems: Number of Islands, Course Schedule, Shortest Path in Binary Matrix, and Network Delay Time.

The styling looked fine on a small demo graph and fell apart when I loaded the station network. Labels overlapped. I adjusted the font, label size, node colors, and layout spacing until I could follow the search on the full graph.

<details>
<summary>Watch the A* search animation</summary>
<figure>
  <img src="{{ '/assets/img/projects/seoul-graph-routing.gif' | relative_url }}" alt="A* search progressing over the Seoul subway graph, followed by an algorithm-comparison table" loading="lazy">
  <figcaption>The visualizer shows the visited stations as the search runs, then compares the algorithms.</figcaption>
</figure>
</details>

## Gangnam to Hapjeong
{: #study-results }

I ran all four algorithms through `/eval` on Gangnam → Hapjeong. They returned the same 6 hops and total cost of 26, but visited different amounts of the graph.

| Algorithm | Hops | Total cost | Nodes visited | Iterations |
| --- | ---: | ---: | ---: | ---: |
| A* | 6 | 26 | 7 | 7 |
| DFS | 6 | 26 | 25 | 25 |
| BFS | 6 | 26 | 27 | 27 |
| Dijkstra | 6 | 26 | 40 | 40 |

A* visited 7 stations, compared with Dijkstra's 40. Its heuristic directed the search toward Hapjeong. I don't think DFS's 25 means much beyond this route, though. The neighbor-list order may simply have sent it in a useful direction. I haven't varied that order to check.

All four found the same cost here, but BFS and DFS don't generally guarantee an optimal route when edges have different costs. This is one query, not a benchmark across the network.

## The comparison I still need
{: #study-next }

I'd run more start–goal pairs, check the heuristic bounds against the edge costs, and measure runtime as well as visited nodes. Varying DFS's neighbor order would help show how much of this result came from the particular query.

The visualizer already gave me something the small examples didn't: I could see where the algorithms spent their search effort, and where my display stopped being readable. That made both kinds of problems easier to investigate.

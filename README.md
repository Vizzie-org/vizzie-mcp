# Vizzie MCP server

A remote MCP server that lets an AI assistant find open government data for a place,
work out which geography a column of area codes belongs to, fetch licence-cleared
boundaries, draw the area within reach of a place, and make a map that opens in
[Vizzie](https://www.vizzie.org) with one link. It is free, needs no key, and every
answer carries the data's licence.

```
https://api.vizzie.org/mcp
```

Streamable HTTP. The same calls are a REST API at `https://api.vizzie.org/v1`
([docs](https://www.vizzie.org/api/), [OpenAPI](https://api.vizzie.org/v1/openapi.json)).

Vizzie is a web-based tool for visualising and mapping open data, connected to 280+
government data portals across 40+ countries. This repository describes the server's
interface; the server itself runs at the address above.

## Add it to an assistant

- **Claude:** Settings → Connectors → Add custom connector, and paste the URL above.
- **Any other client with remote MCP support:** add a server with that URL. No key is
  needed.
- **Clients without MCP:** the REST API takes plain JSON over HTTPS, so a script or a
  tool-calling agent can use it directly.

Searching and making maps need no sign-in. Saving, editing and publishing a map need a
free Vizzie account: the first time you ask, the assistant opens Vizzie's sign-in
(OAuth), where you can also sign up.

Then ask for things like:

- "Map median weekly rent by SA2 in Ballarat, Victoria."
- "Find street tree data for Fitzroy, Victoria."
- "Show everywhere within a 15-minute walk of Flinders Street Station."
- "What geography are these codes?" (with a column of LSOA, SA2, tract or NUTS codes)
- "Here's a CSV of tract-level values for Seattle. Map it."

The answer ends in an `app.vizzie.org` link that opens the map. Viewing it needs no
account.

## Tools

| Tool | What it does |
| --- | --- |
| `search_datasets` | Open datasets and census maps for a topic and place, across the portals Vizzie connects to. Each result has its licence (and how sure that is), whether Vizzie can map it, and a link to open it. Census topics come first as ready-made map links. Send the place's two-letter country code: place names repeat across countries. |
| `describe_dataset` | A dataset's columns (meaning, type, role, unit), its licence and the boundaries it joins to. Example values only when the licence allows passing them on. |
| `resolve_geography` | Which boundary level and vintage a column of area codes is, the share that match (over every code, not a sample), some matched areas by name, and whether the boundaries can be fetched. |
| `get_boundaries` | Boundary polygons as GeoJSON for a list of codes or a bounding box, with attribution. Up to 2,000 areas per call, and only where the licence allows. |
| `get_catchment` | The area within reach of an address or of up to 1,000 points or shapes: a straight-line distance, or how far you can walk or drive along roads (up to 100 origins). 0.1–10 km or 0.1–5 mi. Returns GeoJSON and a link that opens it in Vizzie. |
| `create_map` | A link that opens a map in Vizzie: a census map of a topic for a place; your own table of area codes and values joined to boundaries; a dataset's preview; or up to five of these as layers in one map. |
| `save_map` | Keeps a map from `create_map` in your Vizzie account, so its link doesn't expire. **Needs sign-in.** |
| `edit_map` | Changes a map in place (title, colours, classes, key, a census layer's topic or place, a table's values, layers added, removed or reordered). Same link. **Needs sign-in.** |
| `publish_map` | Saves the map and returns a link that opens it in Vizzie with the Share panel open, one click from a public link. **Needs sign-in.** |

The first four are read-only. A token issued to an assistant works with the three
sign-in tools only, not with the rest of the account.

## Licences

Search results and dataset descriptions are metadata and cover everything Vizzie
catalogues. Data and geometry are passed on only when the source's licence is confirmed,
declared and allows commercial use; share-alike sources come with a `share_alike` flag
and their attribution. A map made with `create_map` is drawn inside Vizzie, so its
boundaries stay on Vizzie's side.

## Limits

| What | Limit |
| --- | --- |
| Search, describe, resolve | 60 calls per 5 minutes each |
| Boundaries, maps, catchments | 20 calls per 5 minutes each |
| Isochrone origins | 50 a day per address |
| All calls | 2,000 a day per address |
| Layers in a map | 5 |
| Maps from your own rows | 5,000 rows and 1 MB across the map, 200 maps a day; links last 30 days unless saved |
| Saved maps | 200 per account |

## Contact

hello@vizzie.org · [www.vizzie.org](https://www.vizzie.org) ·
[Privacy](https://www.vizzie.org/privacy.html) · [Terms](https://www.vizzie.org/terms.html)

This README and `server.json` are MIT licensed. The data the server returns carries
each publisher's own licence, stated in every response.

# Kylito's Way

A third-person 3D neighborhood explorer and multiplayer game. Explore any real-world address with actual building footprints, streets, and terrain elevation.

## Features

- **Real-world maps** - Enter any address and explore it in 3D
- **Terrain elevation** - Actual hills and slopes from elevation data
- **Dynamic world streaming** - Infinite exploration with chunk-based loading
- **Vehicles** - Drive sedans, SUVs, and trucks
- **Edit mode** - Place buildings, roads, fences, walls, hedges, parking lots
- **Top-down view** - Bird's eye editing mode
- **Paintball mode** - Shoot paintballs with physics and splats
- **Mobile support** - Touch controls for phones/tablets

## Quick Start

Just open `index.html` in a browser. No build step, no server required.

1. Enter any address (e.g., "1600 Pennsylvania Ave, Washington DC")
2. Click GO
3. Explore with WASD, mouse to look
4. Press E to enter/exit vehicles
5. Press TAB for edit mode
6. Press P for paintball mode

## Controls

| Key | Action |
|-----|--------|
| WASD | Move |
| Mouse | Look |
| Shift | Sprint |
| Space | Jump |
| E | Enter/exit vehicle |
| TAB | Toggle edit mode |
| M | Toggle minimap |
| P | Toggle paintball mode |
| F3 | Toggle debug panel |

## Tech Stack

- Three.js for 3D rendering
- OpenStreetMap data via Overpass API
- Open-Meteo for elevation data
- Nominatim for geocoding
- Pure vanilla JS, single HTML file

## Roadmap

See [KYLITOS_WAY_HANDOFF.md](KYLITOS_WAY_HANDOFF.md) for the full vision including:
- Multiplayer deathmatch (GoldenEye style)
- Community town curation
- Advanced building editor

## License

MIT

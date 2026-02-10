# Kylito's Way — Project Handoff Document

## Overview

**Kylito's Way** (formerly StreetCraft) is a third-person 3D neighborhood explorer and multiplayer game built as a single self-contained HTML file. Players can explore any real-world address with actual building footprints, streets, and terrain elevation, then customize it and play paintball.

The goal is to evolve this into a **multiplayer platform** where communities cultivate accurate 3D replicas of their neighborhoods and play GoldenEye-style deathmatch games in them.

**Current state:** Single HTML file, fully client-side, ~1,600 lines. No server, no accounts, no multiplayer. Ready to be split into a proper project.

---

## Current Codebase

### File
- **`streetcraft6.html`** — the entire application (attached alongside this document)
- Self-contained: HTML + CSS + JS in one file
- Only external dependency: Three.js r162 loaded from CDN (`cdnjs.cloudflare.com`)
- Fonts: Orbitron + Share Tech Mono from Google Fonts

### Architecture (Current — Monolithic)

```
streetcraft6.html
├── CSS (~120 lines)
│   ├── Launch screen styles
│   ├── HUD (compass, minimap, address, elevation)
│   ├── Edit mode bar + property panels
│   ├── Paintball mode (crosshair, color picker, HUD)
│   ├── Mobile touch controls
│   └── Responsive breakpoints
├── HTML (~50 lines)
│   ├── Launch screen (address input, GPS, saved games)
│   ├── HUD overlay (compass, minimap, edit toggle, paint toggle)
│   ├── Edit bar (tools: select, house, building, road, tree, yard, delete)
│   ├── Property panel (size/color/roof editing)
│   ├── Paintball HUD (color swatches, custom picker)
│   ├── Touch controls (joystick, look zone, action buttons)
│   └── Save modal
└── JavaScript (~1,400 lines)
    ├── Constants & state variables
    ├── Three.js initialization
    ├── Geocoding (6-fallback strategy)
    ├── Terrain elevation (Open-Meteo API)
    ├── OSM data fetch (Overpass API)
    ├── World building (buildings, roads, parks, trees, barriers)
    ├── Vehicle system (create, enter, exit, physics)
    ├── Edit mode (place, select, modify, delete, save/load)
    ├── Paintball mode (shoot, physics, splats)
    ├── Controls (desktop keyboard/mouse, mobile touch)
    ├── Minimap & compass
    ├── Game loop (physics, camera, HUD updates)
    └── Startup flow
```

---

## Feature Inventory

### World Generation
- **OSM Integration:** Fetches buildings, roads, parks, water, landuse, barriers, tree rows, individual trees from Overpass API
- **Geocoding:** 6-fallback strategy using Nominatim — structured query → free-form → partial → city-level → coordinates
- **Terrain:** 21×21 elevation grid from Open-Meteo API, bilinear interpolation, vertex-colored terrain mesh
- **Buildings:**
  - Extruded from OSM polygon footprints
  - Height from `building:height` tag → `building:levels` tag → footprint area estimation
  - Colors from `building:colour` tag → `building:material` tag → type-based palette
  - Residential gets peaked roofs (cone), commercial gets flat roofs
  - Roof antenna details on tall buildings
- **Roads:** Subdivided into 6m segments to drape over terrain, width varies by highway classification
- **Trees:** From `natural=tree` nodes and `natural=tree_row` ways, trunk + leaf sphere
- **Barriers:** Fences (with posts), walls, hedges (bush spheres) from `barrier=*` tags — *queried from OSM but currently showing nothing in test areas (sparse tagging)*
- **Streetlamps:** Random placement with point lights

### Player & Vehicles
- **Player:** Capsule character, WASD movement, sprint (Shift), jump (Space), gravity
- **Vehicles:** Sedan/SUV/truck types with distinct proportions
  - Enter/exit with E key
  - Acceleration, braking, steering with speed-dependent turn radius
  - Terrain-following with pitch tilt on slopes
  - Collision detection with buildings
  - Speedometer HUD (mph)
- **Camera:** Third-person follow with lerp, adjustable pitch/yaw, auto-follow mode, terrain clipping prevention

### Edit Mode
- **Tools:** Select, House, Building, Road, Tree, Yard, Delete
- **Click-to-select:** Buildings and vehicles get property panels
- **Property panels:** Width/depth/height sliders, color picker, roof type (peaked/flat/hip), wall material presets
- **Persistence:** localStorage per location key, JSON export/import
- **Community versions:** Save with author name, browse saved versions

### Paintball Mode
- **Desktop:** Free cursor aiming (crosshair follows mouse), left-click shoots, right-drag rotates camera, WASD moves simultaneously
- **Mobile:** Full-screen tap-to-shoot with raycast to tap position, drag to look
- **Physics:** Projectile with velocity + gravity (speed: 120, gravity: -6)
- **Splats:** CircleGeometry on ground (rotated flat) or walls (face camera), with drip effects, up to 300 persistent
- **Colors:** 8 presets + custom color picker

### Navigation
- **Minimap:** Canvas-rendered, expandable to full screen, shows buildings/roads/player/vehicles, street names, scale bar
- **Compass:** Rotating needle with N/S/E/W, heading text
- **Address detection:** Real-time nearest building address + nearest street name
- **Elevation HUD:** Current terrain height and player Y position

### Mobile Support
- **Touch controls:** Virtual joystick (left), look zone (right), action buttons (jump/run/car or gas/brake/exit)
- **Responsive CSS:** Smaller minimap/compass on mobile, adapted panel sizes
- **Performance:** Reduced geometry segments, fewer lights, limited vehicle count on mobile

---

## Key Technical Details

### Coordinate System
```
World:  x = east(+)/west(-)    z = south(+)/north(-)    y = up
Grid:   row-major grid[row*N + col]
        row 0 = south (z = +half)    row N-1 = north (z = -half)
        col 0 = west (x = -half)     col N-1 = east (x = +half)

World ↔ LatLon conversion:
  ll(lat, lon) → {x, z}:  x = (lon-cLon)*111320*cos(cLat)    z = -(lat-cLat)*111320
  Inverse:                 lat = cLat - z/111320               lon = cLon + x/(111320*cos(cLat))
```

### APIs Used
| API | Purpose | Rate Limits |
|-----|---------|-------------|
| Overpass API (overpass-api.de + kumi.systems fallback) | OSM building/road/park/barrier data | Shared public, no auth |
| Nominatim (nominatim.openstreetmap.org) | Geocoding addresses to lat/lon | 1 req/sec, no auth |
| Open-Meteo (api.open-meteo.com) | Elevation data grid | Free, no auth |

### CORS Handling
Uses a `cfetch` wrapper that tries direct fetch first, then two CORS proxies:
1. `corsproxy.io`
2. `api.allorigins.win`

Needed for Overpass API when running from `file://` protocol.

### Browser Compatibility
- Safari/iOS: Fully tested, avoids `Object.assign` on read-only Three.js properties, uses compatible geometry constructors
- Three.js r162 loaded with multiple CDN fallbacks
- No ES modules, no build step, no localStorage for game state beyond saves

### Key State Variables
```javascript
cLat, cLon          // Center coordinates
RADIUS = 500        // World radius in meters (1km diameter)
terrainElevs[]      // 21×21 elevation grid (meters relative to center)
rData[]             // Road segment data for vehicle spawning
buildingAddrs[]     // Building address data for detection
userEdits[]         // Edit mode changes (serializable)
vehicles[]          // All vehicle meshes in scene
paintBalls[]        // Active projectiles
paintSplats[]       // Persistent splat meshes
```

### Building Collision
`ptInBuilding(x, z)` — 2D point-in-polygon test against all building footprint coordinates stored during world generation.

---

## V7+ Roadmap (Owner's Vision)

### Priority 1: GoldenEye-Style Multiplayer Deathmatch
- **Player avatars** visible to other players with name tags
- **Weapon system** — paintball guns with proper hit detection (raycasting, not proximity)
- **Health/lives** — take hits, respawn at spawn points
- **Scoreboard** — kills, deaths, K/D ratio, match timer
- **Game modes:** Free-for-all, team deathmatch
- **Lobby system** — create/join games, pick a town

### Priority 2: Advanced Building Editor
- **Paintbrush-style editing** — drag to mass-modify buildings, not one-at-a-time
- **Replace buildings** — swap OSM building with custom geometry
- **Add fences, walls, hedges** manually where OSM data is sparse
- **Parking lots** — flat textured areas with line markings
- **Playgrounds** — swing sets, slides, benches as prefab objects
- **Separate editor page** — dedicated full-screen building tool outside the game, possibly with top-down or isometric view

### Priority 3: Community Curation System
- **Variation system** — each town can have multiple user-created versions
- **Blank canvas vs. popular versions** — new users choose starting point
- **Submit changes** — propose edits to a shared community version
- **Merge/review** — version owners approve/reject community submissions
- **"Home town"** — users claim a town as their base for hosting games
- **Leaderboard** — most popular/detailed town variations

### Priority 4: Multiplayer Infrastructure
- **WebSocket or WebRTC** server for real-time player sync
- **Player state sync:** position, rotation, vehicle, shooting, hits
- **Database** for user accounts, town variations, saved edits, scores
- **Authentication** — accounts or anonymous with persistent identity
- **Hosting** — players host games in their cultivated towns

### Stretch Goals
- **More vehicle types** — motorcycles, bikes
- **Weather effects** — rain, fog, day/night cycle
- **Sound effects** — engine, footsteps, paintball shots/splats
- **Better building detail** — windows, doors, textures
- **Interior spaces** — enter buildings
- **NPC pedestrians/traffic**

---

## Suggested Project Structure (V7)

```
kylitos-way/
├── client/
│   ├── index.html              # Game client entry
│   ├── editor.html             # Standalone building editor
│   ├── css/
│   │   ├── game.css
│   │   ├── editor.css
│   │   └── shared.css
│   ├── js/
│   │   ├── main.js             # Entry point, game loop
│   │   ├── world.js            # OSM fetch, building/road/tree generation
│   │   ├── terrain.js          # Elevation API, terrain mesh, getTerrainY
│   │   ├── player.js           # Player movement, physics, camera
│   │   ├── vehicles.js         # Vehicle creation, driving physics
│   │   ├── paintball.js        # Shooting, projectiles, splats, scoring
│   │   ├── editor.js           # Edit mode tools, property panels
│   │   ├── minimap.js          # Minimap + compass
│   │   ├── controls.js         # Desktop + mobile input handling
│   │   ├── geocoding.js        # Address lookup with fallbacks
│   │   ├── networking.js       # WebSocket client, state sync
│   │   ├── ui.js               # HUD, menus, modals
│   │   └── utils.js            # Coordinate math, CORS fetch
│   └── assets/
│       └── (textures, sounds eventually)
├── server/
│   ├── index.js                # Express + WebSocket server
│   ├── game-server.js          # Game state, rooms, matchmaking
│   ├── db.js                   # Database connection (Supabase/Firebase/Postgres)
│   ├── routes/
│   │   ├── auth.js             # User accounts
│   │   ├── towns.js            # Town variations CRUD
│   │   └── scores.js           # Leaderboards, match history
│   └── models/
│       ├── user.js
│       ├── town.js
│       └── match.js
├── shared/
│   └── constants.js            # Shared between client/server
├── package.json
└── README.md
```

---

## Migration Notes

### Splitting the Monolith
The current file has natural module boundaries. Here's how functions map to the proposed structure:

| Current Function(s) | → Module |
|---------------------|----------|
| `fetchOSM`, `buildWorld`, `bldColor`, `isResidentialArea`, `addTree`, `addLamp` | `world.js` |
| `fetchElevation`, `getTerrainY`, `buildTerrain` | `terrain.js` |
| `createVehicle`, `spawnVehicles`, `enterV`, `exitV`, vehicle physics in game loop | `vehicles.js` |
| `togglePaintball`, `shootPaintball`, `shootAtScreen`, `firePaintball`, `updatePaintballs`, `createSplat` | `paintball.js` |
| `toggleEdit`, `setTool`, `placeAtScreen`, `deselectEdit`, save/load functions | `editor.js` |
| `setupDesktop`, `setupMobile`, `updateBtns` | `controls.js` |
| `updateMinimap`, `updateCompass`, `toggleMinimap` | `minimap.js` |
| `doGeocode`, fallback chain | `geocoding.js` |
| `ll`, `cfetch`, `ptInBuilding`, `esc` | `utils.js` |
| `initThree`, `gameLoop`, camera logic | `main.js` |

### State That Needs to Be Shared/Synced for Multiplayer
```
Per player (sync to all clients):
  - position {x, y, z}
  - rotation (yaw)
  - inVehicle (which vehicle ID or null)
  - animation state (walking, running, jumping)
  - health/score

Per paintball (broadcast on fire):
  - origin, direction, color, shooterID

Per splat (broadcast on hit):
  - position, color, surface type

Per town variation (database):
  - base location (lat, lon)
  - userEdits[] array
  - author, contributors, rating, play count
```

### Recommended Tech Stack
- **Frontend:** Vanilla JS + Three.js (keep it dependency-light), or migrate to Vite for dev tooling
- **Server:** Node.js + Express + ws (WebSocket library)
- **Database:** Supabase (free tier, Postgres + auth + realtime) or Firebase
- **Hosting:** Fly.io, Railway, or Render for the WebSocket server; Vercel/Netlify for static client
- **Auth:** Supabase Auth or Firebase Auth (supports anonymous + email + social)

---

## Known Issues & Tech Debt

1. **Fences not rendering** — OSM barrier query is in place but Bentonville has sparse `barrier=*` tagging. The rendering code works but needs areas with better data to test.
2. **Building collision is 2D only** — `ptInBuilding` checks x/z but not height. Paintballs can "hit" buildings at any altitude.
3. **Road segments are flat planes** — they approximate terrain but don't actually conform to slope angle (each segment is horizontal at its center elevation).
4. **No sound** — everything is silent.
5. **Edit mode crosshair conflict** — edit mode creates a separate `#edit-crosshair` div; paintball mode uses `#crosshair`. Works but messy.
6. **localStorage save keys** — changed from `streetcraft_*` to `kylitosway_*` in V6, breaking old saves.
7. **No object pooling** — paintball splats create new meshes each time; at 300 limit this is fine but would need pooling at scale.
8. **Mobile paint mode** — full-screen touch zone works but competes with joystick zone for touch events in edge cases.
9. **Camera in paintball mode** — right-click drag for camera rotation works on desktop but isn't obvious to users.

---

## How to Test

1. Open `streetcraft6.html` in Safari or Chrome
2. Type any address (e.g. "215 south liberty street harrison" or "1489 barberry lane bentonville")
3. Click GO → world loads with terrain, buildings, roads
4. WASD to walk, E to enter/exit cars, P for paintball mode
5. TAB for edit mode, M for minimap

No build step, no server, no install. Just open the HTML file.

---

*Document prepared Feb 9, 2026. Current version: Kylito's Way V6.*

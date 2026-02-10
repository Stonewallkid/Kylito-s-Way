# Kylito's Way - Development Map

## Current Version: V7

---

## Completed Features

### Core
- [x] Real-world map generation from OSM data
- [x] Address geocoding with fallback chain
- [x] Terrain elevation from Open-Meteo API
- [x] Chunk-based world streaming (300m chunks)
- [x] Chunk terrain elevation
- [x] Player movement (WASD, sprint, jump)
- [x] Third-person camera with auto-follow

### Vehicles
- [x] Sedan, SUV, truck types
- [x] Enter/exit vehicles
- [x] Driving physics with terrain following
- [x] Speedometer HUD
- [x] Collision with buildings

### Edit Mode
- [x] Place buildings (house, commercial)
- [x] Place roads
- [x] Place trees
- [x] Place fences, walls, hedges
- [x] Place parking lots with line markings
- [x] Select and modify objects (size, color, roof type)
- [x] Move/drag placed objects
- [x] Delete objects
- [x] Top-down bird's eye view for editing
- [x] Preview mesh for placement
- [x] Save/load to localStorage
- [x] Export/import JSON

### Paintball Mode
- [x] Projectile physics with gravity
- [x] Splat effects on surfaces
- [x] Color picker (8 presets + custom)
- [x] Desktop: mouse aim + click to shoot
- [x] Mobile: tap to shoot

### UI/HUD
- [x] Minimap (expandable)
- [x] Compass with heading
- [x] Current address detection
- [x] Elevation display
- [x] Chunk loading status
- [x] Mobile touch controls (joystick + buttons)

---

## In Progress

### Performance
- [x] Predictive chunk loading (load ahead of movement direction)
- [ ] Terrain elevation consistency (chunk vs main terrain)

---

## Planned Features

### Phase 1: Performance & Polish
- [x] Predictive chunk loading based on velocity
- [x] LOD (Level of Detail) for distant buildings (basic - skip windows/roof detail beyond 200m)
- [ ] Instanced rendering for trees/lamps
- [x] Object pooling for paintballs/splats
- [ ] Web Workers for OSM parsing

### Phase 2: Backend Infrastructure
- [ ] Node.js server for API proxying
- [ ] Redis cache for OSM tiles
- [ ] PostgreSQL + PostGIS for world data
- [ ] User accounts (Supabase Auth)
- [ ] Cloud save for worlds/edits

### Phase 3: Multiplayer Foundation
- [ ] WebSocket server setup
- [ ] Player position sync
- [ ] Player avatar visible to others
- [ ] Name tags above players
- [ ] Basic lobby/room system

### Phase 4: Combat System
- [ ] Health/lives system
- [ ] Hit detection (server authoritative)
- [ ] Respawn at spawn points
- [ ] Kill feed
- [ ] Scoreboard
- [ ] Match timer

### Phase 5: Game Modes
- [ ] Free-for-all deathmatch
- [ ] Team deathmatch
- [ ] Capture the flag
- [ ] King of the hill

### Phase 6: Community Features
- [ ] World variation voting
- [ ] Submit edits for review
- [ ] Leaderboards
- [ ] Player profiles
- [ ] Friends list
- [ ] "Home town" claiming

### Phase 7: Polish & Content
- [ ] Sound effects (footsteps, engine, shots, splats)
- [ ] Day/night cycle
- [ ] Weather effects
- [ ] More vehicle types (motorcycle, bike)
- [ ] Building interiors
- [ ] NPC pedestrians/traffic
- [ ] Better building textures

---

## Known Bugs

- [ ] Terrain elevation sometimes inconsistent between chunks
- [ ] Fences not showing in areas with sparse OSM barrier data
- [ ] Mobile paint mode touch conflicts with joystick in edge cases
- [ ] Camera rotation in paintball mode not obvious to users

---

## Technical Debt

- [ ] Split monolithic HTML into modules
- [ ] Add TypeScript for type safety
- [ ] Add build system (Vite)
- [ ] Add unit tests for coordinate math
- [ ] Document coordinate system thoroughly

---

## Ideas / Backlog

- [ ] VR support
- [ ] AR mode (overlay on camera)
- [ ] Replay system for matches
- [ ] Custom game modes (editor)
- [ ] Mod support
- [ ] Map sharing via URL
- [ ] Screenshot/clip capture
- [ ] Twitch integration
- [ ] Tournament system

---

## Session Log

### 2026-02-09
- Created V7 with chunk-based streaming
- Added terrain elevation for chunks
- Fixed terrainMinElev normalization
- Fixed chunk terrain overlap detection
- Added debug logging for elevation
- Set up GitHub repo + Pages
- Created DEVMAP.md
- Added predictive chunk loading
- Fixed duplicate variable declaration bug (game wouldn't start)
- Added object pooling for paintballs and splats
- Added basic LOD - skip window/roof details for distant buildings
- Added debug panel (F3) - shows FPS, chunks, terrain info, position, velocity, draw calls

---

## Notes

### Coordinate System
```
World:  X = east(+)/west(-)    Z = south(+)/north(-)    Y = up
Chunk:  300m x 300m tiles, coords like (0,0), (1,0), (-1,2)
```

### API Rate Limits
- Overpass: No auth, shared public server (be nice)
- Nominatim: 1 req/sec
- Open-Meteo: Generous, no auth

### Key Constants
```javascript
RADIUS = 500        // Main terrain radius (meters)
CHUNK_SIZE = 300    // Chunk size (meters)
CHUNK_LOAD_DIST = 1 // Load chunks within this distance
CHUNK_UNLOAD_DIST = 3 // Unload chunks beyond this
```

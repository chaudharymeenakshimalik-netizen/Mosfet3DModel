# X-Z Gantry Screw-Tightening Digital Twin

React 18 + react-three-fiber digital twin of the MOSFET + heatsink tightening line (3 jobs per batch, 7 automatic machine stages, 12 screws per batch).

The automatic machine track (27.6 s at 1.8 s/screw) is simulated with a separate operator workflow: load three assemblies (6.0 s), confirm start/clear the guarded area (1.5 s), then unload and sort the completed batch (4.5 s). The worker animation transfers each part from the staging bench into the three Y-table nests, waits outside the guard during machine motion, and carries verified parts to the green OK or red NOK bin. The operator body is collision-constrained to the front-side floor corridor; shoulder/elbow/wrist IK tracks the handled part while keeping the hands above the work surface and fixture approach plane. Loading and sorting use staged approach, grip, lift/carry, lower, release, and retract motion with animated fingers and wrist pose. For a single table this is a 39.6 s repeating cycle; the Excel takt model assumes overlapping operator work at a separate, safety-interlocked load/unload station. The dashboard recalculates the theoretical parallel takt from the selected shift target and OEE; at the default 2,800 good parts/shift, required batch takt is 26.23 s and the 27.6 s automatic machine cycle is flagged over takt. The spreadsheet's 30.6 s takt corresponds to 2,400 parts/shift, not its displayed 2,800 target.

**Engineering note:** The Three.js assembly is a nominal-scale visual digital twin, not a release-ready manufacturing CAD model. Physical dimensions, tolerances, torque limits, safety distances, and any separate/interlocked operator load-unload access needed for parallel handling must be confirmed against approved machine drawings and a safety assessment before construction.

## Run
```bash
npm install
npm run dev        # Vite on :5173 + Express/WebSocket telemetry on :4000
```
Open http://localhost:5173. The HDRI (`Environment preset="warehouse"`) and the HMI text font load from a CDN, so the first load needs internet.

## Dependencies
three, @react-three/fiber, @react-three/drei, zustand, framer-motion, lucide-react, tailwindcss (+ postcss, autoprefixer), vite, express, ws, cors.

## Layout
- `src/store/useSimulationStore.js`  stage machine, `computePose(stage, t)` (every animated value), worker stalls, KPIs
- `src/components/Scene.jsx`         Canvas, lighting, HDRI, OrbitControls, camera presets, floor
- `src/components/GantryMachine.jsx` frame, acrylic, light curtain, X-Z gantry, spindle, cable carriers, HMI, e-stop, signal tower
- `src/components/Conveyor.jsx`      Y-table, fixtures, clamps, poka-yoke beams, parts, OK/NOK bins
- `src/components/Feeder.jsx`        bowl feeder, pedestal, feed tube + travelling screw
- `src/components/Worker.jsx`        procedural operators (0-5 per station)
- `src/components/DashboardUI.jsx`   KPI bar, camera/overlay panel, controls, 8-stage monitor
- `server/index.js`                  REST: GET /api/status, /api/stats, /api/telemetry/history, POST /api/telemetry; WS: /ws

Per-frame values live in `live` (not React state), so the 3D scene never re-renders on every tick.

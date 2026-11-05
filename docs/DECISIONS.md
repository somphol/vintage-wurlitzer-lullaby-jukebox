# Decisions

Short architecture decision records. Each one explains a constraint that is easy to break by accident.

## ADR-001 — No build step

**Decision:** ship `index.html`, `style.css`, `script.js` and one JSON file; load Three.js from a
pinned CDN import map.

**Why:** the product is a single static page on GitHub Pages. A bundler would add a toolchain, lockfile,
CI cache and release artifact for no user benefit.

**Consequence:** no npm dependency, no JSX, no tree-shaking. The import map must stay in sync with the
Three.js version used by the addons.

## ADR-002 — Pin Three.js 0.169.0 exactly

**Decision:** `three@0.169.0` for both `three` and `three/addons/`.

**Why:** the composer, RoomEnvironment and OrbitControls APIs move between releases; an unpinned CDN
can break the page without a commit.

**Consequence:** upgrades are deliberate: change both URLs, rerun the browser suite, capture new
screenshots.

## ADR-003 — Procedural cabinet, no model files

**Decision:** build the Wurlitzer from Three.js primitives, an extruded arch and canvas textures.

**Why:** a GLB would be a large binary, a loading dependency, a licence question and impossible to
diff. Procedural geometry keeps the repository text-only and lets the scene react to audio.

**Consequence:** the model is an evocation, not a replica. Changing proportions means editing
coordinates in `buildCabinet`.

## ADR-004 — Two audio engines, one graph

**Decision:** synth and archive streams share `musicBus`, EQ, compressor, master and timer gain.

**Why:** the sleep timer, mute, EQ and analyser must apply to both. Separate graphs would duplicate
that logic and drift.

**Consequence:** streams must be CORS-enabled to pass through Web Audio. The manifest validator
enforces https and provenance; failures are surfaced, not silently hidden.

## ADR-005 — Smoothstep timer fade, not linear

**Decision:** `fadeGain` uses `x*x*(3-2x)` over 120 s.

**Why:** a linear fade has an audible corner at the start and a hard stop at the end; smoothstep is
perceptually gentler and the midpoint is exactly 0.5, which is easy to test.

**Consequence:** the engine test asserts 0.5 at 60 s; changing the curve requires updating docs, tests
and the self-test.

## ADR-006 — 3D rack shows A/B only

**Decision:** the physical paper rack renders the eight synth records (A-1…B-4). The DOM rack renders
16 slots including archive C and empty D.

**Why:** eight canvas textures stay legible at the cabinet's screen size. Sixteen would shrink the
type below readability and add GPU memory for no interaction benefit.

**Consequence:** archive streams are selected from the DOM or the C keypad code, not by clicking a 3D
strip. This is documented in the help copy.

## ADR-007 — Engine markers and `new Function` tests

**Decision:** pure logic lives between `/* ENGINE:START/END */` and is re-evaluated in Node.

**Why:** a single-file static app still deserves fast, dependency-free unit tests. The marker boundary
is the contract.

**Consequence:** DOM, window, THREE, fetch and `new Date()` are forbidden inside the block; static
tests enforce it. Export timestamps are passed in as ISO strings.

## ADR-008 — Mock keychain in the browser harness

**Decision:** launch Chrome with `--use-mock-keychain` and `--password-store=basic`, temporary profile.

**Why:** the development machine's Chromium otherwise asks macOS Keychain for encryption keys on every
launch, which is noisy and blocks automated runs.

**Consequence:** no credential is read. The flags must stay in `tests/lib/harness.mjs` and any
screenshot script.

## ADR-009 — Settings are normalised, not trusted

**Decision:** one `normalizeSettings` gate clamps ranges, rejects unknown languages, deduplicates
favourites and caps lengths.

**Why:** `localStorage` is user-editable and survives version changes. Validation also prevents a
corrupt export from poisoning the UI.

**Consequence:** new settings must be added to `DEFAULT_SETTINGS`, `normalizeSettings`, the state schema
and the export docs.

## ADR-010 — Bilingual copy is handwritten and symmetrical

**Decision:** EN and TH dictionaries have identical keys; Thai copy is written for the product.

**Why:** machine translation reads badly in a sleep product, and asymmetrical dictionaries produce
silent English leaks.

**Consequence:** a static test compares sorted key arrays; every new UI string needs both languages.

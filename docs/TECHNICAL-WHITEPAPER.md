# SimFarm V0: technical and research notes

## Scope and evidence labels

This note records an engineering review of the current browser reconstruction. It
distinguishes four kinds of statements:

- **Source verified** means the behavior is present in the checked-in JavaScript
  and can be exercised by the project's harnesses.
- **Historically documented** means a contemporary SimFarm manual or catalog
  record describes it.
- **Inference** means the conclusion follows from multiple source facts but is
  not directly stated by a historical source.
- **Proposal** means a possible future design, not a feature in this build.

The project is a JavaScript reconstruction of the 1993 Maxis farming game. It
draws the original-style 640 × 480 interface on an HTML Canvas and has no game
server or DOS emulator. It reads and writes native `.SFM` / `.SSM` byte layouts.
The reconstruction is not a claim of complete or pixel-perfect parity.

## Research record

### Internet Archive

Advanced Search for `title:"SimFarm"` returned 22 records. The DOS catalog
entry was verified dynamically as
[`msdos_SimFarm_1993`](https://archive.org/details/msdos_SimFarm_1993). Its
[metadata endpoint](https://archive.org/metadata/msdos_SimFarm_1993) identifies
Maxis Software Inc. as publisher, Leaping Lizard Software, Inc. as developer,
and 1993 as release year. The entry contains a 1.2 MB ZIP, emulator screenshots,
and metadata; it is a software preservation record, not a manual. No game image
or executable was downloaded for this review.

The catalog search also found Maxis's German DOS manual and quick-start card:
[`simfarm-handbuch`](https://archive.org/details/simfarm-handbuch) and
[`simfarm-schnellstartkarte`](https://archive.org/details/simfarm-schnellstartkarte).
Their metadata describes 1993 German-language documentation. The handbook's
public OCR was consulted for the map, finance, and futures descriptions. It
describes a Map window with overview and land information, a Balance Sheet
listing crop, machinery, land, futures, and livestock income, and futures as a
way to manage weather risk. The OCR is imperfect; details not legible in it are
not treated as historical fact. The handbook does not establish insurance
coverage rules, and a text search failing to find the word “insurance” is not
proof the feature was absent from every edition.

### Other public implementations

GitHub search surfaced [dkim0419/SimFarm](https://github.com/dkim0419/SimFarm),
a 2014 C# classroom project. Its description and file listing identify a
separate grid-based application, rather than a reconstruction of Maxis SimFarm.
It has no implementation or save-format work suitable for direct adaptation.
Search did not surface a licensed, compatible decompilation or reconstruction
that would improve this code. No third-party code was copied and no dependency
was added.

### Current-source findings

- **Source verified:** the fixed farm viewport uses 16-pixel map cells; farm
  drawing, placement, hit tests, crop footprints, disaster overlays, and the
  cursor all share those assumptions. The Map window supplies an overview and
  analytical views, and its pointer navigation selects a camera origin. Arrow
  controls and edge scrolling also move the camera.
- **Source verified:** harvest contracts (futures) have price settlement and
  balance-sheet entries. Weather and disaster routines damage crops. There is
  no insurance policy, premium, claim, or payout state or flow in `src/`.
- **Source verified:** land parcels have ownership state, and buying and selling
  land are implemented. There is no conservation enrollment or recurring
  contract state.
- **Inference:** adding zoom would require coordinated changes to rendering,
  map-cell transforms, object placement, overlays, and pointer hit testing.
  Because no primary reference reviewed here establishes original zoom and the
  current Map window already gives broad navigation, zoom is not justified as a
  fidelity-preserving correction.
- **Inference:** insurance or conservation contracts could be designed as new
  gameplay, but storing them in unused native save bytes would be unsafe without
  proving those bytes are unused across real saves. Adding them only to runtime
  state would lose them on save/load. They are therefore not introduced as
  disconnected or nonportable mechanics.

## Architecture and state flow

The build script concatenates the 16 ordered sections in `src/sections.json`
into `game.js`. The sections share one private closure, so state and helper
functions are not independently imported modules. `index.html` provides the
canvas and save-file input. `src/16-input-and-calendar.js` handles pointer and
keyboard input, calendar advancement, and the animation loop. Rendering is in
`src/10-rendering.js`; map and irrigation logic is in
`src/07-map-and-irrigation.js`; pointer-to-cell tools are in
`src/13-land-and-tools.js`. Finance and reports live in
`src/05-interface-and-finance.js`, inventory transactions in
`src/06-market-and-inventory.js`, harvest settlement in `src/08-machinery.js`,
and native save parsing/writing in `src/11-scenarios-and-saves.js`.

The saved world is a byte array interpreted using offsets in
`data/save-states.json`. Crop definitions are decoded into `data/crops.json`.
The project preserves byte-level formats and reconstructed native behavior;
therefore unknown save bytes cannot safely be claimed as free extension space.

## Camera, map coordinates, and zoom decision

The farm map is a 96 × 96 cell world rendered through a fixed Edit viewport.
The viewport shows a cropped region; the `camera` coordinates are its origin,
with a historical `x + 1` descriptor convention reflected in the renderer and
input handlers. The Map overview converts two overview pixels to one map cell
and clamps the selected origin to the area supported by the Edit viewport.
Separate arrow and edge-scroll controls provide incremental panning. Tests in
`tests/gameplay.cjs` cover overview extremes, inclusive boundaries, eight map
modes, dragging, moved windows, scaling, cancellation, and that navigation
does not charge farm funds.

There is no zoom level in the current camera model. Zoom anchoring would need a
new transform shared by rendering and all world interactions; adjusting only
the canvas scale would make the displayed cell disagree with the cell selected
for planting, construction, or object interaction. The historical handbook
supports overview navigation, not a zoom feature. The implementation decision
is to retain fixed cell size and the established Map window. A future zoom
proposal should first define one canonical screen-to-world transform and test
its inverse, cursor anchoring, limits, and clamping before changing rendering.

## Economy and accounting

The current economy is split across native-format counters and runtime cash.
Purchases, sales, land transactions, bank operations, and seasonal tax handling
update these structures at different call sites. Crop harvest settlement in
`src/08-machinery.js` calculates value from crop quantity, commodity price, and
harvest quality. Futures settlement records the difference between contracted
and market value, posts the contracted crop sale, updates taxable sale income,
and credits cash. The Balance Sheet derives its revenue and expense columns
from native counters in `src/05-interface-and-finance.js`.

This review did not establish a safe, isolated accounting correction against a
reference save or original executable. In particular, native counters with
different units or reporting semantics should not be “fixed” by assuming they
must match cash one-for-one. The existing test fixtures exercise eight scenario
save round trips and many transaction flows, but they do not represent every
farm history. Any future accounting correction should state its unit and
transaction invariant, reproduce the issue in a fixture, and compare the
resulting cash, ledger, and save bytes.

## Crop loss, insurance, and conservation proposal

The current game has crop losses from drought, flood, frost, tornado, locust,
and other disasters, plus futures contracts that manage price risk. The source
does not show a crop-insurance mechanic. The current code also does not model a
conservation program. No gameplay or regulation is added by this review.

### Proposed optional extension format

If these mechanics are pursued, keep the native `.SFM` payload byte-for-byte
intact and add an explicit, versioned companion format (for example `.SFMX`): a
small header, version, original save bytes, and validated extension data. Old
`.SFM` files would load with empty extension state; `.SFMX` round trips would
preserve it. Do not write speculative data into unverified reserved offsets.
This option must be validated against real legacy saves before implementation.

### Proposed fictional crop protection

An optional policy could cover one named crop on selected farm-owned fields for
one growing season. At enrollment, record expected yield and a reference crop
price. Charge 5% of expected crop value as the premium. At harvest, compute
loss as `max(0, (expected yield - actual yield) × reference price)`, subtract a
deductible equal to 20% of expected crop value, and pay no more than 50% of
expected crop value. Only one claim per insured field-season may settle. No
payout occurs when actual yield meets or exceeds expected yield; invalid,
negative, or non-finite values are rejected. The game must record premium
expense, claim payout, and policy state separately. This is a fictional game
rule, not an implementation of federal crop-insurance regulations.

Required tests would cover no loss, partial and total loss, zero coverage,
invalid policy values, unaffordable premium, payout cap, duplicate claim,
harvest ordering, save/load round trip, and old-save defaults. Deterministic
calculation should be separate from rendering.

### Proposed conservation contracts

USDA FSA's current CRP description says eligible acreage is placed in
resource-conserving cover in exchange for annual rental payments, with typical
contracts of 10–15 years. A fictional SimFarm version could allow enrollment
of one owned parcel that has no active field or placed building. It would
replace the parcel with non-plantable conservation cover for three simulated
years and pay 100 credits at the start of each simulated year. Selling an
enrolled parcel would be blocked. Voluntary withdrawal would end future
payments and charge one annual payment as a cancellation fee. Store the
enrollment year, remaining term, next payment year, and parcel identity. These
short terms and fixed payments are game abstractions, not CRP eligibility,
rates, or regulation. The FSA overview is linked below.

Because native save bytes and compatibility are central to the reconstruction,
these additions should wait for a tested versioned extension design. The
`.SFMX` container above is a concrete alternative that can preserve old saves
without conflating new gameplay state with native bytes.

## Save compatibility and testing

`src/11-scenarios-and-saves.js` reads and writes the fixed state byte array and
applies runtime defaults for process-only data. `tests/save_compat.cjs` checks
round trips for all eight included scenario fixtures, saved speed/pause cadence,
serialized season, process-only state, malformed imports, local cache failures,
and export failures. This supports compatibility with those fixtures only; it
does not prove every historical `.SFM` variant is compatible.

`npm ci`, `npm run format:check`, `npm test`, and `npm run build` passed. The test
command passed all nine suites, including generated-bundle consistency,
JavaScript syntax, save compatibility, field placement, machinery, weather,
crop loading, lifecycle, window behavior, and basic map gameplay. The first
format-check attempt failed because the local `prettier` executable was absent;
installing the locked development dependencies resolved it. Interactive
browser testing was not performed as part of this headless review.

## Limitations and next work

1. Obtain and test additional legally accessible native save fixtures from
   multiple versions and platforms before making broader compatibility claims.
2. Build an instrumented transaction harness that records cash and each ledger
   before and after representative purchases, sales, futures, and taxes.
3. If zoom is desired as a new nonhistorical option, implement a shared camera
   transform and migrate every drawing and input consumer together.
4. Implement optional insurance or conservation only after the versioned save
   extension format has a migration policy, user-facing explanation, and full
   regression coverage.
5. Run a browser smoke test on desktop and touch layouts after camera or input
   changes; the Node harnesses do not replace visual play testing.

## References

- Maxis Software Inc., _SimFarm – Deutsch – Handbuch_ (1993), Internet Archive
  item metadata and OCR: [item record](https://archive.org/details/simfarm-handbuch),
  [metadata API](https://archive.org/metadata/simfarm-handbuch),
  [OCR text](https://archive.org/download/simfarm-handbuch/SimFarm%20-%20Deutsch%20-%20Handbuch_djvu.txt).
- Maxis Software Inc., _SimFarm – Deutsch – Schnellstartkarte_ (1993), Internet
  Archive: [item record](https://archive.org/details/simfarm-schnellstartkarte).
- Internet Archive, _SimFarm_ DOS software record:
  [catalog item](https://archive.org/details/msdos_SimFarm_1993),
  [metadata API](https://archive.org/metadata/msdos_SimFarm_1993).
- USDA Farm Service Agency, [Conservation Reserve Program overview](https://www.fsa.usda.gov/resources/conservation/conservation-reserve-program)
  (accessed 2026-10-19). Used only to describe the real program's broad purpose
  and typical term; no detailed regulation is encoded in this game.
- Separate 2014 classroom project: [dkim0419/SimFarm](https://github.com/dkim0419/SimFarm).

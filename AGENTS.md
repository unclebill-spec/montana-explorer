# Montana Explorer — agent handoff

Static Leaflet map for a nurse household, the same shared app as the Kentucky, Tennessee, Massachusetts, Maine, Vermont and Wyoming Explorers (`app.js`, `areas.js`, `perm.js`, `profiles.js`, `extras.js`, `style.css` are byte-identical copies of `/workspace/kentucky/explorer/*`; another worker edits them there, so run `scripts/sync_shared.sh` right before every build/publish and log any shared-file edit in KY `explorer/AGENTS.md`). `sw.js` differs only in its cache prefix (`mtx-`). Montana-specific settings live in `explorer/build.py` `STATE` (ported from VT by `scripts/port_build.py`, idempotent, marker PORT_ST).

Live: https://unclebill-spec.github.io/montana-explorer/ · repo unclebill-spec/montana-explorer · progress log: `/workspace/montana/STATUS.md` (newest first, ET).
Montana and Wyoming share ONE code base: every script in `scripts/` is identical in both folders and picks the state from the folder it runs in (`scripts/common.py` ST).

## Caps (Bill, Oct 3 2026: "add montana and wyoming, increase the housing price to 550k for both")
5+ acres $300k–$550k; 1+ acre 3bd/2ba < $550k; near-hospital 1,600+ sqft 3bd/2ba < $550k (townhomes/condos OK, good condition, ≤ 10 min of a hospital with a 10+ bed ER). No cabin category.

## Blocks
56 counties (`scripts/common.py` -> `data/statewide/raw/mt_blocks_500k.zip`).

## Data pipeline (run from /workspace/montana; pandas scripts use /workspace/kentucky/.venv/bin/python, the rest /usr/bin/python3)
1. `scripts/hospitals_research.py` (CMS + state trauma list + border Level I/II) -> `data/hospitals.json`; `scripts/layers.py` -> block CSVs, schools (CCD 2024-25 + SEDA), RN wages (O*NET/BLS May 2025; no employment counts: BLS API daily limit + bls.gov downloads blocked).
2. `scripts/appeal_build.py` (OSRM drives, cached) -> appeal shading; `scripts/er_beds.py` -> `data/hospital_er_beds.json` (10+ bed ERs for the near-hospital rule).
3. Climate normals -> `data/clim.json`; activities `data/osm/wd_act.py` (Wikidata) + `data/osm/wp_cat_act.py` (waterfalls/trails); colleges `data/osm/nces_post.json`.
4. Compare areas: `compare/areas_build.py` + `land.py` -> `data/areas.json` (Billings, Missoula, Great Falls, Bozeman, Butte).
5. Homes: `scripts/zsearch.py` (Zillow county searches at the $550k caps) -> `data/zsearch/`; `scripts/listings_build.py` (PER_COUNTY 10) -> `listings.json` + `listing-photos/` (log `data/lb.log`); `scripts/bargains.py` (state Zillow comps + national ZHVI from /workspace/kentucky/data); `scripts/top_lists.py`.
6. Permanent RN jobs: `scripts/perm_jobs.py` -> `data/perm_jobs.json` (reuses the KY readers in /workspace/kentucky/scripts/perm_jobs_collect.py via source rewriting). Sources: Intermountain Workday imh (St. Vincent, St. James, Holy Rosary; city queries), Logan Health Workday, Benefis Workday BHS, Community Medical Center (Lifepoint Oracle ORC, locationId 300000006719143), St. Peter's (Oracle ORC CX_2), Bozeman Health (Workday; it answered 403 to the box on Oct 3, so the script falls back to `data/bozeman_websearch.json`, 13 openings found by web search — refresh that list by web search, never work around the block).
7. Travel RN jobs: helpers in `/workspace/tj_mt` (Vivian + Advantis), then `scripts/travel_jobs.py` -> `data/travel_jobs.json`.
8. Phase 4: `data/airports/airports.py`; `data/attractions/wd2.py` + `make_attractions.py` (MANUAL_LL from Nominatim); Crexi `data/forsale/crexi_list.py`, `st_forsale.py`, `make_forsale.py`; thumbnails `explorer/fetch_thumbs.py` (after a build).
9. Border items: `/workspace/border` (`scripts/static.py MT WY`, `scripts/make_border.py MT WY`), copy `out/MT.json` -> `explorer/border.json`. Montana's north side is Canada (BC/AB/SK): no Canadian homes/jobs/schools, only the US neighbors (ID, ND, SD, WY).
10. Ski areas + peaks: `/workspace/mtn/scripts/make_state.py MT` -> `explorer/mtn.json` + `img/mtn/` (see /workspace/mtn/PROGRESS.md). Ticket prices come only from skiresort.com.
11. Publish: `sh scripts/sync_shared.sh && cd publish && PATH=/usr/bin:$PATH ./publish.sh -m "msg"` (flock, pull, build, minify, secscan of dist + full history, push, waits for Pages).
12. Tests (state from the folder): `perf/smoke.py BASE TAG`, `perf/test_homes.py`, `perf/test_perm.py`, `perf/test_p4.py`, `perf/loadtime.py URL`, `perf/sw_check.py URL...`; screenshots in `perf/shots/`.

## Known gaps (Oct 3 2026)
- RN employment counts are null (BLS limits). County history layer is empty (no wiki_history.json; fetch_history.py not run).
- HCA (researched last, Oct 3 6:35 PM ET by web search): HCA has no hospitals in Montana or Wyoming (careers.hcahealthcare.com U.S. locations lists neither); its nearest, Eastern Idaho Regional Medical Center in Idaho Falls, is outside the 15-mi border band. Nothing to add.
- Not collected: Billings Clinic (careers site blocks fetches), Providence St. Patrick (Missoula) + St. Joseph (Polson) (JS-only site), small critical-access hospitals.
- Trauma levels: Billings Clinic + St. Vincent Level I, Benefis + St. Patrick Level II; Logan Health Kalispell's level is ambiguous in the sources. Sacred Heart Spokane is missing from CMS border data.
- Peaks list lacks Electric Peak, Trapper Peak, Sacagawea, Hyalite, Jumbo, Square Butte, Sleeping Giant (no article/coordinates). No ticket price: Bridger Bowl, Blacktail, Showdown, Teton Pass. Near-hospital homes thin (60).


### 50+ acre lots under $250k (Oct 4, 2026 ~10:31 AM ET, big-land worker)
- Black-star layer `big-land` (50+ ac, < $250k, land or home), "50+ ac" button, Map key row, card; shared code from the KY explorer (see KY explorer/AGENTS.md, same date). build.py (marker BIGLAND) merges `/workspace/montana/bigland.json`.
- Refresh: `/usr/bin/python3 /workspace/bigland/bigland.py MT --refresh` before build/publish (keeps the old file if Zillow blocks). Notes: /workspace/bigland/PROGRESS.md.

### Caves and waterfalls on the property (Oct 4, 2026, cave/falls worker)
- Bill, Oct 4 2:12 PM: "add any properties that have a cave or waterfall to the maps, have a small waterfall for the waterfall and a small bat for the caves, the flying type of bat".
- Layer `cvf` (categories `cave` / `falls`; `cf` = kinds, `cfq` = the listing's own words): flying-bat pin for caves (also used when a listing has both), waterfall pin, groups, "Cave" / "Falls" buttons (hidden when the map has none), Map key row, card. A listing already on the map in another category keeps it and gets `cf`/`cfq` (shows under both). Shared code from the KY explorer (KY explorer/AGENTS.md, same date). build.py (marker CAVEFALLS) merges `/workspace/montana/cavefalls.json`; border items come from make_border.py (`CFCAP`).
- Refresh: `/usr/bin/python3 /workspace/cavefalls/cavefalls.py MT --refresh` before build/publish (Zillow keyword search + listing text check + dedupe of the same land listed twice; keeps the old file if Zillow blocks; `--offline` re-checks from the cache). Notes: /workspace/cavefalls/PROGRESS.md.

## Target stores layer (Oct 7, 2026 ~5:15 PM ET, Target worker) — shared app.js + new shared target_build.py + build.py hook
- Bill: a Target stores layer on all 11 maps. Every Target in the state + stores within ~15 mi outside the state line (border rule: `bst`/`bco`/`bmi`, no `county` field, so never in county stats).
- **Data:** `/workspace/target/` (log: PROGRESS.md). Source = Target's own store directory (`target.com/store-locator/store-directory/<state>`, the official per-state list and count) + each store's page (`target.com/sl/<slug>/<id>`: JSON-LD geo, address, regular hours, phone, services). OpenStreetMap (Overpass, brand:wikidata=Q1046951) is only used to find neighbor-state stores near the line and as a count cross-check. Scripts: `scripts/fetch_dir.py` → `fetch_sl.py <STs>` → `parse_sl.py` → `make_target.py <STs>` (→ `out/<ST>.json`, copy to `<state>/explorer/target.json`) → build → `scripts/drives.py <statedir>` (OSRM free-flow minutes from each listing to its fastest of the 3 nearest stores, written into target.json `drv`, cache `cache/osrm.json`) → build again.
- **Build:** `explorer/target_build.py` (shared, md5-identical everywhere; in each sync_shared.sh list). build.py line right after `border_build.add(data, HERE)`: `import target_build; target_build.add(data)` (re-apply: `/workspace/target/patch/hook_build.py <statedir>`). Writes `K.targets` (whole rows in core.js, ~250 B each) + `K.meta.target`, and `nearby.tg` on every property (OSRM minutes when `drv` matches the listing id + spot, else straight-line × 1.3 "approx"). No target.json → no-op.
- **App (block "Target stores" before `function kyxMtn`; re-apply `/workspace/target/patch/patch_app.py app.js`, idempotent, marker `function kyxTarget(`):** icon `G.tgt` (small red bullseye), layer key `tgt` (on by default, icons from `FULL.tgt` = 9, red faint dots below, grouped like the other kinds), right-stack button "Target" (also in the landscape wheel; shows in Bill's P1–P4 and Anna mode — profiles don't filter it), Layers row, Map key row, search ("Target …"), card `R.target` (name, address, phone, store number, regular hours, services, target.com link; border stores get the 🧭 state tag), share `#target=<store number>` (index.html#…; no share page), Back stack like every card, property cards' Nearby row "🎯 Target · ~N min drive". `BST_NAME` gained the western states (border tags now say "South Dakota" instead of "SD").
- **Test:** `/usr/bin/python3 /workspace/target/test_target.py BASE TAG [anna]` (412×915 touch + 1280×720: button, solo on/off, zoom tiers + groups, pin tap, share links in-state + border, Nearby link + ‹ Back, Bill profile, Anna mode when the map has it, wheel over the column, 3 whole pills, no console errors). Screenshots `/workspace/target/shots/`.
- **Refresh:** Target opens/closes stores rarely; rerun the scripts above (fetch_sl.py only downloads pages not in cache/sl/; delete a page to refetch it).

## Active filter on top: kyxFoc (Oct 7, 2026 ~9 PM ET) — shared app.js/style.css
- The active right-side button / Anna button / open Top 10 list draws its pins 1.4x larger (groups 1.15x) and above everything; other pins (trauma, airports, cities...) go 0.72x and underneath while it is on. No filter = unchanged. Details: /workspace/kentucky/explorer/AGENTS.md "Active filter on top"; test /workspace/filterfocus/smoke.py BASE TAG.
- Oct 8, 2026: "Hospitals" right-side button (all hospitals incl. border, existing icons; trauma centers topmost; hidden in Anna). Details: KY explorer/AGENTS.md "Hospitals button: kyxHosp".

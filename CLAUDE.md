# CLAUDE.md — Bailey Engineering Standards & Design Process

Working brief from the Naveen + Eric sessions (Session E1 and follow-ups). Read before editing.
Purpose: (1) source material to build the **Bailey Design Standards** web page, (2) project memory for a
cloud Claude Code session working the standards repo. Save this into the standards repo as `CLAUDE.md`.

---

## Who & context
- **Naveen Vinta** — President & CEO, Bailey Specialty Cranes & Aerials (Muskego, WI). Sole NetSuite admin + sole SuiteScript author. Drives this initiative.
- **Eric** — VP Engineering (walked the current process).
- **Tony, Keegan** — mechanical engineers. Co-author the standard; do not hand it down.
- Company on **SolidWorks PDM Standard** (no PDM/vault API). NetSuite account 5712744, subsidiary 2, single location. `baileycranes.com` is a .NET site; internal apps run React/TS + Supabase on Railway.

## Core problem
Too much rework, caught too late at assembly. Plan: map **As-Is → To-Be**, bridge in **phases**, and replace
tribal knowledge with an **enforced standard**.

---

## Design process (As-Is)
- **Stage 0 — Concept / fit-form-function.** Rough "Lego-block" massing; how big does it need to be. No part numbers (a few legacy parts).
- **Stage 1 — Calculations / physics.** Stress, strain, loads vs service factor (≥ 2.75 for a grade). Material ladder Gr.50 → Gr.80 → Gr.100, then thicken. Loads cascade platform → base, recomputed by hand. Single-body FEA to fine-tune; multi-body for full assembly. Tools: spreadsheets + manual moment calcs; FEA owned but underused.
- **Stage 2 — Part numbers + drawings.** Assigning the part number is the gate 1 → 2. Detailed drawings with every mfg dimension. Vendors set on the drawing and in SolidWorks metadata → NetSuite via CAD link. Design review, then release.
- **Beyond Stage 2:** manuals, schematics, electrical.

## Challenges with the As-Is
- Interference check exists in SolidWorks but is run manually and rarely.
- No FEA report required before a stage advances.
- Spreadsheets disconnected; load cascade is manual, not linked formulas.
- No automated check that a drawing meets a standard before release.
- No hold-point sign-off — fabrication leans on shop-floor tribal knowledge.
- No release gate (Standard; a gate needs PDM Pro).
- Manual dimensioning — easy to miss a hole on a complex part.
- Design vs manufacturing dimensioning conflict — center-relative vs ordinate-from-corner; round vs rectangular zero-position.

## To-Be (target state)
- Every drawing passes an automated standards check before release, enforced at a PDM-Pro release gate that stamps **Meets / Does Not Meet**.
- The quick-win checks become systematic, not ad hoc.
- Materials from one standardized list; design→mfg dimensioning converts automatically.
- SolidWorks/PDM stays system of record; purchasing/production/Naveen read status via the Engineering Audit — parent assembly, vendor, last-touched-by, compliance — with no SolidWorks seat.
- Drawing generation (Hanomi or successor) is the final layer on a clean base.
- Net effect (HARDEN): the process runs on an enforced standard, not on any one engineer's memory.

## Iterative path (As-Is → To-Be)
- **4.1 Quick-win SOPs now (no software):** mandatory interference check before release · FEA report before Stage 1→2 · hold-point sign-off · whole-print review. Run hard 3–4 months.
- **4.2 Bailey Standard + native audit (parallel):** write the standard; audit native SolidWorks/PDM capability before custom code. GD&T lives here.
- **4.3 One-file proof of concept:** SolidWorks API — read metadata → browser → edit → write back → pass/fail. **Cap: one file, one write-back.**
- **4.4 Engineering Audit app:** NetSuite-native (see ADR below).
- **4.5 Hanomi / successor:** last, on a solid base.
- **Parallel track — Stage-1 calc:** parametric load-cascade automation; separate, must not block Stages 2–3.
- **Principles:** #1 risk is scope creep; decouple cheap wins from the app; co-author with Tony/Keegan; HARDEN.

---

## Deep dives (from the discussion)

### Interference detection
- **Native** in SolidWorks: `Tools → Interference Detection`. Set a **clearance tolerance** (e.g., flag anything closer than 1.5"). Returns interfering pairs + gap; can exclude components you don't care about.
- **UI vs API give the same answer** on a complete model — geometry is geometry. The **API's advantage is enforcement, not accuracy**: run it automatically, gate publish, or batch nightly.
- **Decision leaning:** start with **mandatory UI check + engineer sign-off** (4.1, no code); add API gating later.
- **OPEN:** on failure, **block** publish or **allow proceed + log**? (Leaning proceed-with-log, matching the compliance model.)

### GD&T (Geometric Dimensioning & Tolerancing)
- The language that locks a part's size **and** position/shape together (position, perpendicularity, runout, profile, flatness…), referenced to **datums**. Standard in aerospace/defense: **ASME Y14.5** (US), **ISO 1101**.
- **In SolidWorks:** add symbols on the drawing via `Insert → Geometric Tolerance` + datum references. SW displays it; it can't author it for you.
- **What it takes for our engineers:** (1) **knowledge** — learn the symbols and when to use each; (2) a **Bailey GD&T decision table** — which control for which feature type (holes → position; shoulders → perpendicularity; rotating parts → runout; mating surfaces → profile); (3) **annotation time** on every print.
- **Cost is time + discipline**, not software. Belongs in **4.2 (standard)**, not 4.1. Hanomi can auto-generate GD&T later from a clean 3D model.
- **Goal:** GD&T on all prints. Sequence: write the standard → train → enforce.

### Hold points
- Engineer marks **critical dimensions "HOLD"**, signs off; fabrication cannot proceed on those until checked to tolerance. Replaces "it's the print, I'm good" tribal knowledge with a signature.

---

## Bailey Design Standards — proposed outline (build the page from this)
Co-authorship: one living document, one chapter at a time; when a chapter changes, propagate cross-references
so there are **no conflicts or ambiguities**. Confirm/renumber before drafting.

1. **Scope & fundamentals** — what it covers (mechanical parts/assemblies/drawings; not electrical/software); stages where it applies; who enforces.
2. **Drawings & documentation** — what every drawing must carry (title block, material, surface finish, all dimensions, GD&T, critical-dimension call-outs, vendor/part #, revision); format/layout; the "missing dimension" checklist.
3. **GD&T** — decision table by feature type; datum selection; tolerance stack-up ↔ service factor.
4. **Materials & fasteners** — approved material list (Gr.50/80/100, aluminum grades); service-factor → material mapping; fastener grades, locking, torque.
5. **Tolerances & fits** — standard hole/shaft tolerances; critical ("HOLD") dimensions and bands; interference vs clearance.
6. **Manufacturing & fabrication** — assumed capabilities (press brake, plasma, welding, outsourced machining); DFM rules (min bend radius, wall thickness, edge treatment); hold-point sign-off.
7. **Assembly & fit** — interference checking (tolerance, method, pass/fail); sub-assembly hierarchy ↔ part numbering; where-used / BOM structure.
8. **Compliance & sign-off** — the automated validator (what it checks/logs; tamper-evident record); engineer attestation for non-automatable rules; release gate; revision control + audit trail.

---

## Engineering Audit — architecture (ADR 0001 summary)
- **NetSuite-native:** a Suitelet renders HTML beside the inventory-adjustment helper.
- **Data at rest in NetSuite** (item custom fields). NetSuite is store + display. **No live reads of a Windows box.**
- **SolidWorks pushes; NetSuite reads** — at publish via CAD link / a RESTlet writer (print-packet pattern).
- **One SolidWorks API:** drop the PDM API (absent on Standard). Properties via **Document Manager API**; geometry validation via the **full SolidWorks API on a seat**.
- **Easy half is already in NetSuite:** parent assembly = BOM/where-used; vendor/MPN = already synced.
- **Compliance:** automated validator computes a result; **non-blocking** (publish allowed non-compliant) but a **permanent log** is attached. Authoritative, not engineer-editable.
- **Standard constraints:** only machine-checkable rules auto-validate (judgment rules = attestation); a truly read-only in-file field isn't guaranteed on Standard → approximate with a **hash + append-only NetSuite record locked to a system role**, re-stamped each run (**tamper-evident, not tamper-proof**); auto-trigger at check-in is cleanest on Pro.
- **PDM Pro signal** recurs: true immutability, auto-trigger, and the release gate all point at Pro.
- **OPEN:** verify CAD-link sync scope; accept tamper-evident on Standard vs buy Pro.

---

## Infrastructure decisions
- **SOP / decisions page:** React/TS + Supabase on Railway at `engineering.baileycranes.com`. `/sop` built; `/audit` stub. Conference-room loop: say "go ahead and update" → insert `decisions` row (service-role key) → Supabase realtime → TV updates, no refresh.
- **Bailey Design Standards page:** start **static HTML on GitHub Pages** — one file, clickable TOC + scrollspy, Bailey styling; **separate URL** from the SOP. Migrate to a Railway app later only if collaborative editing is needed.
- **Cloud Claude Code (desired):** run headless in the cloud off the GitHub repo — laptop off — to edit/commit/deploy the standards page.

## Bailey design tokens (don't drift)
Red `#E30613` · ink `#14171A` · steel `#5B6670` · hairline `#E3E6EA` · panel `#F6F7F9`. Body/display: Arial.
Data/timestamps: monospace. Red table-header rows. Section numbers + decision ticks in red. Logo: the real
Bailey mark recolored to `#E30613`.

## Recurring principles
- **Native-first** — map what SolidWorks/PDM already do before building.
- **Standard → train → enforce**, in that order.
- **Scope discipline** — POC is one file, one write-back; `/audit` stays a stub until 4.2 + 4.3.
- **Compliance is non-blocking but permanently logged** (technical control; true immutability needs Pro).
- **KEEP / HAND OFF / HARDEN** — HARDEN = make a function survive without one person. Note: NetSuite-native concentrates dependence on Naveen (sole SuiteScript author/admin) — a deliberate **KEEP**, not a harden.

## Open questions / next actions
1. **Verify CAD-link sync scope** — which SolidWorks properties (vendor/MPN, revision, modified-by, dates) land on the NetSuite item record. [gates the Audit]
2. Compliance: accept **tamper-evident on Standard** or buy **PDM Pro**.
3. Interference on failure: **block** or **proceed + log**.
4. Build the **Bailey GD&T decision table** (Chapter 3).
5. Confirm/renumber the **8-chapter outline**, then draft Chapter 1.
6. Stand up **cloud Claude Code + GitHub** for the standards page.

## How Naveen works
Terse, verdict-first, numbered options. Proceed autonomously on routine ops; surface trade-offs, don't pad.
Apply the KEEP / HAND OFF / HARDEN lens to every change.

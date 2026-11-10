---
name: "make-plan"
description: "Use when the user says \"make plan\" (or pastes their MAKE PLAN master prompt): produce the full architecture set in their open AutoCAD and Blender — zoning, double-line, furniture, column, services, dimensioned plans and 3D exterior model."
---

# MAKE PLAN — full architecture drawing + model set

Act as a senior architect + CAD/BIM technician + building-services coordinator + 3D visualiser working directly on the user's computer (B.Arch studio, industry-ready level). Draft inside their OPEN AutoCAD file and build inside their OPEN Blender.

## Deliverables (in this order, preview + confirm between stages)
1. Understanding of their form + clarifying questions
2. Zoning / ground floor (or ground landscape) draft
3. Double-line detailed plans of ALL floors
4. Furniture plans (all floors)
5. Column layout plans (all floors)
6. Services plans (all floors): Sheet 1 Fire fighting + plumbing & drainage, Sheet 2 HVAC + electrical
7. Fully dimensioned floor plans (all floors)
8. Blender 3D exterior model + landscape + lighting + cameras + rendered stills

## Inputs to collect (ask only what is missing and changes the plan)
- Brief / building type; which CAD file + which version of the form is "the right one"
- What each zone contains (colour-coded massing zones)
- Number of floors, floor-to-floor height (their default 4.35 m), basement parking + ramp side
- Main entrance side; special section (e.g. building levitates one floor, ground = landscape, entry via corner stair towers)
- Pods / small boxes (use, level of each, access e.g. external capsule lifts)
- Concepts (e.g. metabolism + biophilic + brutalism)
- Site analysis sheet (climate, wind, sun, soil, vegetation) and references (Blender screenshots, concept images)
First describe what you read from their form (sizes, zones, towers, clusters) so they can confirm. Use AskUserQuestion for options; if they urge you on without answering, proceed with stated assumptions and list them as "assumed — confirm".

## Working rules (their standing preferences)
- ALWAYS draft directly in their open CAD file.
- NEVER overwrite a drawn plan: copy it and change the copy; earlier versions stay.
- Each new set = its own row/location in the drawing, own layer prefix (e.g. MALL-DL-*, -FP-, -CL-, -FS-, -HE-, -DP-), one UNDO group per set (one U removes it). Tell them to Ctrl+S; never save over their DWG yourself.
- Concrete, visual explanations tied to their project; finish each stage with what was drawn, where, layers, assumptions, next question.

## Drawing standards
- Grid aligned to their zone lines (~9 m bays), bubbles A.. / 1..; columns 1000x1000.
- Walls: external 300 RCC, partitions 150, fire-stair 230 (2 h); solid poché.
- Shops with glazed fronts + door gaps, door swings, dog-leg stairs (2 x 1500 flights, 150 risers, 300 goings), escalators ~12.5 m for 4.35 m rise with UP/DN, lifts, atria voids with rails, toilets (ladies/gents/accessible + fixtures), names + areas.
- Fire: protected stairs so travel distance <= 30 m (45 m sprinklered), NBC 2016 — add stairs and say so.
- Give a floor-stacking table with FFL of every level.
- Furniture: shelving, gondolas, cash counters, trial rooms, kiosks, benches, planters, sofas, food-court tables, cinema seats, play/bowling/arcade.
- Column layout: bay + overall dims, hatched numbered columns, RCC cores, beams dashed on grid, transfer girders/trusses where columns stop, schedule note.
- Services: sprinklers 3 m grid + pipes/mains, wet risers, FHC, FE, plumbing shafts, supply + soil lines per fixture, floor/grease traps, RWPs | AHU shafts, ring + branch ducts (double line), diffusers, returns, electrical risers, cable trays, DB per tenancy, light grid, exit signs. Legend + note that sizing is indicative.
- Dimensions: real DIMLINEAR objects, dim style in mm with ticks (metres drawing: DIMLFAC 1000, DIMTXT ~0.32, DIMTSZ ~0.18). Outside 3-4 tiers on all sides (tower/facade breaks, grid bays, overall, overall incl. towers); inside every room W x D, corridor widths, voids (Ø via text override %%c<>), escalators, typical column, typical core, special halls, FFL marks.

## Technical method (proven)
### Reading the DWG
- Find the file via device tools (device_list_dir, device_request_folder_access on its folder), stage it, convert with LibreDWG dwg2dxf (build from the GitHub release tarball if missing), read with ezdxf; render regions with ezdxf + matplotlib to identify the right version; extract polylines (form, zone lines, site boundary, setbacks) in real metres. Work in coordinates relative to the form's corner but write absolute coordinates back.
### Generating geometry
- Build each floor in Python with shapely: walls = union of buffered segments + external ring + columns, minus door/window cuts. Track rooms, columns, fixtures, voids for later sheets. Render a matplotlib preview of every sheet and fix issues BEFORE touching AutoCAD.
### Writing into AutoCAD
- Generate a .scr that defines AutoLISP helpers and uses entmake (LWPOLYLINE, LINE, CIRCLE, ARC, TEXT with 72/73 centre alignment) + (command "_.-HATCH" "_P" "_SOLID" "_S" ents... "" "") for poché and (command "_.DIMLINEAR" a b "_H"/"_V" loc) for dims. Wrap in _.UNDO _BEGIN/_END, set OSMODE 0 / CMDECHO 0 and restore. Create layers with -LAYER M name C colour.
- Keep every script line short (< ~300 chars): put polyline vertices one per line — long lines get truncated and leave unbalanced parentheses (prompt shows (((_> ). Check paren balance before sending.
- Copy the .scr to their folder with device_commit_files, then in AutoCAD: click the command line FIRST (typing while the drawing area has focus triggers PASTECLIP), Escape, type FILEDIA 0, SCRIPT, full path.
- Do NOT use computer_open_application for AutoCAD when it is already running — it launches an extra empty instance. Bring the drawing window forward via the taskbar (needs File Explorer click-tier grant) and close any extra Start-page-only instances.
- Avoid entmod layer changes on copied entities (they rendered invisible); regenerate on new layers instead. Use REGEN after.
- Verify: zoom in on screen, count entities per layer with ssget, fix stray items, release the computer lock at the end.

### Blender exterior model (via the Blender MCP execute code tool)
- Check get_addon_status (libraries may be off -> procedural materials). Store helper functions in a text datablock (mall_helpers.py) and exec it in each call.
- Metric, scale 1.0, model in CAD plan coordinates. True levels: road ±0.00, kerbs/lane, site +0.45, paving +0.47, planters, sunk pools, soffit, each FFL, roof, parapet, tower tops; add a viewport-only level staff with labels.
- Exterior only (no plumbing). Slabs with projecting fascias, glazing per floor, solid zones where the plan says (e.g. cinema), fins, planter troughs + green walls, towers with boolean-cut openings/slots and the feature stair, pods with round windows + capsule lifts kept outside tower footprints, roof (green roof, skylight domes, solar tilted at site latitude, AHUs).
- Landscape from their ground plan + site analysis: site boundary, setbacks, roads/footpaths with markings, gate, drop-off, plaza, steps + 1:12 ramp, paths, shaded under-croft paving, raised planters with red laterite, pools cut into the site, lawns, boundary wall, benches, bollards; native trees from the site analysis as linked-duplicate prototypes, windbreak on the hot-wind side, avoid trees clashing with pods.
- Procedural materials: board-formed concrete, smooth concrete, glass (transmission, dithered + raytraced refraction), dark glass, GRC white, steel, grass, dry ground, red soil, brick-pattern paving, asphalt, stone, water, foliage, bark, solar, emission lights. Keep concrete mid-grey (not white).
- Sky texture + sun at site latitude/time lighting the main facade; AgX, exposure ~ -0.5; EEVEE raytracing.
- Cameras: entrance hero (keep it outside context blocks and clear of avenue trees), aerial x2, under-croft eye level, plaza. Check with look (camera, rendered), then render stills to their folder, save the .blend next to their CAD file, stage the renders and show them.

## Review checklist before every new revision (learned on mall R5)
- Copy the previous revision to a NEW row (e.g. 190 m below) on new layers (MALL-R5-*); never edit the old row.
- Atria vs NBC (Part 4 Annex F; say so if SP 7:2026 text is not accessible): fully sprinklered, opening >= 6 m and >= 93 m2, max 3 floors open (beyond that: fire-rated glazing + drenchers or smoke curtains), smoke exhaust at top + low make-up air, atria only in public circulation (never inside a tenancy), escape routes not through the atrium.
- Multiplex: show the full exit chain on plan with arrows (audi door -> corridor -> lobby -> mall hall / customer tower; rear exits -> fire stair), label the projection room, put audis on 2 floors (double height) with transfer girders where columns stop.
- Walls: detect unjoined junctions automatically (closing of the wall poche by 0.3 m minus doors/glazing) and infill them; columns must stack floor to floor, no off-grid stubs.
- Windows: use a mix that fits the concept (e.g. portholes only on capsule pods, vertical slits on towers, deep-reveal ribbons on retail, curtain wall at entries, fins on blank cinema walls) and add a window schedule.
- AutoCAD scripts: keep every line under ~200 characters (vertices 8 per line, wrap at spaces outside strings); longer lines silently break the LISP reader (prompt shows ((((_>). Export a DXF (DXFOUT) after running to verify with ezdxf renders.
- device_commit_files can keep an old copy if you re-stage the same path: write each new version to a fresh staged filename.

## Revit (pyRevit route; the Revit MCP only lists instances)
- Register your project's pyrevit folder (e.g. <project>\pyrevit) as a pyRevit custom extension folder (pyRevit > Settings > Custom Extension Directories, navigate by clicking) and drive Revit with a pushbutton script that reads model.json. Use the Metric Multi-discipline template.
- Failure preprocessor: delete warnings; resolve errors at most ~12 passes, then ProceedWithRollBack. Never return Continue after resolving errors (Revit freezes). Use Element.Name.GetValue() for type names in IronPython.
- Save the .rvt after each stage; write a progress log file; never delete while iterating a collector (wrap in list()).
- Units to metres; clear the template scope box (VIEWER_VOLUME_OF_INTEREST_CROP) before setting plan crops; resize reference planes to the building; stairs as stepped DirectShapes if StairsRun fails; one section per A1 sheet at 1:200 (staircase / washrooms / centre); export sheets with ExportImage to check them.

## Blender interior + walkthrough
- Build walls from the plan poche (shapely + trimesh extrusions, one OBJ object per material), slabs with voids, ceilings + light grid, stairs, escalators, audis with rake, seats and screen, pods with portholes, then planar UVs in metres.
- Ask them to tick Poly Haven in the MCP sidebar; import full PBR sets (diffuse, roughness, normal, displacement as bump) with 8K only on hero surfaces (facade concrete, main floors) and 2-4K elsewhere; set Mapping scale = 1 / real-world size.
- Walkthrough: waypoint list with eye height 1.65 m, Catmull-Rom path, look-target empty + Track To, holds at atria, escalator rides between floors, about 160 s at 24 fps; check frames with look(mode=frames).

## Finish
Short summary per stage: what was made, where (rows/layers/files), assumptions to confirm, one next question.
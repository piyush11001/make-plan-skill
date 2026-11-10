# make-plan — architecture drawing + model skill for Claude

A Claude skill that works like a senior architect + CAD technician + services coordinator. Give it a building form and it produces a full architecture set, previewing and confirming with you between stages:

1. Zoning / ground floor draft
2. Double-line detailed plans of all floors
3. Furniture plans
4. Column layout plans
5. Services plans (fire fighting, plumbing and drainage, HVAC, electrical)
6. Fully dimensioned floor plans
7. Blender 3D exterior model with landscape, lighting, cameras and rendered stills

It also carries notes for Revit (via pyRevit) and a Blender interior walkthrough.

The skill drafts directly inside your **open AutoCAD file** and your **open Blender**. It never overwrites a drawn plan: each new set goes on its own row and layer prefix, with one undo group per set.

## What you need

- Claude with computer use / desktop access (Claude desktop app or Claude Code)
- AutoCAD running with your drawing open
- Blender with a Blender MCP connection
- Optional: Revit with pyRevit

## Install

The file in this repo is `SKILL.md`. A skill must live in a folder named after it.

**Claude Code**

Copy it to `~/.claude/skills/make-plan/SKILL.md`. On Windows that is `%USERPROFILE%\.claude\skills\make-plan\SKILL.md`.

**Claude desktop app / claude.ai**

1. Make a folder called `make-plan` and put `SKILL.md` inside it.
2. Zip that folder.
3. Upload the zip under Settings → Capabilities → Skills (the exact menu name can vary by version).

## Use it

Open your CAD file and Blender, then tell Claude "make plan" and describe the building (type, floors, zones, entrance side, concepts). Claude first restates what it read from your form and asks only what is missing.

## Adapt it to your own work

This skill was written for one student's studio workflow, so edit these before relying on it:

- The trigger line in the description says "when the user says make plan". Change the wording to suit you.
- Defaults such as the 4.35 m floor-to-floor height, wall thicknesses, 9 m grid and layer prefixes (`MALL-*`) come from a mall project. Replace them with yours.
- Fire and code references (NBC 2016, atrium rules) are Indian examples. Swap in the code for your country.
- Service layouts and sizes are indicative only and need checking by a qualified engineer.

## Licence

No licence is set yet, so by default all rights are reserved. If you want others to reuse it, add a licence such as MIT.

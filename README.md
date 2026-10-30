# Nova Archipelago Compute

The Compute edition of Nova Archipelago: every sol is on a shrinking clock, and the warnings get in your way. Play it by opening `index.html` in a browser. See [Compute: the turn timer](#compute-the-turn-timer) and [Doom warnings](#doom-warnings) for what is different.

A turn-based hex colony builder. Land on a barren water world, build an island colony, and terraform the planet from bare rock to a **Living World** in as few sols (days) as you can.

- **Genre:** placement puzzle + light production chain + terraforming race
- **Platform:** a single self-contained HTML file (`index.html`), no build step, no server
- **Map:** 180 x 136 hexes (24,480 tiles), procedurally generated from a seed
- **Inspirations:** Islanders, Planet Crafter, Factorio, Satisfactory, Oxygen Not Included, Dyson Sphere Program

---

## Contents

1. [Premise](#premise)
2. [How to play](#how-to-play)
3. [Controls](#controls)
4. [The screen](#the-screen)
5. [Core loop and turn structure](#core-loop-and-turn-structure)
6. [Scoring and end conditions](#scoring-and-end-conditions)
7. [Difficulty](#difficulty)
8. [Terraforming stages](#terraforming-stages)
9. [Resources](#resources)
10. [Efficiency, upkeep and morale](#efficiency-upkeep-and-morale)
11. [Buildings](#buildings)
12. [Placement and adjacency](#placement-and-adjacency)
13. [Terrain](#terrain)
14. [Deposits and discoveries](#deposits-and-discoveries)
15. [Progression: milestones and research](#progression-milestones-and-research)
16. [Hazards](#hazards)
17. [Saving and records](#saving-and-records)
18. [Audio](#audio)
19. [Visuals](#visuals)
20. [Design choices](#design-choices)
21. [Known gaps / ideas for alpha](#known-gaps--ideas-for-alpha)

---

## Premise

Your colony charter sends you to **Kepler-442c**, a water world scattered with island chains. Each map is a named sector (for example "Vela Shoals" or "Hydra Reach") built from a random seed. You start with a Colony Lander on a spot picked for nearby ore and terrain variety.

The planet changes as you work. Every terraforming stage tints the land and sea toward lush colours. Farms, Biodomes and Arboretums green the ground around them, Heaters thaw nearby tundra and glacier, and Algae Vats colour the shelf water. Some biomes get new names once they are lush enough: Silt Shelf becomes **Coral Shelf**, Regolith Plains becomes **Moss Plains**, and Cryo Tundra becomes **Thawed Tundra**.

## How to play

1. Open `index.html` in a modern desktop or mobile browser. Fonts load from Google Fonts. Without a connection the game falls back to system fonts.
2. On the **Colony charter** screen, pick a difficulty: Relaxed, Standard or Hard.
3. Click **Launch colony**. **New world** rerolls the seed. The seed is shown as an 8-digit hex code.
4. If a save exists, a **Continue** button appears with the save's difficulty, sol and stage.

The first time you play, the **Field notes** help panel opens automatically.

## Controls

| Action | Mouse / keyboard | Touch |
|---|---|---|
| Pan | Drag, or `W` `A` `S` `D` / arrow keys | Drag |
| Zoom | Mouse wheel, `+` / `=` / `E` in, `-` / `Q` out | Pinch |
| Select a structure | Click its card, or `1`-`9`, `0` (first 10 cards only) | Tap card |
| Place | Left-click a hex | Tap the same hex twice |
| Cancel selection | `Esc` or right-click | Tap the card again |
| Repair a damaged structure | Click it | Tap it |
| End the sol | `Space`, `Enter`, or the **End sol** button | Button |
| Research tree | `R`, or the flask button | Button |
| Music on/off | `M` | Button |
| Jump to the lander | `H` | Button |
| Field notes | `?` or `/` | Button |
| Menu / close panels | `Esc` | Button |
| Jump camera | Click or drag on the minimap | Same |

The menu has separate **Music** and **Effects** volume sliders, plus **Restart** (same seed), **New world** and **End colony**.

## The screen

- **Terraformation box (top left):** total Ti, Ti per sol, the current stage, progress to the next stage, an estimate of sols remaining, any component floors still needed, and running totals for Heat, Pressure and Biomass.
- **Economy panel (top):** meters for Power, Oxygen, Crew (jobs vs housing) and Morale. Below them are stockpiles for ore, alloy, crystal, components and rations, each with its net change per sol. Alert chips cover brownouts, low rations, thin air, understaffing, damaged structures, and active or incoming hazards.
- **Milestones / Research (left):** the milestone checklist, then the active research with progress, sols remaining and the queue.
- **Build tray (bottom):** one card per unlocked structure with its current cost and count built. Unaffordable cards are dimmed. New unlocks get a "NEW" tag.
- **Info panel (bottom left, while placing):** site rule, radius, link range, what the structure draws, base yield, every adjacency rule, and cost.
- **Minimap (bottom right):** structures, monolith markers and the camera frame. The sector name and the hovered hex coordinates appear below it.
- **Tooltip:** hover any hex to see its biome, deposit (with ferrite purity), survey status and the structure's live output and efficiency. While placing, it shows the projected yield, cost, and power/O2/crew budgets after building.

All HUD numbers are a projection of the next sol, so you see the effect of a placement before you end the sol.

## Core loop and turn structure

The game is turn-based, but each sol is on the clock (see [Compute](#compute-the-turn-timer)).

1. **Build.** Spend alloy, crystal and components to place structures inside the dashed build zone. You can place as many as you can afford.
2. **End the sol.** The simulation resolves in this order:
   1. Efficiency is computed from power, oxygen, staffing and morale (see [Efficiency](#efficiency-upkeep-and-morale)).
   2. Drills add ore. Smelters convert ore to alloy 1:1, up to total smelting capacity. The lander (+1) and Skyports add alloy directly.
   3. Assemblers turn 1 alloy + 1 crystal into 1 component, up to assembly capacity and free component storage.
   4. Sail Launchers spend 1 component per sail launched.
   5. Ore is capped at storage. Rations get production minus consumption, capped at storage and never below 0.
   6. Battery charge updates: shortfalls drain it, surplus power refills it.
   7. Terraformers add Heat, Pressure and Biomass to Ti.
   8. Data flows into research.
   9. Morale changes.
   10. The sol counter advances and a summary appears in the message feed.
   11. Hazards advance. A new hazard can start, and a meteor shower strikes immediately.
   12. Terraforming stages are checked. Several can be gained at once.
   13. Loss conditions are checked.
3. **Repeat** until the planet reaches Living World or the colony fails.

### Compute: the turn timer

*Added on the `compute-timer` branch, mostly as a joke.* Compute is a resource that is simply the time you have to plan the current sol. It shows first in the meter row as seconds left out of the sol's budget, with the next sol's budget underneath.

| | |
|---|---|
| Budget | `10 + 110 × 0.95^(structures − 1)` seconds, rounded. The lander counts as 1. |
| Examples | 1 structure: 120 s · 10: 79 s · 20: 52 s · 40: 25 s · 60: 15 s · floor 10 s |
| Refill | Full budget at the start of every sol, recalculated from your structure count then. Building during a sol does not cut the current clock, only the next one. |
| Paused | The menu, the start screen and the victory screen stop the clock. Research and field notes do not. |
| Running out | The sol ends on its own and morale drops 5. |
| Last 10 s | The meter turns amber and ticks each second; the last 5 s blink red and tick higher. |

### Doom warnings

Warnings escalate in three levels, and each is louder than the last.

| Level | Trigger | What happens |
|---|---|---|
| 1 · Warning | Morale below 25, rations running out, oxygen short, or 10 s of compute left | Red alert chips shake and flash, the screen edges pulse red, an alarm every 6 s |
| 2 · Almost dead | Morale at 0, morale hitting 0 next sol, or under 10 sols to the charter review | A flashing **YOU ARE ALMOST DEAD** box roams the screen and drifts toward your cursor, blocking clicks. Clicking it makes it dodge somewhere else. The HUD shakes and a klaxon sounds every 2.4 s |
| 3 · Dead | Collapse or charter loss | The same box reads **YOU ARE DEAD** and wanders over the game-over screen, klaxon every 3.5 s |

## Scoring and end conditions

**Score = the number of sols taken to reach stage 8, Living World (30,000 Ti).** Fewer is better. The best result for each difficulty is saved and shown on the charter screen and the end-of-game report.

| Outcome | Condition |
|---|---|
| **Win: Living World** | Reach stage 8. A victory screen offers **Keep building** or **New world**. |
| **Loss: evacuation** | Morale stays at 0 for 5 consecutive sols (Standard) or 3 (Hard). Never on Relaxed. |
| **Loss: charter withdrawn** | The sol counter passes 300 (Standard) or 240 (Hard) before Living World. No deadline on Relaxed. |
| **Ended** | You chose **End colony** from the menu. |

The colony report shows final Ti, stage, sol and best sols. **View colony** closes the report so you can look at the map. You cannot build after that.

After a win, the charter deadline no longer applies. The morale collapse check still does.

## Difficulty

Every difficulty setting lives in one `DIFFS` table.

| Setting | Relaxed | Standard | Hard |
|---|---|---|---|
| Construction cost multiplier | x0.85 | x1.0 | x1.2 |
| Starting alloy | 46 | 40 | 34 |
| Starting rations | 20 | 16 | 14 |
| Rations eaten per crew per sol | 0.35 | 0.5 | 0.6 |
| Morale recovery per fed sol | +7 | +5 | +4 |
| Morale loss multiplier | x0.6 | x1.0 | x1.35 |
| Morale efficiency floor (code value) | 0.60 | 0.35 | 0.25 |
| Sols at 0 morale before evacuation | never | 5 | 3 |
| Charter deadline | none | sol 300 | sol 240 |
| First hazard | sol 26-30 | sol 16-20 | sol 12-16 |
| Gap between hazards | 24-34 sols | 15-23 sols | 10-16 sols |
| Hazard warning | 3 sols | 2 sols | 1 sol |
| Meteors per shower | 1 | 2 | 3 |

## Terraforming stages

Ti (terraformation index) is the sum of three components: **Heat**, **Pressure** and **Biomass**. To reach a stage you need its total Ti, and some stages also need a minimum in one component.

| # | Stage | Total Ti | Component floor | Perk |
|---|---|---|---|---|
| 0 | Barren Rock | 0 | - | - |
| 1 | Thin Haze | 60 | - | Milestone: unlocks Vapor Drill and Tidal Turbine |
| 2 | Blue Sky | 260 | Pressure 60 | Opens research tier 2 |
| 3 | Lichen Bloom | 800 | Biomass 100 | Hydro Farms +1 rations (built ones included) |
| 4 | Liquid Seas | 2,000 | Heat 800 | Opens research tier 3 |
| 5 | Moss Carpet | 4,200 | Biomass 900 | Morale recovers twice as fast |
| 6 | Breathable Air | 8,000 | Pressure 2,500 | Opens research tier 4. Nothing needs O2 any more. |
| 7 | Forests | 15,000 | Biomass 5,000 | Biodomes and Arboretums +2 Ti/sol (built ones included) |
| 8 | Living World | 30,000 | - | Victory |

**Ti sources:**

| Source | Component | Amount |
|---|---|---|
| Heater | Heat | placement value per sol |
| Vapor Drill, Atmospheric Processor | Pressure | placement value per sol |
| Biodome, Arboretum | Biomass | placement value per sol |
| Solar sails (Sail Launcher) | Heat | +0.25 per sail in orbit, per sol |
| Orbital Mirrors (repeatable tech) | Heat | +3 per level, per sol |
| Decoded Precursor Monolith | Heat | +3 per monolith, per sol |

Building output is scaled by efficiency. Sails, mirrors and monoliths are not.

## Resources

| Resource | Kind | Notes |
|---|---|---|
| **Ti** (Heat / Pressure / Biomass) | Score track | Cumulative. Never spent. |
| **Power** | Per-sol budget | Supply vs demand. Not stored, except in Battery Banks. |
| **Oxygen (O2)** | Per-sol budget | Supply vs demand. Free from stage 6. |
| **Crew** | Per-sol budget | Housing vs jobs. Every housed crew member eats rations. |
| **Morale** | 0-100 | Starts at 75. |
| **Ore** | Stockpile | Cap 40 + 20 per Ferrite Drill. Excess is lost. |
| **Alloy** | Stockpile | No cap. The main building currency. |
| **Crystal** | Stockpile | No cap. |
| **Components** | Stockpile | Cap 20 + 15 per Assembler. Assemblers pause when it is full. |
| **Rations** | Stockpile | Cap 50 + 25 per Hydro Farm. Excess is lost. |
| **Data** | Flow | Goes to the active research. When nothing is queued, up to 20 is banked. |
| **Solar sails** | Count | Each sail in orbit gives +0.5 power and +0.25 Heat per sol, permanently. |

**Production chain:** Ferrite Drill (ore) -> Smelter (alloy) -> Assembler (+ crystal from a Crystal Drill) -> components -> Sail Launcher (sails). Components also pay for advanced buildings and some research.

## Efficiency, upkeep and morale

Each structure lists what it **draws** (power, O2, crew). Every sol:

- **Power efficiency** = (supply + battery discharge) / demand, max 100%. It applies to structures that draw power.
- **O2 efficiency** = supply / demand, max 100%.
- **Staffing** = housing / jobs, max 100%.
- **Morale factor** = 100% at morale 50 or more. Below 50 it is `max(floor, 0.35 + 0.65 x morale / 50)`.
- **Crew efficiency** = staffing x morale factor x O2 efficiency. It applies to structures that draw crew.

A structure's output = its placement value x power efficiency (if it draws power) x crew efficiency (if it draws crew) x hazard modifier. Damaged structures produce nothing and draw nothing.

Housing from Hab Pods, Commons and Arcologies is not scaled by efficiency. Their power and O2 draws only add to demand.

**Morale change per sol:**

| Condition | Change |
|---|---|
| Rations covered consumption | +recover (+7 / +5 / +4), doubled from stage 5 |
| Rations ran out | -16 x loss multiplier (instead of recovery) |
| O2 short | -20 x (1 - O2 efficiency) x loss multiplier |
| Power efficiency below 70% | -3 x loss multiplier |
| Commons Plazas | +2 each, +8 maximum total |

## Buildings

### Summary

Costs are first-copy prices on Standard. See [cost scaling](#cost-scaling) for later copies.

| Building | Unlock | Cost | Site | Draws | Yields (base) | Radius |
|---|---|---|---|---|---|---|
| Colony Lander | Starting | - | Fixed | - | 8 power, 6 O2, 4 housing, +1 alloy/sol | - |
| Hab Pod | Start | 5 alloy | Open land | 1 power, 2 O2 | 2 housing | 2 |
| Solar Array | Start | 4 alloy | Open land | - | 4 power | 1 |
| Hydro Farm | Start | 5 alloy | Open land | 1 power, 1 crew | 2 rations/sol | 2 |
| Heater | Start | 5 alloy | Open land | 2 power | 2 Ti/sol (Heat) | 2 |
| Ferrite Drill | Start | 6 alloy | Ferrite node | 2 power, 1 crew | 0 + purity (2/3/5) ore/sol | 1 |
| Smelter | Start | 8 alloy | Open land | 2 power, 2 crew, 1 O2 | 2 smelting/sol | 2 |
| Relay Pylon | Start | 6 alloy, 1 crystal | Open land | 1 power | Zone +5, survey 8 | - |
| Electrolyzer | Milestone: smelt 10 alloy | 6 alloy | Open land | 2 power | 3 O2 | 1 |
| Crystal Drill | Milestone: house 8 crew | 8 alloy | Crystal vein | 2 power, 1 crew | 1 crystal/sol | 1 |
| Vapor Drill | Milestone: reach Thin Haze | 8 alloy | Open land | 3 power | 2 Ti/sol (Pressure) | 1 |
| Tidal Turbine | Milestone: reach Thin Haze | 6 alloy | Shelf water | - | 2 power | 1 |
| Assembler | Milestone: mine 8 crystal | 10 alloy, 2 crystal | Open land | 3 power, 2 crew, 1 O2 | 1 assembly/sol, +15 component storage | 2 |
| Research Lab | Milestone: assemble 5 components | 10 alloy, 2 crystal, 1 comp. | Open land | 2 power, 2 crew, 1 O2 | 3 data/sol | 2 |
| Geothermal Plant | Tech: Geothermal Power (T1) | 10 alloy, 2 comp. | Thermal vent or steam geyser | 1 crew | 6 power (+2 on a geyser) | 1 |
| Algae Vat | Tech: Algae Bioreactors (T1) | 8 alloy, 1 crystal | Shelf water | 1 power | 3 O2 | 1 |
| Battery Bank | Tech: Power Storage (T1) | 8 alloy, 2 crystal | Open land | - | 12 power storage | 1 |
| Biodome | Tech: Lichen Seeding (T1) | 10 alloy, 2 crystal, 2 comp. | Open land | 2 power, 1 O2, 1 crew | 6 Ti/sol (Biomass) | 2 |
| Commons Plaza | Tech: Communal Living (T2) | 8 alloy, 1 crystal | Open land | 1 power | 1 housing, +2 morale/sol | 2 |
| Drone Hub | Tech: Drone Logistics (T2) | 12 alloy, 4 comp. | Open land | 2 power, 1 crew | 0 smelting/sol, zone +4, survey 7 | 3 |
| Point Defense | Tech: Meteor Defense (T2) | 12 alloy, 2 crystal, 4 comp. | Open land | 3 power, 1 crew | Meteor cover, 4 hexes | - |
| Atmospheric Processor | Tech: Atmospheric Engineering (T2) | 16 alloy, 4 crystal, 6 comp. | Open land | 5 power, 2 crew, 1 O2 | 12 Ti/sol (Pressure) | 3 |
| Fusion Reactor | Tech: Fusion Power (T3) | 16 alloy, 4 crystal, 6 comp. | Open land | 3 crew, 1 O2 | 14 power | 2 |
| Arcology | Tech: Arcologies (T3) | 18 alloy, 4 crystal, 6 comp. | Open land | 3 power, 3 O2 | 4 housing | 3 |
| Signal Spire | Tech: Precursor Linguistics (T3) | 14 alloy, 6 crystal, 4 comp. | Open land | 2 power, 1 crew | 3 data/sol | 3 |
| Arboretum | Tech: Engineered Forests (T3) | 20 alloy, 6 crystal, 8 comp. | Open land | 2 power, 2 crew | 12 Ti/sol (Biomass) | 2 |
| Skyport | Tech: Orbital Freight (T4) | 20 alloy, 6 crystal, 10 comp. | Open land | 4 power, 3 crew, 1 O2 | 3 alloy/sol | 2 |
| Sail Launcher | Tech: Dyson Swarm (T4) | 24 alloy, 10 crystal, 12 comp. | Open land | 6 power, 3 crew | 1 sail/sol (costs 1 component each) | 2 |

"Open land" means a land hex that is not Crystal Peaks and has no deposit on it. O2 draws drop to 0 at stage 6. Closed-loop Life Support cuts the Hab Pod's O2 draw to 1.

### Adjacency rules

Each rule counts matching neighbours within the building's radius. "(adj)" means the rule only counts hexes directly adjacent, even when the radius is larger. "Open X" means an empty hex of biome X with no deposit or structure on it.

| Building | Bonuses | Penalties | Notes |
|---|---|---|---|
| Hab Pod | Hab +1, Commons +2, Arcology +1 | Smelter -1, Ferrite Drill -1, Assembler -1, Fusion Reactor -2 | Crew like neighbours and dislike industry. |
| Solar Array | Open Salt Flats / Ember Waste / Obsidian Field / Cryo Tundra +1 each | Solar Array -2 | Dust storms cut output to 35%. |
| Hydro Farm | Flora +1 (adj), open Silt Shelf +1 (adj), open Fungal Marsh +1 (adj) | Hydro Farm -1 (adj), Smelter -2 | Cold snaps halve output unless a Heater is adjacent. |
| Heater | Thermal vent +2 (adj), ice +1 (adj), open Obsidian +1 (adj) | Heater -2 | |
| Ferrite Drill | Ferrite node +1 | Ferrite Drill -1 | Base = purity: impure 2, normal 3, pure 5. |
| Smelter | Ferrite Drill +1, Drone Hub +1 | Smelter -1 | Capacity, not output. Alloy is limited by ore. |
| Electrolyzer | Ice +2, open Silt Shelf +1 | Electrolyzer -1 | |
| Crystal Drill | Crystal vein +1 | Crystal Drill -1 | |
| Vapor Drill | Ice +1, steam geyser +3, open Glacier +1, open Cryo Tundra +1 | Vapor Drill -3 | |
| Tidal Turbine | Open Silt Shelf +1, open Abyssal Sea +2 | Tidal Turbine -3 | Not affected by storms. |
| Assembler | Smelter +1, Crystal Drill +1, Drone Hub +1 | Assembler -1 | |
| Research Lab | Relic +3, Monolith +6, Crystal Drill +1 | Research Lab -2 | |
| Geothermal Plant | Thermal vent +2, steam geyser +2 | - | +2 base when built on a geyser. |
| Algae Vat | Open Silt Shelf +1, black smoker +3 | Algae Vat -2 | |
| Battery Bank | Solar Array +2 storage | Battery Bank -2 | |
| Commons Plaza | Hab +1, Arcology +2 | Commons -4 | Morale +2/sol regardless of placement. |
| Biodome | Flora +1, spore pods +1, Hydro Farm +1 | Biodome -3, Smelter -2 | |
| Drone Hub | Ferrite Drill +1, Smelter +1, Crystal Drill +1, Assembler +1 | - | Adds smelting capacity. |
| Fusion Reactor | Ice +2, open Silt Shelf +1 | Fusion Reactor -8 | |
| Atmospheric Processor | Steam geyser +3, ice +1, Vapor Drill +2 | Atmospheric Processor -10 | |
| Arcology | Hab +1, Commons +2, Biodome +2, Hydro Farm +1 | Fusion Reactor -3, Smelter -2, Arcology -6 | |
| Signal Spire | Relic +3, Monolith +8, Research Lab +1 | Signal Spire -8 | |
| Arboretum | Flora +1, spore pods +1, Biodome +2 | Arboretum -5, Smelter -2 | |
| Skyport | Smelter +1, Arcology +1 | Skyport -12 | |
| Sail Launcher | Fusion Reactor +1, Geothermal Plant +1 | Sail Launcher -3 | |

Output never goes below 0. After Fusion Containment, newly placed Hab Pods and Arcologies ignore the reactor penalty.

### Cost scaling

```
cost = ceil(base x difficulty multiplier x copy multiplier x (0.8 if Lean Fabrication))
```

- **Terraformers** (Heater, Vapor Drill, Biodome, Atmospheric Processor, Arboretum): copy multiplier = 1.12^n, compounding.
- **Everything else:** copy multiplier = 1 + 0.1 x n, linear.

`n` is the number already built.

**Repair** of a meteor-damaged structure costs 40% of that structure's current build cost, rounded up.

## Placement and adjacency

- **Build zone (dashed line).** You can only build inside it. Every structure extends it by its link range: 2 hexes by default, Lander 4, Drone Hub 4, Relay Pylon 5. Long-range Linking adds +1 to all.
- **Survey.** Structures reveal land around them. Survey range is the link range + 4 by default, or Lander 9, Relay Pylon 8, Drone Hub 7. Seam Mapping adds +2. Unsurveyed land is drawn dim and its deposits are hidden.
- **Fixed placement values (Islanders-style).** A structure's value is calculated once, when you place it, from its neighbours at that moment. Building neighbours later does not change it. Tech and stage bonuses marked "built ones included" are added to existing structures.
- **Placement preview.** While placing, the game shows a ghost of the structure, shades its radius, puts a +N or -N badge on every neighbour that counts, and shows the total yield above the hex. The outline is amber when valid, orange when the site is fine but you cannot afford it, and red when the site is invalid. The tooltip gives the reason, such as "Occupied", "Outside the build zone", "Blocked by flora", "Build on shelf water", "Too steep to build" or "Needs 3 alloy more".

## Terrain

Land is generated from elevation, temperature and moisture noise. Cold biomes cluster toward the top and bottom of the map. Sea fills the lowest ground. Hexes have stepped heights, and peaks and glaciers stand tallest.

| Biome | Lush name | Buildable | Common deposits (chance per hex) | Open-tile bonuses for |
|---|---|---|---|---|
| Abyssal Sea | - | No | Black smoker 1.2% | Tidal Turbine (+2) |
| Silt Shelf | Coral Shelf | Tidal Turbine and Algae Vat only | Black smoker 1.2% | Hydro Farm, Electrolyzer, Tidal Turbine, Algae Vat, Fusion Reactor (+1 each) |
| Salt Flats | - | Yes | Ferrite 6% | Solar Array |
| Regolith Plains | Moss Plains | Yes | Flora 8%, ferrite 5% | - |
| Lumen Jungle | - | Yes | Flora 38%, spore pods 5% | - |
| Fungal Marsh | - | Yes | Spore pods 26%, flora 8%, geyser 5% | Hydro Farm |
| Ember Waste | - | Yes | Vent 10%, ferrite 6%, geyser 3% | Solar Array |
| Obsidian Field | - | Yes | Vent 16%, ferrite 8% | Solar Array, Heater |
| Cryo Tundra | Thawed Tundra | Yes | Ice 12%, geyser 3%, crystal 3% | Solar Array, Vapor Drill |
| Glacier | - | Yes | Ice 22%, crystal 6% | Vapor Drill |
| Basalt Ridge | - | Yes | Crystal 14%, ferrite 8%, vent 4% | - |
| Crystal Peaks | - | Only Crystal Drills on veins | Crystal 45% | - |

On empty, buildable land hexes there is also a 0.5% chance of a Precursor relic and a 0.4% chance of Ark debris.

Lushness only changes colours and names. It has no effect on gameplay.

## Deposits and discoveries

Deposits block ordinary construction. Most of them give adjacency bonuses to nearby structures.

| Deposit | Used for |
|---|---|
| Ferrite node | Ferrite Drill site. Purity: about 45% impure, 38.5% normal, 16.5% pure. Purity pips are drawn on the tile. |
| Crystal vein | Crystal Drill site |
| Thermal vent | Geothermal Plant site. Heater and Geothermal bonuses. |
| Steam geyser | Geothermal Plant site. Vapor Drill, Atmospheric Processor and Geothermal bonuses. |
| Ice deposit | Heater, Electrolyzer, Vapor Drill, Fusion Reactor and Atmospheric Processor bonuses |
| Lumen flora | Hydro Farm, Biodome and Arboretum bonuses |
| Spore pods | Biodome and Arboretum bonuses |
| Black smoker | Algae Vat bonus (+3) |
| Precursor relic | Research Lab and Signal Spire bonuses (+3). Discovery bonus. |
| Ark debris | Discovery bonus only |
| Precursor Monolith | Research Lab (+6) and Signal Spire (+8) bonuses. Discovery bonus. |

**Map guarantees:** at least one crystal vein within 4 hexes of the lander, an attempt to place two ferrite nodes (one normal purity) within 3 hexes, and four Precursor Monoliths spread far apart, each more than 16 hexes from the landing site.

**Discoveries** trigger once, when a tile is first surveyed:

| Find | Reward |
|---|---|
| Ark debris | One of: +12-20 alloy (40%), +6-10 crystal (30%), +3-5 components (20%), +20 rations (10%) |
| Precursor relic | +15 data, banked for research |
| Precursor Monolith | Instantly completes the active research, or banks +60 data if none is active. Also gives +3 Heat Ti every sol, permanently. |

Unsurveyed monoliths appear as glowing cyan **?** beacons and are ringed on the minimap. The Orbital Scan tech shows every deposit on the map, but it does not survey tiles, so it does not trigger discoveries.

## Progression: milestones and research

### Milestones (early game)

These are shown as a checklist until all five are done.

| # | Goal | Unlocks |
|---|---|---|
| 1 | Smelt 10 alloy (cumulative) | Electrolyzer |
| 2 | House 8 crew (current housing) | Crystal Drill |
| 3 | Reach Thin Haze (stage 1) | Vapor Drill, Tidal Turbine |
| 4 | Mine 8 crystal (cumulative) | Assembler |
| 5 | Assemble 5 components (cumulative) | Research Lab |

### Research

- Click a tech to add it to the queue. Click a queued tech to remove it.
- Labs and Spires add data every sol. Leftover data rolls into the next tech in the queue.
- Techs with a component cost take the components when they start. The queue waits until you have enough.
- **Tier gates:** tier 1 needs the Research Lab unlocked. Tiers 2, 3 and 4 open at Blue Sky, Liquid Seas and Breathable Air, and each also needs 2 techs from the previous tier.
- The flask button shows a badge when the labs are idle and a tech is available.

| Tier | Tech | Data | Components | Effect |
|---|---|---|---|---|
| 1 | Geothermal Power | 24 | - | Unlocks Geothermal Plant |
| 1 | Algae Bioreactors | 20 | - | Unlocks Algae Vat |
| 1 | Power Storage | 24 | - | Unlocks Battery Bank |
| 1 | Lichen Seeding | 40 | - | Unlocks Biodome |
| 1 | Thin-film Photovoltaics | 28 | - | Solar Arrays +2 power (built ones included) |
| 1 | Closed-loop Life Support | 28 | - | Hab Pods need 1 O2 instead of 2 |
| 1 | Seam Mapping | 24 | - | Ferrite Drills +1 ore. Survey radius +2. |
| 1 | Long-range Linking | 28 | - | Every structure links 1 hex further |
| 2 | Communal Living | 50 | - | Unlocks Commons Plaza |
| 2 | Drone Logistics | 60 | - | Unlocks Drone Hub |
| 2 | Meteor Defense | 55 | - | Unlocks Point Defense |
| 2 | Atmospheric Engineering | 80 | 4 | Unlocks Atmospheric Processor |
| 2 | Induction Smelting | 60 | - | Smelters +1 capacity (built ones included) |
| 2 | Greenhouse Gases | 70 | - | Heaters +2 Ti/sol (built ones included) |
| 2 | Orbital Scan | 50 | - | Reveals every deposit on the planet |
| 2 | Hydroponics | 55 | - | Hydro Farms +1 rations (built ones included) |
| 3 | Fusion Power | 110 | 6 | Unlocks Fusion Reactor |
| 3 | Arcologies | 110 | 4 | Unlocks Arcology |
| 3 | Precursor Linguistics | 100 | 4 | Unlocks Signal Spire |
| 3 | Engineered Forests | 140 | 8 | Unlocks Arboretum |
| 3 | Lean Fabrication | 100 | - | All construction 20% cheaper |
| 3 | Gene-tailored Lichen | 120 | - | Biodomes and Arboretums +3 Ti/sol (built ones included) |
| 3 | Precision Assembly | 100 | - | Assemblers +1 capacity (built ones included) |
| 4 | Orbital Freight | 170 | 10 | Unlocks Skyport |
| 4 | Dyson Swarm | 220 | 16 | Unlocks Sail Launcher |
| 4 | Fusion Containment | 150 | - | Reactors +4 power. New homes ignore reactors. |
| 4 | Quantum Cores | 150 | - | Labs and Spires +2 data (built ones included) |
| 4 | Orbital Mirrors | 140 (+40 per level) | - | Repeatable. +3 Heat Ti/sol per level. |

## Hazards

Hazards are scheduled ahead of time. A warning chip appears 1-3 sols before one arrives, depending on difficulty. Each roll picks storm 45%, cold snap 30% or meteor shower 25%.

| Hazard | Duration | Effect | Counter |
|---|---|---|---|
| Dust storm | 2-3 sols | Solar Arrays at 35% output | Battery Banks, Tidal, Geothermal, Fusion |
| Cold snap | 3 sols | Hydro Farms at 50% unless directly adjacent to an undamaged Heater | Heaters beside farms |
| Meteor shower | 1 sol | 1 / 2 / 3 random structures (never the lander) are knocked offline | Point Defense within 4 hexes, running at 50% efficiency or more. Otherwise click to repair. |

The screen takes on an orange tint during storms and a blue tint during cold snaps.

## Saving and records

- The game autosaves to browser `localStorage` after every action (key `nova-archipelago-v4`). **Continue** on the charter screen restores the save, including the camera position.
- The save is deleted when the colony ends (loss or **End colony**).
- Other stored keys: `nova-best-sols-<difficulty>` (best score), `nova-diff` (last difficulty), `nova-audio` (music on/off and volumes), `nova-help-seen`.
- **Restart** replays the same seed from landing. **New world** returns to the charter screen with a new seed.

## Audio

All sound is generated live with the Web Audio API. There are no audio files.

- **Music:** an endless ambient score in D major with a Lydian colour. It cycles six chords (9 seconds each) on detuned saw/triangle pads with a slow filter sweep, plus a sine bass, plucked melody notes with echo, occasional FM bells, a filtered-noise ocean wash, and a 5-second reverb.
- **Music follows progress:** each terraforming stage opens the pad filter (620 Hz + 140 Hz per stage), makes melody notes more frequent, and widens their range.
- **Sound effects:** placement (pitch rises with the placement value), invalid action, card select, end sol, milestone/stage fanfare, research complete, discovery, repair, and a hazard alarm.
- Audio starts on the first click or keypress, pauses when the tab is hidden, and has separate Music and Effects sliders in the menu.

## Visuals

- Everything is drawn on HTML canvas. The terrain is extruded hex columns with stepped heights and a vignette. Terrain only redraws when the camera or map changes.
- Every structure and deposit has its own procedural art. At low zoom they become coloured dots, and hex grid lines appear when zoomed in.
- **Animations:** a pulsing dashed build-zone border, sonar rings on the lander and relays, drones flying between a Drone Hub and industry within 3 hexes, sail launch trails, meteor streaks and impacts, smoke and sparks on damaged structures, and floating yield popups.
- The UI is a sci-fi HUD using the Oxanium, Chakra Petch and IBM Plex Mono fonts on a dark teal and amber palette.
- The layout adapts for screens 760 px wide or less, and transitions are turned off when the system asks for reduced motion.

## Design choices

- **Variety over spam.** Terraformer costs compound (x1.12 per copy) and most structures penalise copies of themselves nearby. Infrastructure costs rise only 10% per copy.
- **Placement is the puzzle.** Values are fixed where you place them, as in Islanders, so build order matters. Put the neighbours down first, then the structure that benefits from them.
- **Turn-based race.** There is no real-time pressure. The challenge is reaching Living World in as few sols as possible, and the HUD always shows the next sol's projection.
- **Upkeep is a real threat.** Rations, brownouts, thin air and morale can spiral, and hazards punish thin margins. According to the design notes, balance was tuned with scripted bots so that random play does not win.
- **Gated progression.** Milestones teach the production chain one link at a time. After that, research takes over, and its tiers are tied to terraforming stages so the economy and the planet advance together.
- **Three Ti components.** Stage floors on Heat, Pressure and Biomass mean you cannot win by building one kind of terraformer.
- **One difficulty table.** Every difficulty number lives in `DIFFS`, so it is easy to tune.

---

## Known gaps / ideas for alpha

> Everything in this section is a **suggestion or an observation from the code**, not an implemented feature.

**Things that look stale, inconsistent or unfinished**

- **Win sol is off by one.** `state.wonSol` is recorded after the sol counter has advanced. Reaching Living World while ending sol 120 records and displays "121 sols". Best scores share the same offset, so comparisons are still fair.
- **The Hard morale floor has no effect.** `moraleFactor` is `max(floor, 0.35 + 0.65 x morale / 50)`, and the second term is never below 0.35. Hard's 0.25 floor therefore behaves the same as Standard's 0.35.
- **Unused sound.** `Sound.crate` is defined but never called. It is left over from the earlier "supply crate" design.
- **No doctype.** The file starts at `<title>` (artifact format), so opening it directly runs the page in browser quirks mode. Adding `<!doctype html>` and a viewport meta tag would make standalone play reliable.
- **Crystal Peaks are "too steep to build" except for Crystal Drills.** The feature-site check runs before the peak check, so drills can be placed on peak crystal veins. This may be intentional, but no in-game text says so.
- **Some blurbs imply live adjacency.** "Nearby drills add capacity" (Smelter) and "Solar Arrays beside it add storage" (Battery Bank) only count structures that already exist when you place them.
- **Repair cost scales with copies built.** It uses the current build price, so repairing an early Heater late in the game can be expensive.
- **A monolith can complete any active research for free,** including 220-data techs. This is easy to exploit by queuing the most expensive tech before surveying one.
- **The "two ferrite nodes" guarantee is probabilistic** (a 30% roll per tile within 3 hexes). It almost always works, but it is not certain.
- **Hotkeys cover only the first 10 tray cards.** The full game unlocks 28 structures.
- **Field notes leave out some controls:** `Enter` to end the sol, `0`, arrow keys, `E`/`Q`/`+`/`-` zoom, minimap jumping, and double-tap to place on touch.
- **The "about N sols" estimate** in the Ti box ignores component floors.

**Natural next steps**

- In-game wiki: show this README (or a condensed form) in the Field notes panel. The repo guidelines list this as nice to have.
- A seed input on the charter screen so players can share or replay worlds. The seed is already displayed.
- Ways to demolish or relocate structures, and a way to re-evaluate placement values.
- A per-sol history graph (Ti, morale, stockpiles) on the colony report.
- More difficulty work: hazard intensity scaling with stage, and a custom difficulty using the existing `DIFFS` fields. The design notes say the user plans more difficulty work.
- More uses for late-game sails and alloy surplus, such as a second intermediate product from the design notes' Satisfactory/Factorio inspirations.
- Hazard variety: for example, events tied to biomes or stages.
- Accessibility: a colour-blind check on the adjacency badges, which use green and red, and full keyboard placement without a mouse.

# Inflatable Loot Goblin - Build Status & Handoff Notes (v2)

## ⚠️ READ THIS FIRST

This document is a companion to `goblin_progress_summary_v10.md` and `animation_reference.md`. Read both before continuing development.

---

## 🛠️ CURRENT BUILD STATUS

**First build is working.** She spawns, stays in world, displays texture, walks, stands, and can be tamed. The following is confirmed working and confirmed not working as of the current build.

---

## ✅ CONFIRMED WORKING

- Spawns and stays in world
- Poppy texture displays correctly
- `idle.stand` plays
- `move.walk` plays
- Can be tamed with golden carrot, golden apple, glistering melon slice
- Follows owner after taming
- Hostile to players without gold armor
- Non-hostile to players wearing gold armor

---

## ❌ CONFIRMED NOT WORKING / NOT YET TESTED

- `move.jump`, `move.fall`, `move.land` — chain not activating (fix needed)
- `idle.anim_2` — not yet tested
- `lock.sitting_0_1` — not yet tested (needs `loot:idle_pose` set manually)
- Texture cycling — all goblins currently use Poppy only
- No script running yet — all script-driven features non-functional

---

## 🔧 KNOWN FIXES NEEDED

### Jump/Fall/Land chain not activating
The animation controller uses `query.is_jumping` and `query.is_on_ground` but these aren't triggering correctly. Try replacing the jumping transition condition with:
```json
{ "jumping": "query.is_on_ground == false" }
```
And falling transition with:
```json
{ "falling": "!query.is_jumping && query.is_on_ground == false" }
```

### Sitting not testable yet
`loot:idle_pose` needs to be set to 1 for sitting to trigger. Without the script this can't be tested properly yet. Add to next build priorities.

---

## 📋 CURRENT BUILD FILE LIST

### What's in the pack RIGHT NOW:

**BP:**
- `manifest.json` — has data + script modules, @minecraft/server dep
- `entities/goblin_zero.json` — stage 0 only, no transformation yet
- `loot_tables/entities/goblin.json` — empty loot table
- `scripts/goblin_script.js` — blank placeholder (`import {} from "@minecraft/server"`)

**RP:**
- `manifest.json` — resources module, BP dependency
- `entity/zero.client.json` — stage 0 client entity
- `render_controllers/goblin_render_controller.json` — simplified, Poppy default
- `animation_controllers/goblin.animation_controllers.json` — 7 states (stand, walk, sit, anim_2, jump, fall, land)
- `models/entity/goblins.geo.json` — all 6 geometries
- `animations/goblins.animation.json` — 30 animations
- `textures/entity/name/` — 6 named textures
- `sounds/sound_definitions.json`
- `texts/en_US.lang`

---

## 🎯 NEXT BUILD PRIORITIES (IN ORDER)

1. **Add `combat.punch`** to animation controller — she attacks but no animation plays
2. **Fix jump/fall/land chain** — see fix above
3. **Add texture cycling** — script sets random named texture at spawn
4. **Test `idle.anim_2`** — confirm random trigger works
5. **Test sitting** — once idle_pose can be set via script

---

## 🧩 ENTITY PROPERTIES (goblin_zero.json)

Currently declared:
```json
"loot:hair_index": { "type": "int", "range": [0,15], "default": 0, "client_sync": true },
"loot:name": { "type": "int", "range": [-1,5], "default": -1, "client_sync": true }
```

Still needed (add in future builds):
```json
"loot:digestion_timer": { "type": "float", "range": [0.0,25.0], "default": 20.0, "client_sync": false },
"loot:idle_pose": { "type": "int", "range": [0,3], "default": 0, "client_sync": true },
"loot:locked": { "type": "bool", "default": false, "client_sync": true },
"loot:lock_anim": { "type": "int", "range": [0,5], "default": 0, "client_sync": true },
"loot:attacking": { "type": "bool", "default": false, "client_sync": true },
"loot:weapon_state": { "type": "int", "range": [0,3], "default": 0, "client_sync": true },
"loot:crossbow_phase": { "type": "int", "range": [0,2], "default": 0, "client_sync": true },
"loot:hyper": { "type": "bool", "default": false, "client_sync": true },
"loot:dizzy": { "type": "bool", "default": false, "client_sync": true }
```

**CRITICAL**: `loot:digestion_timer` default MUST be `20.0` not `20` — Bedrock silently kills entire properties block if float default is written as integer.

---

## ⚠️ CRITICAL TECHNICAL NOTES

1. **Float defaults** — always `20.0` not `20`
2. **`has_property` not `is_property`** in behavior file filters
3. **Animation keys** use `animation.category.name` format
4. **No spaces in bone names** — confirmed clean in current geo
5. **`pre_animation` in client entity scripts** — not in render controller
6. **Never auto-deploy Bridge to com.mojang** — causes duplicate packs
7. **`minecraft:pushable` removed** — not valid in current Bedrock versions
8. **`consume_item` not valid** in interact interactions — use `use_item: true`
9. **`despawn_from_distance`** wrapper required for min/max distance values
10. **Script module must be in BP manifest** — without it script never runs

---

## 📐 STAGE TRANSITION TEMPLATE

When writing `goblin_one` through `goblin_three`, use this transformation structure:

```json
"minecraft:transformation": {
    "into": "loot:goblin_one",
    "transformation_sound": "goblin.rumble",
    "keep_owner": true,
    "keep_level": false,
    "drop_equipment": false,
    "drop_inventory": false,
    "preserve_equipment": true
}
```

Change `"into"` for each stage. Regression uses same structure pointing to previous stage.

---

## 💡 SCRIPT PRIORITIES (When Writing goblin_script.js)

Build the script in this order — test after each addition:

1. **Texture randomization at spawn** — `query.variant`, `query.skin_id`, `loot:hair_index`, `loot:name`
2. **Nametag checking** — maps name to `loot:name` index
3. **Digestion timer** — lean ticks, belly rub −4s, hit +1s, regression at 0
4. **Hyper/dizzy timers** — 15s each, revert to stand/tired after
5. **Healing bias** — stage-based honey vs food percentage
6. **Stage transformation** — fires correct minecraft:transformation event
7. **Pop system** — explosion timing, fake-out unlock
8. **Bartering** — gold payment detection, stolen loot drop
9. **Death loot** — 50% per slot on death
10. **Mercy tame** — detect crouching right-click during pop_fake

**Nametag map:**
```javascript
const NAMETAG_SKINS = {
    "Poppy": 0, "Tristana": 1, "Malvah": 2,
    "Smug": 3, "Gobu": 4, "Jackie": 5
};
```

**Entity identifiers:**
```javascript
const GOBLIN_IDS = [
    "loot:goblin_zero", "loot:goblin_one",
    "loot:goblin_two", "loot:goblin_three"
];
```

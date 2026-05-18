# Inflatable Loot Goblin - Full Design Reference (v10)

## 📋 PROJECT OVERVIEW

**Platform**: Bedrock Edition (Behavior Pack + Resource Pack + Scripting API)  
**Editor**: VS Code with Blockception extension + Error Lens  
**Model Format**: Blockbench (`.geo.json`, `.animation.json`)  
**Source Mod**: Expandable Goblins (Java, Forge 1.20.1) by irritator-fan-nsfw  
**Pack Name**: Inflatable Loot Goblin  
**Namespace**: `loot:`  
**BP UUID**: a1b2c3d4-e5f6-7890-abcd-ef1234567890  
**RP UUID**: d4e5f6a7-b8c9-0123-defa-234567890123  

---

## 🧠 HOW IT ALL FITS TOGETHER

The Goblin Girl is a tameable mob with four stages of growth driven by honey consumption. She starts slim and hostile, grows larger and tankier as she drinks, and regresses over time through rest and player interaction. Wild goblins are greedy and dangerous — they pick up gold items, steal loot, and can only be reasoned with through gold. Tamed goblins are loyal companions that fight for their owner, accept normal food, and are protected from player-induced popping. The core gameplay loop is: encounter → wear gold to approach safely → tame or barter → feed and manage her size → keep her comfortable through belly rubs and rest.

---

## ✅ CONFIRMED ASSETS

### Geometry File (`goblins.geo.json`)
| Identifier | Used By | Notes |
|---|---|---|
| `geometry.goblin_zero` | `loot:goblin_zero` | Stage 0 |
| `geometry.goblin_one` | `loot:goblin_one` | Stage 1 |
| `geometry.goblin_two` | `loot:goblin_two` | Stage 2 |
| `geometry.goblin_three` | `loot:goblin_three` | Stage 3 |
| `geometry.honey` | `loot:bottle` | Honey bottle prop |
| `geometry.food` | `loot:food` | Food item prop |

### Texture System
Temporary: six named textures used until layered system is implemented.

**Target layered system (future):**
| Layer | Files | Count | Query |
|---|---|---|---|
| Skin + eyes | `skin/goblin0–9.png` | 10 | `query.variant` |
| Outfit | `outfit/cloth0–9.png` | 10 | `query.skin_id` |
| Hair | `style/hair0–15.png` | 16 | `loot:hair_index` |

### Nametag Easter Eggs
| File | Trigger |
|---|---|
| `name/poppy.png` | "Poppy" |
| `name/tristana.png` | "Tristana" |
| `name/malvah.png` | "Malvah" |
| `name/smug.png` | "Smug" |
| `name/gobu.png` | "Gobu" |
| `name/jackie.png` | "Jackie" |

- Script sets `loot:name` (-1=none, 0–5=override)
- Render controller checks `loot:name` first

### Animation File (`goblins.animation.json`)
30 animations across four categories. See `animation_reference.md` for full details.

**Categories:** `animation.idle.*`, `animation.lock.*`, `animation.move.*`, `animation.combat.*`

---

## 🗡️ GOBLIN TYPES & WEAPON SYSTEM

### Spawn Variants
| Type | Spawn Chance | Starting Weapon |
|---|---|---|
| Normal | 65% | None |
| Ranged | 25% | Crossbow (pre-loaded) |
| Mace | 10% | Mace |

### Weapon AI Priority
1. **Mace** — closes to melee, uses `combat.mace` ✅
2. **Crossbow** — ranged, `combat.load` → `combat.shoot` layered on `combat.stand` / `combat.walk`, can move while firing
3. **Gold weapon / tool** — melee, uses `combat.swing`
4. **Unarmed** — fallback, uses `combat.punch`

### Gold Item Pickup
| Item | Effect |
|---|---|
| Honey bottle | Hand-fed only → drinking → stage advance |
| Gold food | Floor or hand-fed → displays 5s → inventory |
| Normal food | Hand-fed tamed only → eating animation |
| Gold sword / axe / tools | Equipped in `rightItem` |
| Mace | Equipped in `rightItem` |
| Gold ingot | Kept — valid barter currency |
| Gold nugget | Kept — no barter value |
| Any other item | Stored as stolen loot |

---

## 🤝 TAMING

- **Method**: Hand-fed golden carrot, golden apple, or glistering melon slice
- **Chance**: ~15% per attempt
- **Any stage** — goblins self-drink so can be found at any stage
- Wild goblins non-hostile when player wears any gold armor piece

### Mercy Tame (Stage 3 pop_fake only)
- During `lock.pop_fake` (30s window), nearby **mobs cannot target her**
- Player crouches + right-clicks → **instant guaranteed tame**
- Works whether triggered by player or self-healing
- Expires after 30s → returns to hostile if no mercy rub

---

## ⚔️ PIGLIN-STYLE GOLD BEHAVIOR

- Wild goblins hostile by default
- Non-hostile when player wears any gold armor piece
- **Any goblin can be force-fed honey** — even while hostile (pop-for-defense mechanic)
- Tamed goblins never hostile to owner

---

## 💰 BARTERING SYSTEM

- Wild goblins pick up and store items as stolen loot
- Valid gold payment → drops all stolen non-gold loot
- **Valid**: gold ingot, sword, axe, tools, mace, gold food
- **Invalid**: gold nugget
- On death: 50% chance per slot to drop stolen loot

---

## 🔢 STAGE SYSTEM

| Stage | Entity ID | Speed | HP |
|---|---|---|---|
| 0 | `loot:goblin_zero` | 0.30 | 20 |
| 1 | `loot:goblin_one` | 0.27 | 24 |
| 2 | `loot:goblin_two` | 0.24 | 28 |
| 3 | `loot:goblin_three` | 0.21 | 32 |

- Each stage = separate entity
- `minecraft:transformation` handles advances and regression
- `keep_owner: true`, `preserve_equipment: true`, `drop_inventory: false`

### Stage 3 Honey Behavior
| Scenario | Outcome |
|---|---|
| Tamed — player hand-feeds | Always `pop_fake` (30s) |
| Tamed — self-induced | 75% fake / 25% real |
| Untamed — either source | 50% fake / 50% real |

---

## 🍯 HONEY BOTTLE MECHANICS

- Hand-fed only — works on any goblin including hostile
- Bottle entity visible until animation ends
- Movement locked during drink and pop animations
- Spawn inventory: 5 honey bottles
- Triggers `idle.dizzy` for 15 seconds after drinking

---

## 🍎 FOOD MECHANICS

| Recipient | Food Type | Effect |
|---|---|---|
| Tamed | Normal food | `lock.eat` immediately + `idle.hyper` 15s |
| Tamed | Gold food | Inventory — backup heal |
| Wild | Gold food | Displays 5s → inventory |
| Wild | Normal food | Rejected |

- Gold food floor pickup displays 5s then stored
- `idle.hyper` only triggers from normal food given by player — not gold food

---

## 💊 HEALING SYSTEM

HP ≤ 8 triggers self-heal. Stage-based bias:

| Stage | Honey % | Food % |
|---|---|---|
| 0 | 100% | 0% |
| 1 | 75% | 25% |
| 2 | 50% | 50% |
| 3 | 25% | 75% |

- Honey heals to full HP
- Food heals to just above 50% HP
- Falls back to available option if preferred unavailable

---

## 📉 REGRESSION SYSTEM

### Digestion Timer (`loot:digestion_timer` — **must be `20.0` not `20`**)

| Event | Effect | Cap |
|---|---|---|
| `lock.rest` passive | −1s per second | Floor: 0 |
| Belly rub (crouch + RC, sitting, tamed) | −4s | Floor: 0 |
| Taking any hit | +1s | Ceiling: 25s |
| Timer hits 0 | Transform to previous stage, reset to 20s | — |

### Idle Pose Bias
| Stage | Sit % | Rest % |
|---|---|---|
| 0 | 100% | 0% |
| 1 | 75% | 25% |
| 2 | 50% | 50% |
| 3 | 25% | 75% |

---

## 🪑 INTERACTIONS TABLE

| Input | Goblin State | Effect |
|---|---|---|
| RC (empty hand) | Standing / resting | Force sit |
| RC (empty hand) | Sitting | Stand up |
| Crouch + RC (empty hand) | Sitting, tamed | Belly rub → −4s timer |
| Crouch + RC (empty hand) | Untamed, `pop_fake` | Instant mercy tame |
| RC (honey bottle) | Any, stages 0–2 | `lock.drink` |
| RC (honey bottle) | Stage 3, tamed | `lock.pop_fake` |
| RC (honey bottle) | Stage 3, untamed | 50/50 pop |
| RC (normal food) | Tamed | `lock.eat` + `idle.hyper` |
| RC (gold food) | Any | Displays 5s → inventory |
| RC (taming item) | Wild, not in pop_fake | 15% tame attempt |

---

## 💥 POP SYSTEM (Stage 3 Only)

| Outcome | Animation | Duration | Result |
|---|---|---|---|
| Real pop | `lock.pop_real` | 4s | Dies from own explosion |
| Fake-out | `lock.pop_fake` | 30s | Recovers OR mercy tame |

- Explosion: `breaks_blocks: false`, `fire: false` — cosmetic only
- Script fires `loot:apply_explosion` at 3.5s mark

---

## 🔊 SOUNDS

| Key | Triggered By |
|---|---|
| `goblin.drink` | `lock.drink` keyframe |
| `goblin.eat` | `lock.eat` keyframe |
| `goblin.pop` | `lock.pop_real` keyframe |
| `goblin.burp` | Script — on regression |
| `goblin.hurt` | Script — on damage |
| `goblin.rumble` | Script — on stage advance |

---

## 🎨 FUTURE FEATURES

| Feature | Notes |
|---|---|
| Layered texture system | Skin + outfit + hair — deferred |
| Belly rub reaction animation | Jiggle + ear wiggle |
| Teleport to owner when too far | Standard tamed behavior |
| Dye cosmetic system | Collar / outfit colors |
| Sit transition animation | `lock.sit_down` — deferred |
| Campsite structure | Far future |

---

## 📁 FILE STATUS

### Behavior Pack
| File | Status |
|---|---|
| `manifest.json` | ✅ |
| `entities/goblin_zero.json` | ✅ |
| `loot_tables/entities/goblin.json` | ✅ (empty) |
| `scripts/goblin_script.js` | ⚠️ Placeholder |
| `entities/goblin_one–three.json` | ❌ |
| `entities/bottle.json` | ❌ |
| `entities/food.json` | ❌ |

### Resource Pack
| File | Status |
|---|---|
| `manifest.json` | ✅ |
| `entity/zero.client.json` | ✅ |
| `render_controllers/goblin_render_controller.json` | ✅ (simplified) |
| `animation_controllers/goblin.animation_controllers.json` | ⚠️ Partial (7 states) |
| `models/entity/goblins.geo.json` | ✅ |
| `animations/goblins.animation.json` | ✅ (30 animations) |
| `textures/entity/name/` (6) | ✅ |
| `textures/entity/skin/` (10) | ✅ |
| `textures/entity/outfit/` (10) | ✅ |
| `textures/entity/style/` (16) | ✅ |
| `sounds/sound_definitions.json` | ✅ |
| `particles/burp_bubble.json` | ⚠️ Needs format fix |
| `entity/one–three.client.json` | ❌ |
| `entity/bottle.client.json` | ❌ |
| `entity/food.client.json` | ❌ |
| `texts/en_US.lang` | ✅ |

---

## 🎯 NEXT BUILD STEPS

1. Test current build — stand, walk, jump, fall, land, anim_2, sitting, punch
2. Fix jump/land chain if still not working
3. Add texture cycling through named textures (script)
4. Write full animation controller (all 30 animations)
5. Write `goblin_one` through `goblin_three` behavior files
6. Write remaining client entities
7. Write prop entity files (bottle, food)
8. Write full script

---

## 💡 DESIGN NOTES

| # | Note |
|---|---|
| 1 | Four separate entities — `minecraft:transformation`, no flicker |
| 2 | 30 animations, four categories — see animation_reference.md |
| 3 | Stage 3 uses pop instead of drink |
| 4 | Tamed stage 3 = always pop_fake |
| 5 | Untamed stage 3 = 50/50 |
| 6 | pop_fake = 30s |
| 7 | `lock.rest` passive digestion |
| 8 | Belly rub −4s, hits +1s, cap 25s |
| 9 | No pose activation delay |
| 10 | Honey hand-fed only, works on hostile |
| 11 | Normal food tamed only — triggers hyper |
| 12 | Gold food → inventory, no hyper |
| 13 | Healing bias mirrors rest/sit ratio |
| 14 | Bartering all gold except nugget |
| 15 | 50% death loot drop |
| 16 | Mercy tame via pop_fake belly rub |
| 17 | Mob protection during pop_fake |
| 18 | `move.hurt` always layers, no exceptions |
| 19 | `move.target` blocked by dizzy, death, anims, rest, pops, eat, drink |
| 20 | `idle.tired` replaces stand at ≤50% HP |
| 21 | `idle.dizzy` 15s after honey, blocks target |
| 22 | `idle.hyper` 15s after normal food from player |
| 23 | `move.swim` cancels combat AI |
| 24 | `combat.death` overrides everything |
| 25 | `combat.mace` complete ✅ |
| 26 | Layered textures deferred — using name textures |
| 27 | `loot:digestion_timer` must be `20.0` not `20` |
| 28 | Use `has_property` not `is_property` |
| 29 | Never auto-deploy Bridge to com.mojang |
| 30 | Texture cycling through named variants until layered system ready |

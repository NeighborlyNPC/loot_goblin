# Inflatable Loot Goblin - Animation Reference Sheet (v2)

## 📋 ANIMATION FILE
**File**: `goblins.animation.json`  
**Total animations**: 30  
**Naming convention**: `animation.category.name`

---

## 🎭 LAYERING RULES

Some animations always layer on top of others regardless of current state:

| Animation | Layers Over | Excluded From |
|---|---|---|
| `move.target` | Everything | `lock.eat`, `lock.drink`, `lock.pop_real`, `lock.pop_fake`, `idle.anim_1`, `idle.anim_2`, `lock.rest`, `idle.dizzy`, `combat.death` |
| `move.hurt` | Everything | Nothing — always applies |
| `idle.blink` | Idle states only | `lock.rest` |

---

## 🧍 IDLE CATEGORY

### `animation.idle.stand`
- **Loop**: Yes
- **Trigger**: Default standing state — plays when stationary, healthy, and no other idle active
- **Replaces**: Nothing — this is the base
- **Notes**: Procedural Molang physics on all major bones — breathing, ear wiggle, head sway, belly/chest physics

### `animation.idle.blink`
- **Loop**: No (0.25s)
- **Trigger**: Layered on top of idle states randomly — fires every 5–10 seconds
- **Excluded from**: `lock.rest`
- **Notes**: Scales eye/pupil bones, drops eyelash positions

### `animation.idle.hyper`
- **Loop**: Yes
- **Trigger**: Tamed goblin receives **normal food** from player → plays for 15 seconds → reverts to `idle.stand` or `idle.tired`
- **Notes**: Excited/energized idle. Gold food does NOT trigger this — goes to inventory silently. Only stationary — walking uses `move.walk` normally during hyper.

### `animation.idle.dizzy`
- **Loop**: Yes
- **Trigger**: Any goblin drinks honey → plays for 15 seconds → reverts to `idle.stand` or `idle.tired`
- **Notes**: Dazed/unfocused idle. Blocks `move.target` — she loses focus while dizzy. Honey always heals to full so `idle.tired` won't activate after dizzy unless she takes damage.

### `animation.idle.tired`
- **Loop**: Yes
- **Trigger**: HP drops to 50% or below → replaces `idle.stand` as default standing idle
- **Deactivates**: When HP rises above 50%
- **Notes**: Replaces `idle.stand` only — does not affect walking or other states. Zombie piglin inspired — looks exhausted and barely alive.

### `animation.idle.anim_1`
- **Loop**: No (short)
- **Trigger**: Random timer — **tamed goblins only**
- **Notes**: Curious or playful gesture. Returns to `idle.stand` or `idle.tired` when complete. `move.target` does not layer over this.

### `animation.idle.anim_2`
- **Loop**: No (short)
- **Trigger**: Random timer — **wild goblins only / hostile state**
- **Notes**: Threatening or aggressive gesture. Returns to `idle.stand` or `idle.tired` when complete. `move.target` does not layer over this.

---

## 🔒 LOCK CATEGORY
All lock animations set movement speed to 0 for their duration.

### `animation.lock.rest`
- **Loop**: Yes
- **Trigger**: Idle pose system rolls lean/rest based on stage bias. Passive digestion ticks while active.
- **Stage bias**: Stage 0: 0% / Stage 1: 25% / Stage 2: 50% / Stage 3: 75%
- **Notes**: Renamed from leaning. Player hit interrupts and returns to stand. `move.target` and `idle.blink` do not layer over this.

### `animation.lock.drink`
- **Loop**: No — hold_on_last_frame (~2s)
- **Trigger**: Player hand-feeds honey bottle. Stages 0–2 only.
- **Notes**: Bottle entity visible until complete. Stage advances via `minecraft:transformation` after.

### `animation.lock.eat`
- **Loop**: No — hold_on_last_frame (~2s)
- **Trigger**: Tamed goblin receives normal food (immediate) OR self-heals with gold food from inventory.
- **Notes**: Food entity visible until complete. No stage change.

### `animation.lock.sitting_0_1`
- **Loop**: Yes
- **Trigger**: Idle pose system rolls sit OR tamed owner force-sits. **Stages 0 and 1 only.**
- **Notes**: Belly rub available via crouch + right-click. Right-click again to stand.

### `animation.lock.sitting_2_3`
- **Loop**: Yes
- **Trigger**: Idle pose system rolls sit OR tamed owner force-sits. **Stages 2 and 3 only.**
- **Notes**: Same interactions as `sitting_0_1`.

### `animation.lock.pop_real`
- **Loop**: No — hold_on_last_frame (4s)
- **Trigger**: Stage 3 honey — real pop outcome.
- **Notes**: Entity scales to 0 at 3.5s. Script fires explosion at 3.5s mark. Natural death — no kill command.

### `animation.lock.pop_fake`
- **Loop**: No — hold_on_last_frame (30s)
- **Trigger**: Stage 3 honey — fake-out outcome. Tamed player-fed = always this.
- **Notes**: Fully immobilized. Nearby mobs cannot target goblin. Tamed owner crouches + right-clicks for instant mercy tame. `move.target` does not layer over this.

### `animation.lock.admire`
- **Loop**: No — hold_on_last_frame (~2–3s)
- **Trigger**: Goblin picks up a gold item from the ground.
- **Notes**: Examines item with greedy excitement. Uses `minecraft:admire_item` component.

---

## 🏃 MOVE CATEGORY

### `animation.move.walk`
- **Loop**: Yes
- **Trigger**: `query.modified_move_speed > 0.1` and not in combat stance
- **Notes**: Full body walk cycle with hair and belly physics.

### `animation.move.target`
- **Loop**: Yes
- **Trigger**: Goblin has an active target — layered on top of most states
- **Excluded from**: `lock.eat`, `lock.drink`, `lock.pop_real`, `lock.pop_fake`, `idle.anim_1`, `idle.anim_2`, `lock.rest`, `idle.dizzy`, `combat.death`
- **Notes**: Allows head to track target independently from body movement.

### `animation.move.hurt`
- **Loop**: No (short)
- **Trigger**: Goblin takes any damage — always layers regardless of current state
- **Notes**: Brief flinch/recoil. Interrupts visually but does not stop current animation state from continuing underneath.

### `animation.move.jump`
- **Loop**: No
- **Trigger**: Goblin leaves ground
- **Notes**: Chains into `move.fall` if airborne longer than the jump animation.

### `animation.move.fall`
- **Loop**: Yes
- **Trigger**: Goblin is airborne after jump
- **Notes**: Loops while falling. Chains into `move.land` on ground contact.

### `animation.move.land`
- **Loop**: No (short ~0.3–0.5s)
- **Trigger**: Goblin hits ground after falling
- **Notes**: Impact reaction. Returns to `idle.stand` or `move.walk` after completion.

### `animation.move.swim`
- **Loop**: Yes
- **Trigger**: Goblin is in water
- **Notes**: Surface doggy paddle. Cancels all combat AI while active — she can barely swim so all focus goes to staying afloat. No water damage.

---

## ⚔️ COMBAT CATEGORY

### `animation.combat.stand`
- **Loop**: Yes
- **Trigger**: Has active target AND stationary AND crossbow equipped (`loot:weapon_state == 2`)
- **Notes**: Crossbow raised combat stance. `combat.load` and `combat.shoot` layer on top.

### `animation.combat.walk`
- **Loop**: Yes
- **Trigger**: Has active target AND moving AND crossbow equipped (`loot:weapon_state == 2`)
- **Notes**: Crossbow raised walking stance. Allows movement while loading/shooting.

### `animation.combat.punch`
- **Loop**: No
- **Trigger**: Melee attack — unarmed (`loot:weapon_state == 0`)
- **Notes**: Basic arm swing forward.

### `animation.combat.mace`
- **Loop**: No
- **Trigger**: Melee attack — mace equipped (`loot:weapon_state == 3`)
- **Notes**: Heavy downward smash. ✅ Complete.

### `animation.combat.swing`
- **Loop**: No
- **Trigger**: Melee attack — sword, axe, or tool equipped (`loot:weapon_state == 1`)
- **Notes**: Overhead slash/swing.

### `animation.combat.load`
- **Loop**: No — hold_on_last_frame (~0.7s)
- **Trigger**: Crossbow charge — `loot:crossbow_phase == 1`. Layered on `combat.stand` or `combat.walk`.
- **Notes**: Arms pull back loading crossbow.

### `animation.combat.shoot`
- **Loop**: No — hold_on_last_frame (~0.7s)
- **Trigger**: Crossbow fire — `loot:crossbow_phase == 2`. Layered on `combat.stand` or `combat.walk`.
- **Notes**: Arms release and recoil.

### `animation.combat.death`
- **Loop**: No — hold_on_last_frame
- **Trigger**: Goblin dies — overrides and cancels ALL current animations
- **Notes**: Applies to both tamed and wild goblins. `move.target` does not layer over this. Entity removed after animation completes naturally.

---

## 🎮 ANIMATION CONTROLLER STATES SUMMARY

| State | Animations | Condition |
|---|---|---|
| `standing` | `idle.stand` + `idle.blink` + `move.target` (layered) | Default stationary, healthy |
| `tired` | `idle.tired` + `move.target` (layered) | HP ≤ 50% AND stationary |
| `hyper` | `idle.hyper` + `move.target` (layered) | `loot:hyper == true` AND stationary |
| `dizzy` | `idle.dizzy` | `loot:dizzy == true` AND stationary |
| `anim_1` | `idle.anim_1` | Random timer, tamed only |
| `anim_2` | `idle.anim_2` | Random timer, wild only |
| `sitting_01` | `lock.sitting_0_1` | `loot:idle_pose == 1`, stages 0–1 |
| `sitting_23` | `lock.sitting_2_3` | `loot:idle_pose == 1`, stages 2–3 |
| `resting` | `lock.rest` | `loot:idle_pose == 2` |
| `walking` | `move.walk` + `move.target` (layered) | Moving, no crossbow combat |
| `swimming` | `move.swim` | In water |
| `jumping` | `move.jump` | Left ground |
| `falling` | `move.fall` | Airborne |
| `landing` | `move.land` | Hit ground after fall |
| `combat_stand` | `combat.stand` + load/shoot layers | Has target, stationary, crossbow |
| `combat_walk` | `combat.walk` + load/shoot layers | Has target, moving, crossbow |
| `melee` | `combat.punch` / `combat.swing` / `combat.mace` | Has target, melee weapon |
| `locked` | `lock.drink` / `lock.eat` / `lock.pop_real` / `lock.pop_fake` / `lock.admire` | `loot:locked == true` |
| `death` | `combat.death` | On death |

---

## 📝 PROPERTIES USED BY ANIMATION CONTROLLER

| Property | Type | Values | Purpose |
|---|---|---|---|
| `loot:idle_pose` | int | 0=stand, 1=sit, 2=rest | Current idle pose |
| `loot:locked` | bool | true/false | Movement locked |
| `loot:lock_anim` | int | 0=drink, 1=eat, 2=pop_real, 3=pop_fake, 4=admire | Which lock animation plays |
| `loot:weapon_state` | int | 0=unarmed, 1=sword, 2=crossbow, 3=mace | Current weapon |
| `loot:crossbow_phase` | int | 0=idle, 1=load, 2=shoot | Crossbow animation phase |
| `loot:hyper` | bool | true/false | Post normal-food excited state |
| `loot:dizzy` | bool | true/false | Post-honey dazed state |
| `loot:attacking` | bool | true/false | Currently attacking |

---

## ⚠️ NOTES

- `move.hurt` is the only animation that layers over absolutely everything with no exceptions
- `idle.blink` does not play during `lock.rest`
- `idle.tired` replaces `idle.stand` only — all other states unaffected by health
- `idle.dizzy` and `idle.hyper` are mutually exclusive in practice — honey heals to full so tired won't follow dizzy unless damage is taken
- Jump → fall → land chain managed by `query.is_on_ground`
- `animation.combat.death` overrides everything and is not interrupted by anything
- `move.swim` cancels all combat AI — she cannot attack while swimming
- `lead` bone present for leash attachment

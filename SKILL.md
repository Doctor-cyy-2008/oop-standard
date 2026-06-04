---
name: oop-standard
description: >
  Apply object-oriented programming standards when writing, reviewing, or refactoring
  JavaScript/TypeScript code. Use this skill whenever the user asks you to write classes,
  review OOP code, refactor legacy code into clean architecture, or mentions class design,
  naming conventions, SOLID principles, code organization, or "clean code" patterns.
  Also triggers on requests to "improve code structure", "make this more reusable",
  "extract modules", or any task involving converting procedural code to object-oriented.
---

# OOP Coding Standard

Enforce consistent, modern object-oriented programming conventions in all JavaScript/TypeScript code.

---

## The Prime Directive: Preserve Behavior

> **When refactoring existing code, every feature, animation, edge case, and side effect
> must survive the transformation.** The goal is to reorganize, not to rewrite.

If you are refactoring an existing codebase, the flow is:

1. **Read everything first.** Don't touch code until you understand every function, every
   DOM interaction, every event listener, every animation callback.
2. **Catalog all behaviors.** Make a mental (or written) list: "ground textures, harvest
   animations, gold flying effect, touch support, save/load, weather particles…"
3. **Refactor incrementally.** Wrap existing code in class structures — do NOT delete and
   rewrite. The original line of code should still execute; it just lives inside a method now.
4. **Verify after each step.** After extracting one class, check that all features still work.
5. **The original code IS the specification.** If the old code does something, the new code
   must do it too, even if it seems like a weird edge case.

**Why this matters:** A "clean" rewrite that loses half the features is worse than messy code
that works. Users don't see your elegant architecture — they see missing features.

---

## Refactoring Workflow (For Existing Code)

Follow these steps when transforming procedural / monolithic code into object-oriented:

### Step 1: Merge and Understand
- If code is split across multiple `<script>` files, merge them into one first so you can
  see the full picture.
- Read the ENTIRE codebase before making any changes.
- Identify: global state, DOM dependencies, 3D/rendering dependencies, and cross-cutting
  concerns (save/load, animations, event handlers).

### Step 2: Catalog Features
List every observable behavior. For a game, this includes:
- Rendering features (textures, sprites, animations, particle effects)
- Interaction features (click, hover, drag, touch, tool modes)
- Data features (save/load, reset, offline compensation)
- UI features (panels, toasts, tooltips, progress bars)

### Step 3: Identify Class Boundaries
Group related functions and data. Good boundaries:
- **Data classes**: configuration, registry, state
- **Logic classes**: operations on that data
- **Render classes**: visual representation
- **Orchestrator**: wires everything together

Do NOT force every single function into a class. Module-level constants and utility
functions are fine. The goal is organization, not dogmatism.

### Step 4: Wrap, Don't Rewrite
- Take the original function body and move it into a method. The logic stays identical.
- Replace `var` with `let`/`const` (safe transformation).
- Add JSDoc to the method signature.
- Test that the behavior is unchanged.

### Step 5: Verify Completeness
After refactoring, go through your catalog list and verify each feature is present.
If anything is missing, add it back — even if it doesn't fit the "clean" architecture.
A working feature behind a slightly awkward delegation is better than a missing feature.

---

## Environment Considerations

The module system you use depends on the target environment:

**Browser with server** (`http://` or `https://`):
- ES modules (`import`/`export`) work fully.
- Use `<script type="module">`.

**Electron / `file://` protocol:**
- ES modules are blocked by CORS when loading from `file://`.
- Solution: bundle all JS into a single `<script>` block (no imports).
- Remove `import`/`export` statements, order classes by dependency, and embed inline.
- Keep the architecture clean — the bundling is a deployment concern, not a design concern.

**General rule:** Design with ES modules. Bundle for deployment when targeting `file://`.

---

## Naming Conventions

### Class names → PascalCase (大驼峰)

Classes are constructor functions — PascalCase signals "this creates objects" instantly.

```javascript
// Correct
class UserAccount {}
class CropManager {}
class WeatherSystem {}

// Wrong
class userAccount {}
class crop_manager {}
```

### Method and property names → camelCase (小驼峰)

```javascript
// Correct
class CropManager {
  plantedCells;
  getGrowthRatio(index) {}
  harvestAll() {}
}

// Wrong — inconsistent styles from legacy code
class CropManager {
  PlantedCells;
  get_growth_ratio(index) {}
  HarvestAll() {}
}
```

### Constants → UPPER_SNAKE_CASE (全大写+下划线)

```javascript
// Correct
const DAY_SECONDS = 24 * 60;
const MAX_RETRY_COUNT = 3;
const DEFAULT_FERTILIZER_COUNT = 20;

// Wrong
const daySeconds = 24 * 60;
const maxRetryCount = 3;
```

**Real-world example** — extracting magic numbers from legacy code:
```javascript
// Before (scattered magic values)
function buyFertilizer() {
  if (window.goldCount < 100) { showToast('need 100 gold!'); return false; }
  // ...
}
function buyWater() {
  if (window.goldCount < 5) { showToast('need 5 gold!'); return false; }
  // ...
}

// After (extracted constants)
const FERTILIZER_PRICE = 100;
const WATER_PRICE = 5;
// Now used in buyFertilizer() and buyWater()
```

---

## Architecture Principles

### Single Responsibility (单一职责)

A class should have exactly one reason to change.

**Real-world example** — splitting a monolithic crop system:
```javascript
// Before: one 642-line file with everything mixed together
// crop-system.js contained:
//   - 10 crop definitions (data)
//   - Planting/harvesting logic (business logic)
//   - Texture loading/caching (rendering)
//   - UI population (dom manipulation)
//   - Buy/sell operations (shop)
//   - Save/load (persistence)
//   All as global functions sharing global variables.

// After: separate concerns
class CropRegistry {
  // Only crop data definitions
  getAll() { return this.#entries; }
}
class CropManager {
  // Only planting/harvesting/growth logic
  plant(index, cropId) { /* ... */ }
  harvest(index) { /* ... */ }
}
class Inventory {
  // Only seed/fruit/resource counts
  consumeSeed(name) { /* ... */ }
}
class SaveManager {
  // Only persistence
  save() { /* ... */ }
  load() { /* ... */ }
}
```

### Prefer Composition over Inheritance (组合优于继承)

Inheritance couples subclass to superclass. Composition lets you mix behaviors at the instance level.

```javascript
// Correct — compose behaviors
class PositionComponent {
  #x; #y;
  update(deltaTime) { /* ... */ }
}
class Player {
  #position = new PositionComponent();  // has-a
  #inventory = new Inventory();         // has-a
}
class Enemy {
  #position = new PositionComponent();  // has-a
  #ai = new AIBehavior();              // has-a
}

// Wrong — deep inheritance chain
class GameObject {}
class LivingEntity extends GameObject {}
class Humanoid extends LivingEntity {}
class Player extends Humanoid {}
// What happens when you need a Robot that shares Player behavior but isn't a LivingEntity?
```

**When inheritance IS appropriate:** Only for true "is-a" relationships with max depth 2.
`Player extends GameObject` is fine. `Player extends Humanoid extends LivingEntity extends GameObject` is not.

---

## Documentation

### All public methods must have JSDoc

```javascript
// Correct
class CropManager {
  /**
   * Plant a crop on the specified grid cell.
   * @param {number} index - Grid cell index (0-99)
   * @param {number} cropId - Crop type ID from CropRegistry
   * @returns {{ success: boolean, message?: string }}
   */
  plant(index, cropId) {
    // implementation
  }
}
```

**Guideline:**
- `@param` with type for every parameter
- `@returns` with type for return values
- `@throws` for explicitly thrown errors
- Private methods don't require JSDoc but benefit from inline comments

---

## Encapsulation

### Private fields use `#` prefix

```javascript
// Correct — true private fields
class Wallet {
  #gold = 0;

  getGold() { return this.#gold; }
  add(amount) { this.#gold += amount; }
}

// Wrong — pseudo-private convention
class Wallet {
  _gold = 0;  // anyone can still access wallet._gold
}
```

**When wrapping procedural code in classes:** Existing global variables that become
instance properties should use `#` if they're internal state. Variables that are
genuinely shared across multiple classes (e.g., a Three.js scene reference) may
remain as constructor-injected references.

---

## Quick Checklist

When writing or refactoring a class, verify:

- [ ] Class name is PascalCase?
- [ ] All methods/properties are camelCase?
- [ ] Constants extracted and UPPER_SNAKE_CASE?
- [ ] Class has exactly one responsibility?
- [ ] Behaviors are composed, not inherited?
- [ ] Every public method has JSDoc?
- [ ] Internal state uses `#` private fields?
- [ ] **For refactoring: ALL original features preserved?**
- [ ] **For refactoring: Original code wrapped, not rewritten?**

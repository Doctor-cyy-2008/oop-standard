# oop-standard

> A Claude Code skill that enforces modern OOP conventions in JavaScript/TypeScript — for writing new code, reviewing existing code, and refactoring legacy procedural code into clean, composable classes.

## Quick Start

```bash
mkdir -p ~/.claude/skills/oop-standard
cp SKILL.md ~/.claude/skills/oop-standard/
```

Triggers automatically when you ask Claude to write classes, review OOP code, or refactor legacy code. Or invoke explicitly: `/oop-standard`

## Rules Enforced

| # | Rule | Good | Bad |
|---|------|------|-----|
| 1 | Class names → **PascalCase** | `class InventoryManager` | `class inventory_manager` |
| 2 | Methods/properties → **camelCase** | `getGrowthRatio()` | `get_growth_ratio()` |
| 3 | Constants → **UPPER_SNAKE_CASE** | `const MAX_ITEMS = 1000` | `const maxItems = 1000` |
| 4 | **Single Responsibility** | `CropManager` only does crops | One class doing crops + shop + save |
| 5 | **Composition over Inheritance** | `Player` composes `Inventory` | 4-level inheritance chain |
| 6 | Public methods → **JSDoc** | `@param` / `@returns` / `@throws` | No documentation |
| 7 | Private fields → **`#` prefix** | `#balance` | `_balance` or `this.balance` |

## Two Modes

### Writing New Code

All 7 rules enforced at generation time.

### Refactoring Existing Code

Includes a **Prime Directive**: *preserve all existing behavior.*

1. Read everything first
2. Catalog all features
3. Wrap — don't rewrite
4. Verify each feature survives

This prevents the most common failure: clean code that lost functionality.

## Evaluation Results

Tested on 3 real-world JavaScript refactoring tasks:

| Metric | With Skill | Without |
|--------|:----------:|:-------:|
| Overall Pass Rate | **100%** | 43% |
| Naming conventions | 3/3 | 2/3 |
| JSDoc coverage | 3/3 | 0/3 |
| `#` private fields | 3/3 | 0/3 |
| Single Responsibility | 3/3 | 1/3 |
| Composition over inheritance | 3/3 | 1/3 |

## Companion Skill

**[oop-framework-first](https://github.com/cheng/oop-framework-first)** — contract-driven, abstraction-first OOP framework development. An 8-step methodology for building new systems from scratch with DDD, SOLID, design patterns, and TDD against interfaces.

## License

MIT © 2026 cheng

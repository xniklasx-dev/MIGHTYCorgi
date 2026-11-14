# ADR-01 Modular Monolith

**Status:** accepted
**Date:** 14.11.2026
**Author:** Andreas

**Context.** Our game consists of distinct domain areas: Character, Combat, Item, Enemy, World, and Progress. We need an architectural style that allows our team to work in parallel on different game features without constant merge conflicts, while keeping the game logic clean and maintainable.

**Considered options.**
1. Modular Monolith (cut by domain / game systems).
2. Layered Architecture (cut by technical concerns like Rendering, Logic, Data).

**Decision.** We will use option 1, the Modular Monolith. It allows for easier changes to individual systems (e.g., updating the Combat system doesn't require touching World or Progress) and provides a clearer separation for our game-specific use cases than a classic Layered Architecture.

**Consequences.**
* Good: High cohesion within game systems; developers can take ownership of specific modules.
* Bad: We need strict dependency rules (e.g., who is allowed to access the Combat module) to avoid a tangled web of dependencies between the modules.

# Pipe-works

> **Building tools for accountable, procedural interactive fiction**

Welcome to **pipe-works**—an open-source ecosystem of interconnected tools for creating online interactive fiction that emphasizes determinism, procedural generation, and player agency. All projects are GPL-3.0 licensed and designed around a core philosophy: **systems should be comprehensible, reproducible, and resistant to optimization**.

---

## The Pipe-works Ecosystem

`pipe-works` is not a single application but a collection of specialized, composable tools that support interactive fiction development. It is composed of three distinct layers:

1. **The Tools**: A suite of standalone Python libraries for generation and simulation.
2. **The Workshop**: `pipe-works.org`, a public space for live demonstrations and development.
3. **The Game**: *The Daily Undertaking*, a playable MUD that integrates all the tools.

---

### 📰 The Daily Undertaking

**The project that started it all.**

The Daily Undertaking is the living, playable interactive fiction world designed as a personal project first that `pipe-works` was built to support. The tools in this organization exist because they were needed to solve real problems encountered during its development—a process of building the pickaxe because we needed to mine the mountain. It is the "why" behind the "how."

It is a complete, standalone game composed of two primary parts:

1. A **Python-based backend** (the `pipeworks_mud_server`) that runs the deterministic game world.
2. A **vanilla HTML/CSS/JS frontend** ([the_daily_undertaking_ui](https://github.com/aa-parky/the_daily_undertaking_ui)) that provides the vintage, newspaper-style player interface.

This is **pipe-works in practice**, where the entire ecosystem is integrated to create a cohesive experience:

- Names are supplied by `pipeworks_name_generation`.
- Character identity is shaped by `pipeworks_entity_state_generation`.
- Portraits are generated as consequences by `pipeworks-image-generator`.
- The world itself is managed and served by `pipeworks_mud_server`.

This is not a demo. It is an actual interactive fiction world that depends on the `pipe-works` ecosystem functioning as a whole. It is what happens when you use the tools to build something real. Public release planned once multiplayer code is complete.

Open, free, GPL-3.0.

> **Goblin Warning**: The current edition is *cough* broken.

---

### 🧬 pipeworks_entity_state_generation

[Repository](https://github.com/pipe-works/pipeworks_entity_state_generation)

This is about **entities as states, not objects**.

Instead of static characters or items, this repo explores:

- entities as layered conditions,
- mutable states over time,
- history as part of identity.

Nothing here is "finished".

Entities carry what happened to them.

---

### 🔤 pipeworks_name_generation

[Repository](https://github.com/pipe-works/pipeworks_name_generation)

This repo treats naming as:

- generation under constraint,
- signal, not decoration.

Names are produced with:

- phonetic structure,
- cultural pressure,
- repeatability without sameness.

It deliberately resists "cool name" thinking.

Names are **tools**, not labels.

---

### 🖼️ Pipeworks-image-generator

[Repository](https://github.com/pipe-works/pipeworks-image-generator)

This explores a key inversion:

> Images are not decoration — they are consequences.

Instead of illustrating lore after the fact, images are:

- generated from actions,
- conditioned by states,
- treated as outputs with provenance.

This makes visual artefacts auditable, not ornamental.

---

### 🧵 pipeworks_mud_server

[Repository](https://github.com/pipe-works/pipeworks_mud_server)

This is the **pressure chamber**.

The MUD server exists to:

- let people interact *before* systems are complete,
- surface social and narrative friction early,
- allow mistakes to happen in public, cheaply.

It is intentionally minimal.

Conversation comes first.

Rules emerge later.

---

### 🌐 pipe-works.org

[Repository](https://github.com/pipe-works/pipe-works-org)

**The public workshop.**

This is not a complete game or a polished platform. It is a deliberate exercise in building with the garage door up, featuring:

- Interactive demonstrations of `pipe-works` components.
- Documented failures and design choices.
- The working philosophy made visible.
- Goblin laws enforced.

The site resists "product" thinking. It shows the tools mid-development, mistakes included, as a living record of what happens when you commit to transparency over presentation.

---

## Core Philosophy

### Determinism Over Optimization

`pipe-works` is built on deterministic systems, not optimized ones. Given the same inputs, conditions, and state, the system must behave the same way every time. Outcomes must be traceable, reproducible, and explainable after the fact. Optimization focuses on engagement and retention; determinism focuses on causality and accountability. `pipe-works` chooses inspectability.

### Conditions as Axes, Not Flags

Entities exist on interpretive spectrums, not in binary states. The system doesn’t ask, "Does this character have the condition?" It asks, **"Where along this axis does interpretation tilt?"** A "weary" character suggests fragility but doesn’t prevent action; it colors perception. This is bias, not prescription.

### Failure as Data

Actions are resolved through axes (e.g., Timing, Precision, Stability). Attributes don’t determine success—they determine **how you fail**. Every failure is recorded in an immutable ledger, creating accountability and narrative.

### Code-First, Not Node-Based

Code is explicit, version-controllable, and infinitely extensible. Visual node editors are not. The same conditional axis system that generates entity states also generates image prompts programmatically. The tools are built for code-based generation, with Gradio used as a development interface for testing, not as a final product. Tools like ComfyUI and InvokeAI served as sketch pads - useful for testing prompting strategies against various models, but fundamentally incompatible with version control and programmatic generation.

### Programmatic Truth, Narrative Interpretation

Game mechanics are **authoritative** and deterministic. They define what can and cannot happen. This authority never belongs to an LLM. Large Language Models are **non-authoritative** and are used only for flavor text, description, and ambient context under strict programmatic constraint. They do not decide outcomes.

---

## How They Connect

```text
┌─────────────────────────────────────────────────────────────────┐
│                    pipeworks_mud_server                         │
│              (The Undertaking - Main Game Server)               │
│                                                                 │
│  Integrates:                                                    │
│  • pipeworks_name_generation → Character name issuance          │
│  • pipeworks_entity_state_generation → Character quirks/failing │
│  • pipeworks-image-generator → Character portraits              │
│  • pipeworks_character_sheet_generator → Sheet generation       │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │
                    ┌─────────┴─────────┐
                    │                   │
        ┌───────────▼──────┐  ┌─────────▼─────────┐
        │ pipeworks_       │  │ pipeworks-        │
        │ name_generation  │  │ image-generator   │
        │                  │  │                   │
        │ Phonetic names   │  │ AI portraits      │
        └──────────────────┘  └───────────────────┘
                    │
        ┌───────────▼──────────────────┐
        │ pipeworks_entity_state_      │
        │ generation                   │
        │                              │
        │ Character conditions,        │
        │ facial signals, occupations  │
        └──────────────────────────────┘
```

Each tool can be used independently, but they're designed to compose together for building complete interactive fiction systems.

## How to Contribute

This project thrives on community involvement. We encourage contributions of all kinds, from documentation improvements to new features.

All repositories in the `pipe-works` organization follow a unified set of development standards, including:

- **Python 3.12+**
- **pytest** with high test coverage
- **black/ruff/mypy** for code quality
- **Conventional Commits** for version control
- **GPL-3.0-or-later** license

To get started, please read the `CONTRIBUTING.md` file in the specific repository you are interested in. For organization-wide standards, issue templates, and reusable CI/CD workflows, we are centralizing our efforts in a dedicated [`.github` repository](https://github.com/pipe-works/.github)

## Documentation

- **To Explore the Philosophy**: Visit [pipe-works.org](https://pipe-works.org)
- **To Generate Names**: See [pipeworks_name_generation documentation](https://pipeworks-name-generation.readthedocs.io/en/latest/)
- **To Generate Character Conditions**: See [pipeworks_entity_state_generation documentation](https://pipeworks-entity-state-generation.readthedocs.io/en/latest/)
- **To Generate Images**: See [pipeworks-image-generator documentation](https://pipeworks-image-generator.readthedocs.io/en/latest/)
- **To Build a MUD**: See [pipeworks_mud_server documentation](https://pipeworks-mud-server.readthedocs.io/en/latest/)

---

## Acknowledgments

- Inspired by classic MUDs and the magic of tinkering with systems
- Built with Python, FastAPI, Gradio, HuggingFace, and love for procedural generation
- Design and development with assistance from Claude (Anthropic)

---

## Why GPL-3.0?

We chose GPL-3.0-or-later because:

1. **Derivative works stay open** - If you build on pipe-works, share your improvements
2. **Community ownership** - No company can privatize this work
3. **Long-term sustainability** - The ecosystem remains accessible to everyone
4. **Alignment with philosophy** - Open tools for open systems

---

Maybe see you over at the [Crooked Pipe](https://www.pipe-works.org/crooked-pipe.html)!

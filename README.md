# 🔥 3JS-GFX-Playbook

**An agent-ready playbook for proposing particle effects, post-processing, shaders and atmosphere in React + Three.js games.**

Built for 3D third-person dungeon crawlers, and useful for any Three.js / React Three Fiber project that needs visual effects that fit its design.

---

## 🧭 What is this?

`3JS-GFX-playbook.md` is a structured reference document for an **AI coding agent** (or a developer) that has full visibility of a game's codebase or design plans.

When you ask an agent *"what effects could we add?"*, it usually answers with a generic list. This playbook gives the agent a repeatable process to follow instead:

1. **Discover** the project: three.js version, R3F/drei usage, WebGL vs WebGPU, existing VFX, art style, performance targets, event system and quality settings.
2. **Decide** on approaches using decision trees and art-style matching.
3. **Select** effects from tiered catalogues (**Cheap / Standard / Premium**).
4. **Propose** them in a consistent format that states why each effect fits, where it integrates, what it costs and what the alternatives are.

The result is a set of proposals that fit your architecture, art direction and performance budget.

---

## 📦 What's inside

| Section | Contents |
|---|---|
| **0. Ground rules** | How the agent should behave: discover first, reuse, tier options, state costs |
| **1. Discovery checklist** | 16 questions to answer from the codebase before proposing anything |
| **2. Decision framework** | Particle approach tree, post-processing choice, art-style matching table |
| **3. Tier definitions** | Cheap / Standard / Premium cost expectations |
| **4. Particle & VFX catalogue** | Environment, combat, magic, loot and interaction effects, with options at each tier |
| **5. Post-processing catalogue** | Bloom, N8AO/SSAO, vignette, LUTs, outlines, shockwaves, recommended pass order |
| **6. Atmosphere & lighting** | Fog, flickering torches, baked lighting, light and shadow budgets |
| **7. Shader effects library** | Dissolve, fresnel rim, hit flash, scrolling UVs, polar runes, heat haze, soft particles |
| **8. Game-feel** | Screen shake, hit-stop, damage numbers, FOV kicks, time slow |
| **9. Performance budgets** | Per-platform limits and hard rules for R3F performance |
| **10. Recommended architecture** | `VFXManager`, pooled emitters, event-driven triggers, quality-aware post stack, with code samples |
| **11. Library quick reference** | drei, postprocessing, three.quarks, wawa-vfx, CSM, TSL/WebGPU, rapier, profiling tools |
| **12. Proposal template** | The exact output format the agent should produce |
| **13. Default first-pass bundle** | Six high-impact, low-cost recommendations to use when information is limited |

---

## 🚀 How to use it

### With an AI coding agent (Claude Code, Cursor, Copilot, etc.)

1. Drop `3JS-GFX-playbook.md` into your repo (e.g. `/docs`), or give it to the agent as context.
2. Prompt the agent, for example:

   > Read `3JS-GFX-playbook.md`, then analyse this codebase and produce a GFX proposal for the combat system following §12.

   > Using the playbook, suggest atmosphere improvements for the dungeon level that fit our art style and mobile performance target.

3. Review the proposal, pick the options you want, and ask the agent to implement them.

### As a human reference

Use the catalogues and tables as a checklist when planning a VFX pass. The performance rules (§9) and architecture patterns (§10) apply whether or not an agent is involved.

---

## 🛠️ Tech coverage

- **three.js** (WebGLRenderer and WebGPURenderer / TSL)
- **React Three Fiber** and **@react-three/drei**
- **postprocessing** / **@react-three/postprocessing** (including N8AO)
- **three.quarks**, **wawa-vfx**, **three-custom-shader-material**
- **GPUComputationRenderer** (GPGPU) and TSL compute particles
- **@react-three/rapier** for physics-driven VFX
- **r3f-perf** / **stats-gl** for profiling

> ⚠️ The Three.js ecosystem moves quickly. The playbook tells agents to verify current package versions and APIs before recommending them.

---

## 🎮 Designed for dungeon crawlers

The examples are tuned for dark, enclosed, combat-heavy 3D games. Examples include torch fire and embers, dust in light shafts, weapon trails, hit sparks, dissolve-on-death, spell projectiles, AoE telegraphs, loot rarity beams and interactable outlines. The discovery and decision sections are genre-agnostic, so the playbook adapts to other Three.js projects too.

---

## 📄 Files

```
3JS-GFX-Playbook/
├── 3JS-GFX-playbook.md   # The playbook
└── README.md             # This file
```

---

## 🤝 Contributing

Suggestions, new effect recipes, corrections for API changes and additional art-style mappings are welcome. Open an issue or submit a pull request at [github.com/MushroomFleet/3JS-GFX-Playbook](https://github.com/MushroomFleet/3JS-GFX-Playbook).

---

## 📚 Citation

### Academic Citation

If you use this codebase in your research or project, please cite:

```bibtex
@software{3js_gfx_playbook,
  title = {3JS-GFX-Playbook: An agent-ready playbook for proposing particle effects, post-processing, shaders and atmosphere in React + Three.js games},
  author = {Drift Johnson},
  year = {2026},
  url = {https://github.com/MushroomFleet/3JS-GFX-Playbook},
  version = {1.0.0}
}
```

### Donate:

[![Ko-Fi](https://cdn.ko-fi.com/cdn/kofi3.png?v=3)](https://ko-fi.com/driftjohnson)

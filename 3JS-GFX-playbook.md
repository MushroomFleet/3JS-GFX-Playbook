# Three.js / React GFX Playbook
**Particle effects, post-processing, shaders and atmosphere for a 3D third-person dungeon crawler**

> **Audience:** an AI agent (or developer) with read access to the codebase and/or design plans.
> **Purpose:** quickly turn "what effects could we add?" into concrete, design-fitting, budget-aware proposals.
> **How to use:** follow §1 → §2 → pick from §4–§8 → present using §10. Don't propose anything before completing §1.

---

## 0. Ground rules for the agent

1. **Discover before proposing.** Every recommendation must cite something found in the codebase or plans (renderer, art style, perf target, existing systems).
2. **Fit the existing architecture.** Extend the patterns already in use (folder layout, state management, component style). Don't introduce a second particle library if one exists.
3. **Offer tiers, not one answer.** For each effect give a Cheap / Standard / Premium option where it makes sense (§3).
4. **State the cost.** Every proposal lists draw calls, fill-rate risk, new dependencies and mobile impact.
5. **Prefer reuse.** One pooled emitter system serving many effects beats many bespoke components.
6. **Verify package versions** (npm / changelogs) before naming APIs. The R3F, drei, postprocessing and three.js WebGPU APIs move quickly.

---

## 1. Discovery checklist (run first)

Scan the repo and plans, and fill this table in. Use the answers to filter every later section.

| # | Question | Where to look | Why it matters |
|---|---|---|---|
| 1 | three.js version? | `package.json`, lockfile | WebGPU/TSL availability (r16x+), API differences |
| 2 | R3F used? Version? | `@react-three/fiber` in deps | Component-style vs imperative effects |
| 3 | drei / postprocessing / rapier present? | deps | Free ready-made effects; physics-driven debris |
| 4 | Renderer: `WebGLRenderer` or `WebGPURenderer`? | Canvas setup, `gl` prop | Decides GLSL vs TSL, compute particles |
| 5 | Existing particle/VFX code? | grep `Points`, `InstancedMesh`, `quarks`, `nebula`, `ShaderMaterial`, `vfx` | Extend, don't duplicate |
| 6 | Existing post-processing? | grep `EffectComposer`, `Bloom`, `postprocessing` | Merge into one composer |
| 7 | Art direction | design docs, textures, materials, README | Realistic / stylised / toon / low-poly / pixel |
| 8 | Lighting approach | grep `PointLight`, `shadow`, lightmaps, `aoMap` | Light budget, bake vs real-time |
| 9 | Target platforms & FPS | docs, settings menus, `dpr` prop | Budget tier (§9) |
| 10 | Quality settings system? | settings store, options menu | Hook effects into quality levels |
| 11 | State management | zustand / redux / context / ECS | How gameplay events trigger VFX |
| 12 | Game event system | combat, damage, death, pickup handlers | Integration points for VFX triggers |
| 13 | Camera rig | third-person controller code | Screen shake, DoF, occlusion handling |
| 14 | Asset pipeline | `public/`, GLTF loaders, texture formats (KTX2?) | Sprite sheets, atlases, compression |
| 15 | Build tool / shader import | Vite/webpack config, `vite-plugin-glsl` | How shaders are authored |
| 16 | TypeScript? Test setup? | `tsconfig`, test dirs | Code style of proposals |

**Output:** a short "Project Profile" (5–10 lines) at the top of any proposal.

---

## 2. Decision framework

### 2.1 Pick the particle approach

```
Is three.quarks / another VFX lib already installed?
 ├─ yes → extend it.
 └─ no
     ├─ Need designer-friendly authoring, many varied effects? → three.quarks
     ├─ Small project, R3F-native, quick wins?                  → drei helpers + wawa-vfx
     ├─ Few effect types, max control/perf?                     → custom Points/InstancedMesh + ShaderMaterial
     └─ Huge counts (100k+), swarms, sims?
          ├─ WebGPURenderer available → TSL compute particles
          └─ WebGL only               → GPUComputationRenderer (GPGPU)
```

### 2.2 Pick the post-processing approach

```
Using R3F? → @react-three/postprocessing (single <EffectComposer>)
Vanilla?   → `postprocessing` (pmndrs) preferred over three/examples EffectComposer
             (it merges effects into fewer passes)
WebGPU?    → three.js TSL post-processing nodes (PostProcessing + pass())
```

### 2.3 Match the art style

| Art style | Lean towards | Avoid |
|---|---|---|
| Realistic / dark fantasy | SSAO/N8AO, bloom, fog, soft particles, LUT grading, film grain | Hard outlines, pixelation |
| Stylised / painterly | Bloom, toon ramps, stylised flipbook fire, rim light | Heavy grain, realistic smoke |
| Toon / cel | Outline pass, stepped lighting, shape-based particles (stars, puffs) | SSAO noise, DoF |
| Low-poly | Flat-shaded debris (instanced), simple fog, gentle bloom | Photo-real textures |
| Retro / pixel / PS1 | Pixelation, dithering, vertex snapping, low-res render target, limited palette | High-res soft effects, DoF |

---

## 3. Tier definitions

Use these labels on every option.

| Tier | Meaning | Typical cost |
|---|---|---|
| **Cheap** | Sprite/billboard, CPU-light, no new deps | <1 draw call per effect type, works on mobile |
| **Standard** | Library-based or custom shader, pooled | 1–5 draw calls, desktop + mid mobile |
| **Premium** | GPGPU/compute, extra render passes, volumetrics | Extra full-screen passes, desktop-first, quality-gated |

---

## 4. Particle & VFX catalogue (dungeon crawler)

Each entry lists the gameplay hook, then the options by tier.

### 4.1 Environment / ambience
| Effect | Hook | Cheap | Standard | Premium |
|---|---|---|---|---|
| Torch fire | Static props | Flipbook sprite + flickering PointLight | three.quarks fire + embers + smoke | Shader fire (noise-raymarched) + heat-haze distortion |
| Embers / sparks | Torches, braziers | drei `<Sparkles>` | Points + upward drift shader | GPGPU with turbulence field |
| Dust motes | Light shafts, rooms | `<Sparkles>` low opacity | Points shader with slow curl noise | Volumetric light + motes lit by shaft |
| Dripping water | Caves | Instanced drops + ripple decal | Splash burst on impact | Screen-space ripples on puddles |
| Fog / mist | Floors, pits | `FogExp2` | drei `<Clouds>` / layered planes with scrolling noise | Raymarched volumetric fog |
| Cobwebs / floating debris | Corridors | Static alpha planes | Wind-sway vertex shader | — |

### 4.2 Combat
| Effect | Hook | Cheap | Standard | Premium |
|---|---|---|---|---|
| Weapon trail | Attack anim | drei `<Trail>` | Custom ribbon mesh with gradient texture | Distortion trail (refraction) |
| Hit sparks | `onHit` | Burst of 10–20 billboard quads | quarks burst + light flash | Physics debris (rapier) + sparks |
| Blood / ichor | `onDamage` | Sprite burst | Burst + `<Decal>` on floor | Fluid-ish metaball splats |
| Hit flash | `onDamage` | Material emissive pulse | Uniform-driven flash shader | + chromatic aberration pulse |
| Death | `onDeath` | Fade + smoke puff | **Dissolve shader** + ember burst | Dissolve + particles spawned on mesh surface (MeshSurfaceSampler) |
| Block / parry | `onBlock` | Spark burst | Shield fresnel flash | Shockwave distortion |
| Critical hit | `onCrit` | Larger burst + text | Hit-stop + camera shake | + radial blur |

### 4.3 Magic / abilities
| Effect | Hook | Cheap | Standard | Premium |
|---|---|---|---|---|
| Projectile (fireball etc.) | Spell cast | Glowing sprite + trail | Mesh core + quarks trail + point light | GPGPU trail + distortion |
| AoE ground circle | Telegraph | Decal/plane with scrolling texture | Animated rune shader (polar UVs) | + vertical light pillars |
| Buff aura | Status effects | `<Sparkles>` around character | Fresnel shell mesh + orbiting particles | Volumetric aura shader |
| Heal | `onHeal` | Rising green sparkles | Spiral particle emitter | + bloom pulse |
| Teleport / blink | Ability | Fade + puff | Dissolve out/in | + afterimage ghosts (instanced snapshots) |
| Portal | Level exits | Rotating textured plane | Swirl shader + particles | Render-target "window" into another scene |

### 4.4 Loot & interaction
| Effect | Hook | Cheap | Standard | Premium |
|---|---|---|---|---|
| Item glow by rarity | Drop spawn | Emissive + bloom | Light beam pillar (Diablo-style) | + sparkles + outline |
| Chest open | `onOpen` | Burst of coins (instanced) | Physics coins + golden light | — |
| Interactable highlight | Proximity | Emissive tint | Outline effect (postprocessing `Outline`) | Through-wall silhouette |
| Footstep dust | Movement | Small puff sprite | Surface-dependent puffs (stone/water/dirt) | — |

---

## 5. Post-processing catalogue

Order matters. Typical composer order: **AO → (God rays) → Bloom → DoF → Chromatic Aberration → Color grading/LUT → Vignette → Noise → (Pixelation) → Tone mapping (often handled by renderer or a final ToneMapping effect).**

| Effect | Dungeon use | Cost | Notes |
|---|---|---|---|
| **N8AO / SSAO** | Contact shadows, heavy corners | Medium–High | N8AO is usually better quality per ms. Bake AO for static geometry if possible |
| **Bloom** | Fire, magic, loot | Medium | Use `mipmapBlur`, threshold > 1 with emissive/HDR. Selective bloom via luminance threshold, not layers, when possible |
| **Vignette** | Mood, low-HP pulse | Very low | Drive darkness/colour from health |
| **Chromatic aberration** | Damage, poison, curse | Low | Pulse briefly; never leave on strongly |
| **Depth of field** | Menus, cutscenes, dialogue | High | Disable in gameplay or keep very subtle |
| **God rays** | Light through grates | Medium–High | Needs a light-source mesh |
| **LUT / colour grading** | Per-level mood | Low | Swap LUTs per biome; lerp on transition |
| **Noise / film grain** | Grit | Very low | Premultiply, low opacity |
| **Outline** | Interactables, enemies | Medium | Limit selection set size |
| **Pixelation / dither** | Retro style | Low | Or render at low res + nearest upscale |
| **Shockwave** | Explosions, slams | Low | Short-lived |
| **Tone mapping** | Everything | Very low | ACES / AgX; be consistent with the renderer setting |
| **SMAA / FXAA** | AA when MSAA off | Low | Composer often disables MSAA; add SMAA |

---

## 6. Atmosphere & lighting

| Technique | Cost | Guidance |
|---|---|---|
| `FogExp2` | ~0 | Default for dungeons; match fog colour to background |
| Flickering PointLights | Per light | Animate intensity with noise in `useFrame`; **cap real-time lights (~4–8 active)**; cull by distance |
| Emissive + bloom instead of lights | Low | Most "glows" should be this, not real lights |
| Baked lightmaps / AO maps | ~0 runtime | Best for static rooms; requires Blender bake pipeline |
| Shadow casting | High | One shadow-casting light near the player; others no shadows. Consider `<ContactShadows>` or blob shadows |
| Volumetric spot (drei `<SpotLight volumetric>`) | Medium | Hero moments, not everywhere |
| Light probes / env map | Low | `<Environment>` with a dark custom HDR for PBR reflections |

---

## 7. Shader effects library

| Shader | Use | Technique summary |
|---|---|---|
| **Dissolve** | Death, spawn, teleport | Noise texture vs `uProgress`; `discard` below threshold; emissive edge band |
| **Fresnel rim** | Ghosts, shields, auras | `pow(1.0 - dot(N, V), k)` added to emissive |
| **Hit flash** | Damage feedback | `mix(color, flashColor, uFlash)`; decay `uFlash` per frame |
| **Scrolling UV** | Lava, water, energy | `uv + uTime * speed`; layer two scrolling noises |
| **Polar UV runes** | AoE circles, portals | Convert to polar coords, rotate, mask rings |
| **Heat haze** | Above fire | Screen-space distortion via normal map offset of scene texture |
| **Vertex sway** | Webs, banners, plants | Sine/noise offset weighted by vertex height/UV |
| **Soft particles** | Smoke touching geometry | Fade alpha by depth difference (needs depth texture) |
| **Toon ramp** | Stylised lighting | Step lighting via gradient map (`MeshToonMaterial`) |
| **Vertex snap / affine** | PS1 style | Quantise clip-space positions |

**Authoring options:** `onBeforeCompile` patches (keep standard lighting), `three-custom-shader-material` (CSM, extend built-in materials cleanly), raw `ShaderMaterial`, or **TSL node materials** (WebGPU and WebGL-compatible).

---

## 8. Game-feel (non-rendering, high impact)

| Technique | Implementation |
|---|---|
| Screen shake | Trauma-based (`trauma²` × noise) offset on camera rig; drei `<CameraShake>` for ambient |
| Hit-stop | Pause animation mixers / time scale for 40–80 ms on heavy hits |
| Damage numbers | Pooled drei `<Text>` (troika) billboards, float + fade |
| Camera FOV kick | Brief FOV change on dash / heavy attack |
| Time slow | Global timescale on last-enemy kill |
| Controller rumble | Gamepad API `vibrationActuator` paired with shake |

---

## 9. Performance budgets & rules

### 9.1 Budget tiers (adapt to discovered target)
| Target | Active particles | Full-screen passes | Real-time lights | Shadow lights |
|---|---|---|---|---|
| Mobile / low | ≤ 2k | ≤ 2 | ≤ 3 | 0–1 |
| Desktop mid | ≤ 20k | ≤ 5 | ≤ 8 | 1 |
| Desktop high | 100k+ (GPGPU) | ≤ 8 | ≤ 12 | 1–2 |

### 9.2 Hard rules
- **No React state per frame.** Mutate refs/uniforms inside `useFrame`.
- **Pool everything** (emitters, decals, damage text). No mount/unmount per hit.
- **Additive/transparent particles:** `depthWrite={false}`, correct `blending`, sort only if needed.
- **One draw call per particle type** via `Points` / `InstancedMesh`; use **texture atlases / flipbooks**.
- **Fill rate is the killer:** large overlapping transparent quads cost more than particle counts. Keep big smoke sprites few.
- **Cap `dpr`** (e.g. `[1, 1.5]`); consider lower-res effect buffers.
- **Dispose** geometries, materials, render targets on unmount.
- **Gate Premium effects** behind the quality-settings system (§1 #10). Create one if missing.
- **Measure:** `r3f-perf` / `stats-gl`, `renderer.info` (calls, triangles), Spector.js for frame capture.

---

## 10. Recommended architecture (if none exists)

```
src/vfx/
  VFXManager.tsx          // single mount point, owns pools, listens to game events
  pools/                  // generic pooled emitters (burst, trail, decal, text)
  effects/                // definitions: hitSparks.ts, fireball.ts, dissolve.ts ...
  shaders/                // GLSL/TSL chunks (dissolve, fresnel, noise)
  post/PostFX.tsx         // the ONE EffectComposer, quality-aware
  quality.ts              // low/med/high flags consumed by everything above
```

- **Event-driven triggers:** gameplay emits `vfx.play('hitSparks', { position, normal, scale })` through the existing state/event system (zustand store, event bus, or ECS). VFX never reaches into gameplay logic.
- **Data-driven definitions:** effects are config objects (count, lifetime, colours, curves, texture) so they can be tuned without code changes.
- **Single composer:** all post effects live in `PostFX`; gameplay modulates them via store values (e.g. `damagePulse`, `lowHealth`).

### Minimal patterns

**Pooled burst (R3F, instanced):**
```tsx
const MAX = 512;
function BurstPool() {
  const mesh = useRef<THREE.InstancedMesh>(null!);
  const data = useMemo(() => Array.from({ length: MAX }, () => ({
    life: 0, pos: new THREE.Vector3(), vel: new THREE.Vector3(),
  })), []);
  const dummy = useMemo(() => new THREE.Object3D(), []);
  let cursor = 0;

  useVFXEvent('burst', ({ position, count = 16 }) => {
    for (let i = 0; i < count; i++) {
      const p = data[cursor++ % MAX];
      p.life = 1; p.pos.copy(position);
      p.vel.randomDirection().multiplyScalar(2 + Math.random() * 3);
    }
  });

  useFrame((_, dt) => {
    data.forEach((p, i) => {
      if (p.life > 0) { p.life -= dt * 2; p.vel.y -= 9.8 * dt; p.pos.addScaledVector(p.vel, dt); }
      dummy.position.copy(p.pos);
      dummy.scale.setScalar(Math.max(p.life, 0) * 0.1);
      dummy.updateMatrix();
      mesh.current.setMatrixAt(i, dummy.matrix);
    });
    mesh.current.instanceMatrix.needsUpdate = true;
  });

  return (
    <instancedMesh ref={mesh} args={[undefined, undefined, MAX]} frustumCulled={false}>
      <planeGeometry />
      <meshBasicMaterial color="orange" transparent depthWrite={false} blending={THREE.AdditiveBlending} />
    </instancedMesh>
  );
}
```
*(`useVFXEvent` = whatever event hook matches the project's state system.)*

**Dissolve fragment core:**
```glsl
float n = texture2D(uNoise, vUv).r;
if (n < uProgress) discard;
float edge = smoothstep(uProgress, uProgress + 0.05, n);
gl_FragColor.rgb += (1.0 - edge) * uEdgeColor * 4.0; // HDR edge for bloom
```

**Quality-aware post stack:**
```tsx
<EffectComposer multisampling={0}>
  {q.ao && <N8AO aoRadius={2} intensity={2} />}
  <Bloom mipmapBlur luminanceThreshold={1} intensity={q.high ? 1.2 : 0.8} />
  <Vignette darkness={0.5 + lowHealth * 0.3} />
  {q.high && <Noise opacity={0.03} />}
  <SMAA />
</EffectComposer>
```

---

## 11. Library quick reference

| Package | Role | When to propose |
|---|---|---|
| `@react-three/drei` | Sparkles, Trail, Clouds, Decal, Text, CameraShake, volumetric SpotLight, Instances | Almost always (if R3F) |
| `@react-three/postprocessing` / `postprocessing` | Screen effects, merged passes | Any post-processing |
| `n8ao` (bundled in r3f postprocessing) | High-quality AO | Realistic/dark styles |
| `three.quarks` | Full particle system + editor | Many varied authored effects |
| `wawa-vfx` | Lightweight R3F VFX | Small/medium R3F projects |
| `three-custom-shader-material` | Extend built-in materials with custom shader code | Dissolve, flash, rim on PBR materials |
| `three/examples` GPUComputationRenderer | GPGPU particles (WebGL) | 100k+ particles on WebGL |
| three.js TSL / `WebGPURenderer` | Node materials, compute particles, node post-FX | WebGPU projects / future-proofing |
| `@react-three/rapier` | Physics debris, coins | Physical VFX |
| `r3f-perf` / `stats-gl` | Profiling | Always during VFX work |
| `meshline` / drei `<Trail>` | Ribbons | Weapon/projectile trails |

*Verify current versions and compatibility before recommending.*

---

## 12. Proposal template (what the agent outputs)

```markdown
# GFX Proposal: <scope, e.g. "Combat feedback pass">

## Project profile
- three r___, R3F ___, renderer: WebGL/WebGPU
- Existing VFX/post: ___
- Art style: ___ | Target: ___ fps on ___
- Event system: ___ | Quality settings: yes/no

## Recommendations (priority order)
### 1. <Effect name>, <Tier>
- **Why it fits:** <link to design doc/art style/gameplay>
- **Approach:** <library or technique>
- **Integration point:** <file/function, e.g. `src/combat/applyDamage.ts`>
- **Cost:** draw calls ~__, passes +__, new deps: __, mobile: ok/gated
- **Alternatives:** Cheap: ___ / Premium: ___
- **Effort:** S / M / L

### 2. ...

## Shared infrastructure needed
- <e.g. VFXManager + pooled bursts, quality flags>

## Risks & open questions
- <fill-rate on mobile, art assets needed (sprite sheets, noise textures, LUTs), ...>

## Suggested implementation order
1. Infrastructure → 2. Highest-impact cheap wins → 3. Premium, quality-gated
```

---

## 13. Default "first pass" bundle

If the agent needs a fast recommendation with minimal info, propose this (adjusting to the art style):

1. `FogExp2` + flickering torch lights (capped, distance-culled)
2. Emissive materials + **Bloom** + **Vignette** (+ **N8AO** on desktop)
3. Pooled **hit sparks**, **weapon trails**, **hit flash** and **screen shake**
4. **Dissolve** on enemy death with an ember burst
5. Loot rarity glow (emissive + light beam) and interactable **outline**
6. Quality settings flags gating AO, noise and Premium particles

These six items give the most dungeon atmosphere and combat feel for the least cost and complexity.

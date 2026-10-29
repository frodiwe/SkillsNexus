# Blender Modeling Skills: Research, Conclusion & Plan

Goal: give the agents that drive Blender through Blender MCP (`execute_blender_code`, `get_scene_info`,
viewport screenshots, Poly Haven / Hyper3D integrations) the working knowledge a professional 3D artist
uses: how they plan, block out, model, topologize, unwrap, bake, texture and deliver assets. Animation and
rigging are out of scope, except where they constrain modeling (deformation topology).

---

## Part 1: How professionals actually work

### 1.1 The universal pipeline (every asset type)

Every discipline in the research follows the same coarse-to-fine order. The details change per asset
type, the order doesn't:

1. **Brief & reference.** Purpose, camera distance, platform, style, budget. Artists collect many reference
   images (photos, orthographic blueprints, disassembly/review videos for mechanical things) *before* touching
   the mesh. For known objects they work from real dimensions or a bounding box.
2. **Blockout.** Primitives at **real-world scale** that settle proportion, footprint, silhouette, pivot and
   camera readability. Everything later depends on this; late proportion changes ripple into UVs,
   textures, collision and rigs, so the blockout is where iteration is cheap.
3. **Primary → secondary → tertiary forms.** Primary forms define the silhouette, secondary forms give
   individuality (panels, muscle groups, folds), tertiary forms carry surface story (scratches, pores,
   stitches). Silhouette comes first and is checked as a flat black shape; a common rule of thumb is that
   most of a design's readability comes from silhouette.
4. **Topology appropriate to purpose.** Deforming meshes (characters, cloth) need loops that follow
   muscle/deformation; static hard-surface needs topology that *shades* cleanly; real-time low-poly spends
   triangles on silhouette and lets normal maps carry the rest.
5. **UVs.** Seams in hidden places and on hard edges, consistent **texel density**, straightened islands,
   efficient packing with padding.
6. **Bake high → low** (when there is a high-poly): normal, AO, curvature, ID, etc.
7. **Texturing / materials.** PBR-correct values, then storytelling (wear where things get touched, dirt
   where it collects).
8. **Optimization & delivery.** Triangle budget, LODs, collision, naming, pivots, applied transforms,
   export format and engine conventions.

The step agents most often skip is 2–3. They jump straight to detailed geometry on top of wrong
proportions, which gives what I'll call **"primitive soup"**: unbeveled, unjoined, intersecting primitives
at arbitrary scale.

### 1.2 Detail allocation: geometry vs normal map vs texture

Professionals decide *where* each level of detail lives:

| Detail level | Lives in | Example |
|---|---|---|
| Silhouette-changing | Geometry (low poly) | Rifle barrel, ear shape, roof outline |
| Medium relief, doesn't change silhouette | Normal map (baked from high poly) or geometry for non-baked pipelines | Panel lines, bolts, wrinkles, muscle striations |
| Micro surface | Texture (roughness/normal detail/albedo) | Scratches, pores, wood grain, fabric weave |

For an agent producing assets without a sculpting step, the practical version is: geometry for silhouette
and medium forms, *bevels on every exposed edge*, and procedural/texture detail for the micro level.

### 1.3 Topology fundamentals (shared vocabulary)

- **Quads** are the default because they subdivide predictably and form edge loops. Triangles are fine on
  final real-time meshes (everything is triangulated in-engine) and in flat, non-deforming areas.
  N-gons are fine on flat planar faces of *non-subdivided* hard-surface meshes and cause pinching on
  curved or subdivided surfaces.
- **Poles** (vertices with ≠4 edges): 3- and 5-poles are normal and are how edge flow is redirected.
  6+ poles should be avoided. Place poles where the surface is flat or deforms least, never in the middle
  of a highlight or a joint.
- **Support/holding loops** control sharpness under subdivision: the closer the loop to the edge, the
  tighter the corner. Edge crease is the non-destructive alternative.
- **Edge density follows need.** Dense where curvature, deformation or silhouette needs it, sparse in flat
  areas. Even density everywhere is a sign of auto-remesh, not craft.
- **Edge flow follows form.** Loops follow the direction of the shape (muscle direction, cylinder rings,
  panel borders).

### 1.4 Hard-surface modeling

Two established workflows, picked per project:

**A. Subdivision (SubD) workflow**, for smooth high-poly or film-quality hard surface:
- Model a low-density quad cage, Subdivision Surface modifier on top.
- Control sharpness with support loops or creases. Keep terminations and poles on flat areas.
- Booleans need cleanup into quads around the intersection.

**B. Boolean + bevel + custom normals (non-SubD)**, the dominant modern Blender game-art workflow:
- Cut shapes with Boolean modifiers (cutters kept as hidden objects for non-destructive edits).
- Bevel modifier (limit by angle or bevel weight, 2–3 segments, profile ~0.5, *Harden Normals*); the Arc
  setting gives smoother miters.
- **Weighted Normal** modifier (Face Area, *Keep Sharp*) at the end of the stack to fix shading on large
  flat faces with chamfered edges.
- N-gons on flat faces are acceptable; triangulate on export.
- Common artifact: bevel weights bleeding onto loops between a boolean's edges. Only the boolean
  edge itself should carry the weight.

Principles shared by both:
- **Nothing in reality is perfectly sharp.** Every exposed edge gets a chamfer or bevel sized relative to
  the object (a millimetre on a phone, centimetres on a crate). Highlight-catching bevels are what
  separates "CG toy" from "real object".
- **Big/medium/small shape rhythm and rest areas.** Detail clusters contrast with calm surfaces; uniform
  detail density reads as noise.
- **Form follows function.** Parts must look manufacturable and assemblable: fasteners where parts join,
  hinge clearances, panel gaps, plausible thickness.
- **Floating geometry** (bolts, vents laid on the surface without merging) is fine in a high poly that will
  be baked.
- Vehicles and other large curved hard surfaces: work from orthographic blueprints; separate *form* from
  *topology* by defining the shape with guide curves first, then snapping/shrinkwrapping clean topology
  onto it. Panel gaps go in after the main surface is smooth.

### 1.5 Characters

#### Proportions
- Average adult ≈ **7.5 heads** tall; the idealized canon is **8 heads**; heroic/comic ≈ 8–9.
  Stylized characters shrink the body and enlarge the head (chibi ≈ 2–3 heads).
- 8-head landmarks: chin at head 1, nipples at 2, navel at 3, crotch at 4 (the body's midpoint), knee
  around 6.
- Face: eyes on the vertical midpoint of the head; face in thirds (hairline–brow, brow–nose base,
  nose base–chin); one eye-width between the eyes; ears between brow line and nose base.
- Model in **A-pose** (arms ~45°) or T-pose with slightly bent elbows/knees; A-pose deforms the shoulder
  better.
- Stylization means picking the defining traits and pushing them. Too much exaggeration, or clutter,
  hurts readability.

#### Body topology (deformation)
- Loops wrap every joint: shoulder, elbow, wrist, hip, knee, ankle, knuckles.
- **Elbows/knees**: at least 2–3 loops at the bend so the volume holds and joint placement is forgiving.
- **Shoulder** is the hardest area: a loop at the arm root plus loops flowing across the deltoid from
  chest to back.
- **Torso**: horizontal loops (it bends along its length); pectoral/scapula flow on hero meshes.
- **Hips/buttocks**: radial loops around the leg root; a common problem area when the leg lifts.
- **Hands**: each finger is its own tube (6–8 sides on hero meshes, fewer on low poly) with tighter
  loop spacing at the knuckles; fingers join the palm through a planned loop distribution; the thumb roots
  deep in the palm, not at its edge.
- Symmetry via Mirror modifier with clipping; center loop vertices exactly on X=0.

#### Head & face
- Concentric loops around the **eyes** (orbicularis oculi) and **mouth** (orbicularis oris). All
  other facial flow connects to these anchors. Loops *encircle* the eye rather than spiral.
- **Nasolabial loop**: from the nose wing around the mouth corners to the chin. It is what makes smiles
  deform properly.
- Poles go in low-deformation zones (cheeks, temples, forehead), never on the lip line or eyelid edge.
  The mouth has no single correct pole layout; it depends on the expressions needed.
- Typical classroom density: ~5 loops around each eye and ~4 around the mouth for a mid-res head.
- **Separate meshes**: eyeball (sclera + concave iris), a transparent **cornea/eye-wet** shell that bulges
  over the iris, tear line at the lid margin, eye occlusion shell, teeth, gums, tongue, inner mouth bag,
  eyelash and eyebrow cards. Separate meshes rig and take materials independently.

#### Hair
Three families, chosen by style and target:
1. **Hair cards** (real-time realistic): build strands as curves, bake an **atlas** (color, alpha, depth,
   root–tip gradient, ID, flow, normal, AO) onto a texture sheet, then place UV-mapped cards:
   - **Inside out**: an opaque **base/scalp layer** sets silhouette and base color (and keeps overdraw
     cheap), then 2–3 progressively more transparent, flowier layers, then single strays/flyaways.
   - Place big clumps first, hairline → crown, following the flow of the hair volume; clumps of ~3 cards
     arranged in rows; fill gaps last.
   - Avoid a few large flat cards, which look polygonal; split into smaller ones.
   - Root at the top of the UV, tip at the bottom; transfer normals from a smooth hair-volume proxy for soft
     shading.
2. **Stylized/sculpted hair**: large clump shapes as tapered tubes or curve-with-bevel-and-taper,
   designed as shape language (big-medium-small clumps) rather than strands.
3. **Strand grooms** (offline render): Blender Curves hair object + geometry-node grooming. For games it
   gets converted to cards or a mesh.

#### Clothing & accessories
- Garments sit *over* the body with a consistent offset, never coplanar with skin; hidden body parts under
  tight clothing are often deleted to stop clipping.
- Thickness via Solidify (rim only where the edge is visible: hems, collars, cuffs). Folds are secondary
  forms that follow tension (from anchor points: shoulders, waist, knees).
- Pro workflow: pattern-based sim (Marvelous Designer or Blender cloth), then retopo. Carry the posed
  shape onto flat-pattern retopology with Surface Deform/shape keys. Merge layers into as few contiguous
  meshes as possible for games.
- Straps, belts, cables as curves with a profile; buckles and armor plates follow hard-surface rules.

### 1.6 Props (small → big)

| Class | Examples | Typical budget* | Characteristics |
|---|---|---|---|
| Small / handheld | Cup, phone, tool, weapon | 300 – 5k tris (weapons more in FPS) | Seen close: bevels and texel density matter most; unique UVs |
| Medium | Chair, crate, barrel, machine | 1k – 10k | Clear functional logic, wear storytelling, often a unique texture |
| Large | Vehicle, statue, market stall | 10k – 100k+ | Mix of unique + tiling materials + trim sheets; multiple materials; LODs essential |

\* Budgets vary by platform, camera distance and LOD use. These are community ranges, not standards.
Mobile is roughly 1.5–5k for a whole asset; console/PC hero characters run 15–100k.

Prop principles: real dimensions first; "how was it made and used?"; silhouette test; chamfer all edges;
pivot where it's used (floor center, hinge axis, grip); wear where hands touch and edges get hit, dirt in
crevices; kitbash and reuse parts; check readability from in-game camera distance.

### 1.7 Environments

- **Greybox first**: simple shapes at real scale, with a 1.8 m human mannequin and an eye-height camera
  (~1.6–1.7 m). Scale blocks to the planned buildings/props, test layout, then derive the **asset list**
  from the blockout.
- **Composition**: a focal point/landmark, foreground/midground/background layering, leading lines, value
  contrast, rest areas. Landmarks guide orientation.
- **Big/medium/small distribution**: clusters, not uniform scatter. Set dressing follows gravity, use and
  story (things accumulate in corners, along walls, where people work).
- **Modular kits**: a strict grid (e.g. 1 m base unit; walls at 2/4/8 m; power-of-two friendly sizes),
  pivots on grid corners, snap increments matching the grid, pieces named by size. Seams hidden by
  trims/pillars. Common failure: wall UVs that don't line up across pieces.
- **Trim sheets & tiling textures**: one shared texture with horizontal strips (trims) and tiling
  surfaces; faces are UV-mapped onto named regions. Unique detail goes on the hero pieces only.
- **Breaking repetition**: vertex-color material blending, decals, props, world-aligned textures,
  rotation/scale variation of reused meshes.
- **Rocks & cliffs**: blockout → sculpt/displace (or merge boolean-carved shapes and voxel-remesh into one
  cohesive mesh) → decimate/retopo → bake; check from all angles because rocks get reused and rotated;
  tiling rock materials + vertex blend + world-aligned top-layer (moss/snow, Z-up mask).
- **Vegetation**: trees from curves/generators + leaf cards with alpha; grass as card clumps; scatter with
  geometry nodes (density masks from painted weights, Poisson spacing, scale/rotation variation).
- **Architecture standards**: door ~0.9 × 2.1 m, ceilings 2.4–3 m (residential), stair riser ~17–18 cm /
  tread ~28 cm, table 72–76 cm, chair seat 43–46 cm, counter ~90 cm, interior walls 10–15 cm,
  exterior 20–30+ cm, handrail 0.9–1.0 m.

### 1.8 UVs, texel density, baking, materials

**UVs**
- Seams on hard edges, in occluded areas, and along natural material boundaries; never across a
  highly visible smooth surface.
- **Texel density** consistent across an asset and across a scene. Players notice mismatches more than
  absolute resolution. Rule-of-thumb targets: ~512 px/m third-person, ~1024 px/m first-person, higher for
  hero props and characters.
- Straighten rectangular islands (less stair-stepping, better packing); a few pixels of padding so
  mipmaps don't bleed; mirror or overlap symmetric parts to save space (offset the overlapped copies out
  of 0–1 for baking); second non-overlapping UV channel for lightmaps when the engine needs it.

**Baking (high → low)**
- Hard edges (sharp) ↔ UV seams: every hard edge must be a UV seam (not every seam a hard edge).
- Triangulate the low poly before baking so the engine's triangulation matches.
- Use a cage, or a small extrusion (start ~0.01 units and adjust) plus max ray distance. Cage and low poly
  must share topology.
- Blender: Cycles, *Selected to Active* (high selected, low active). Bake at a higher resolution or 32-bit
  and downsample. Name-match high/low pairs and bake exploded or in parts to avoid cross-occlusion.
- **Normal map convention**: Blender/glTF/Unity use OpenGL (Y+); Unreal uses DirectX (Y−), so flip green
  on export to Unreal.

**PBR materials**
- Albedo/base color holds no lighting. Non-metal sRGB ≈ 30/50 – 240; metals 180 – 255 (≈ 70–100 %
  reflectance).
- **Metalness is binary** (0 or 1); intermediate values only in transitions (dirt over metal).
- Roughness carries most of the realism: vary it with wear, fingerprints and dust, never a single
  constant.
- Color spaces: base color = sRGB; normal/roughness/metal/AO/height = Non-Color.
- Channel packing: ORM (R = occlusion, G = roughness, B = metallic), as glTF and Unreal expect.
- In-Blender procedural texturing: masks from AO, Pointiness/curvature, Bevel node and position; layered
  BSDFs mixed by masks; then **bake procedural materials to image textures** for export.

### 1.9 Delivery / engine conventions
- Apply transforms (scale 1, rotation 0) before export; unapplied non-uniform scale breaks normals and
  collision. The Blender origin becomes the engine pivot: place it deliberately (floor center for props,
  hinge for doors, grid corner for modular pieces).
- Units: metric, 1 unit = 1 m. Mind engine axis conventions (glTF +Y up, Blender +Z up; the exporter
  converts).
- Collision meshes by prefix (Unreal: `UCX_<MeshName>_##`, convex). LODs at roughly 50 % triangle
  reduction per level. Common naming: `SM_` static mesh, `SK_` skeletal, `M_`/`MI_` materials,
  `T_<Name>_D/_N/_ORM` textures.
- glTF/GLB for web and general interchange, FBX for Unreal/Unity pipelines.

### 1.10 Blender-specific technique (what changed recently, and agents get wrong)

- **Auto Smooth was removed in 4.1.** `mesh.use_auto_smooth`, `auto_smooth_angle` and `calc_normals_split`
  are gone; use the *Smooth by Angle* modifier (geometry-node asset) or the shade-smooth-by-angle operator;
  custom normals (Weighted Normal) no longer need Auto Smooth. Models trained on older code still emit
  `use_auto_smooth` and crash.
- **Blender 5.0**: Boolean solver `FAST` renamed to `FLOAT`; UV selection moved to shared attributes
  (`MeshUVLoopLayer.vertex_selection`, `BMLoopUV.select` removed); `scene.node_tree` → compositing node
  group; `mathutils` now float32; `brush.sculpt_tool` → `sculpt_brush_type`; Geometry Nodes modifier
  property access changed.
- Edge crease and bevel weight are generic attributes since 4.0 (`crease_edge`, `bevel_weight_edge`).
- Retopology tools: Voxel remesh (cleanup and merging intersecting parts, Adaptivity/Fix Poles), Quad
  remesh (Quadriflow: slower, for SubD/multires, not a fix for intersecting geometry), Decimate (reduce
  a clean mesh), Shrinkwrap (project new topology onto a source). The manual itself says animated
  character topology must still be done by hand.
- `bpy` is not thread-safe; MCP add-ons queue work to the main thread. Wrap agent code runs in an undo
  push; run small, idempotent steps; prefer the data/`bmesh` API over context-dependent `bpy.ops` (use
  `context.temp_override` when an operator is unavoidable).
- Hair: the Curves hair object (3.5+) and geometry-node grooming; converting to game meshes needs
  Curve to Mesh with a profile or a card generator.
- Eevee Next (4.2+) replaced blend modes with *Render Method* (Dithered/Blended), which matters for hair
  cards and foliage alpha.

---

## Part 2: Conclusion

1. **The knowledge gap is mostly process, not tools.** Agents can call any bpy function. What they
   lack is the professional order of operations (reference → blockout at scale → form hierarchy →
   purpose-driven topology → UV → bake → material → delivery) and the *quality gates* between stages.
   Skills should be built around stages and gates, not around Blender features.
2. **Most perceived quality comes from four things agents routinely skip**: real-world scale and
   proportion; silhouette; bevels/chamfers on every edge; and PBR-plausible materials with roughness
   variation. One skill should encode just these as non-negotiable defaults.
3. **The agent's modeling modality is code, not a stylus.** It can't sculpt freehand well, so each skill
   has to turn the professional technique into code-friendly methods:
   - hard surface → primitives + bmesh + modifier stacks (Bevel, Boolean, Weighted Normal, Mirror,
     Solidify, Subdivision);
   - organic → parametric base meshes (Skin modifier, curves, subdivided cages), **base mesh libraries**
     (e.g. MPFB/MakeHuman for human bodies with production topology), shrinkwrap + displacement,
     voxel remesh;
   - environments/vegetation → Geometry Nodes generators and scatter, modular kits on a grid;
   - AI-generated meshes (Hyper3D) → treated as a *high poly / reference* that is remeshed, unwrapped and
     re-baked, never shipped raw.
4. **Verification has to be visual and numeric.** Pros constantly orbit, check silhouette, and look at
   wireframes. The agent needs a fixed review protocol (orthographic front/side/top + 3/4 perspective,
   matcap, wireframe overlay, black silhouette, side-by-side with reference) plus scripts that measure
   dimensions, triangle count, n-gons, non-manifold edges, flipped normals, unapplied scale, UV overlap
   and texel density. "Looks done in the code" is not done.
5. **Topology rules depend on purpose.** Three regimes (deforming organic, SubD hard surface, game
   low-poly / non-SubD hard surface) have different, sometimes opposite rules (n-gons ok vs never). Skills
   must state the regime up front so the agent doesn't apply character rules to a crate or vice versa.
6. **Numbers make skills usable.** Vague advice ("use appropriate density") fails with agents. Every skill
   should carry concrete defaults: proportions, dimensions, bevel widths, texel densities, budgets, PBR
   value ranges, naming patterns. These go in reference files so SKILL.md stays short.
7. **Blender version drift is a real failure source.** Reference API notes for 4.1+ and 5.x belong in a
   shared technique skill, not repeated in every modeling skill.

---

## Part 3: Skill list

Format proposal: one directory per skill under `blender/`, each with a short `SKILL.md` (trigger
description, workflow with gates, rules, failure modes, verification checklist) plus `references/`
(numbers, tables, topology diagrams described in text) and `scripts/` (bpy snippets the agent pastes into
`execute_blender_code`). The orchestrator routes to specialists; specialists hand off to the shared
UV/bake/material/delivery skills.

### Tier 0: Foundation (used on every asset)

| # | Skill | Covers | Why |
|---|---|---|---|
| 1 | `blender-asset-pipeline` | Orchestrator. Turns a request into an **asset spec** (type, style, target platform, scale, tri budget, texel density, material count, deliverables), runs the stage order with gates, routes to specialist skills. | Fixes the #1 problem: no process. Also the entry point the existing pipeline calls. |
| 2 | `blender-bpy-craft` | How to drive Blender via MCP code: small idempotent steps, undo pushes, naming/collections, bmesh vs ops, `temp_override`, modifier stacks (non-destructive until delivery), 4.1+/5.x API changes, units/scale, camera & viewport setup for screenshots. | Stops wasted runs from API errors and fragile scripts; shared by all skills. |
| 3 | `blender-model-review` | Visual critique protocol (standard views, matcap, wireframe, silhouette, reference comparison) + validation scripts (dimensions, tris, n-gons, non-manifold, normals, transforms, UV overlap, texel density) + a scoring rubric. | Verification at every gate; the agent gets eyes and a definition of done. |

### Tier 1: Core disciplines (cross-cutting)

| # | Skill | Covers | Why |
|---|---|---|---|
| 4 | `form-and-proportion` | Reference gathering, real-world dimension tables, blockout at scale, primary/secondary/tertiary forms, silhouette, shape language, stylization rules (realistic / stylized / low-poly flat). | Biggest quality lever; agents go straight to detail on wrong proportions. |
| 5 | `topology-fundamentals` | Quads/tris/n-gons by regime, poles, edge flow, support loops vs creases, density distribution, mirror/symmetry, retopology methods (manual shrinkwrap, Quadriflow, voxel, decimate) and when each applies. | Shared vocabulary for hard-surface and character skills; prevents shading and subdivision artifacts. |
| 6 | `hard-surface-modeling` | SubD and boolean+bevel+weighted-normal workflows, bevel sizing by scale, panel lines, cutouts, fasteners, mechanical logic, curved-surface/vehicle workflow (blueprints, guide curves, shrinkwrap), cleanup of boolean artifacts. | Most props, weapons, vehicles and architecture are hard-surface; this is where agent output looks "toy-like". |
| 7 | `procedural-geometry-nodes` | Building parametric generators and scatter systems with Geometry Nodes from Python: distribute, instance, randomize, density masks, exposing parameters, realizing instances for export. | The most agent-friendly modeling modality: deterministic, parametric, re-runnable. Used by environment, vegetation and hair skills. |
| 8 | `uv-unwrapping` | Seam placement rules, texel density targets and scripts, straightening, packing/padding, mirroring/overlaps, trim-sheet UV mapping, UDIM vs atlas, lightmap UV channel. | Bad UVs ruin baking and texturing; agents default to Smart UV Project everywhere. |
| 9 | `baking-high-to-low` | Making the low poly (silhouette budget), hard edges ↔ seams, triangulation, cage/extrusion/ray distance, Cycles bake setup via bpy, bake map set, OpenGL/DirectX, bake-by-parts, troubleshooting artifacts. Includes **refining AI-generated (Hyper3D) meshes** as high poly → clean low poly. | The bridge between detailed and game-ready assets; many silent failure modes. |
| 10 | `pbr-materials-texturing` | PBR value ranges, metalness/roughness rules, node-based procedural texturing (AO/curvature/position masks, edge wear, dirt, layering), Poly Haven texture use, baking materials to images, color spaces, channel packing, texture resolution choice, stylized/hand-painted variant. | Flat single-color Principled BSDFs are the second most visible quality problem. |
| 11 | `game-ready-export` | Tri budgets per class/platform, LOD generation, collision meshes, naming conventions, pivots, applying transforms, FBX/glTF settings per engine, final checklist. | Assets must actually work downstream; easy to automate with scripts. |

### Tier 2: Characters

| # | Skill | Covers | Why |
|---|---|---|---|
| 12 | `character-anatomy-proportions` | Head-unit canons (realistic, heroic, stylized, chibi), body landmarks, facial proportions, A-pose/T-pose, body masses and gesture, base-mesh strategy (MPFB/base mesh library, Skin modifier, box modeling), creature adaptations. | Proportion errors are the most visible character failure; base meshes give the agent a correct starting point it can't sculpt itself. |
| 13 | `character-body-topology` | Deformation loops for shoulders, elbows, wrists, hips, knees, ankles; torso flow; hands and feet; density budgets by tier (hero/NPC/mobile); low-poly game character layout; feet/shoe interface. | Body deformation rules are specific and not inferable from general topology. |
| 14 | `character-head-face` | Face loop anatomy (eye, mouth, nasolabial, nose, ears), pole placement, head density tiers, eyeball/cornea/tear line/occlusion construction, teeth/gums/tongue/mouth bag, eyelash and brow cards, stylized face simplifications. | The face is the densest set of rules and where viewers judge quality. |
| 15 | `character-hair` | Choosing cards vs stylized clumps vs strand grooms; hair-card atlas creation; card layering (base → layers → strays); clump shape language for stylized hair; Curves hair + geometry-node grooming; brows/beards/lashes; alpha and shading setup (Eevee Next render method, normal transfer). | Hair is its own discipline with its own geometry types and material tricks. |
| 16 | `character-clothing-accessories` | Garment construction over the body (offsets, thickness, layering, hidden-body deletion), fold logic, cloth-sim draping in Blender, retopo of simulated garments, seams/stitches/hems, straps/belts via curves, shoes, armor/hard-surface accessories. | Clothing is a large share of character work and fails by clipping and "painted-on" looks. |

### Tier 3: Props & environments

| # | Skill | Covers | Why |
|---|---|---|---|
| 17 | `prop-modeling` | Small/medium/large prop workflows and budgets, real-dimension tables for common objects, functional and manufacturing plausibility, wear and storytelling, organic props (food, fabric, cushions via cloth sim), kitbashing and part reuse, pivots by use. | Props are the most frequent request; size class changes everything (detail, UV strategy, materials). |
| 18 | `environment-blockout-composition` | Greybox at human scale, camera/eye height, composition (focal point, layering, leading lines, rest areas), asset list from blockout, big/medium/small set-dressing, storytelling clusters, scene lighting for review. | Environment quality is mostly layout and composition, not individual meshes. |
| 19 | `modular-kits-trim-sheets` | Grid systems and piece lists, pivot/snap conventions, seam hiding, trim sheet design and UV mapping, tiling textures, vertex-color blending, decals, architectural dimension standards, building/facade generators. | Buildings and interiors need a system, otherwise seams and scale break. |
| 20 | `nature-terrain-vegetation` | Rocks/cliffs (displace, voxel-merge, decimate, bake, reuse-from-any-angle), terrain from displaced grids/GN, trees (curve/generator trunks + leaf cards), grass/bush cards, scatter rules, world-aligned/Z-up material blending, water surfaces. | Organic environments rely on procedural and card techniques that differ from hard-surface. |

**Why 20 and not fewer:** each skill matches a distinct body of knowledge that professionals also
specialize in, and each must stay short enough to load into context next to the task. The shared Tier 0–1
skills keep the specialist skills from repeating UV/bake/material/export rules. If 20 is too many to
maintain, the safe merges are 7 into 20, 12 into 13, and 19 into 18.

**Deliberately excluded:** rigging, weight painting, animation, simulation-for-motion, rendering/lighting
for final shots (only review lighting is covered), and external tools (ZBrush, Substance, Marvelous)
except as context for what the Blender-native equivalent replaces.

---

## Part 4: Build plan

### Phase 0: Align with your setup (before writing skills)
Decisions that change the content (defaults in brackets are what I'll assume if unanswered):
- Target output: real-time game assets, offline renders, or both? [real-time, glTF + FBX]
- Target engine(s) for conventions (normal-map Y, naming, collision)? [engine-agnostic + Unreal/Unity notes]
- Default art style? [support realistic PBR and stylized; style set in the asset spec]
- Blender version on the MCP host? [4.2 LTS+ with 5.x notes]
- Which Blender MCP fork, and does it have viewport screenshots / render-to-image? [ahujasid's, with `get_viewport_screenshot`]
- Allowed add-ons (MPFB for human base meshes, Quad Remesher paid, etc.)? [Blender built-ins + free add-ons only]
- Is Hyper3D/AI generation part of the pipeline? [optional input, always refined]
- Where should the skills live (this repo, `blender/` folder) and in what format (Claude Code `SKILL.md`
  directories)? [yes]

### Phase 1: Foundation (skills 1–3) + script library
Highest return: process, reliable code execution, and a definition of done. Deliverables: the three
skills, and `scripts/` with `validate_mesh.py`, `capture_views.py` (ortho/persp/matcap/wireframe/silhouette),
`measure.py` (dimensions vs spec), `texel_density.py`, `apply_and_clean.py`.

### Phase 2: Core disciplines (skills 4–11)
Order: `form-and-proportion` → `topology-fundamentals` → `hard-surface-modeling` → `pbr-materials-texturing`
→ `uv-unwrapping` → `baking-high-to-low` → `game-ready-export` → `procedural-geometry-nodes`.
At the end of this phase the pipeline can make good props.

### Phase 3: Props & environments (skills 17–20)
Uses Phase 2 heavily; mostly domain rules, reference tables and generator recipes.

### Phase 4: Characters (skills 12–16)
Last because it depends on topology, UV and bake skills and is the hardest for a code-driven agent.
Base-mesh strategy first, then head/face, body, clothing, hair.

### Phase 5: Evaluation loop (runs alongside every phase)
- A **benchmark brief set**, e.g.: wooden crate, coffee mug, medieval chair, sci-fi pistol, stylized
  treasure chest, car (blockout level), rock cluster, stylized tree, modular dungeon corridor kit, small
  diorama scene, stylized character head, full low-poly game character, hair-card hairstyle.
- Run each brief **with and without** the skills; score with the `blender-model-review` rubric
  (proportion/scale, silhouette, topology regime correctness, shading artifacts, UV/texel, material
  plausibility, budget, export validity) plus screenshot side-by-sides.
- Revise skills where scores don't move; every recurring failure becomes a rule or a script.

### Skill authoring template (every SKILL.md)
1. Frontmatter `name` + a trigger-rich `description` (asset types, keywords).
2. When to use / when to hand off to another skill.
3. Inputs required from the asset spec.
4. Workflow: numbered stages, each ending in a **gate** (screenshot + check script + criteria).
5. Rules (do/don't) with concrete numbers; regime stated explicitly.
6. Code recipes (bpy, tested on the target Blender version).
7. Agent failure modes seen in evaluation, with fixes.
8. Final checklist.

---

## Sources & reliability note

The Polycount wiki (the usual canonical reference for face/limb topology, texel density and normal maps)
returned HTTP 503 during this research, so much of the evidence comes from Blender manual pages, Blender
release notes, Epic/Unity docs, and practitioner forum posts and breakdowns. Many are community opinions or
vendor pages; numeric values above are marked as rules of thumb where sources disagree. The synthesis also
draws on long-established industry practice.

- Blender MCP tool surface: [ahujasid/blender-mcp addon](https://glama.ai/mcp/servers/@ahujasid/blender-mcp/blob/9cefe8e6ab578ed293cbca564f8794274249fe41/addon.py), [naab007/blender_mcp](https://github.com/naab007/blender_mcp), [Blender MCP setup guide](https://glama.ai/mcp/servers/@shdann/mcp-blend/blob/258fea314c7db701d8091bac57141f64926bb8b6/SETUP_GUIDE.md), [Blender MCP screenshots](https://macmyths.com/how-to-capture-blender-screenshots-with-mcp/), [blender.org on MCP safety](https://www.blender.org/?p=96198)
- Hard surface: [Blender Artists – boolean/bevel on curved surfaces](https://blenderartists.org/t/boolean-and-bevel-on-curved-surface/1308394), [MESHmachine](https://superhivemarket.com/products/meshmachine), [Hardsurface Helper Pro](https://superhivemarket.com/products/hardsurface-helper-pro), [Boolean Bevel reviews](https://superhivemarket.com/products/boolean-bevel/ratings), [Foolproof Car Modelling](https://superhivemarket.com/products/foolproof-car-modelling-in-blender), [GFM Editor](https://blendermarket.com/products/gfm-editor), [G4M bomber WIP](https://blenderartists.org/t/g4m-betty-bomber-wip-mini-tutorial/1277358.md)
- Topology (face/body): [UW CSE458 head modeling](https://courses.cs.washington.edu/courses/cse458/10au/content/projects_458/project3/index.html), [UW modeling crash course](https://courses.cs.washington.edu/courses/cse458/21au/content/projects/project2/resources/modeling_crash_course.pdf), [Shoot First – topology](https://www.shootfirst.art/toplogy), [Gnomon – topology for animated characters](https://www.thegnomonworkshop.com/workshops/topology-for-animated-characters), [Blender Artists – face topology](https://blenderartists.org/t/learning-face-topology/647631), [Blender Artists – retopology help](https://blenderartists.org/t/help-with-retopology/680960), [Polycount – limb topology](https://polycount.com/discussion/comment/2065612), [Blender Artists – hand topology](https://blenderartists.org/t/best-way-to-topologise-this-hand/604296), [Blender Artists – character topology](https://blenderartists.org/t/character-for-animation-topology/563479), [Contour Retopo FAQ](https://superhivemarket.com/products/contour-retopo/faq)
- Proportions & stylization: [Wikipedia – body proportions](https://en.wikipedia.org/wiki/Body_proportions), [Life Drawing Academy](https://lifedrawing.academy/drawing-lessons/human-body-proportions), [Proko – idealistic figures](https://proko.com/course-lesson/human-proportions-idealistic-figures/notes), [Shape language (RocketBrush)](https://rocketbrush.com/blog/shape-language-in-game-character-design-how-to-make-characters-readable-and-consistent), [Primary/secondary/tertiary forms](https://steveyang.readthedocs.io/Modeling/123/), [Stylization guide](https://rtuckerkoepp.notion.site/Stylization-28872e9174ea80438dd2d9485725389d), [80.lv stylized character workflow](https://80.lv/articles/001agt-stylized-character-art-workflow-from-blockout-to-render)
- Eyes: [CryEngine eye shader](https://cryengine.com/docs/static/engines/cryengine-3/categories/1114113/pages/15728675), [Blender Artists – physically correct eye](https://blenderartists.org/t/modeling-a-physically-correct-human-eye/620146), [Tripo – modeling eyes](https://www.tripo3d.ai/vi/blog/explore/3d-modeling-eyes)
- Hair: [Games Artist – curly hair](https://gamesartist.co.uk/curly-hair-ls/), [Vertex School – hair card layers](https://www.vertexschool.com/hair-cards-building-the-layers-establishing-the-strays), [Polycount – hair cards](https://polycount.com/discussion/comment/2609200), [Polycount – base layer](https://polycount.com/discussion/comment/2555801), [Reallusion hair mesh production](https://wiki.reallusion.com/Content_Dev:CC_3.4_Hair_Mesh_Production), [CGMA – Jeff Hindsbol Hansen](https://www.cgmasteracademy.com/blog/jeff-hindsbol-hansen), [Simple Hair Cards](https://superhivemarket.com/products/simple-hair-cards), [FiberBake](https://superhivemarket.com/products/fiberbake-hair-cards), [Blender devtalk – hair curves to mesh](https://devtalk.blender.org/t/convert-new-hair-curves-to-mesh/25760)
- Clothing: [Polycount – MD to Blender retopo](https://polycount.com/discussion/comment/2745044), [Polycount – game clothing layering](https://polycount.com/discussion/comment/2698752), [80.lv – Retopo Planes](https://80.lv/articles/get-this-procedural-retopology-blender-tool-for-garments)
- Props & budgets: [The Rookies – game-ready assets workflow](https://www.therookies.co/blog/breakdowns/developing-game-ready-assets-a-workflow), [Polycount – prop reference](https://polycount.com/discussion/comment/2808922), [Unreal answers – triangle counts](https://answers.unrealengine.com/questions/29063/view.html), [Polycount budgeting guide (Tripo)](https://www.tripo3d.ai/blog/explore/polycount-budgeting-for-realtime-assets), [Game Ready One-Click docs](https://superhivemarket.com/products/game-ready-one-click/docs)
- Environments: [Level Design Book – blockout](https://https-book_leveldesignbook_com.proxy.bugsmash.io/process/blockout), [Epic – greyboxing in UEFN](https://dev.epicgames.com/documentation/en-us/uefn/greyboxing-in-unreal-editor-for-fortnite), [Unity – blockout from concept art](https://learn.unity.com/course/high-fidelity-game-visuals-environment/tutorial/creating-a-prototype-blockout-from-concept-art), [Roblox – environmental art](https://create.roblox.com/docs/tutorials/curriculums/environmental-art.md), [Roblox – modular environments](https://create.roblox.com/docs/tutorials/use-case-tutorials/modeling/assemble-modular-environments.md), [Polycount – modular UV seams](https://polycount.com/discussion/comment/2776494), [Modular blueprint kit](https://tuily.itch.io/modular-blueprint-kit), [TrimForge docs](https://superhivemarket.com/products/trimforge--trim-sheet-workflow/docs), [ModularKit](https://www.artstation.com/marketplace/p/dLg1B/modularkit-grid-snapping-socket-kit-builder), [80.lv – mountainous environment](https://80.lv/articles/how-to-model-texture-a-vibrant-storytelling-mountainous-environment/), [80.lv – waterfall bridge](https://80.lv/articles/waterfall-bridge-asset-workflow-in-zbrush-for-environment-art), [Geometry Nodes from scratch log](https://community.gamedev.tv/t/geometry-nodes-from-scratch/212175), [GN scatter rocks](https://blenderartists.org/t/geometry-nodes-scatter-rocks-3-6-and/1591695)
- UV / texel density / baking: [Foundry – texel density](https://learn.foundry.com/modo/12.0/content/help/pages/uving/texel_density.html), [Polycount – texel density](https://polycount.com/discussion/comment/1740645), [Texture resolution guide](https://www.strayspark.studio/blog/texture-resolution-guide-games-512-1k-2k-4k), [BlenderNation – baking with a cage](https://www.blendernation.com/2021/10/22/how-to-bake-normal-maps-with-a-cage-in-blenderhow-to-bake-normal-maps-with-a-cage/), [Polycount – bake bias](https://polycount.com/discussion/comment/2350623), [Bake high poly to PBR in Blender](https://www.strayspark.studio/learn/tutorials/bake-high-poly-to-pbr-maps-blender), [Blender devtalk – baking issue](https://devtalk.blender.org/t/issue-baking-normal-map-with-high-and-low-poly-please-help/13671)
- PBR: [Cocos PBR guide](https://docs.cocos.com/creator/manual/en/shader/effect-builtin-pbr.html), [PBR albedo chart](https://shinsoj.artstation.com/blog/Q9j6/pbr-color-space-conversion-and-albedo-chart), [Smush material research](https://github.com/ScanMountGoat/Smush-Material-Research/blob/master/Textures.md), [ACES PBR validator](https://cirocardoso.gumroad.com/l/vmcfzk)
- Export/engine: [Blender to UE5 pipeline](https://sarahhyperdense.substack.com/p/blender-to-ue5-the-complete-export), [Epic FBX static mesh pipeline](https://docs.unrealengine.com/udk/Three/FBXStaticMeshPipeline.html), [UE FbxStaticMeshImportData](https://docs.unrealengine.com/5.0/en-US/PythonAPI/class/FbxStaticMeshImportData.html)
- Blender API/manual: [Blender 5.0 Python API release notes](https://developer.blender.org/docs/release_notes/5.0/python_api/), [Auto smooth replaced by node group (PR #108014)](https://projects.blender.org/blender/blender/pulls/108014), [Auto smooth regression discussion](https://projects.blender.org/blender/blender/issues/116922), [Blender manual – retopology](https://docs.blender.org/manual/en/4.4/modeling/meshes/retopology.html), [Quad Remesher + shrinkwrap workflow](https://blenderartists.org/t/quadremesher-auto-retopologizer-now-available-for-blender/1170913/117)
- Existing agent skills for reference: [arjun988/blender-skills – stylized-style](https://skills.sh/arjun988/blender-skills/stylized-style)

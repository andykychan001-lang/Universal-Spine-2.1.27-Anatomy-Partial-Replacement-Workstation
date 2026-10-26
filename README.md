# Universal-Spine-2.1.27-Anatomy-Partial-Replacement-Workstation
A Zero-Dependency, Non-Destructive Web Workstation for Darkest Dungeon Modding.
Universal Spine 2.1.27 Anatomy & Partial Replacement Workstation
A Zero-Dependency, Non-Destructive Web Workstation for Darkest Dungeon Modding

Language: English | 繁體中文

🎯 Overview & Motivation
Creating or modifying character sprites for Darkest Dungeon requires converting static character artwork into multi-part texture atlases compatible with the game's legacy Spine 2.1.27 runtime (.skel, .atlas, and .png). Modders traditionally face several major bottlenecks:

Legacy Binary Lock-In: The game engine strictly requires Spine 2.1.27 binary .skel files. Manually editing .atlas coordinates or resizing regions without re-exporting from the legacy Spine editor often results in broken UV mappings, distorted meshes, or immediate crashes to desktop (CTD).

Joint Seam Gaps During Motion: Slicing a static standing illustration into torso and limb layers leaves hollow cut seams exposed whenever bones rotate during walk cycles or combat animations.

Destructive Partial Edits: Replacing only a character's head or torso inside a complex skill animation (attack_sk*) or multi-pose sheet (combat) usually forces modders to unpack, re-slice, and repack every untouched weapon, slash FX, and secondary attachment.

Universal Spine 2.1.27 Workstation is a single-file, zero-dependency browser application designed around a Non-Destructive, Minimum-Intervention Pipeline. Load any existing Master Host bundle (.png + .atlas + .skel), paint part zones and underlay joint putty on your target artwork, preview it live on the host's actual .skel bone/mesh rig, and export a 100% compliant 3-in-1 bundle in one click.

✨ Key Features
🛡️ Non-Destructive & 1:1 Native Spec Compliance

Default 0-Filter Pass-Through: Preserves 100% of your original pixel data by default. Optional edge-peeling and white-defringe filters are strictly opt-in and engineered to never erode Darkest Dungeon's signature pitch-black ink outlines (RGB 0,0,0).

Strict Spine 2.1.27 Invariants: Operates at native 1:1 resolution, preserving host orig dimensions and calculating legal positive offset tight-trims without altering the binary .skel structure.

🔒 Unmapped Slot Pass-Through (Partial Replacement Mode)

Supports all animation types (walk, combat, idle, camp, defend, attack_sk*, fx_*). Any host slot not bound to a custom zone—such as weapons, slash effects, or ground shadows (shadow)—can either Keep Host Original (Pass-Through) or Convert to a 1×1 Transparent Safe Anchor.

🧱 Joint Putty Brush & Auto-Seam Coloring

Paint underlay joint extensions directly across limb sockets using the Putty Brush, then click Auto-Color Putty to automatically sample interior seam colors and fill gaps during bone rotation.

Generational-Loss-Free Scaling: Zone masks and hand-painted putty are stored in a 1:1 Master Space with 3×3 morphological closing smoothing—scaling your character between 70% and 150% never degrades or fragments putty edges.

⚡ 60FPS Zero-Lag Interactive Inverse Baking

Dragging part textures on Canvas B or adjusting offset/rotation sliders uses an analytical bounding-box projection (< 0.5ms per frame), deferring full pixel tight-trim scans until mouse release.

📐 Oversized / HD Texture Sample Support

Switch seamlessly between Standard (900×750), Large (1400×1100), and XL (2000×1600) 1:1 surgical canvases, switch origin presets (Foot Origin vs. Center FX Origin), and force-reveal hidden Setup Pose attachments.

📦 Custom Naming & 3-in-1 Bundle Export (.png + .atlas + .skel)

Customize your export base name with automatic first-line .atlas image header synchronization (preventing white-box missing texture bugs) and optional one-click packaging of the original .skel binary.

🚀 Quickstart Workflow
Launch: Open index.html directly in any modern web browser (Chrome, Edge, Firefox)—no local server or installation required.

Load Master Host (Section A): Upload the reference .png, .atlas, and .skel files from an existing hero or mod skill.

Load Target Artwork: Upload your character sprite. Use Paint Zone Mode (or click ⚡ Auto-Generate Zones from Host Slots) to assign body parts to host slots.

Add Joint Putty & Fine-Tune Alignment (Sections A & B):

Select 🧱 Putty Brush Mode to paint rounded joint extensions at limb cut lines, then click 🎨 Auto-Color Putty.

In Section B, drag textures directly with the left mouse button or use 🔗 Snap Yellow Pivot to Host Bone to align joints against the live .skel rig.

Export 3-in-1 Bundle: Enter your target filename (e.g., crusader.sprite.walk or my_hero.sprite.attack_sk1) and click 📦 Export 3-in-1 (.png + .atlas + .skel).

📂 Recommended Repository Structure
index.html — English UI Edition (GitHub Pages ready)

workstation_zh.html — Traditional Chinese UI Edition (繁體中文版)

README.md — Project overview & quickstart (English)

README_ZH.md — Project overview & quickstart (Traditional Chinese)

TECHNICAL_REPORT.md — Mathematical models & engineering architecture report

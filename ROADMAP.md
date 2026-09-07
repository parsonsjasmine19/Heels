# Futuristic-shoe heel-video pipeline — roadmap

**Goal:** Take a real clip of the creator walking in, twisting a foot to show the shoe,
and walking out. Replace the real shoe with an **AI-generated futuristic heel** that
stays consistent as the foot twists (inside vs. outside of the foot show different
sides of the same shoe). Target quality: **good enough for social** (convincing at
phone-screen scale and scrolling speed).

## Core design decision

Generate the shoe **once, as a single consistent asset** (a 3D model, or a multi-view
"turntable" set of images from one design), then render/show the **correct side** of
that one asset in each frame based on the foot's orientation. This is what keeps the
shoe from morphing through the twist — every angle is the same object.

The whole pipeline therefore hangs on one anchor: **knowing the foot's position and
orientation in every frame.**

## Phases

### Phase 0 — Tracking feasibility  ✅ (done)
- SAM 2 tracks the foot/shoe region across the clip from a single boxed frame.
- Findings: the **foot** tracks cleanly (big, opaque, high-contrast). A sparkly
  strappy shoe is hard to segment on its own (thin straps, skin showing through,
  low contrast on dark floor). A second shoe drops out when occluded behind the
  front foot — expected.
- Conclusion: **anchor to the foot, not the shoe.**

### Phase 1 — Foot anchor (position + orientation)  ← NEXT
- Per frame: foot mask (have it from SAM 2) + an **orientation signal**.
- First cheap version: principal-axis angle + elongation of the foot mask (PCA /
  image moments) — no new model, reuses Phase 0 output.
- If that signal is too coarse, upgrade to keypoint tracking (heel / toe / ankle
  via CoTracker) or a foot-pose model.
- Success = a smooth, readable orientation curve through the twist that we can use
  to pick which side of the shoe to show.

### Phase 2 — Shoe generation
- AI-generate a futuristic heel as a **consistent asset**:
  - text-to-3D (a 3D model we can render from any angle), or
  - multi-view image generation (a turntable of the same design).
- One design → all viewing angles, so the twist stays consistent by construction.

### Phase 3 — Placement & render
- For each frame, choose/interpolate the shoe view matching the foot orientation
  from Phase 1, position and scale it to the foot anchor, and **occlude** it with
  the foot mask (foot passes in front of straps where it should).
- MVP: sprite/billboard compositing driven by orientation (light, social-quality).
- Later: full per-frame 3D render with matched camera and lighting.

### Phase 4 — Polish
- Light video-diffusion pass for lighting/shadow blending and temporal smoothing.
- Contact shadow where the heel meets the floor.
- Keep the fixed background and body motion from the real footage untouched.

### Phase 5 — Package
- Simple flow: input a real clip + a text prompt ("chrome cyber stiletto") →
  output the finished video. Reuse across all your clips since the setup is
  identical every time (fixed camera, same background, same choreography).

## Honest risk list
- **Temporal consistency** — the illusion breaks if the shoe flickers frame to
  frame. Driving from one asset + a smooth orientation signal is the main defense.
- **The edge-on twist moment** — the hardest frames; the shoe turns thin/side-on.
- **Foot-shoe contact & shadows** — grounding the heel believably on the floor.
- **Occlusion realism** — skin showing correctly through/around straps.

## Not doing (and why)
- Per-frame image generation with no shared asset → shoe morphs across the twist.
- Training a video model from scratch → needs data/compute far beyond one creator;
  every viable path adapts existing models or uses classical CV + rendering.

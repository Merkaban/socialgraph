# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a single-file interactive visualization ("The shape of nearness") — a self-organizing social graph where nodes drift under force-directed physics, relationships decay without contact, and energy/synergy rules apply symmetrically to all nodes. No build tools, no dependencies, no tests.

## Development

Open `index.html` directly in a browser. No server required (though one can be used for live reload):

```
open index.html
# or
python3 -m http.server 8000
```

## Architecture

Everything lives in `index.html` — HTML structure, CSS styles, and a canvas-based physics simulation in a single IIFE:

- **Nodes** (~12-15 people): defined as data at the top of the script with properties (id, name, baseR, color, aura, texture pattern). Easy to edit.
- **Edges**: weighted connections between any pair of nodes (not just star topology). Each edge has a nominal `strength`, mutable `currentStrength` (decays/heals over time), visual width, color, and optional dashing for draining bonds.
- **Physics loop** (`tick`):
  - Computes edge activation based on distance (full within OPTIMAL, linear falloff over FALLOFF range)
  - **Edge decay/healing**: `currentStrength` drifts toward a floor when endpoints are apart; heals back toward nominal when close
  - **Force-directed drift**: attraction proportional to currentStrength toward a resting distance, repulsion for negative edges, mild all-pairs repulsion, velocity damping. Dragged nodes are pinned.
  - **Energy**: each node's energy = f(sum of active connection contributions), with super-linear synergy when 2+ positive bonds are active simultaneously. Applied symmetrically to ALL nodes.
- **Render loop** (`draw`): draws edges (thickness/color reflect currentStrength and activation), nodes (per-type texture functions), auras, labels, and updates the top-bar energy gauge (tracks "You" node).
- **Interaction**: pointer events for drag; canvas auto-sizes via `fitCanvas` on resize/orientationchange. iOS Safari canvas init hardened with double-rAF and fallback sizing.
- **Controls**: Pause/Play (stops drift + decay), Reset (positions + strengths), Shuffle (scatter to watch self-organization).

## Tuning Constants

Key constants exposed near the top of the script for easy adjustment:

| Constant | Default | Purpose |
|----------|---------|---------|
| `DECAY_RATE` | 0.003 | How fast neglected edges weaken per frame |
| `HEAL_RATE` | 0.008 | How fast maintained edges recover per frame |
| `STRENGTH_FLOOR` | 0.08 | Minimum residual strength for decayed positive edges |
| `ATTRACT_K` | 0.0004 | Attraction force multiplier |
| `REPULSE_K` | 800 | All-pairs repulsion constant |
| `NEG_REPULSE_K` | 1500 | Extra repulsion for negative edges |
| `REST_DIST` | 165 | Resting distance for attraction (px at scale=1) |
| `DAMPING` | 0.92 | Velocity damping per frame (lower = more friction) |
| `BOUNDARY_PAD` | 30 | Margin to keep nodes inside canvas |

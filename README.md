# experimental-UI

UI experiments based on scientific publications. Original interface components whose movement is computed from a real model
rather than drawn as a curve. Dependency free, no build step, no framework, no
CDN, dark page.

Live at https://amyleesterling.github.io/experimental-ui

## Why this is not in scifi-ui

Its sibling repo, [scifi-ui](https://github.com/amyleesterling/scifi-ui), holds
components extracted from software that actually ships, and it keeps that rule
strictly: never approximate a component that exists somewhere else and call it
extracted. Everything here is the other thing. It is original, it is
experimental, and none of it was lifted from a product. Keeping the two apart
means the claim each repo makes stays unambiguous.

Nothing here represents recorded data. Every trace, tissue, embryo, connectome
and readout is generated for the purpose of moving an interface.

## The set

### Motion, from a system

| Component | What moves it |
| --- | --- |
| `wave-progress` (`holowave`) | a wrapped gaussian packet crossing a track or a border |
| `sync-cluster` (`holosync`) | Kuramoto coupled oscillators that lock by themselves |
| `spring-motion` (`holospring`) | a damped harmonic oscillator, integrated per frame |

### Spatial, from five papers

| Component | The job |
| --- | --- |
| `atlas-field` (`holoatlas`) | a value that only means something in its place |
| `channel-wipe` (`holochannel`) | compare layers without losing registration |
| `scale-bridge` (`holobridge`) | jump six orders of magnitude and stay oriented |
| `projection-matrix` (`holoproject`) | show what a summary throws away |
| `section-stack` (`holostack`) | prove a set of layers is one object |

### Interaction states

| Component | The job |
| --- | --- |
| `action-state` (`holoaction`) | one control, four states, each moved by the model that suits it |
| `arrival` (`holoarrive`) | arriving and being unread are two different things |
| `veil` (`holoveil`) | a change with a definite instant |

## Pages

- `index.html`, a landing assembled from the set, with a boot sequence that is
  computed rather than choreographed.
- `components/motion.html`, the three models with their controls exposed.
- `components/spatial.html`, the five paper derived components.
- `components/applied.html`, the experiments placed in EyeWire 2: a
  proofreading wipe, a connectivity table that morphs onto its cell, EM
  sections closing into a volume, and a mapping from every component to a
  named panel in the eyewire-ii-community branch of ng-extend.
- `components/models.html`, nine models worn as interface: a
  contribution sunflower packed by phyllotaxis, a Poisson seeded celebration, a flow field
  empty state, a Fourier epicycle loader, avatars grown by reaction
  diffusion, dendritic progress by DLA, a Voronoi coverage map, batch
  actions that flock home, and a counter that glides on exponential
  decay. Under each element the original concept runs live, so every
  section pairs the UI with an interactive of the science behind it.
- `components/all.html`, everything on one page: the interaction states,
  the three motion models, the five spatial components, the nine model
  elements and the three applied mockups, in one scroll.

## Using one

Every component is two files that work alone. Take the pair, nothing else is
required except `hologram-tap.js` if you want the touch path.

```html
<link rel="stylesheet" href="components/spring-motion.css">
<script src="components/spring-motion.js"></script>
<script>
  holoSpring(document.getElementById("panel"), { label: "Detail" });
</script>
```

`hologram-tap.js` comes from scifi-ui unchanged. It is what puts `.holo-on` on
a tapped container, so every hover treatment in this repo is reachable on a
phone. See AGENTS.md for why that is not optional.

## Rules

Working rules, and the things that cost time to learn, are in
[AGENTS.md](AGENTS.md). Read it before adding anything.

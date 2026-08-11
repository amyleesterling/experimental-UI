# AGENTS.md, version 4

Working knowledge for anyone contributing to **experimental-ui**. Read it
before writing a line.

Version this file. When you learn something a future agent would otherwise pay
for again, add it and bump the version at the bottom.

---

## 1. What this repo is

Original interface components whose movement is computed from a real model
rather than drawn as a curve. Dependency free, no build step, no framework, no
CDN. Live at https://amyleesterling.github.io/experimental-ui, deployed by
pushing `main`.

- `index.html`: a landing assembled from the set.
- `components/`: one component per file pair, plus the demo pages.
- `components/models.html`: nine sketches sharing one page. A sketch is an
  interactive demonstration of a model, not yet a reusable component, so the
  one file per component rule does not apply to it. The promotion path is the
  rule instead: the moment a sketch is wanted in a second place, it gets
  extracted into its own css and js pair with the full contract, exactly as
  the travelling wave once was. Sketches still follow every behaviour rule:
  ambient loops only in view, one shots with backstops, reduced motion lands
  true, hover ships its tap path.
- `hologram-tap.js`: taken unchanged from scifi-ui, see section 4.

## 2. The claim this repo makes, and the one it does not

Its sibling [scifi-ui](https://github.com/amyleesterling/scifi-ui) holds
components extracted from software that actually ships, and it holds that rule
strictly: never approximate something that exists elsewhere and call it
extracted. This repo is the opposite case on purpose. Everything here is
original. Nothing is claimed to be extracted from anything.

That split is the reason both repos are readable. If you port something, it
belongs in scifi-ui and it has to be a real port. If you invent something, it
belongs here and it has to say so in its file header.

**Nothing here represents recorded data.** Traces, tissue, embryos,
connectomes and readouts are generated to move an interface. Where a component
borrows a way of thinking from published work, name the work and say plainly
that the drawing is schematic. Never let a component imply it is showing a
measurement.

## 3. Motion has to earn its place

The test a component passes before it exists: does it do a job an easing curve
could not? Carry state, encode a real quantity, or produce structure you would
otherwise hand author. A model used only because it sounds impressive is worse
than a transition, because it costs more and says less.

Three examples of the test being passed:

- A spring carries position **and velocity**, so a control grabbed halfway
  through has a correct answer already. A bezier has a fixed duration and can
  only snap or restart.
- Coupled oscillators produce coherence, which is a real state worth showing.
  Lights blinking in lockstep from the first frame are a decoration.
- A travelling packet gives waiting a direction. A pulse in place does not.

**Only ambient things loop.** Everything else runs once. An ambient loop also
stops when it scrolls out of view, because a loop running under a page nobody
is looking at is pure cost.

**Prefer a control that can take the evidence away.** Registration off,
collapse to a matrix, drag the divider back. A demo that can only show the good
state is a claim. One that can show the degraded state beside it is a
demonstration, and it usually costs one toggle.

## 4. Every hover state ships its tap path, in the same commit

Inherited from scifi-ui, and not negotiable here either. A hover treatment
hidden behind `@media (hover: none)` is not a decision, it is a group of users
getting a static page.

`hologram-tap.js` puts `holo-on` on the nearest activatable ancestor of a tap
and takes it off the previous one. Every `:hover` rule in this repo names
`.holo-on` as a second selector. Add `data-holo-tap` to a container you want
activatable, never to a control that already has `:active` and
`:focus-visible`. Whatever answers to hover also answers to focus, so extend
`:focus-within` at the same time. `prefers-reduced-motion` still wins: add the
class selector to the reduced motion rules too, or a tap resurrects what the
preference suppressed.

## 5. Two components must never share a base class

Grep the set before naming anything. Two base classes colliding is invisible
until a page loads both stylesheets, and that day is when a page tries to be
complete.

## 6. Reduced motion has to land somewhere true

Not frozen mid flight, and not simply switched off. Each component holds the
state its inputs would have reached: the oscillator cluster sits locked when
coupling is high and scattered when it is not, the spring jumps to its target,
the ladder shows every rung reached, and the veil still performs its swap.

The corollary, learned the hard way: anything that is **information** rather
than decoration keeps working under reduced motion. The unread mark in
`arrival` decays on its own clock in every mode, because it tells somebody
returning to a tab what they missed.

## 7. Teardown, and the backstop

Every `requestAnimationFrame` loop and canvas effect carries a teardown, plus a
timeout backstop. A loop that only tears down inside itself never tears down if
the first frame never arrives, and a full screen canvas then sits over the page
for its lifetime.

Anything with side effects outside the animation, like `holoveil` swapping
content, must run those effects exactly once on **every** exit path: the normal
finish, the backstop, and the reduced motion path. A transition that can drop
its swap leaves a region showing content that is no longer true, which is worse
than having no transition at all.

## 8. Verification, and how to do it here

Assert on behaviour, not on the stylesheet, whenever a real browser is
available. Playwright composites frames, so motion here has been verified by
driving it: the oscillator cluster measured going from 0.14 coherence to 0.95
after Connect and back to 0.15 after Scatter, and the underdamped spring
sampled across 160 frames to confirm it passes its target at 1.079 before
settling to exactly 1. Say which you did, and say plainly what you did not see.

**Always check overflow.** `scrollWidth` must equal `clientWidth` at 375 and
1280, measured with any page level `overflow-x: hidden` turned off so the
result is not masked, and measured in the **active** state with every tappable
lit. Un-hiding a hover treatment on a small screen is precisely how overflow
gets reintroduced.

**Auditing section 4 with the CSSOM needs one guard, or it silently passes.**
The obvious walker is wrong. In current Chrome a plain `CSSStyleRule` exposes
an empty `cssRules` list, because of nested CSS, so a walker that does
`if (rule.cssRules) { recurse; return; }` returns early on every style rule and
reports zero hover rules and zero problems. Test `rule.selectorText` first and
only recurse when `rule.cssRules.length` is non zero. Note also that `cssRules`
throws on a `file://` page, so serve the directory before auditing.

## 9. Things that cost time, so they do not cost it twice

**Offsetting a centreline needs the normal, not the tangent.** Written as
`a + Math.PI / 2` the offset direction becomes the tangent, so both edges slide
along the curve instead of away from it and the shape renders as a blade with a
fat middle and hairline ends. It looks like a thickness bug and it is a
direction bug. For a point at angle `a` on an arc the outward normal is
`(cos a, sin a)`. Two passes were spent tuning thickness numbers before the
direction was checked.

**Normalised canvas fractions are not a coordinate system.** Multiplying a 0 to
1 coordinate by width and height separately stretches any shape by the box
aspect. Anything that must keep its proportions needs a square space fitted and
centred in the canvas, `S = Math.min(W, H)`, with offsets taken from the
centre. Width and height fractions are only safe for things that should
stretch, like a full bleed gradient.

**A throttled readout needs its first report to be outside the window.**
Initialising `lastReport = 0` and painting once at setup computes `0 - 0`,
which is not greater than the interval, so the element ships without an
`aria-valuenow` until the loop happens to start. Invisible on screen, obvious
to a screen reader. Initialise the marker far in the past, and reset it before
any path that repaints outside the loop.

**An entrance and an unread mark are two different jobs.** Driving both from
one timeline quietly breaks the case that matters. The entrance is over in half
a second and only serves somebody who was looking. The mark is what serves
somebody who was not, so it needs its own clock, it clears on attention rather
than on a timer alone, and it survives reduced motion.

**One spring integrator can serve press, arrival and failure.** There are no
shake keyframes in this repo. The failure state in `action-state` is the same
integrator given an initial velocity and too little damping, so it rings
because that is what an underdamped system does and it settles honestly.

**Use two properties when two things move one element.** `action-state` takes
`transform` for the press scale and `translate` for the failure ring, so a
control that fails mid press composes instead of one animation clobbering the
other. This comes up any time a state machine can overlap its own states.

## 10. Feel

The components are meant to be oddly satisfying, and satisfaction is not a
coat of polish applied at the end. It comes from a handful of mechanics, all
cheap, all already in the set. Use them, and do not stack them: one felt
moment per event.

**Chase, do not jump.** A displayed value follows its truth through an
exponential approach, `disp += (truth - disp) * (1 - exp(-dt * k))`, and the
loop stops on settle. The embryo folds a beat behind the finger, the wipe
divider trails the pointer, the bar's fill glides to its number. The settle at
the end is most of the satisfaction, so never clamp it away. Keep the truth
and the display as separate variables: aria values, range inputs and readouts
report the truth, paint draws the chase.

**Completions arrive, they do not stop.** The wave bar at 100 percent sends
its packet out to the far end, blooms once, then rests with a quiet settled
glow and no animation at all. Done is a state with its own look, not the
absence of the busy look.

**Celebrate once.** The cluster blooms at the instant coherence crosses the
lock threshold, the button releases one ring on success, an arriving row
glints as its spring settles. Always one shot, never a loop, and the class
that fires it either comes off on a timer or marks an event that can only
happen once, per the replay rule. Celebrations are decoration: under reduced
motion they are suppressed entirely, while the state they celebrate still
lands.

**Draw marks, do not fade them in.** The check and the cross draw from tip to
tail through `stroke-dashoffset`, driven by the same spring that scales them,
with `min()` clamping the overshoot so the draw finishes clean. Set
`pathLength="1"` on the path and the stylesheet needs no measured lengths.

**Settle as a cascade.** When several siblings move to one target, give each
its own chased copy with a slightly different time constant. The section
stack closes like a liquid rather than a plate. The constants differ by
depth, so the cascade means something: nearer settles sooner.

**Spend time at the moment of meaning.** The veil's travel is eased, so the
packet is at its slowest exactly where the swap happens. Where an animation
carries an instant that matters, shape the timing so that instant gets the
most of it.

Every chase and settle loop follows section 7: it runs only while unsettled,
and it carries a backstop that lands the display on the truth.

## 11. Non-negotiables

- **No authentication, ever.** Inputs in demos are inert: no `<form>`, no field
  with a `name`, nothing submitted. Say "visual component only" in the copy.
- **No third party logos or wordmarks.**
- **Respect `prefers-reduced-motion`** in every component, verified by stubbing
  the preference rather than assuming the media query works.
- **Accessibility is not optional**: real anchors with real `href`s, labelled
  controls, `aria-pressed` and `aria-expanded` where they apply, live values on
  `role="meter"` and `role="progressbar"`, visible focus.
- **No em or en dashes in any prose.** Commas and periods. Amy's rule across
  every project.

---

Version 2, 2026-08-11. Version 1 split the set out of scifi-ui. Version 2
added section 10 with the feel pass, after every component was retuned to
chase, settle, arrive and celebrate.

Version 3, 2026-08-11. Added the sketches page and its promotion rule, and
two lessons from building it. Simulations under reduced motion should land on
a settled state computed synchronously, which means the grid or step budget
has to be chosen so that synchronous compute stays under a second; the
reaction diffusion grid is 104 for exactly that reason. And a pointer driven
canvas needs touch-action none on the stage, or the first drag on a phone
scrolls the page instead of driving the sketch.

Version 4, 2026-08-11. Two rules from assembling the everything page and the
concept panes. First, a model element ships with its concept: every section
on the models page pairs the UI element with an interactive of the original
science, in a `.concept` block, so a reader can drive the model that drives
the interface. Second, the section 5 rule about shared base classes now has
its proof: when the applied sections were lifted onto the everything page
without their `.stage canvas { width: 100%; height: 100% }` rule, the
canvases sized themselves from the container each frame while widening it,
a resize feedback loop that grew two canvases to 38,000 px and took
`getImageData` out of memory with them. A canvas that a script sizes from
its container must always carry the CSS that pins it back to that
container, and a section lifted onto another page must bring that CSS
along. That is also why the concept panes use `.cstage` and `.cread`
rather than the `.stage` and `.readout` the applied page already claims.

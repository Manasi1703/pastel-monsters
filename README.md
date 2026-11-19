# action cures anxiety.

Scribble a worry onto paper with oil pastels, press **bring it to life!**, and watch it grow legs, blink its googly eyes and wander off.

It's a single-page toy built with [three.js](https://threejs.org/) and WebGL. All the visuals, including the paper, the pastel strokes, the monsters and the music, are generated in the browser. There are no image or audio files.

---

## Run it

The page loads three.js from a CDN using an ES module import map, so it needs to be served over HTTP rather than opened as a file:

```bash
cd pastel-monsters
python3 -m http.server 5177
```

Then open <http://localhost:5177/>.

Any static file server works. There is nothing to install or build.

---

## How to play

1. **Pick a pastel** from the box at the bottom (or press `1`–`9`).
2. **Pick a stroke size**: thin, medium or fat.
3. **Scribble** a shape anywhere on the paper.
4. Press **bring it to life!** (or `Enter`). The drawing lifts off the paper and becomes a monster.
5. **Click a monster** to poke it and make it do something silly.

The paper is endless, so you can keep making monsters and go find them later.

### Controls

| Action | Mouse / trackpad | Keyboard | Touch |
| --- | --- | --- | --- |
| Draw | Left-drag | — | One-finger drag |
| Move around the paper | Scroll, right/middle-drag, or hold `Space` + drag | Arrow keys | Two-finger drag |
| Choose pastel | Click a crayon | `1`–`9` | Tap a crayon |
| Bring to life | **bring it to life!** button | `Enter` | Tap the button |
| Clear un-lifted drawing | **clear** button | `C` | Tap the button |
| Jump to a monster | **find one** button | `F` | Tap the button |
| Music on / off | Top-right pill | `M` | Tap the pill |

A pen or stylus that reports pressure makes lighter or heavier strokes.

---

## What makes it feel real

### Oil pastel on paper
- **Paper tooth:** the paper has a seamless, tiling surface texture with fibres. Pastel catches on the raised parts of the paper first and leaves specks of white in the dips.
- **Wax build-up:** going over an area again fills those dips and builds up thickness, the way real wax pastel packs into paper.
- **Pigment mixing:** overlapping colours mix the way real pigments do, so yellow over blue makes green.
- **Stroke character:** edges are ragged, stroke direction leaves wax streaks, and stray crumbs land near the line.
- **Lighting:** the paper and wax are lit as a height map, with a soft waxy sheen on thick areas.

### Monsters
When you bring a drawing to life, the page looks at its silhouette and gives it a body type and a personality:

| Shape | Body | Tends to… |
| --- | --- | --- |
| **Tall** | Long legs, long strides | do jumping jacks, wave, moonwalk |
| **Wide** | 3–4 stubby legs, waddles | sit, wiggle |
| **Round** | Bouncy gait | dance |
| **Spiky / scraggly** | Fast, noodly arms | throw tantrums, run |

The colour you used most also nudges its mood. Red gets hot-headed, yellow and orange dance more, blue and purple are calmer, green is sporty, pink waves and black moonwalks.

Monsters cycle through **walk, run, moonwalk, idle, dance, wiggle, tantrum, jumping jacks, wave and sit**. Each action comes with its own faces: happy closed eyes, a surprised "O", a zig-zag grumble and crossed eyes. Their googly eyes follow your cursor, and they blink.

They also keep a little personal space. They turn around rather than bump into each other, and nudge apart if they overlap.

### Music
A slow, soft ambient piece is generated live with the Web Audio API: warm pad chords (C, Am, F and G with a 7th or 6th) plus occasional kalimba-like notes, all through reverb. It starts after your first click or key press, because browsers block sound until you interact. It pauses when the tab is hidden and remembers whether you turned it off.

---

## How it's built

Everything lives in **`index.html`** (HTML, CSS, GLSL shaders and JavaScript).

| Part | What it does |
| --- | --- |
| **Paper texture** | A seamless 1024px paper texture is generated once by a shader and repeated across the whole sheet. |
| **Stroke layer** | The stroke you're drawing is collected in a screen-sized buffer, so overlapping parts of one stroke don't double up. |
| **Pigment tiles** | Finished strokes are saved into 512px tiles that are only created where you draw. Each tile overlaps its neighbours slightly so the edges blend without a visible line. |
| **Display** | Combines paper, pigment and the stroke in progress, and applies the lighting. |
| **Lifting a monster** | The pigment under your drawing is stitched into its own texture, and a 64×64 mask of it is analysed to decide the body type and where the hips, shoulders and eyes go. |
| **Monster rig** | The body bends, squashes and wobbles, with a slight hand-drawn jitter. Legs and arms are pastel-textured lines driven by a simple two-joint leg solver. Hands have three fingers. Eyes are shader-drawn. |
| **Behaviour** | A weighted random choice of actions, with every limb easing smoothly between poses. |
| **Music** | Web Audio oscillators, a generated reverb and a filter. No audio files. |

**Dependencies:** three.js `0.160.0` (loaded from unpkg) plus the Google Fonts **Averia Serif Libre** (title) and **Inter** (everything else).

---

## Tweaking

All in `index.html`:

- **Crayon colours:** edit the `CRAYONS` array (`hex` and `mood`).
- **Which actions monsters do:** edit `ACTIONS`, or the weights set in the `Monster` constructor.
- **How many monsters at once:** `MAX_MONSTERS` (default 24; the oldest fades out).
- **Paper whiteness:** `paperCol` in `DISPLAY_FRAG`.
- **Music:** the `CHORDS` and `SCALE` arrays, and the loudness the master volume fades up to (`0.32`) in `music.start()`.

---

## Browser support

Works in current desktop Chrome, Edge, Firefox and Safari. It needs WebGL2 and support for drawing into half-float textures. On phones it works with touch, but large drawings use more GPU memory.

## Known limitations

- Drawings that haven't been brought to life are cleared when you press **clear** or bring another drawing to life.
- Nothing is saved: reloading the page starts a fresh sheet.
- Very large drawings (over 2400px across) are discarded rather than turned into monsters.

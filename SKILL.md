---
name: "storify"
description: "Turn a scenario, case study or knowledge base into a hand-drawn animated explainer film with original robot characters, as an HTML file, artifact and/or MP4. Use when the user runs /storify or asks to storify or animate a story, case or lesson."
---

# /storify: animated story films in the "sketchbook diorama" style

You turn the user's story into a short, hand-drawn explainer film starring an original robot
cast. Every frame is drawn in JavaScript on a canvas, with procedural music and sound. The film
teaches: it directs the viewer's eye, tells a real story, and ends with a recap.

## Parameters

`/storify` accepts optional arguments. Any you're given skip the matching question:

| Argument | Values | Effect |
|---|---|---|
| `outputs=` | `html`, `artifact`, `mp4` (comma-separated) | skips the output question |
| `type=` | `before-after`, `how-it-works`, `problem-fix`, `comparison`, `cautionary`, `lessons` | forces the story type instead of analysing it |
| `characters=` | `1`–`4` | forces the cast size instead of analysing it |
| `mode=` | `click` (default) or `auto` | starting playback mode of the film |
| `title="…"` | text | forces the film's title |

Anything after the arguments, or any attached file, is the story.
Example: `/storify outputs=artifact,mp4 characters=2 -- <story text>`

## Workflow

Follow these steps in order. Keep a task list, because this is a long job.

### Step 1: Ask for the outputs (checkboxes)
Unless `outputs=` was given, ask with **AskUserQuestion**, `multiSelect: true`:
- question: "Which outputs should I create?"
- options:
  - **Artifact (Recommended)**: a shareable page with the interactive player
  - **HTML file**: the same film as a single .html file
  - **MP4 video**: a rendered video with the soundtrack, for slides, LMS or social posts

Create only what was chosen. If nothing is chosen, default to Artifact.

**Where you're running changes what's available.** The Artifact tool and SendUserFile exist in
the Claude app but not in the Claude Code CLI. If the Artifact tool isn't available, drop that
option (recommend **HTML file** instead). If SendUserFile isn't available, write the outputs to the
current working directory (or a folder the user names) and give their paths.

### Step 2: Ask for the story
Unless story text or files came with the command, ask in one short message:
"Paste your story, or attach it. It can be a scenario, a case study, or a knowledge base you
want to turn into a lesson. If there are numbers that must appear exactly, list them too."
Then wait for the reply. Read any attached files in full.

### Step 3: Analyse (before any code)
Work out five things from the story and write them down as a **plan**:

1. **Source kind**:
   - **Scenario:** events that happen to someone. Tell it as it happened.
   - **Case study:** a situation, decisions and outcomes. Put the key decision at the turning point.
   - **Knowledge base:** facts, concepts or procedures with no plot. You have to *build* a story
     around it: give the protagonist (Dennice) a goal that needs this knowledge (learning it,
     applying it, or getting it wrong first). Pick the 3–6 ideas that matter most. Turn abstract
     parts into physical metaphors (section "Visual style"). Use only facts from the source.
2. **Story type**: see "Storytelling", step 1. Say why you chose it in one line.
3. **Cast**: see "Cast analysis". Give each character's name, role and why they're in it.
4. **Facts ledger**: every number, name and claim that will appear on screen, each taken
   word for word from the source. Nothing else may appear as fact.
5. **Outline**: chapters, with a one-line purpose and a beat count for each (20–35 beats in
   total), the ending type (moral / key takeaway / what to remember / when to use which),
   and a draft of the ending sentence.

Show the plan and ask with **AskUserQuestion**: "Build it" (Recommended) / "Change the plan".
If the user is away or the run is unattended, state the plan at the top of your reply and build.

### Step 4: Build the film
Write one self-contained `.html` file, following every rule in the sections below. Start from
the reference code at the end of this skill: the helpers, the character drawer, the bubble and
the player. Keep the timeline and beats as data. Build the MP4 capture support in from the
start (see "MP4 rendering"), even if MP4 wasn't chosen; it's harmless in normal viewing. A
complete, working film is at `examples/regulatory-report-film.html` (next to this SKILL.md);
read it when you need a full reference. If the user has an earlier storify film of their own,
you may read that too.

### Step 5: Check it (one pass)
Use Playwright with Chromium to render 8–10 frames across all chapters, including the intro
card, one frame per character, and the recap. Scrub via the range input. Check:
- bubbles sit next to the speaker and never cover the focal point
- camera framing and no overlaps, including the chapter title against any top-center tracker
- no clipped text and no console errors
- every on-screen number matches the facts ledger
Fix, then check once more. Don't loop beyond that.

### Step 6: Deliver each chosen output
- **Artifact:** publish the HTML with the Artifact tool (`icon: "film"`) and a one-sentence
  description. Mention that it's private until shared.
- **HTML file:** send it with SendUserFile. If the user's computer is linked and they named a
  folder, also save it there.
- **MP4:** render it with the pipeline in "MP4 rendering". Files over 30 MB can't be sent in
  chat, and files over 20 MB can't be saved to the user's computer. So always also make a
  delivery copy with `ffmpeg -i film.mp4 -c:v libx264 -preset slow -crf 24 -pix_fmt yuv420p -c:a copy -movflags +faststart film-share.mp4`
  (about 13 MB per minute), raising `-crf` to 28–30 if it's still over 20 MB. Send it with
  SendUserFile, and save it to the named folder if the computer is linked.

Finish with one line per output, and one line on the story type and cast you chose.

---

## Cast analysis (how many characters)

1. List every **actor** in the source: people, roles, teams or personified systems that
   *decide, act or speak*.
2. An actor becomes an on-screen **character** only if at least one of these is true:
   - it acts or speaks in 2 or more beats
   - the story's tension depends on it (the reviewer who catches the error, the customer who
     complains, the new hire who has to learn)
   - it owns one side of a comparison
3. Everything else stays a **prop or label**: tools, systems, documents, products, companies.
   Claude and other real products are always shown as props and labels (shelves, crates,
   pipes, signs). **Never draw a real brand's logo, mascot or character.**
4. Size of cast (the roster has 26, but a film should use only what the story needs):
   - 1 is the default.
   - Use 2–3 when roles interact (hand-offs, reviews, teaching, conflict).
   - Use at most 4, and no more than 3 in one shot.
   - Merge minor roles into one ("the team").
5. Assign characters from the **roster**. Each character has a **role hint**: a casting hint
   only.
   - When a story role matches a hint (a dbt developer, a QA tester, a manager), cast that
     character.
   - If several match, pick one and keep them for the whole film.
   - If none match, cast any unused character. For example, a customer or regulator can be any
     character not already in the film.
   - **Never show or mention a character's role hint unless the story itself needs that role.**
     Beth is head of training, but in a story where she's simply a teammate, nobody says so.
   - The protagonist is **Dennice** by default, unless the story's main character clearly fits
     another role better.
6. **Names and pronouns:**
   - Dennice and Unix are girls (she/her).
   - For the other characters, use pronouns only if the user gives them. Otherwise, refer to
     them by name.
   - If the story gives real people's names, use the roster characters' names instead. Say in
     the plan who plays whom.

**Roster (26):**

| Character | Role hint | Look |
|---|---|---|
| **Dennice** (she) | analyst, protagonist, the doer or learner | periwinkle, 80×70, antenna with a yellow bulb |
| **Keith** | data engineer | coral, tall 74×86, twin antennae, "DATA" badge |
| **Unix** (she) | baby developer (junior, new to the job) | mint, small 62×56, leaf sprout |
| **Karan** | manager | mustard, boxy 82×72, visor face, blue tie |
| **Ian** | full-stack developer | plum, wide 96×62, dish antenna, round glasses |
| **Chris** | lead data engineer | sky blue, tall and narrow 64×92, spring antenna |
| **Isa** | senior data & martech consultant | rose, round 76×66, gold star antenna |
| **Jeson** | data engineer | olive, stocky 94×74, red cap |
| **Joeen** | data engineer | cocoa, 72×76, headphones |
| **Vlada** | dbt developer | tangerine, 70×74, hair bun |
| **Cannon** | QA tester | steel gray, 84×70, green check-flag antenna |
| **Wilber** | VP of engineering | navy, tall 78×94, hair tuft, red bow tie |
| **Samantha** | BI developer | peach, round 72×68, flower antenna |
| **Leo** | BI developer | lime, 70×72, blue beanie with pompom |
| **Rachelle** | dbt developer | aqua, 68×80, twin buns |
| **Jomari** | Salesforce | ice blue, wide 86×66, little cloud antenna |
| **Mark** | founder | crimson, 82×82, gold crown |
| **Jack** | martech consultant | green, 76×74, wifi antenna |
| **David** | solutions architect | sand, boxy 86×80, yellow hard hat |
| **Ryan** | Salesforce developer | cobalt, 72×70, `</>` sign antenna |
| **Beth** | head of training | magenta, 74×78, graduation cap |
| **Dean** | data engineer | sage gray, 88×72, spinning gear antenna |
| **Carlos** | data director | burgundy, tall 80×88, glasses and mustache |
| **Diego** | AI director | violet, 78×84, orbit (atom) antenna |
| **Myk** | HR lead | salmon, round 74×70, heart antenna |
| **JM** | dbt developer | mauve, 76×72, red propeller beanie |

Exact colors and shapes are in `CAST` in the character drawer code. A reference sheet of the
whole cast is at `examples/cast-sheet.png` (next to this SKILL.md), and an animated cast reel
at `examples/cast-reel.html`.

All characters share the same construction, sketch style, moods and poses, so they belong to
one world. They differ in silhouette, color and head detail, so they can be told apart even at
a glance or in grayscale.

**Staging with more than one character:**
- Each character has a **home side** of the stage and keeps it. This keeps screen direction
  consistent.
- Characters face whoever they're talking to. Pupils look at the speaker or the focal point.
- **Only one speaks per beat.** The bubble points at the speaker, and when the cast is larger
  than 1 it carries a small name chip in the speaker's color.
- The camera frames the speaker and what they're acting on. Use a two-shot (zoom about 1.15)
  for exchanges.
- A beat object gains `who:'keith'` (any CAST key) for the speaker, plus optional per-character
  pose and mood overrides.

---

## Storytelling

### Step 1: Story type (skip if `type=` was given)
Don't default to before/after. Match the story's own shape:

| Type | Choose when the story… | Intro kicker | Chapter arc | Ending banner |
|---|---|---|---|---|
| **Before and after** | shows a change and its measurable effect | `A BEFORE-AND-AFTER STORY` | before → turning point → after → result | MORAL OF THE STORY |
| **How it works** | explains a process, system or pipeline | `HOW IT WORKS` | overview → one chapter per stage → the whole thing running | KEY TAKEAWAY |
| **Problem and fix** | has one problem, its cause and its fix | `A PROBLEM AND ITS FIX` | symptom → cause → fix → confirmed | WHAT TO REMEMBER |
| **Comparison** | weighs two or more options | `A SIDE-BY-SIDE` | option A → option B → head-to-head → verdict | WHEN TO USE WHICH |
| **Cautionary tale** | shows a mistake and what it cost | `A CAUTIONARY TALE` | normal day → the mistake → the consequence → what should have happened | MORAL OF THE STORY |
| **Lessons** | teaches a set of tips or principles | `KEY TAKEAWAYS` | one short chapter per lesson, acted out | KEY TAKEAWAYS |

If a story mixes types, pick the **main** one: the shape it builds towards. Only use a moral
when the story has a real lesson about choices. Never force one.

### Step 2: Tell it as a story
- **Advance organizer (pre-training):** always open with an **intro card** that says what the
  film shows and what to watch for, before any action starts.
- **And, But, Therefore (Randy Olson):** the intro card is the story in three lines, each with a
  small mono label: the setup (**and**, ink), the tension (**but**, red), and what to watch for
  (**therefore**, teal). Labels depend on the type:

  | Type | Setup | Tension | What to watch |
  |---|---|---|---|
  | Before and after | THE JOB | THE PROBLEM | WATCH FOR |
  | How it works | THE SYSTEM | THE MYSTERY | WATCH HOW |
  | Problem and fix | THE SETUP | WHAT BROKE | WATCH THE FIX |
  | Comparison | THE CHOICE | THE CATCH | WATCH FOR |
  | Cautionary tale | THE ROUTINE | THE SLIP | WATCH WHAT IT COSTS |
  | Lessons | THE GOAL | THE TRAP | WATCH FOR |

  Above the three lines: the kicker in small mono, and a 40px title in the type's shape.
- **Pixar story spine:** "Once upon a time… Every day… One day… Because of that… Until finally…
  And ever since…". Every film has a normal state, a disturbance, a chain of consequences,
  a payoff and a meaning.
- **Signpost the arc:** the chapter titles at the top-left use the type's words, e.g.
  `before · months 1–2 in chat` or `stage 2 · compile`.
- **Show the tension; don't just state it.** Characters' moods and props carry it: tired and
  sweating, red flags and an alert face, puzzled, happy.

### Step 3: Chapters every film shares
0. **Intro · what this film shows:** the intro card on a wooden board, one line revealed per
   beat (hold ~3 s each). The stage is visible behind it, with no characters yet. The card
   fades out as the story starts.
1. **Setup:** introduce the protagonist, the task or system, and its parts.
2. **Body chapters:** the type's arc. Give each chapter 3–8 beats. Show a repeated pattern
   slowly once, then speed through the repeats.
3. **Payoff:** a hand-drawn chart board (if the story has numbers), the finished system
   running, or the verdict. Characters react.
4. **Recap (always last):**
   - The protagonist says one opener ("Let's recap what happened.") and steps to the edge of
     the frame.
   - A big board drops in on ropes ("what happened" / "how it works" / "what we learned") with
     3–4 numbered points revealed one per beat. The chapter names are the headings, in their
     semantic colors, and the points carry the facts-ledger numbers.
   - Then the ending banner reveals last, with a chime: 27px text, amber border, warm fill,
     one or two sentences, based on the story, with no numbers repeated.
   - The chapter title and ring fade out during the recap.
   - Recap beats hold 3–4 s.

---

## Directing the viewer's eye (most important)
- **One idea and one focal point per beat.**
- **Words go in the speaker's speech bubble, beside the action. Never use a caption bar.**
  - The bubble is in screen space above the speaker's head, with a tail pointing at them.
  - It's clamped inside the frame, with y ≥ 100 so it stays below the chapter title.
  - Use `side:true` to put it to the right when something above must stay clear.
  - Text is 25px hand font, at most 2 lines (~400px wide), 12 words or fewer, in first person.
- **Tell, then show.** Every beat has three phases:
  1. **Lead:** the story is frozen, the camera eases to the shot (1 s), and the line types in at
     ~34 characters a second, for 1–1.9 s.
  2. **Play:** the action runs at 0.85× story speed.
  3. **Hold:** 1.2 s in autoplay, or **wait** for a click in click-through.
  Beats may set their own `hold`.
- **Camera:** each beat has `cam:[x, y, zoom]`.
  - Wide (1.0) for a new place or a result.
  - Medium (1.1–1.35) for actions.
  - Close-up (1.45–1.6) for the detail that matters.
  - Ease over 1 s, and clamp so the frame never shows past the edge of the stage.
- **Signaling:**
  - A radial vignette darkens the edges by about 28%.
  - Pupils look toward the focal point.
  - New things pop in, and errors get a red "!" and a buzz.
- **Only the beat's main action animates strongly.** Ambient motion runs on real time and stays
  subtle, so holds and pauses still feel alive.
- This follows Mayer's multimedia principles (signaling, spatial and temporal contiguity,
  segmenting, redundancy, pre-training) and the staging principle from classical animation.

## Engine and player
- Single `.html` file, no build step. Canvas logical size **1280×720**, scaled to the container
  width with devicePixelRatio (cap 2), `aspect-ratio:16/9`.
- **Story time `s` drives everything:** `render(s)` draws the whole frame from `s`.
  - Character x-keyframes, pose intervals, item intervals, flight arcs, beats, chapters and
    sound events are all data at the top of the script.
  - Beats are `{s, e, say, who?, cam, side?, hold?}`. `who` is a CAST key; the default is the
    protagonist.
  - Intro beats use negative story time (e.g. −6…0), so the story itself starts at 0.
- **All real-time reads go through `NOW()`**, never `performance.now()` directly. This is
  what makes the MP4 capture possible:
  `let VCLOCK=null; const NOW=()=>VCLOCK??performance.now();`
- **Player bar:**
  - previous beat / play-pause-replay / next beat
  - scrubber, and a beat counter `7 / 34`
  - mode toggle: **Click-through (default, listed first)** | Autoplay
  - a **speed selector**, a button showing the current speed that opens a menu of
    0.5× / 0.75× / 1× normal / 1.25× / **1.5× (default)** / 2×. It stays visible on phones,
    closes on selection, outside click or Esc, and ↑/↓ move through the options.
    Use the tested code below.
  - sound (off by default), full screen
- Chapter chips sit below the player.
- **Speed selector (tested code).** HTML in the bar (replaces any single speed button):
```html
<div class="speed" id="speedWrap">
  <button id="spd" type="button" aria-haspopup="menu" aria-expanded="false" aria-controls="spdMenu" aria-label="Playback speed 1.5×">1.5×</button>
  <div class="speed-menu" id="spdMenu" role="menu" aria-label="Playback speed" hidden><div class="h">Speed</div></div>
</div>
```
```css
.speed{position:relative}
#spd{min-width:62px;justify-content:center;border:1px solid rgba(255,255,255,.22)}
#spd[aria-expanded="true"]{background:rgba(255,255,255,.16)}
.speed-menu{position:absolute;right:0;bottom:calc(100% + 8px);z-index:5;min-width:132px;padding:6px;border-radius:10px;
  background:var(--bar);border:1px solid rgba(255,255,255,.18);box-shadow:0 8px 24px rgba(0,0,0,.35);display:flex;flex-direction:column;gap:2px}
.speed-menu[hidden]{display:none}
.speed-menu .h{font:600 11px/1 "IBM Plex Mono",monospace;letter-spacing:.06em;text-transform:uppercase;opacity:.6;padding:6px 8px 4px}
.speed-menu button{justify-content:space-between;width:100%;padding:8px 10px}
.speed-menu button[aria-checked="true"]{background:rgba(255,255,255,.16)}
.speed-menu button[aria-checked="true"]::after{content:"✓"}
```
```js
const SPEEDS=[.5,.75,1,1.25,1.5,2],spdMenu=document.getElementById('spdMenu'),spdWrap=document.getElementById('speedWrap');
const spdLabel=v=>v+'×';
SPEEDS.forEach(v=>{const b=document.createElement('button');b.type='button';b.setAttribute('role','menuitemradio');b.dataset.v=v;
  b.textContent=v===1?'1× normal':spdLabel(v);b.onclick=e=>{e.stopPropagation();setSpeed(v);closeSpd(true);};spdMenu.appendChild(b);});
function setSpeed(v){speedMul=v;spd.textContent=spdLabel(v);spd.setAttribute('aria-label','Playback speed '+spdLabel(v));
  [...spdMenu.querySelectorAll('button')].forEach(b=>b.setAttribute('aria-checked',+b.dataset.v===v));}
function openSpd(){spdMenu.hidden=false;spd.setAttribute('aria-expanded','true');
  (spdMenu.querySelector('[aria-checked="true"]')||spdMenu.querySelector('button')).focus();}
function closeSpd(refocus){if(spdMenu.hidden)return;spdMenu.hidden=true;spd.setAttribute('aria-expanded','false');if(refocus)spd.focus();}
spd.onclick=e=>{e.stopPropagation();spdMenu.hidden?openSpd():closeSpd();};
document.addEventListener('click',e=>{if(!spdWrap.contains(e.target))closeSpd();});
spdMenu.addEventListener('keydown',e=>{const items=[...spdMenu.querySelectorAll('button')],i=items.indexOf(document.activeElement);
  if(e.key==='Escape'){e.preventDefault();closeSpd(true);}
  else if(e.key==='ArrowDown'||e.key==='ArrowUp'){e.preventDefault();items[(i+(e.key==='ArrowDown'?1:items.length-1))%items.length].focus();}
  e.stopPropagation();});
setSpeed(speedMul);
```
  The MP4 capture hook sets `speedMul=1` in `begin()`, so rendered videos keep their tested pacing.
- **Click-through:** the film stops after each beat and shows a pulsing "click to continue ▸"
  pill. Click, Space or → advances, and clicking during a beat finishes it.
- **Autoplay:** clicking the canvas pauses and shows a "❚❚ paused" pill. The music keeps
  playing. ← replays the current beat, or goes back one beat.
- `prefers-reduced-motion`: open in click-through mode, with no camera moves and no line boil.
- Wait for the fonts before the first frame (`document.fonts.load`, with a 1.5 s timeout).
- Start the file with `<meta charset="utf-8">`. Without it, local renders (Playwright, MP4)
  garble `·`, `–`, `→` and `Σ`.

## Visual style: "sketchbook diorama"
- **Background (cached offscreen):**
  - Paper `#F1E9D8` with diagonal stripes `#C5D3DB` at 60% opacity (85px band every 170px).
  - Faint protractor guides at `rgba(43,38,33,.10)`.
  - A ground band from y=598 in `#ECE2CA`, with a wobbly ink horizon and sand specks.
  - Paper grain.
  - For a camera that pans across a wide stage, tile a 1360px background (8 stripe periods,
    so tiles join seamlessly).
- **Lines:** every outline goes through `sk()`: densified points with ±1.1px deterministic
  jitter, stroked twice (the second pass at 40%), ink `#2B2621`, 2–2.4px wide.
  - **Line boil:** the jitter seed changes at 6 fps of real time.
- **Materials:** wood `#AE7A45` / `#7E5227` planks with grain and nails, paper labels `#FCF9F1`,
  sand `#E8D6AA`.
- **Props are physical metaphors:**
  - a workspace is a wooden crate with a signpost
  - storage is a shelf with labeled cubbies
  - sources are a mailbox or a stool of scrolls
  - documents are paper stacks, scrolls, folders, calculators or report sheets
  - a pipeline is a row of stations joined by dashed lines
  - a decision is a signpost fork
  - knowledge is a book or a lit lamp
- **Motion:**
  - Objects fly on quadratic arcs with a dashed blue trail (`rgba(60,95,175,.85)`, 6/8) and a
    start dot.
  - Things pop in with ease-out-back, and cleared things leave white puffs.
  - Links are dashed teal lines with a moving dash offset, and items flow along them.
  - Hatched fills show data and completed states.
- **Semantic colors:**

  | Colour | Hex | Meaning |
  |---|---|---|
  | red | `#C8452F` | problem or error |
  | amber | `#D08A2E` | old or manual way |
  | teal | `#2B8C82` | new way or fix |
  | blue | `#4F74B8` | neutral data |
  | accent | `#D2553F` | chapter ring |
  | yellow | `#F2C14E` | glow |

- **Type:** `Patrick Hand` for labels, bubbles and titles, and `IBM Plex Mono` for numbers and
  tags (Google Fonts, with fallbacks).
- **Frame furniture:**
  - The chapter title sits top-left (64, 62) in 26px hand font, lowercase, with an amber
    underline. It fades in only when the chapter changes. Keep it to 24 characters or fewer
    (e.g. `after · months 3–8`), so it never runs into a top-center tracker.
  - The chapter ring sits top-right (1188, 64).
  - An optional progress tracker (e.g. month boxes) can go top center.

## Characters: moods and poses (shared by the whole roster)
- **Moods (face only):**
  - `normal`: oval eyes that blink every ~3.7 s, small smile
  - `happy`: ^ ^ eyes, wide smile
  - `tired`: half-lidded eyes, flat mouth, sweat drop
  - `alert`: wide eyes with pupils, "o" mouth, red "!" badge
  - `puzzled`: one eye small, wavy mouth, "?" badge
  - `calm`: closed, smiling eyes (meditating, content)
- **Poses:**
  - `idle` (sway)
  - `walk` (leg swing, bounce, faces the direction of travel, hops over box walls)
  - `carry` (item held over the head)
  - `type` (laptop)
  - `mag` (magnifying glass)
  - `throw`
  - `point` (one arm out toward the focal point)
  - `talk` (small hand gestures while their bubble is up)
  - `wave` (one arm up, waving)
  - `cheer` / `celebrate` (jumping, confetti)
  - custom actions (dribbling, cooking, juggling…): pass `hands` and draw the props at the
    returned hand positions
- Mood follows the story beat by beat.
- Don't mirror (`dir:-1`) a character that wears text, like Keith's DATA badge; the text
  would read backwards.

## Sound
All sound is procedural Web Audio. It starts only when the viewer turns sound on.
- **Synth functions schedule at `AT??ac.currentTime`** on whatever `ac` and `master` are
  current. The same code then drives live playback and the offline MP4 render (see
  "MP4 rendering").
- **Music:**
  - triangle plucks on a **0.3 s step**, with a sine bass every 8 steps
  - one pattern per chapter: intro gentle and sparse, setup major pentatonic, problem sparse
    minor, turning point rising arpeggio, solution bright major, finale high arpeggio,
    recap warm
  - note volume 0.04–0.05
  - it runs on its own lookahead clock (a 60 ms timer that schedules 0.25 s ahead), so it keeps
    playing while the film is paused or waiting
  - its own gain bus **ducks to 55%** during each beat's lead-in
- **Sound effects:** whoosh, thud, pop, buzz (errors), ding, chime, big (milestones), poof,
  click (typing), tick (clock).
  - They fire from the event list only while playing forward, never while scrubbing.

## Accuracy
- Only facts from the ledger appear on screen, and chart values are drawn to scale.
- Use the source's own terms for labels.
- For a knowledge base, analogies are allowed only if they're visibly metaphors (props). They
  must never introduce new claims.

---

## MP4 rendering (tested)
The MP4 is the **autoplay** version, with sound on, at 1920×1080 and 30 fps. A ~2.5-minute film
renders in about 5 minutes and comes out at around 50 MB.

1. **Build the film with capture support from the start.**
   - Declare the clock and logs near the top:
     `let VCLOCK=null,CAPTURE=false,AT=null;const NOW=()=>VCLOCK??performance.now();const LOG=[],LOGC=[],LOGD=[];`
   - Every real-time read uses `NOW()`, except the rAF `last` bookkeeping. **Don't
     search-and-replace `performance.now()` blindly:** that also rewrites the one inside `NOW`
     itself and causes infinite recursion.
   - `tone()` and `noise()` schedule at `AT??ac.currentTime`. `noise` must call `src.start(t0)`.
   - `sfx()` routes every sound through `fire(n)`. `fire` logs `{t:VCLOCK/1000,name:n}` while
     `CAPTURE` is on, and plays live otherwise. While capturing, `sfx()` must not return early
     just because sound is off.
   - `fit()` and the rAF `frame()` both return immediately while `CAPTURE` is on.
2. **Add the capture hook** (tested code):
```js
// ---------- capture hook for MP4 rendering (inactive in normal viewing) ----------
function wavB64(buf){const ch=buf.numberOfChannels,len=buf.length,sr=buf.sampleRate,dv=new DataView(new ArrayBuffer(44+len*ch*2));
  const w=(o,str)=>{for(let i=0;i<str.length;i++)dv.setUint8(o+i,str.charCodeAt(i));};
  w(0,'RIFF');dv.setUint32(4,36+len*ch*2,true);w(8,'WAVE');w(12,'fmt ');dv.setUint32(16,16,true);dv.setUint16(20,1,true);dv.setUint16(22,ch,true);
  dv.setUint32(24,sr,true);dv.setUint32(28,sr*ch*2,true);dv.setUint16(32,ch*2,true);dv.setUint16(34,16,true);w(36,'data');dv.setUint32(40,len*ch*2,true);
  const data=[];for(let c=0;c<ch;c++)data.push(buf.getChannelData(c));let o=44;
  for(let i=0;i<len;i++)for(let c=0;c<ch;c++){const v=Math.max(-1,Math.min(1,data[c][i]));dv.setInt16(o,v<0?v*0x8000:v*0x7FFF,true);o+=2;}
  const u8=new Uint8Array(dv.buffer);let bin='';for(let i=0;i<u8.length;i+=0x8000)bin+=String.fromCharCode.apply(null,u8.subarray(i,i+0x8000));return btoa(bin);}
window.__storify={
  begin(){CAPTURE=true;cv.width=1920;cv.height=1080;BASE=1.5;mode='auto';paused=false;speedMul=1;VCLOCK=0;
    LOG.length=0;LOGC.length=0;LOGD.length=0;lastChap=-1;duck=-1;enter(0,true);},
  step(ms){VCLOCK+=ms;update(ms/1000);
    const ci=chapAt(s);if(ci!==lastChap){lastChap=ci;chapT0=NOW();LOGC.push({t:VCLOCK/1000,ci});}
    const d=phase==='lead'?.55:1;if(d!==duck){duck=d;LOGD.push({t:VCLOCK/1000,d});}
    render(s);return phase==='end';},
  frame(q=.9){return cv.toDataURL('image/jpeg',q).split(',')[1];},
  async audio(dur){const sr=44100,oc=new OfflineAudioContext(2,Math.ceil(sr*dur),sr);
    const keep=[ac,master,musicBus];ac=oc;master=oc.createGain();master.gain.value=.55;master.connect(oc.destination);
    musicBus=oc.createGain();musicBus.connect(master);
    LOGD.forEach(e=>musicBus.gain.setTargetAtTime(e.d,e.t,.25));
    const chapAtT=tt=>{let c=0;LOGC.forEach(e=>{if(e.t<=tt)c=e.ci;});return c;};
    for(let n=0,tt=.05;tt<dur;n++,tt+=.3){const m=MUS[chapAtT(tt)],note=m.seq[n%m.seq.length];
      if(note)toneAt(mtof(note),.5,'triangle',m.v,tt,musicBus);if(n%8===0)toneAt(mtof(m.root),1.6,'sine',.07,tt,musicBus);}
    LOG.forEach(e=>{AT=e.t;SFX[e.name]();});AT=null;
    const buf=await oc.startRendering();[ac,master,musicBus]=keep;return wavB64(buf);}
};
```
3. **Render with `render.js`** (tested code), run with
   `node render.js film.html out.mp4 [maxSeconds] [fontDir]`.
   - Do a 10–15 s test render first (`maxSeconds=14`), then the full render in the background
     with `nohup`, polling the log.
   - Don't stop a running render with `pkill -f` on a pattern that also appears in your own
     shell command; it kills the shell too. Check with `ps` first.
```js
// Usage: node render.js <film.html> <out.mp4> [maxSeconds] [fontDir]
const { chromium } = require('playwright');
const { spawn } = require('child_process');
const fs = require('fs'), path = require('path');
const [,, film, out, maxArg, fontDir] = process.argv;
const FPS = 30, MAX = maxArg ? +maxArg : 600, TAIL = 2;
(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage({ viewport: { width: 1920, height: 1200 } });
  page.on('pageerror', e => console.error('pageerror', e.message));
  await page.goto('file://' + path.resolve(film));
  if (fontDir) { // local font files, for environments that can't reach Google Fonts
    const faces = fs.readdirSync(fontDir).filter(f => /\.(ttf|otf|woff2?)$/.test(f)).map(f => {
      const fam = /patrick/i.test(f) ? 'Patrick Hand' : /plex/i.test(f) ? 'IBM Plex Mono' : null;
      const wt = /semibold/i.test(f) ? 600 : /medium/i.test(f) ? 500 : 400;
      return fam && `@font-face{font-family:"${fam}";font-weight:${wt};src:url("file://${path.resolve(fontDir, f)}");}`;
    }).filter(Boolean).join('\n');
    await page.addStyleTag({ content: faces });
  }
  await page.evaluate(async () => { try { await document.fonts.load('26px "Patrick Hand"'); await document.fonts.load('600 12px "IBM Plex Mono"'); } catch (e) {} });
  await page.waitForTimeout(800);
  const tmpV = out.replace(/\.mp4$/, '.video.mp4');
  const ff = spawn('ffmpeg', ['-y', '-loglevel', 'error', '-f', 'image2pipe', '-framerate', String(FPS), '-i', '-',
    '-c:v', 'libx264', '-preset', 'medium', '-pix_fmt', 'yuv420p', '-crf', '20', tmpV]);
  ff.stderr.on('data', d => process.stderr.write(d));
  await page.evaluate(() => window.__storify.begin());
  let frames = 0, endAt = null; const t0 = Date.now();
  while (true) {
    const done = await page.evaluate(ms => window.__storify.step(ms), 1000 / FPS);
    const b64 = await page.evaluate(() => window.__storify.frame(.9));
    const buf = Buffer.from(b64, 'base64');
    if (!ff.stdin.write(buf)) await new Promise(r => ff.stdin.once('drain', r));
    frames++;
    if (done && endAt === null) endAt = frames + TAIL * FPS;
    if ((endAt !== null && frames >= endAt) || frames >= MAX * FPS) break;
    if (frames % 300 === 0) console.log(`${frames / FPS}s rendered (${((Date.now() - t0) / 1000).toFixed(0)}s wall)`);
  }
  ff.stdin.end(); await new Promise(r => ff.on('close', r));
  const dur = frames / FPS;
  console.log('video frames', frames, 'duration', dur.toFixed(2));
  const wav = await page.evaluate(d => window.__storify.audio(d), dur);
  const tmpA = out.replace(/\.mp4$/, '.wav'); fs.writeFileSync(tmpA, Buffer.from(wav, 'base64'));
  await browser.close();
  await new Promise((res, rej) => { const m = spawn('ffmpeg', ['-y', '-loglevel', 'error', '-i', tmpV, '-i', tmpA, '-c:v', 'copy', '-c:a', 'aac', '-b:a', '160k', '-shortest', '-movflags', '+faststart', out]);
    m.stderr.on('data', d => process.stderr.write(d)); m.on('close', c => c ? rej(new Error('mux failed')) : res()); });
  fs.unlinkSync(tmpV); fs.unlinkSync(tmpA);
  console.log('wrote', out, 'in', ((Date.now() - t0) / 1000).toFixed(0), 's');
})().catch(e => { console.error(e); process.exit(1); });
```
4. **Fonts:** the sandbox usually can't reach Google Fonts or font downloads, so the video
   would fall back to a plain serif.
   - Check for a `storify-fonts` folder on the user's linked computer (usually on the Desktop).
     If it's there, stage the `.ttf` files and pass the folder as `fontDir`.
   - If it isn't, render anyway. Then tell the user the text uses a stand-in font, and ask them
     to put Patrick Hand and IBM Plex Mono (Medium and SemiBold) `.ttf` files from
     fonts.google.com into `Desktop/storify-fonts`, so you can re-render.
5. **Check the result:**
   - `ffprobe` shows a 1920×1080 video stream and an audio stream of the same duration
   - `volumedetect` gives a mean around −40 dB and a max around −20 dB
   - pull frames at 10%, 50% and 90% and look at them

## Reference code (start from this; adapt, don't rewrite from scratch)

This code is from the reference film and the tested cast drawer. When you adapt it:
- replace `performance.now()` with `NOW()` (except inside `NOW` itself)
- draw every character with `drawChar(CAST[key], …)`, one call per cast member. `drawChar`
  accepts `hands` (base coordinates) for custom actions, and returns the world-space hand
  positions so props can be attached to hands
- have `bubble()` take the speaker's head position and draw a name chip in `CAST[who].body` when
  the cast is larger than 1.
- the helpers already declare `GX`; don't reuse that name for anything else.

### Helpers, sketch primitives and cached background
```js
const HAND = '"Patrick Hand","Comic Sans MS",cursive', MONO = '"IBM Plex Mono",ui-monospace,monospace';
const INK='#2B2621', PAPER='#F1E9D8', STRIPE='#C5D3DB', WOOD='#AE7A45', WOOD_D='#7E5227',
      SAND='#E8D6AA', PAPERW='#FCF9F1', SCREEN='#262439',
      BULB='#F2C14E', RED='#C8452F', AMBER='#D08A2E', TEAL='#2B8C82', BLUE='#4F74B8', RING='#D2553F';
/* ---------- helpers ---------- */
const clamp=(v,a,b)=>Math.max(a,Math.min(b,v));
const seg=(t,a,b)=>clamp((t-a)/(b-a),0,1);
const ease=x=>x<.5?2*x*x:1-Math.pow(-2*x+2,2)/2;
const back=x=>{const c1=1.70158,c3=c1+1;return x<=0?0:1+c3*Math.pow(x-1,3)+c1*Math.pow(x-1,2);};
const inAny=(t,iv)=>iv.some(([a,b])=>t>=a&&t<b);
function track(k,t){if(t<=k[0][0])return k[0][1];for(let i=1;i<k.length;i++){if(t<=k[i][0]){const[a,v0]=k[i-1],[b,v1]=k[i];return v0+(v1-v0)*ease(seg(t,a,b));}}return k[k.length-1][1];}
function hsh(n){n=Math.sin(n*127.1+311.7)*43758.5453;return n-Math.floor(n);}
let BOIL=0, A=0, GX=0;
const jit=(s,i,a)=>(hsh(s*13.37+i*7.13+BOIL*3.1)-.5)*2*a;

function densify(p,closed,step){const o=[],n=p.length,m=closed?n:n-1;for(let i=0;i<m;i++){const a=p[i],b=p[(i+1)%n],d=Math.hypot(b[0]-a[0],b[1]-a[1]),k=Math.max(1,Math.ceil(d/step));for(let j=0;j<k;j++)o.push([a[0]+(b[0]-a[0])*j/k,a[1]+(b[1]-a[1])*j/k]);}if(!closed)o.push(p[n-1]);return o;}
function rr(x,y,w,h,r){r=Math.min(r,w/2,h/2);const p=[],c=[[x+w-r,y+r,-Math.PI/2],[x+w-r,y+h-r,0],[x+r,y+h-r,Math.PI/2],[x+r,y+r,Math.PI]];for(const[cx,cy,a0]of c)for(let k=0;k<=5;k++){const a=a0+k/5*Math.PI/2;p.push([cx+Math.cos(a)*r,cy+Math.sin(a)*r]);}return densify(p,true,10);}
function ell(cx,cy,rx,ry,n=18){const p=[];for(let i=0;i<n;i++){const a=i/n*Math.PI*2;p.push([cx+Math.cos(a)*rx,cy+Math.sin(a)*ry]);}return p;}
function ln(x1,y1,x2,y2){return densify([[x1,y1],[x2,y2]],false,12);}
function sk(pts,o={}){
  const {closed=true,seed=1,amp=1.1,fill=null,stroke=INK,lw=2.2,passes=2}=o;
  if(fill){ctx.beginPath();pts.forEach((p,i)=>i?ctx.lineTo(p[0],p[1]):ctx.moveTo(p[0],p[1]));ctx.closePath();ctx.fillStyle=fill;ctx.fill();}
  if(!stroke)return;
  ctx.strokeStyle=stroke;ctx.lineWidth=lw;ctx.lineJoin='round';ctx.lineCap='round';
  const ga=ctx.globalAlpha;
  for(let q=0;q<passes;q++){ctx.globalAlpha=ga*(q?.4:1);ctx.beginPath();const n=pts.length,L=closed?n+1:n;
    for(let i=0;i<L;i++){const p=pts[i%n],x=p[0]+jit(seed+q*5,i%n,amp),y=p[1]+jit(seed+q*5+99,i%n,amp);i?ctx.lineTo(x,y):ctx.moveTo(x,y);}ctx.stroke();}
  ctx.globalAlpha=ga;
}
function txt(s,x,y,{size=20,color=INK,align='center',font=HAND,base='middle'}={}){ctx.font=size+'px '+font;ctx.fillStyle=color;ctx.textAlign=align;ctx.textBaseline=base;ctx.fillText(s,x,y);}
function dot(x,y,r,c){ctx.beginPath();ctx.arc(x,y,r,0,7);ctx.fillStyle=c;ctx.fill();}
function plank(x,y,w,h,seed,color=WOOD){
  sk(rr(x,y,w,h,3),{seed,fill:color,amp:.8,lw:2});
  ctx.strokeStyle='rgba(70,40,15,.35)';ctx.lineWidth=1.1;
  const horiz=w>=h;
  for(let k=1;k<=2;k++){ctx.beginPath();
    if(horiz){const yy=y+h*k/3;for(let xx=x+6;xx<=x+w-6;xx+=12){const yj=yy+Math.sin(xx*.05+seed+k)*1.2;xx===x+6?ctx.moveTo(xx,yj):ctx.lineTo(xx,yj);}}
    else{const xx=x+w*k/3;for(let yy=y+6;yy<=y+h-6;yy+=12){const xj=xx+Math.sin(yy*.05+seed+k)*1.2;yy===y+6?ctx.moveTo(xj,yy):ctx.lineTo(xj,yy);}}
    ctx.stroke();}
  if(horiz){dot(x+6,y+h/2,1.8,INK);dot(x+w-6,y+h/2,1.8,INK);}else{dot(x+w/2,y+6,1.8,INK);dot(x+w/2,y+h-6,1.8,INK);}
}
function tag(s,x,y,{size=16,seed=7,font=HAND,fill=PAPERW,color=INK}={}){
  ctx.font=size+'px '+font;const w=ctx.measureText(s).width+18,h=size+12;
  sk(rr(x-w/2,y-h/2,w,h,3),{seed,fill,lw:1.6,amp:.6});txt(s,x,y+1,{size,font,color});
}
function hatch(x,y,w,h,color,seed){
  sk(rr(x,y,w,h,2),{seed,fill:color+'33',stroke:color,lw:2,amp:.8});
  ctx.save();ctx.beginPath();ctx.rect(x+2,y+2,w-4,h-4);ctx.clip();ctx.strokeStyle=color;ctx.lineWidth=1.6;ctx.globalAlpha*=.7;
  for(let k=-h;k<w;k+=8){ctx.beginPath();ctx.moveTo(x+k+jit(seed,k,1),y+h);ctx.lineTo(x+k+h+jit(seed+3,k,1),y);ctx.stroke();}ctx.restore();
}

/* ---------- cached background ---------- */
const bg=document.createElement('canvas');bg.width=W*2;bg.height=H*2;
(function(){const b=bg.getContext('2d');b.scale(2,2);
  b.fillStyle=PAPER;b.fillRect(0,0,W,H);
  b.fillStyle=STRIPE;b.globalAlpha=.6;
  for(let k=-H;k<W+H;k+=170){b.beginPath();b.moveTo(k,0);b.lineTo(k+85,0);b.lineTo(k+85-H,H);b.lineTo(k-H,H);b.closePath();b.fill();}
  b.globalAlpha=1;
  b.strokeStyle='rgba(43,38,33,.10)';b.lineWidth=1;
  [230,360].forEach(r=>{b.beginPath();b.arc(660,600,r,Math.PI,2*Math.PI);b.stroke();});
  for(let a=0;a<=72;a++){const g=Math.PI+a/72*Math.PI,r1=360,r2=a%6?368:380;b.beginPath();b.moveTo(660+Math.cos(g)*r1,600+Math.sin(g)*r1);b.lineTo(660+Math.cos(g)*r2,600+Math.sin(g)*r2);b.stroke();}
  b.beginPath();b.moveTo(660,230);b.lineTo(660,600);b.moveTo(250,470);b.lineTo(1080,470);b.stroke();
  b.fillStyle='#ECE2CA';b.fillRect(0,598,W,H-598);
  b.strokeStyle=INK;b.lineWidth=2;b.beginPath();for(let x=0;x<=W;x+=14){const y=598+Math.sin(x*.03)*1.2+(Math.random()-.5)*1.2;x?b.lineTo(x,y):b.moveTo(x,y);}b.stroke();
  for(let i=0;i<260;i++){b.fillStyle='rgba(120,95,60,.35)';b.beginPath();b.arc(Math.random()*W,606+Math.random()*110,Math.random()*1.6+.4,0,7);b.fill();}
  for(let i=0;i<12000;i++){b.fillStyle='rgba(60,45,30,'+(Math.random()*.06)+')';b.fillRect(Math.random()*W,Math.random()*H,1,1);}
})();
```

### Character drawer (whole roster, tested)
```js
// Cast roster (26). All share one construction; they differ by silhouette, color and head detail.
// role = casting hint only; it never has to appear on screen.
function shade(hex,f){const n=parseInt(hex.slice(1),16);const c=[n>>16,(n>>8)&255,n&255].map(v=>Math.max(0,Math.min(255,Math.round(v*f))));return '#'+c.map(v=>v.toString(16).padStart(2,'0')).join('');}
const CAST={
  dennice:{name:'Dennice',role:'analyst · protagonist',body:'#8C95EC',w:80,h:70,r:28,head:'bulb'},
  keith:{name:'Keith',role:'data engineer',body:'#F08A6E',w:74,h:86,r:26,head:'twin',badge:'DATA'},
  unix:{name:'Unix',role:'baby developer',body:'#7CC9A8',w:62,h:56,r:28,head:'leaf'},
  karan:{name:'Karan',role:'manager',body:'#E7B94C',w:82,h:72,r:10,head:'none',visor:true,tie:true},
  ian:{name:'Ian',role:'full-stack developer',body:'#A884D8',w:96,h:62,r:30,head:'dish',glasses:true},
  chris:{name:'Chris',role:'lead data engineer',body:'#62B4E4',w:64,h:92,r:22,head:'spring'},
  isa:{name:'Isa',role:'senior data & martech consultant',body:'#E98FB6',w:76,h:66,r:33,head:'star'},
  jeson:{name:'Jeson',role:'data engineer',body:'#93AA5C',w:94,h:74,r:18,head:'cap'},
  joeen:{name:'Joeen',role:'data engineer',body:'#B8906C',w:72,h:76,r:24,head:'phones'},
  vlada:{name:'Vlada',role:'dbt developer',body:'#F4A259',w:70,h:74,r:26,head:'bun'},
  cannon:{name:'Cannon',role:'QA tester',body:'#98A6B5',w:84,h:70,r:16,head:'flag'},
  wilber:{name:'Wilber',role:'VP of engineering',body:'#4F6DB0',w:78,h:94,r:24,head:'tuft',bowtie:true},
  samantha:{name:'Samantha',role:'BI developer',body:'#F7C3A1',w:72,h:68,r:34,head:'flower'},
  leo:{name:'Leo',role:'BI developer',body:'#BCD65A',w:70,h:72,r:22,head:'beanie'},
  rachelle:{name:'Rachelle',role:'dbt developer',body:'#72D1C8',w:68,h:80,r:30,head:'twinbuns'},
  jomari:{name:'Jomari',role:'Salesforce',body:'#A9D6F5',w:86,h:66,r:32,head:'cloud'},
  mark:{name:'Mark',role:'founder',body:'#D35454',w:82,h:82,r:30,head:'crown'},
  jack:{name:'Jack',role:'martech consultant',body:'#4E9E6E',w:76,h:74,r:20,head:'wifi'},
  david:{name:'David',role:'solutions architect',body:'#D9C58E',w:86,h:80,r:14,head:'hardhat'},
  ryan:{name:'Ryan',role:'Salesforce developer',body:'#3F7FD6',w:72,h:70,r:26,head:'code'},
  beth:{name:'Beth',role:'head of training',body:'#C860B4',w:74,h:78,r:28,head:'gradcap'},
  dean:{name:'Dean',role:'data engineer',body:'#7F9C96',w:88,h:72,r:18,head:'gear'},
  carlos:{name:'Carlos',role:'data director',body:'#9C4A5E',w:80,h:88,r:24,head:'none',glasses:true,mustache:true},
  diego:{name:'Diego',role:'AI director',body:'#7A5CC7',w:78,h:84,r:28,head:'orbit'},
  myk:{name:'Myk',role:'HR lead',body:'#F29E9E',w:74,h:70,r:32,head:'heart'},
  jm:{name:'JM',role:'dbt developer',body:'#B07AA1',w:76,h:72,r:20,head:'propeller'}};
for(const k in CAST){const c=CAST[k];c.key=k;if(!c.dark)c.dark=shade(c.body,.72);}
function heartPath(cx,cy,s){ctx.beginPath();ctx.moveTo(cx,cy+s*.9);ctx.bezierCurveTo(cx-s*1.4,cy-s*.1,cx-s*.7,cy-s*1.2,cx,cy-s*.45);ctx.bezierCurveTo(cx+s*.7,cy-s*1.2,cx+s*1.4,cy-s*.1,cx,cy+s*.9);ctx.closePath();}
function drawHead(c,W2,hb,t,seed){
  const stem=(len)=>sk(ln(0,hb,0,hb-len),{closed:false,seed:seed+6,lw:2.5,passes:1,amp:.3});
  switch(c.head){
  case 'bulb':sk(ln(0,hb,3,hb-15),{closed:false,seed:seed+6,lw:2.5,passes:1,amp:.4});
    ctx.globalAlpha*=.35+.25*Math.sin(t*4);dot(3,hb-20,9,BULB);ctx.globalAlpha=1;sk(ell(3,hb-20,5,5,10),{seed:seed+7,fill:BULB,lw:1.8,amp:.3});break;
  case 'twin':[-1,1].forEach(k=>{sk(ln(k*W2*.4,hb+2,k*W2*.55,hb-12),{closed:false,seed:seed+8+k,lw:2.5,passes:1,amp:.4});
    sk(ell(k*W2*.55,hb-15,4.5,4.5,10),{seed:seed+10+k,fill:c.dark,lw:1.6,amp:.3});});break;
  case 'leaf':ctx.strokeStyle=INK;ctx.lineWidth=2.5;ctx.beginPath();ctx.moveTo(0,hb);ctx.quadraticCurveTo(-2,hb-10,4,hb-15);ctx.stroke();
    ctx.save();ctx.translate(8,hb-17);ctx.rotate(-.5+Math.sin(t*2)*.12);sk(ell(0,0,8,4,12),{seed:seed+12,fill:'#8FD694',lw:1.6,amp:.3});ctx.restore();break;
  case 'spring':ctx.strokeStyle=INK;ctx.lineWidth=2.4;ctx.beginPath();ctx.moveTo(0,hb);
    for(let k=1;k<=6;k++)ctx.lineTo((k%2?-5:5),hb-k*3.2);ctx.stroke();sk(ell(0,hb-23+Math.sin(t*5)*1.5,5,5,10),{seed:seed+30,fill:'#FF8A5B',lw:1.8,amp:.3});break;
  case 'star':{stem(12);const p=[];for(let k=0;k<10;k++){const a=-Math.PI/2+k*Math.PI/5+Math.sin(t*2)*.1,r=k%2?3.6:8;p.push([Math.cos(a)*r,hb-19+Math.sin(a)*r]);}
    sk(p,{seed:seed+32,fill:BULB,lw:1.6,amp:.3});break;}
  case 'cap':sk(rr(-W2*.55,hb-12,W2*1.1,16,8),{seed:seed+33,fill:'#E4572E',lw:2});sk(rr(-W2*.1,hb-2,W2*.95,7,3),{seed:seed+34,fill:'#C0431F',lw:1.8,amp:.4});break;
  case 'phones':ctx.strokeStyle=c.dark;ctx.lineWidth=5;ctx.beginPath();ctx.arc(0,hb+14,W2+2,Math.PI*1.12,Math.PI*1.88);ctx.stroke();
    ctx.strokeStyle=INK;ctx.lineWidth=1.6;ctx.stroke();[-1,1].forEach(k=>sk(rr(k*(W2+2)-7,hb+c.h*.18,14,22,5),{seed:seed+35+k,fill:'#3B3140',lw:1.8,amp:.4}));break;
  case 'dish':stem(8);ctx.beginPath();ctx.arc(0,hb-10,9,Math.PI*1.05,Math.PI*1.95);ctx.closePath();ctx.fillStyle=c.dark;ctx.fill();ctx.strokeStyle=INK;ctx.lineWidth=2;ctx.stroke();break;
  case 'bun':sk(ell(0,hb-7,10,9,14),{seed:seed+40,fill:c.dark,lw:2});sk(rr(-7,hb-2,14,4,2),{seed:seed+41,fill:'#E4572E',lw:1.4,amp:.3});break;
  case 'twinbuns':[-1,1].forEach(k=>sk(ell(k*W2*.55,hb-2,8,8,12),{seed:seed+42+k,fill:c.dark,lw:2,amp:.4}));break;
  case 'tuft':ctx.strokeStyle=c.dark;ctx.lineWidth=3.2;ctx.lineCap='round';[[-7,-11,-12],[0,-1,-18],[7,11,-13]].forEach(([a,b,h])=>{ctx.beginPath();ctx.moveTo(a,hb+1);ctx.quadraticCurveTo(a,hb+h*.6,b,hb+h);ctx.stroke();});break;
  case 'flower':stem(12);for(let k=0;k<5;k++){const a=k/5*Math.PI*2+Math.sin(t*1.5)*.2;sk(ell(Math.cos(a)*5.5,hb-18+Math.sin(a)*5.5,4.4,4.4,10),{seed:seed+44+k,fill:'#FFF1F6',lw:1.3,amp:.2});}dot(0,hb-18,3.2,BULB);break;
  case 'beanie':{const R=W2*.66;ctx.beginPath();ctx.arc(0,hb+9,R,Math.PI,2*Math.PI);ctx.closePath();ctx.fillStyle='#3E6FB0';ctx.fill();ctx.strokeStyle=INK;ctx.lineWidth=2;ctx.stroke();
    sk(rr(-R-2,hb+3,2*R+4,8,3),{seed:seed+50,fill:'#2F5A94',lw:1.6,amp:.3});sk(ell(0,hb+9-R-4,6,6,10),{seed:seed+51,fill:'#fff',lw:1.6,amp:.3});break;}
  case 'cloud':stem(12);[[-6,-18,6],[1,-23,7.5],[8,-18,6],[1,-16,6]].forEach(([a,b,r],i)=>sk(ell(a,hb+b,r,r*.85,12),{seed:seed+52+i,fill:'#fff',lw:1.4,amp:.3,stroke:i==3?null:INK}));break;
  case 'crown':sk([[-13,hb+2],[-13,hb-10],[-6,hb-4],[0,hb-14],[6,hb-4],[13,hb-10],[13,hb+2]],{seed:seed+56,fill:BULB,lw:1.8,amp:.3});dot(0,hb-5,2,RED);break;
  case 'wifi':dot(0,hb-5,3,INK);ctx.strokeStyle=INK;ctx.lineWidth=2.4;[8,14].forEach((r,i)=>{ctx.globalAlpha=.5+.5*Math.max(0,Math.sin(t*5-i));ctx.beginPath();ctx.arc(0,hb-3,r,1.25*Math.PI,1.75*Math.PI);ctx.stroke();});ctx.globalAlpha=1;break;
  case 'hardhat':{const R=W2*.72;ctx.beginPath();ctx.arc(0,hb+7,R,Math.PI,2*Math.PI);ctx.closePath();ctx.fillStyle=BULB;ctx.fill();ctx.strokeStyle=INK;ctx.lineWidth=2;ctx.stroke();
    sk(rr(-R-6,hb+3,2*R+12,6,3),{seed:seed+57,fill:'#D9A520',lw:1.6,amp:.3});sk(ln(0,hb+7-R,0,hb+4),{closed:false,seed:seed+58,lw:2,passes:1,amp:.2});break;}
  case 'code':stem(10);sk(rr(-14,hb-26,28,16,4),{seed:seed+59,fill:SCREEN,lw:1.6,amp:.3});txt('</>',0,hb-17.5,{size:10,font:MONO,color:'#9FE0FF'});break;
  case 'gradcap':sk(rr(-9,hb-7,18,8,2),{seed:seed+60,fill:'#2B2B35',lw:1.6,amp:.3});sk([[-19,hb-9],[0,hb-16],[19,hb-9],[0,hb-2]],{seed:seed+61,fill:'#2B2B35',lw:1.6,amp:.3});
    ctx.strokeStyle=BULB;ctx.lineWidth=2;ctx.beginPath();ctx.moveTo(0,hb-9);ctx.lineTo(15,hb-6+Math.sin(t*3));ctx.lineTo(15,hb+5+Math.sin(t*3));ctx.stroke();break;
  case 'gear':{stem(10);const p=[];for(let k=0;k<16;k++){const a=k/16*Math.PI*2+t*1.2,r=k%2?6:9.5;p.push([Math.cos(a)*r,hb-20+Math.sin(a)*r]);}sk(p,{seed:seed+62,fill:'#B7C0C8',lw:1.6,amp:.2});dot(0,hb-20,2.6,INK);break;}
  case 'orbit':stem(10);dot(0,hb-19,5,BULB);ctx.save();ctx.translate(0,hb-19);ctx.rotate(t*1.5);ctx.strokeStyle=INK;ctx.lineWidth=1.8;ctx.beginPath();ctx.ellipse(0,0,13,4.5,0,0,7);ctx.stroke();
    dot(13*Math.cos(t*4),4.5*Math.sin(t*4),2.4,'#9FE0FF');ctx.restore();break;
  case 'heart':stem(10);heartPath(0,hb-19+Math.sin(t*4)*1.2,8);ctx.fillStyle='#E5537A';ctx.fill();ctx.strokeStyle=INK;ctx.lineWidth=1.6;ctx.stroke();break;
  case 'propeller':{ctx.beginPath();ctx.arc(0,hb+3,12,Math.PI,2*Math.PI);ctx.closePath();ctx.fillStyle='#E4572E';ctx.fill();ctx.strokeStyle=INK;ctx.lineWidth=1.8;ctx.stroke();
    sk(ln(0,hb-9,0,hb-15),{closed:false,seed:seed+67,lw:2,passes:1,amp:.2});const sp=Math.abs(Math.cos(t*14));
    ctx.fillStyle=BULB;ctx.beginPath();ctx.ellipse(-7*sp,hb-16,8*sp+1.5,2.6,0,0,7);ctx.fill();ctx.fillStyle='#3F7FD6';ctx.beginPath();ctx.ellipse(7*sp,hb-16,8*sp+1.5,2.6,0,0,7);ctx.fill();dot(0,hb-16,2,INK);break;}
  case 'flag':stem(22);sk(rr(0,hb-22,17,11,1),{seed:seed+63,fill:'#2E9E5B',lw:1.5,amp:.2});ctx.strokeStyle='#fff';ctx.lineWidth=2;ctx.beginPath();ctx.moveTo(4,hb-17);ctx.lineTo(7,hb-14);ctx.lineTo(13,hb-20);ctx.stroke();break;
  }
}
// drawChar(c, x, fy, o): feet at (x, fy). o = {pose, mood, walking, sw, dir, gx, t(real seconds), hands}
// o.hands overrides the pose's hand positions; give them in base coordinates (an 80x70 body; shoulders near y=-46, feet at 0).
function drawChar(c,x,fy,o){
  const {pose='idle',mood='normal',walking=false,sw=0,dir=1,gx=0,t=0}=o;
  const W2=c.w/2,H=c.h,top=-16-H,kx=c.w/80,ky=H/70,seed=c.name.length*37+c.w;
  ctx.save();ctx.translate(x,fy);ctx.scale(dir,1);
  ctx.save();ctx.globalAlpha*=.25;ctx.fillStyle=INK;ctx.beginPath();ctx.ellipse(0,2,W2*.75,4,0,0,7);ctx.fill();ctx.restore();
  const lx=Math.min(13,W2*.35);
  sk(ln(-lx,-18,-lx+sw,-5),{closed:false,seed:seed+1,lw:4,stroke:c.dark,amp:.4,passes:1});
  sk(ln(lx,-18,lx-sw,-5),{closed:false,seed:seed+2,lw:4,stroke:c.dark,amp:.4,passes:1});
  sk(ell(-lx-1+sw,-4,10,5),{seed:seed+3,fill:c.dark,lw:1.8,amp:.4});sk(ell(lx+1-sw,-4,10,5),{seed:seed+4,fill:c.dark,lw:1.8,amp:.4});
  sk(rr(-W2,top,c.w,H,c.r),{seed:seed+5,fill:c.body,lw:2.4});
  ctx.save();ctx.fillStyle='rgba(255,255,255,.18)';ctx.beginPath();ctx.ellipse(-W2*.45,top+H*.8,W2*.25,6,-.5,0,7);ctx.fill();ctx.restore();
  drawHead(c,W2,top,t,seed);
  const scW=c.visor?c.w*.8:Math.min(56,c.w*.7),scH=c.visor?18:Math.min(34,H*.48),scY=top+9,ey=scY+scH*.42;
  sk(rr(-scW/2,scY,scW,scH,c.visor?7:12),{seed:seed+14,fill:SCREEN,lw:2});
  const ex=c.visor?[-scW*.2,scW*.2]:[-11,11];
  ctx.strokeStyle='#fff';ctx.fillStyle='#fff';ctx.lineWidth=3;ctx.lineCap='round';
  if(c.visor){
    const eh=mood==='happy'?2:mood==='alert'?7:mood==='tired'?2:(t%3.7<.12?1:5);
    ex.forEach(e=>{ctx.fillRect(e-7+gx*dir,ey-eh/2+1,14,eh);});
    ctx.strokeStyle=INK;ctx.lineWidth=2.4;ctx.beginPath();
    if(mood==='happy')ctx.arc(0,scY+scH+8,7,.15*Math.PI,.85*Math.PI);else if(mood==='alert')ctx.arc(0,scY+scH+12,4,0,7);else{ctx.moveTo(-6,scY+scH+12);ctx.lineTo(6,scY+scH+12);}ctx.stroke();
  } else {
    const my=scY+scH*.78;
    if(mood==='happy'){ex.forEach(e=>{ctx.beginPath();ctx.arc(e,ey+3,5,Math.PI*1.1,Math.PI*1.9);ctx.stroke();});ctx.beginPath();ctx.arc(0,my-5,6,.15*Math.PI,.85*Math.PI);ctx.stroke();}
    else if(mood==='calm'){ex.forEach(e=>{ctx.beginPath();ctx.arc(e,ey-2,5,.15*Math.PI,.85*Math.PI);ctx.stroke();});ctx.beginPath();ctx.arc(0,my-6,4,.2*Math.PI,.8*Math.PI);ctx.stroke();}
    else if(mood==='tired'){ex.forEach(e=>{ctx.beginPath();ctx.ellipse(e,ey+2,4.5,2.6,0,0,7);ctx.fill();ctx.beginPath();ctx.moveTo(e-7,ey-2);ctx.lineTo(e+7,ey-2);ctx.stroke();});ctx.beginPath();ctx.moveTo(-5,my);ctx.lineTo(5,my-1);ctx.stroke();}
    else if(mood==='alert'){ex.forEach(e=>{dot(e,ey,6.5,'#fff');dot(e+gx*dir,ey,2.6,SCREEN);});ctx.beginPath();ctx.arc(0,my,3.5,0,7);ctx.stroke();}
    else if(mood==='puzzled'){ctx.beginPath();ctx.ellipse(ex[0]+gx*dir,ey,3.6,6,0,0,7);ctx.fill();ctx.beginPath();ctx.ellipse(ex[1]+gx*dir,ey+1,2.6,3,0,0,7);ctx.fill();
      ctx.beginPath();ctx.moveTo(-6,my);ctx.quadraticCurveTo(-2,my-4,1,my);ctx.quadraticCurveTo(4,my+4,7,my);ctx.stroke();}
    else{const bl=(t%3.7)<.12?1:6;ex.forEach(e=>{ctx.beginPath();ctx.ellipse(e+gx*dir,ey,3.6,bl,0,0,7);ctx.fill();});ctx.beginPath();ctx.arc(0,my-6,5,.2*Math.PI,.8*Math.PI);ctx.stroke();}
    if(c.glasses){ctx.strokeStyle='#E9E4FF';ctx.lineWidth=1.6;ex.forEach(e=>{ctx.beginPath();ctx.arc(e,ey,9,0,7);ctx.stroke();});ctx.beginPath();ctx.moveTo(ex[0]+9,ey);ctx.lineTo(ex[1]-9,ey);ctx.stroke();}
    if(c.mustache){ctx.fillStyle='#fff';ctx.beginPath();ctx.moveTo(0,my-9);ctx.quadraticCurveTo(-9,my-13,-13,my-6);ctx.quadraticCurveTo(-6,my-8,0,my-6);ctx.quadraticCurveTo(6,my-8,13,my-6);ctx.quadraticCurveTo(9,my-13,0,my-9);ctx.fill();}
  }
  if(c.tie){const y0=scY+scH+18;sk([[-4,y0],[4,y0],[6,y0+22],[0,y0+28],[-6,y0+22]],{seed:seed+64,fill:'#2E5E9E',lw:1.6,amp:.2});}
  if(c.bowtie){const y0=scY+scH+9;sk([[0,y0],[-11,y0-6],[-11,y0+6]],{seed:seed+65,fill:RED,lw:1.6,amp:.2});sk([[0,y0],[11,y0-6],[11,y0+6]],{seed:seed+66,fill:RED,lw:1.6,amp:.2});dot(0,y0,2.6,RED);}
  if(c.badge){const by=top+H*.62;sk(rr(-18,by,36,16,3),{seed:seed+15,fill:PAPERW,lw:1.4,amp:.4});txt(c.badge,0,by+8.5,{size:9,font:MONO,color:c.dark});}
  if(pose==='type'){sk(rr(-27,-42,54,20,3),{seed:seed+16,fill:'#3A4260',lw:1.8,amp:.4});ctx.fillStyle='#9FB0FF';ctx.fillRect(-20,-37,26,2.5);ctx.fillRect(-20,-31,34,2.5);
    sk(rr(-33,-23,66,7,3),{seed:seed+17,fill:'#2B3246',lw:1.8,amp:.4});}
  const tap=Math.sin(t*30)*3,wave=Math.sin(t*10)*6,sh=-16-H*.43;
  const P=o.hands||{idle:[[-48,-28+Math.sin(t*2)],[48,-28-Math.sin(t*2)]],carry:[[-22,-106],[22,-106]],type:[[-14,-30+tap],[14,-30-tap]],
    mag:[[-48,-28],[58,-64]],throw:[[-46,-62],[32,-112]],point:[[-48,-28],[66,-58]],talk:[[-48,-28],[52,-48+Math.sin(t*6)*5]],
    wave:[[-48,-28],[40,-112+wave]],cheer:[[-44,-108+wave],[44,-108-wave]],celebrate:[[-44,-110+wave],[44,-110-wave]]}[pose]||[[-48,-28],[48,-28]];
  let hands=P.map(([hx,hy])=>[hx*kx,-16+(hy+16)*ky]);
  if(walking&&pose==='idle'&&!o.hands)hands=[[-(W2+6)+sw*.6,-26*ky],[(W2+6)-sw*.6,-26*ky]];
  [[-(W2-4),sh],[W2-4,sh]].forEach((s0,i)=>{const h=hands[i];sk(ln(s0[0],s0[1],h[0],h[1]),{closed:false,seed:seed+20+i,lw:5,stroke:c.dark,amp:.5,passes:1});
    sk(ell(h[0],h[1],5.5,5.5,10),{seed:seed+22+i,fill:c.dark,lw:1.6,amp:.3});});
  if(pose==='mag'){const h=hands[1];sk(ln(h[0],h[1],h[0]+12,h[1]+10),{closed:false,seed:seed+24,lw:4,stroke:'#7A5230',passes:1,amp:.3});
    ctx.fillStyle='rgba(200,230,255,.5)';ctx.beginPath();ctx.arc(h[0]+26,h[1]+20,17,0,7);ctx.fill();sk(ell(h[0]+26,h[1]+20,17,17,20),{seed:seed+25,lw:3.5,stroke:'#7A5230',amp:.5});}
  if(mood==='tired'){const d=(t*25)%16;ctx.fillStyle='#7EC8F0';ctx.beginPath();ctx.moveTo(W2+4,top-2+d);ctx.quadraticCurveTo(W2+10,top+8+d,W2+4,top+12+d);ctx.quadraticCurveTo(W2-2,top+8+d,W2+4,top-2+d);ctx.fill();}
  ctx.restore();
  const bx=x+W2-4,byy=fy+top-44;
  if(mood==='alert'){sk(rr(bx,byy,26,30,8),{seed:seed+26,fill:RED,lw:2});txt('!',bx+13,byy+16,{size:24,color:'#fff'});}
  if(mood==='puzzled'){sk(rr(bx,byy,26,30,8),{seed:seed+27,fill:BLUE,lw:2});txt('?',bx+13,byy+16,{size:22,color:'#fff'});}
  if(pose==='celebrate'||pose==='cheer'){for(let k=0;k<7;k++){const a=k/7*Math.PI*2+t*1.5,r=78+Math.sin(t*5+k)*10;
    ctx.save();ctx.translate(x+Math.cos(a)*r,fy-60+Math.sin(a)*r*.7);ctx.rotate(t*3+k);ctx.fillStyle=[BULB,RING,TEAL,BLUE][k%4];ctx.fillRect(-4,-2,8,4);ctx.restore();}}
  // world-space hand positions, so props can be attached to hands
  return {headY:fy+top-24,hands:hands.map(([hx,hy])=>[x+hx*dir,fy+hy])};
}
```

### Speech bubble (add a name chip when the cast is larger than 1)
```js
function wrap(str,maxW){ctx.font='25px '+HAND;const words=str.split(' '),lines=[];let cur='';
  words.forEach(w=>{const tr=cur?cur+' '+w:w;if(ctx.measureText(tr).width>maxW&&cur){lines.push(cur);cur=w;}else cur=tr;});if(cur)lines.push(cur);return lines;}
function bubble(b,hx,hy){
  if(!b.say)return;const el=reduce?1e3:(performance.now()-bubbleT0)/1000;
  const pop=reduce?1:back(clamp(el/.25,0,1));const shown=Math.floor(el*40);
  const lines=wrap(b.say,400);ctx.font='25px '+HAND;const tw=Math.max(...lines.map(l=>ctx.measureText(l).width));
  const w=tw+36,h=lines.length*31+26;let bx,by,tail;
  if(b.side){bx=clamp(hx+70,16,W-16-w);by=clamp(hy-10,100,H-16-h);tail=[bx,by+h/2];}
  else{bx=clamp(hx-w/2,16,W-16-w);by=Math.max(100,hy-26-h);tail=[clamp(hx,bx+24,bx+w-24),by+h];}
  ctx.save();ctx.translate(bx+w/2,by+h/2);ctx.scale(pop,pop);ctx.translate(-(bx+w/2),-(by+h/2));
  ctx.fillStyle='rgba(43,38,33,.12)';ctx.beginPath();ctx.roundRect?ctx.roundRect(bx+4,by+5,w,h,16):ctx.rect(bx+4,by+5,w,h);ctx.fill();
  const tp=b.side?[[tail[0]+2,tail[1]-10],[hx+26,hy+18],[tail[0]+2,tail[1]+10]]:[[tail[0]-12,tail[1]-2],[clamp(hx,tail[0]-40,tail[0]+40),Math.min(hy-4,tail[1]+22)],[tail[0]+12,tail[1]-2]];
  sk(rr(bx,by,w,h,16),{seed:960,fill:PAPERW,lw:2.2,amp:.7});
  sk(tp,{seed:961,fill:PAPERW,lw:2.2,amp:.4,closed:false});
  ctx.fillStyle=PAPERW;ctx.fillRect(Math.min(tp[0][0],tp[2][0])+2,Math.min(tp[0][1],tp[2][1])-3,Math.abs(tp[2][0]-tp[0][0])-2||4,Math.abs(tp[2][1]-tp[0][1])+5||5);
  let left=shown;lines.forEach((l,i)=>{const part=l.slice(0,Math.max(0,left));left-=l.length+1;txt(part,bx+18,by+13+15+i*31,{size:25,align:'left'});});
  ctx.restore();}
```

### Player state machine (lead → play → hold/wait)
```js
/* ---------- player: beats, lead → play → hold/wait ---------- */
BEATS.forEach(b=>b.lead=b.say?clamp(b.say.length/34,1.0,1.9):.15);
let bi=0,phase='lead',pt=0,s=0,mode='click',paused=reduce,speedMul=1.5;
let camFrom=BEATS[0].cam.slice(),camT0=0,bubbleT0=0,chapT0=0,lastChap=-1;
const RATE=.85,HOLD=1.2;
function enter(i,instant){const from=camNow();bi=i;s=BEATS[i].s;phase='lead';pt=0;camFrom=instant?BEATS[i].cam.slice():from;camT0=performance.now();bubbleT0=performance.now();}
function update(dt){if(paused||phase==='end'||phase==='wait')return;pt+=dt*speedMul;const b=BEATS[bi];
  if(phase==='lead'){if(pt>=b.lead){phase='play';pt=0;}}
  else if(phase==='play'){const a=s;s=Math.min(b.e,s+dt*RATE*speedMul);sfx(a,s);
    if(s>=b.e){pt=0;phase=bi===BEATS.length-1?'end':(mode==='auto'?'hold':'wait');}}
  else if(phase==='hold'){if(pt>=(b.hold!=null?b.hold:(b.say?HOLD:.1)))enter(bi+1);}}
function next(){if(phase==='end'){enter(0);return;}if(bi<BEATS.length-1)enter(bi+1);}
function prev(){const back_=(phase==='lead'&&pt<.4)||(phase!=='lead'&&s-BEATS[bi].s<.2&&phase==='play');enter(Math.max(0,back_?bi-1:bi));}
function advance(){ // click-through: finish the current beat, or move to the next one
  if(phase==='wait'||phase==='end'){next();return;}
  if(phase==='lead'){phase='play';pt=0;return;}
  if(phase==='play'){s=BEATS[bi].e;phase=bi===BEATS.length-1?'end':'wait';pt=0;}}
```

The reference film this style came from: "Dennice's monthly report", a before-and-after story
with a 1-character cast, 37 beats, a moral ending and the THE JOB / THE PROBLEM / WATCH FOR intro.
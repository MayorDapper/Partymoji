---
name: emoji-party
description: Make a beat-synced emoji-world lyric music video in Blender Grease Pencil from any song in any genre: emoji characters with swag, a world and palette per scene, animated lyrics, full render. Two modes: 2D Grease Pencil, or a 3D toy town where one emoji lives a day in the life.
---

# Emoji Party (beat-synced emoji-world lyric video, Blender Grease Pencil)

The user gives a song in any genre (audio file) and its lyrics, or no lyrics for an instrumental. You deliver a full-length 1080p video: one recognisable emoji face per section, each in its own world and colour scheme, doing something relatable, dancing on the beat, with the lyrics popping on screen and shaking with the bass. Proven on "Mr Me" (150 BPM, 4:12, 15 scenes, ~840 objects, rendered in about 10 minutes).

## 0. Creative rules (learned from the user)
- **Leads are emoji faces that carry an emotion**, a different one per scene (anxious, grin, bandaged, huffing, crying-laughing, shushing, mind-blown, nerd, angry, hot, star-eyes, zipper-mouth, money-mouth, halo, party). Objects (candle, compass, diamond) are props only, never the lead.
- **Swag on every lead**: every lead wears something. For rap that is a chain with pendant, grills, face tattoos, earrings and a hat (cap, durag, bucket, beanie, crown); for other genres see section 0b.
- **Relatable action per scene**: driving, riding a bike, sitting an exam, filming on phones, fishing, DJing, lifting weights, selfie, throwing money.
- **One palette per scene**, chosen from the lyric's mood. Lyric styling stays constant across scenes so the video reads as one piece.
- One recurring lead (sunglasses emoji) owns the chorus and returns in the final chorus with the whole cast behind.
- Draw original emoji-style characters. Do not copy a vendor's emoji set.
- Show a single test scene (or 3 palette options) first and get a yes before building everything. Send preview stills with SendUserFile.

## 0b. Adapt to the music (any genre)
Read the track before choosing anything. Genre is an input, not an assumption.
- **Tempo search**: search 60-180 BPM, then resolve half/double time by where the snare or clap lands (beats 2 and 4). Trap and hip-hop often read as 140-150 but feel half-time; drum and bass reads 170+ but can be animated at half speed. Store both `FPB` (felt beat) and the grid beat.
- **Meter**: default 4 beats per bar; make beats-per-bar a parameter (3 for waltz, 6 for 6/8) and use it in `FR(bar)` and the bar-length expressions.
- **Tempo drift** (live bands, older recordings): if the comb score is weak or drifts, use a tracked beat list and key a `beat_phase` property per beat instead of the fixed-period expressions.
- **Frame rate**: pick the fps that gives whole frames per felt beat; otherwise 25 and accept fractional beats.
- **Movement by feel**:
  - Four-on-the-floor (house, techno, disco, amapiano): even bounce on every beat, steady sway.
  - Half-time (trap, hip-hop, drill, dubstep): big hit on beat 1, lean back on 3, small hat-speed details.
  - Off-beat (reggae, dancehall, afrobeats, ska): bounce on OFFP, hips sway on a 2-beat period.
  - Fast (drum and bass, punk, hardcore): half-speed body, full-speed hands and props.
  - Slow (ballad, R&B, soul, lo-fi): sway only, long periods, little squash, slow lyric fades instead of pops.
  - Swing/triplet (jazz, blues, shuffle): delay the off-beat to two thirds of the beat.
- **Cut rate from energy**: 8-16 bars per scene when calm, 4 bars in choruses, 1-2 bars in drops and peaks.
- **Look from genre and lyrics**: choose palettes, worlds, hats and accessories that belong to the music (chains and grills for rap, headphones and visors for dance music, cowboy hat for country, flowers for folk, studs for rock). Always keep the emoji face as the lead, and keep the swag rule: every lead wears something.
- **Lyric style**: hard pops and shake for bass-heavy tracks; smooth slide and fade for soft tracks; bigger type for chants and hooks. Language follows the lyrics as given (never "correct" dialect or pidgin).
- **Instrumentals**: no lyric layer; use the title, section names or short mood words on drops, and let props carry the story.
- Confirm the read with the user in one line (BPM, feel, section map) before building.

## 1. Analyse the song (workspace)
- `pip install --break-system-packages -q librosa`, then a comb search on low-band (<130 Hz) flux over a BPM range gives exact BPM and first-beat offset; print per-bar kick (K) and loudness (R) to see intro, groove, drops and dropouts. Do not trust the BPM written in the prompt (the Suno prompt said 140; the track was 150).
- Use 25 fps when it gives whole frames per beat (120, 125, 150 BPM do).
- **Lyric timings**: speech models cannot be downloaded in the workspace, and uploads to Descript storage are blocked from the shell. Call `Descript file_upload_ui`, ask the user to drop the audio in, `wait_for_job`, then `export_transcript` as srt. Expect line-level timing for clear rap/spoken parts and block-level (or missing) timing for dense or sung parts; match the user's real lyrics to those timestamps by hand and estimate the rest evenly across bars. Tell the user which sections are estimated.
- Write a scene table: section, start bar, emoji, world, action, palette. Scene boundaries sit on bars (half bars are fine). Add instrumental scenes (title card, drop) where the vocal is silent.

## 2. Blender setup
- Check the connection with `blender__get_scene_info`. Ask before clearing a saved file. Save early (ask for a folder; Blender's Python can read the user's folders directly, so point the sound strip at the audio in place rather than copying it).
- Flat 2D: orthographic camera at y=-20 looking along +Y, `ortho_scale` 16 (view is 16 x 9 units). Everything is drawn in the XZ plane; **depth is the object's Y** (larger Y is further back).
- Each scene lives at **X = index * 40** in its own collection `S01_...`; the camera jumps between them with LINEAR keys on location X (plus a small drift inside each scene) and a timeline marker per cut.
- Colour: `view_transform='Standard'`, look None. Store reusable code in `bpy.data.texts` and `exec` it at the top of each call.

## 3. Grease Pencil rules that cost time to find (Blender 5.x)
- Data is `bpy.data.grease_pencils`; `layer.frames.new(1).drawing`; `drawing.add_strokes([n])`; set `point.position=(x,0,z)` and `point.radius`; call `drawing.tag_positions_changed()`.
- **Fills do not render unless each filled stroke has a non-zero `fill_id`**. Set `stroke.fill_id = unique int` for fills and 0 for lines, and `stroke.hide_stroke = True` on fill-only strokes. Give all loops of one text glyph run the **same** fill_id so letter holes cut out.
- Materials: `bpy.data.materials.create_gpencil_data(m)`; `show_fill` / `show_stroke`; radial gradient via `fill_style='GRADIENT'`. Material colours are **linear**: convert hex with `(v/255)**2.2` or everything looks washed out.
- `layer.use_lights = False` on every layer.
- **Font/curve text objects are hidden behind Grease Pencil in renders.** Convert text to GP: make a temporary FONT curve, `evaluated_get(dg).to_curve(dg)`, sample bezier splines with `mathutils.geometry.interpolate_bezier`, draw the loops as GP strokes (outline strokes first, then fills). Do the temp work in a separate scratch scene so the big scene's depsgraph is not re-evaluated per word (486 words took 3 s this way).
- Translucent fills over other translucent fills look muddy; prefer opaque darker colours for rays and glows.
- Drivers: auto-run Python is usually off, so every driver must be a **simple expression** (`sin abs exp fmod min max pi frame`); assert `driver.is_simple_expression`.

## 4. Beat expressions (F0 = first-beat frame, FPB = frames per beat)
```
PULSE = exp(-fmod(frame-F0+BIG, FPB)/(FPB*0.25))        # 1 on each beat, decays
OFFP  = same, shifted half a beat                       # off-beat
HALF  = exp(-fmod(frame-F0+BIG, 2*FPB)/(FPB*0.45))      # every 2 beats
OFFB  = HALF shifted one beat
SWAY(per,ph) = sin(pi*(frame-F0)/(FPB*per)+ph)
FR(bar) = F0 + round(4*FPB*bar)
```
`dance(obj)`: location Z += bounce*PULSE, scale X *(1+squash*PULSE), scale Z *(1-squash*PULSE), rotation Y += sway*SWAY. Alternate PULSE and OFFP between neighbours. Hands shrug on HALF/OFFB. Camera `ortho_scale = 16*(1 - zoom*PULSE)` with a keyed `zoom` custom property (bigger in drops and choruses).

## 5. Helper library to build once
- Shapes: `ell`, `rect`, `rrect`, `star`, `drop`; `F` (fill), `S` (line), `P` (fill + ink outline).
- `emo(C,name,x,z,r,mood)`: shadow, yellow gradient disc, gloss, then features per mood. Store `o["r"]=r` so accessories scale.
- Swag: `chain(o,pendant,ice)`, `grills(o)`, `tattoo(o,kind,side)` (tear, cross, star, lines, text), `earring(o,side)`, `hat(o,kind,col,col2,tilt)` as a child object, `headphones(o)`, `hand(...)` with rings and optional watch.
- World: `sky(C,X,top,mid,bot)` as 16 colour bands, `ground`, `city` skyline with lit windows, `wheel` with a rotation driver, `phone`.
- `preview(shots)` renders stills at 33% to /tmp and `montage(paths)` tiles them with numpy into one image; view it with `blender__look(image=path)`. Check every scene this way before rendering.

## 6. Lyrics layer
- Data: list of `(kind, t0, t1, text)` with kinds `L` line, `A` ad-lib, `H` hook word, `B` big shout. Store it as JSON in a text block.
- `L`: split into words, wrap to 14.4 units wide at size 0.78 with a 0.44 gap, top of frame. Each word is its own GP object appearing at `start + i*step` (step from line duration, 2 to 8 frames) and hidden at the next line's start or the scene cut.
- Per-word drivers: `scale = vis * pop * (1+0.10*PULSE)` where `vis` is a 2-frame in/out ramp and `pop` is a 6-frame overshoot; `location X/Z += shake*sin(frame*k+ph)*PULSE` for the bass vibrate.
- `A`: pink, tilted, alternating left and right. `H`: yellow, bottom, drops in from above. `B`: gold, size 1.5, top.
- Put each scene's lyric objects in `LYR_Sxx` collections so render chunks can hide the rest.

## 7. Render (fast path)
- A full-scene render is ~1.3 s/frame because every GP object is evaluated. Render **one chunk per scene in parallel** instead: a background Blender per scene that sets `hide_render` on every collection except `Sxx_`, `LYR_Sxx` and `CAMERA`, removes the sound strip, and writes H264 with no audio. Run 5 at a time from a detached runner process (`start_new_session=True`) that logs to `render_log.txt`.
- Then assemble in a factory-empty scene with movie strips at each scene's start frame plus the sound strip, H264 + AAC. **Set that scene's view transform to Standard** or the final video comes out dark and desaturated (factory default is AgX).
- Do not delete chunks until the final file is verified.
- Verify: load the mp4 as a movieclip (frame count, fps, size) and as a sound (rate, channels); pull ~16 frames through a temporary sequencer scene with Standard view transform into a contact sheet and look at it. The `bgl` add-on traceback in background logs is harmless.
- Don't poll in a tight loop; sleep ~5 minutes between checks and tell the user the expected time.

## 8. Deliver
- Say where the mp4 and .blend are (plain words plus a computer:// link), and send the contact sheet.
- State what was not checked (you saw stills, not playback) and which lyric sections are estimated.
- Offer fixes by scene or by lyric line; timeline markers are the cuts and the lyric JSON holds the timings.

## 9. V2: 3D toy-world mode ("a day in the life")
Use when the user asks for 3D instead of Grease Pencil. Proven on "Mr Me" in Blender 5.3 (about 2,800 objects, 13 minutes to render). Everything in sections 0, 0b, 1 and 4 still applies; this replaces sections 2, 3, 5, 6 and 7.

**Look.** A bright toy town: chunky bevelled houses, ball-top trees, puffy clouds, round hills, toy cars, glossy plastic materials (Principled, roughness ~0.22, a little emission so colours stay saturated). Original shapes only; take the feel of toy and platform-game worlds without copying any brand's bricks, characters or logos.

**Story.** One hero emoji travels through ONE continuous town from morning to night. Each song section is a stop along a road where the hero meets a different emoji (the lyric decides who and what they do). Earlier characters return as a waving row in the chorus and as the full cast in the final chorus.

**Layout.**
- Stops sit along +X at a fixed spacing (24 units); stop i is at `X = i*24`. Road runs along X at y=-1.5, landmark building behind each stop at y~6.6, a signpost naming the place, background rows of houses/trees (y 8-12), hills (y 30-40) and clouds along the whole length.
- `Hero` is an empty keyed on location X: it arrives at each stop on the section's first frame, holds, then travels during the last ~60 frames (BEZIER keys). In a cruise section it travels the whole time.
- `Follow` is an empty whose X is driven from the hero (`hx+1.6`, TRANSFORMS variable). Camera pivot, fill/back lights, sun and moon balls, and ALL lyric words are parented to `Follow`, so they stay framed while the town scrolls past.
- Camera: perspective 32 mm, about 13 units in front of the pivot, slight sway on the pivot, lens pulsing 3% on the beat.

**Characters.** `emoji3d(mood)`: a UV sphere plus features placed on the front of the sphere by angle (`on(ax,az)`), built from small spheres, boxes and NURBS-curve tubes. Moods used: cool, joy, grin, anxious, bandage, huff, hush, scream, explode, nerd, angry, hot, star, zip, money, halo, party. Swag as child objects: tilted cap, beanie, bucket hat, crown, durag (cut spheres), curve-tube chain with a disc pendant and extruded letter, earrings, floating hands with ring and watch. Parent everything to one root empty and drive that root: bounce on PULSE, squash, sway, twist; face the NPC toward the hero with a base Z rotation.

**Interactions (keep them simple and readable).** Hero and NPC face each other and dance on opposite beats; a hand from each reaches toward the other on OFFB (fist bump); props react (alarm clock shakes, seedling grows on the beat, clock hand spins, barbell lifts on HALF, diamond spins, package hops, rocket launches on a keyed frame with `0.006*max(0,frame-LF)^2`). For riding, key the hero root up and scale the vehicle in and out with a driver over that section's frames.

**Time of day.** World background is a 3-stop vertical gradient; keyframe the ramp colours, the sun lamp's energy and colour, and the sun/moon ball heights at section starts (predawn, sunrise, day, golden hour, dusk, night, dawn).

**Lyrics.** Extruded FONT objects work in 3D (no need to convert to strokes): white emissive face plus a dark offset copy behind as the outline. Same per-word pop and shake drivers as section 6. Measure word widths once in a scratch scene. Lines at the top of frame (z~5.1, wrap at ~11.6 units, size 0.72), ad-libs pink at the sides, hook word yellow low in the foreground. Put each section's words in `LYR_Sxx` collections. Lyric timings can be loaded from an earlier .blend's text block with `bpy.data.libraries.load`.

**Render.** A frame costs ~2.3 s with the whole town loaded, regardless of resolution, because every driver is evaluated. In each per-section background process, DELETE (`bpy.data.batch_remove`) every object whose top-level parent is more than ~48 units outside that section's hero X range, plus all other sections' lyric collections; keep anything under `Follow` or `Hero` and the ground/road. That brings it to ~0.7 s/frame; run 6 processes at once, longest sections first, then assemble with the audio in a factory scene set to the Standard view transform. Turn EEVEE raytracing off; 12 samples is enough.

**Checks.** Render one contact sheet of every mood on a plain background before building the town, then one contact sheet with a frame per section plus a travel frame. Watch for: the hero hidden behind a prop or vehicle, titles hidden behind the hero, lyric words overlapping while popping.

# Partymoji: the Emoji Party Skill

A Claude skill that turns any song into a beat-synced emoji-world lyric video in Blender (Grease Pencil).

Give Claude an audio file and the lyrics. It reads the tempo and feel of the track, casts a different emoji face for each section, builds a world and colour scheme around each one, animates everything to the beat, puts the lyrics on screen and renders the full video.

## Two looks
- **2D**: flat emoji worlds drawn in Grease Pencil, one world per song section.
- **3D toy town**: one emoji hero travels through a bright toy town from morning to night, meeting a different emoji at every stop.

## What you need
- Blender 5.x with the Blender MCP add-on connected to Claude
- A Descript connection (used for lyric timings)

## Install
Add the `emoji-party` folder as a skill in Claude. The skill is the single file `emoji-party/SKILL.md`.

## Use
> Make an Emoji Party video for this song. Here are the lyrics.

First made for "Mr Me" by Dapper Videos.

## Licence
Apache 2.0. See `LICENSE`.

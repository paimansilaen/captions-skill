# captions-skill

Word-synced captions for vertical short videos (TikTok / Shorts / Reels), burned
into the film with ffmpeg.

Give it a voice-over and a narration script, and it produces captions that appear
word by word, exactly when each word is spoken: 2–3 words per caption, white
uppercase heavy sans at 71.5% frame height, with a hard drop shadow, a 5-frame
pop-in, and key words (written in ALL CAPS in the script) highlighted in yellow.

## What's in here

- `SKILL.md` — the full skill: look, safe-zone rules, steps, and all knobs.
- `scripts/captions.py` — builds the caption sprites and burns them into the film
  (`--burn`), or writes a green-screen clip / ProRes 4444 alpha clip.
- `scripts/words_from_voice.py` — makes `words.json` (word timings) from a
  voice-over file with faster-whisper, aligned to the narration script's spelling.

## Quick use

```bash
# 1. Word timings (default language id; faster-whisper model "small")
python3 scripts/words_from_voice.py voice/narration.wav SCRIPT.txt voice/words.json id

# 2. Burn captions into a 1080x1920 film (10 s test first)
python3 scripts/captions.py voice/words.json out/captions --start 0 --dur 10 \
  --w 1080 --h 1920 --burn out/final/Title.mp4

# 3. Full render: drop --start / --dur
python3 scripts/captions.py voice/words.json out/captions \
  --w 1080 --h 1920 --burn out/final/Title.mp4
```

Outputs: `captions_burned.mp4` (the deliverable), `captions_green.mp4`,
`captions.json`, and with `--alpha`, `captions_alpha.mov`.

## Safe zone

Every caption is checked against the YouTube Shorts safe zone (right 16% of the
width, bottom 25% of the height, top 8%). The script prints
`WARNING outside the Shorts safe zone: <text>` for any caption that crosses it —
never deliver with a WARNING: lower `--size` or raise `--y` a little.

## Requirements

- Python 3, ffmpeg, PIL.
- faster-whisper for word timings (`pip install faster-whisper`; CPU, model
  `small` downloads ~500 MB on first run). Without it, write `words.json` by
  hand from TTS/subtitle timings.

## Attribution

Adapted from the `captions` skill in
[fahniazz/claude-shorts-skills](https://github.com/fahniazz/claude-shorts-skills)
(MIT License, Copyright (c) 2026 Fahni). Changes from the original: burned-in
captions (`--burn`) are the default instead of a green-screen clip for After
Effects; `--key-color` is implemented (upstream documented it but the script
ignored it); `words_from_voice.py` accepts plain-text narration files and fails
with a clear install hint when faster-whisper is missing.

## License

MIT — see `LICENSE`.

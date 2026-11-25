# Word-synced captions for vertical shorts

<!-- Adapted from fahniazz/claude-shorts-skills (MIT License, Copyright (c) 2026 Fahni),
     skill `captions` + scripts `captions.py` / `words_from_voice.py` from
     skills/cartoon-3d-short/templates/scripts/. Changes for this user are listed
     under "What changed from the original" below. -->

## When to use

- Use when the user asks for captions, subtitles, "teks ikut suara", or captioned
  text on a short / vertical video.
- Default delivery on this VM is captions BURNED into the film with ffmpeg
  (`--burn`), because the user edits and posts finished vertical videos, not
  After Effects projects. The green-screen clip and the ProRes alpha clip are
  optional variants, not the default.

## Environment on this VM

- CPU-only Linux VM, no GPU. ffmpeg 8.1.2 at /usr/bin, PIL installed.
- Voice-over TTS: `/opt/hatch/bin/tts speak --text ... --language id --voice <id> --output <file.mp3>`.
  Female narration voice is Aria (`avocado_v2:MAI_01`); male default is Helio (`avocado_v2:MAI_03`).
- Downloads: yt-dlp at `~/workspace/tools/yt-dlp`.
- faster-whisper is NOT installed yet. Install it before making word timings:
  `pip install faster-whisper` (CPU, model `small` downloads ~500 MB on first run).
  On this VM it lives in a venv: run words_from_voice.py with
  `~/workspace/custom-skills/captions/venv/bin/python`.
  If it cannot be installed, fall back to manual word timings (step 1 below).
- No ElevenLabs, no After Effects, no Keylight on this VM. Never promise them.
- Scripts live in this skill dir: `scripts/captions.py`, `scripts/words_from_voice.py`.
  Run them from the project dir, e.g. `python3 ~/workspace/custom-skills/captions/scripts/captions.py ...`.

## Look

- Position: centred horizontally; the middle of the capitals at **71.5 % of the frame height**.
  The middle of the capitals is the anchor, so a comma or a Q never moves a caption.
- White, UPPERCASE, heavy sans, **3.2 % of frame height** (61 px at 1920 high, 82 px at 2560).
  Black outline; **hard** black drop shadow down-right, solid on purpose.
- Punctuation is kept as in the script (MONA LISA. / PARIS, / 1911.).
  `--strip-punct` removes trailing . , ; :
- **2-3 words max** per caption. A caption never crosses a sentence/comma or a pause in the voice,
  never ends on a glue word (the, of, with, yang, di, dengan ...), never splits a name
  (Mona | Lisa), avoids lone single words.
- Each caption appears on the frame its first word is spoken (a 5-frame pop 0.85 -> 1.06 -> 1)
  and holds until the next one. After a pause in the voice it leaves 0.25 s after its last word.
- KEY WORDS: a word the narration script writes in ALL CAPS is rendered in yellow
  (`--key-color FFD21F` by default), the rest white. Write a word in CAPS in the script
  to highlight it; do not edit the colour per word by hand.

## YouTube Shorts safe zone (must hold for every caption, at the pop's 1.06 peak)

- Action buttons: right 16 % of the width.
- Title/channel/music: bottom 25 % of the height.
- Top bar: top 8 % of the height.
- The script measures every caption's ink, prints `WARNING outside the Shorts safe zone: <text>`
  for any that cross a zone, and prints the reach (should be about "83.8 % of width,
  69.8-74.0 % of height"). Chunks are split so they fit `--max-width 0.68`.
  A single long word is shrunk to fit. Never deliver with a WARNING: lower `--size`,
  or raise `--y` a little.

## Steps

1. Word timings: `<project>/voice/words.json`, a list of `{"w": word, "s": start_s, "e": end_s}`.
   Make it with faster-whisper aligned to the script's spelling:
   `python3 scripts/words_from_voice.py <voice.wav> <SCRIPT.md|SCRIPT.txt> <words.json> [en|id]`
   (default language `id`). The script file may be the `| # | Beat | Narasi | Gambar |` table
   or plain narration text; plain text is read whole, beat 0. The times must match the
   film's audio (the final's voice starts at 0 s). If faster-whisper is unavailable,
   write words.json by hand from the TTS timings or the subtitle timestamps instead;
   never guess timings from the script length alone.
2. Test span first (the user wants to SEE a short test before a full render):
   ```
   python3 scripts/captions.py <project>/voice/words.json out/<slug>/captions --start 0 --dur 10 \
     --w 1080 --h 1920 --burn "out/<slug>/final/<Title>.mp4"
   ```
   It prints every caption with its times and the safe-zone check, and writes:
   - `captions_burned.mp4` (film + captions, audio copied — the deliverable)
   - `captions_green.mp4` (pure #00FF00 clip, same size/fps, silent)
   - `captions.json` (each caption's text and in/out seconds)
   - With `--preview <film.mp4>` instead of `--burn`: `captions_preview.mp4`
     (film + keyed captions + sound, for checking only).
3. Check:
   - Pull frames of the widest captions from the burned test with the Shorts zones drawn in red
     (ffmpeg `drawbox`: x=iw*0.84 from y=ih*0.45, y=ih*0.75 to the bottom, the top ih*0.08),
     `hstack` them into one sheet, and look.
   - Send the user the burned test clip and the printed caption list.
4. After the user's OK, render full length: drop `--start`/`--dur` so the clip runs to the
   last word + 1 s, and check the burned file's duration against the film with `ffprobe`.

## Knobs (scripts/captions.py)

- `--w 1080 --h 1920`: frame size. The script defaults to 1440x2560; pass 1080x1920
  for this user's vertical Shorts. `--fps 30` by default.
- `--y 0.715`: caption centre (capitals middle) as a fraction of the frame height.
- `--size 0.032`: font size as a fraction of the frame height.
- `--max-width 0.68`: widest caption as a fraction of the frame width.
- `--font <ttf>`: default is the first heavy sans found on the system
  (DejaVuSans-Bold on this VM, `/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf`;
  NotoSans-Bold is the alternative).
- `--max-words 3`
- `--gap 0.30`: a voice pause this long (s) starts a new chunk. If a breath of just over
  0.30 s splits a phrase, raise `--gap` to 0.35.
- `--hold 0.25`: seconds a chunk stays after its last word when the voice pauses.
- `--lead 0`: frames the caption appears before its word.
- `--no-pop`: static cut instead of the 5-frame pop.
- `--lower`: keep the script's case instead of uppercasing.
- `--strip-punct`: drop trailing . , ; :
- `--key-color FFD21F`: hex colour for KEY WORDS (no leading #).
- `--burn <film.mp4>`: burn the captions into the film (default delivery here).
  The film must match `--w`/`--h`/`--fps` or the script exits with the mismatch.
- `--alpha`: also write `captions_alpha.mov` (ProRes 4444, real transparency, no keying needed).
- `--preview <film.mp4>`: keyed preview clip for checking only.
- `--keep "the snail hole"` (repeatable): never split this phrase across captions; a 4-word
  caption is allowed only for it and must still fit `--max-width`. Read the full caption
  list after every full render and fix names or phrases that got split.

## Variants (only when the user asks)

- **Green-screen clip:** `captions_green.mp4` is written on every run. Key it in an editor
  with Keylight, Screen Colour = pure green. It is yuv444 crf 8, so edges stay clean.
- **Alpha .mov:** add `--alpha`. It needs no key at all (drop it on top). With `--key-color`
  prefer it: a green despill pushes yellow toward orange.
- **captions.json:** lists each caption's text and in/out seconds, for rebuilding them as text layers.

## What changed from the original

- Output path flipped: burned-in captions (`--burn`, new flag) are the DEFAULT delivery;
  green-screen/alpha are optional variants. The original delivered only the green clip
  (plus optional alpha) for After Effects and never burned captions in.
- `--key-color` implemented in `scripts/captions.py` (word-by-word sprite drawing;
  a word is a key word when the script writes it in ALL CAPS). The upstream SKILL.md
  documents `--key-color FFD21F` but the upstream script has no such flag, so upstream
  runs silently ignored it.
- `scripts/words_from_voice.py`: accepts a plain-text narration file as well as the
  `| # | Beat | Narasi | Gambar |` table, and exits with an install hint instead of an
  ImportError when faster-whisper is missing.
- Frame size: the upstream default is 1440x2560; this skill documents 1080x1920 as the
  size to pass for this user's vertical Shorts.
- Dropped as not applicable here: the upstream model note (run captions on Haiku 5.5),
  the After Effects / Keylight workflow as the default, and ElevenLabs voice tooling.
- Verified on this VM (2026-11-26): `python3 -m py_compile` passes on both scripts;
  a 3 s 1080x1920 test film burned 3 chunks with no safe-zone WARNING, ink 80.9 % of
  width and 69.9-74.0 % of height, and the CAPS key word rendered yellow (checked
  pixel-level on the green clip).

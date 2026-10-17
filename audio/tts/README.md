# audio/tts: recorded syllables

One MP3 per Vietnamese syllable, spoken by Google Cloud Text-to-Speech (voice `vi-VN-Standard-A`) in `vi-VN-Standard-A/`.
`decks/audiomanager.js` of `qtng/chunom` plays such a file first (`speak(text, 'auto')`), then Google TTS, and the voice of the browser only as the last resort.
It does not look anything up: it works out the file name from the text and tries to load it, a missing file (404) means "ask Google".

At the moment the folder holds the 757 one-syllable entries of `dict.csv` (the frequent words). Compounds and sentences are not recorded; they go to Google TTS.

## File names

Telex letters followed by the tone digit: `â`=aa `ê`=ee `ô`=oo `ă`=aw `ơ`=ow `ư`=uw `đ`=dd; tone 0 ngang, 1 sắc, 2 huyền, 3 hỏi, 4 ngã, 5 nặng.
`tân` = `taan0.mp3`, `việt` = `vieet5.mp3`, `đi` = `ddi0.mp3`. The rule is the same in `syllableFileName()` (JS) and `file_name()` in `tools/make_tts.py` (Python) of `qtng/chunom`; change both or none.

## Adding or completing files

1. Clone `qtng/chunom` and `qtng/chunom-assets` next to each other.
2. A Google API key with the Cloud Text-to-Speech API enabled, in the environment (not in a file):
   `export GOOGLE_TTS_KEY=...` (an unrestricted key; a key restricted to the websites does not work from a script).
3. In `chunom`: `python3 tools/make_tts.py --list` shows the syllables and file names it would make (no key needed);
   `python3 tools/make_tts.py --out ../chunom-assets/audio/tts/vi-VN-Standard-A` makes the missing ones. Existing and empty files are checked, so it can be run again after `dict.csv` grew or after an interruption.
4. Commit the new files in `chunom-assets` (one commit: Git keeps every version forever, so do not regenerate files that already exist).

Google answers `429` when asked too fast. The script waits and tries again and keeps 0.6 s between requests, about 100 files a minute.

### All syllables, not only those of dict.csv

`js/base_chars.json` of `chunom` has about 7150 syllables. `make_tts.py` reads only `dict.csv` (function `syllables()`). To record all, let `syllables()` also read the keys of `base_chars.json` (`json.load(...)['dict']` is `key:glyphs|key:glyphs|…`; use the keys that start with a letter, lower-case, and put each through `file_name()`). That is about 6400 more files, 35 to 40 MB, 1 to 1.5 hours.
It was tried and stopped on purpose: Google TTS is free for 4 million characters a month (standard voices), a syllable costs a few characters, and a recording only saves the round trip. Decide by speed, not by cost.

## Keep in mind

- The key in `decks/audiomanager.js` is public. Restrict it in the Google Cloud console to the sites (`qatt.org`, `chunom.org`) and to the Text-to-Speech API, and set a budget alert.
- A second voice goes into a folder of its own (`vi-VN-Neural2-A/`, …); `TTS_DIR` in `audiomanager.js` points at the folder in use.

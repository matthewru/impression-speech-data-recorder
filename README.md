# Impression voice recorder

A browser app for recording a text-to-speech training dataset, one sentence at a time. Built for the Impression project, where each member fine-tunes a [Piper](https://github.com/OHF-Voice/piper1-gpl) voice on their own recordings.

Everything runs in the browser. Recordings are saved locally (IndexedDB) and never leave the computer until the person clicks **Export**.

## For members

1. Open the recorder link in **Chrome on a laptop**.
2. Enter your first name and run the **mic check**.
3. Press **Space** to record each sentence, Space again to stop. **150 sentences is the minimum; all 315 is the goal.**
4. Click **Export my dataset** to download `yourname_dataset.zip`.

The zip contains a training-ready dataset in LJSpeech format:

```
dataset/
├── wavs/               one 22,050 Hz mono 16-bit WAV per sentence (e.g. b0427.wav)
├── metadata.csv        filename|transcript, one line per clip
└── recording_info.json name, clip count, minutes of speech, export date
```

Unzip it and upload the `dataset` folder to `MyDrive/my_voice_tts/` in Google Drive. Clips are named by sentence ID, so everyone's datasets line up sentence for sentence.

## The sentences

`sentences.js` holds 315 prompts:

- **300 from [CMU ARCTIC](http://www.festvox.org/cmu_arctic/)**, a sentence set Carnegie Mellon built for making synthetic voices, **reordered so each sentence adds as many new sound pairs (diphones) as possible**. The first 150 cover 86% of the sound pairs in all 1,132 ARCTIC sentences (more than the first 300 in the original order); the full 300 cover 99.5%.
- **15 questions**, one every 20 sentences, because ARCTIC has almost none and the model needs to hear rising question intonation.

To regenerate the list or tweak the selection:

```bash
pip install piper-tts
git clone --depth 1 https://github.com/festvox/festvox.git
python3 tools/select_sentences.py festvox/src/vox_files/prompts/cmuarctic.data
```

It uses the same espeak-ng phonemizer Piper trains with, so the coverage numbers match what the model actually sees.

## What the app does to each take

- Records raw audio with the browser's echo cancellation, noise suppression and auto-gain **turned off**, so the model learns your real voice.
- Trims the key-press clicks and silence at both ends, keeping a little padding.
- Resamples to 22,050 Hz mono, the rate Piper's medium models use.
- Warns (and doesn't auto-advance) if a take clipped, was very quiet, or ran close to the 20-second limit.
- Volume is left as recorded, not normalized per clip, so loudness stays consistent across the dataset. The mic check is what makes sure the level is right.

## Deploying

It's a static site with no build step: `index.html` + `sentences.js`.

- **Vercel:** import the repo, framework preset "Other", no build command. Or run `npx vercel` in this folder.
- Must be served over **https**, or browsers won't allow microphone access. (Opening `index.html` directly from disk also works in Chrome.)

## Credits

Sentences adapted from the CMU ARCTIC database by Carnegie Mellon University (John Kominek and Alan W Black), modified as described in [NOTICE](NOTICE).

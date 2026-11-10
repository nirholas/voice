---
name: avatar-voice
description: Give an avatar or app speech recognition, text-to-speech and ARKit lipsync with @three-ws/voice. Use when transcribing audio, synthesizing a voiced clip, or producing per-frame blendshape visemes for a talking avatar.
---

# Avatar voice with @three-ws/voice

A zero-dependency client (Node 18+ and browsers, uses `fetch`) for three endpoints on `https://three.ws`: `/api/asr` (NVIDIA Riva), `/api/tts/speak` (NVIDIA Magpie, with an OpenAI backstop and ElevenLabs for cloned voices) and `/api/a2f` (Audio2Face-3D). The free NVIDIA lanes need no key and no payment. They are rate limited, and exceeding the limit returns HTTP 429.

```bash
npm install @three-ws/voice
```

## Functions

```js
import { transcribe, speak, lipsync, say, voices, asrInfo, lipsyncInfo, createVoice } from '@three-ws/voice';
```

- `transcribe(audio, { format, language, sampleRate, words, model, signal })` returns `{ text, confidence, language, model, durationSec, words? }`. `audio` is a Blob, ArrayBuffer or Uint8Array. `format` is `wav`, `pcm`, `flac` or `ogg` (WebM/Opus must be decoded to PCM or WAV first). `language` defaults to `en-US`, `sampleRate` to 16000 and is required for raw `pcm`.
- `speak(text, { voice, format, language, speed, model, signal })` returns `{ blob, url, contentType, voice, format, model }`. Default voice is `nova`. Text is at most 4096 characters. Ids: nova, alloy, ash, ballad, coral, verse, echo, fable, onyx, sage, shimmer.
- `lipsync(audio, options)` takes `wav` or `pcm` audio and returns `{ fps, blendShapeNames, frames: [{ t, w }], frameCount, durationSec }`, an ARKit-52 track at 30 fps. `t` is seconds from clip start, so sample by the audio element's `currentTime`.
- `say(text, { voice, language })` does speak and lipsync in one round trip and returns `{ audio, animation }`.
- `voices()` is the live voice catalog. `asrInfo()` and `lipsyncInfo()` return `{ configured, ... }`. When `configured` is false, use an in-browser fallback.
- `createVoice({ baseUrl, apiKey, fetch })` binds one configuration. `THREE_WS_BASE_URL` overrides the default origin.

Failures reject with a typed `ThreeWsError` carrying a `code` such as `not_configured`.

## Worked example

```js
const { text } = await transcribe(audioBlob);
const reply = await speak(`You said: ${text}`, { voice: 'nova' });
const face = await lipsync(reply.blob);   // feed face.frames to @three-ws/avatar
```

To run the full loop against the live endpoints: `node examples/voice-loop.mjs` from a clone, which probes both lanes, speaks, lipsyncs and transcribes a sentence.

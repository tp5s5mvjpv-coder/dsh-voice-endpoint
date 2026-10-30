# dsh-voice-endpoint

Voice input for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) that **ends the take when you pause**, so dictation no longer needs a second click on Stop.

Click the microphone once, speak, and stop talking. After about a second of silence the take ends by itself, the existing SenseVoice provider transcribes it, and the text lands in the composer draft and is submitted for you — no extra click.

> Install [`dsh-voice-endpoint-bundle`](https://www.npmjs.com/package/dsh-voice-endpoint-bundle), not this package directly 鈥?the bundle carries the patch that swaps the stock toolbar row.

## Install

```bash
dsh plugin --profile web add dsh-voice-endpoint-bundle
```

Desktop builds manage the reserved `desktop` profile through their bundled CLI:

```bash
dsh plugin --profile desktop add dsh-voice-endpoint-bundle
```

Or use **Settings 鈫?Plugins 鈫?Add plugin** and enter `dsh-voice-endpoint-bundle`.

Restart the app (a page reload is not enough for a newly composed plugin row).

## Behaviour

| Action | Result |
| --- | --- |
| Speak, then pause ~1 s | Take ends, transcription starts, the text is inserted and the message is **submitted** |
| Speak, then press Stop | Same as stock: transcribe and insert |
| Cancel / Escape | Discards the capture, unchanged |
| Speech shorter than 200 ms | No auto-stop; the take waits for Stop or the duration limit |
| Auto-send disabled | Set `AUTO_SUBMIT_TRANSCRIPT = false` to keep the transcript in the draft for review |

## Tuning

Thresholds are client-side constants at the top of `lib/client.js` (mirrored in the source patch below):

```js
const ENDPOINT_RMS = 0.012        // RMS gate; raise it in a noisy room
const ENDPOINT_POLL_MS = 60       // sample cadence
const ENDPOINT_MIN_SPEECH_MS = 200
const ENDPOINT_SILENCE_MS = 1000  // silence that ends the take
const AUTO_SUBMIT_TRANSCRIPT = true   // false keeps the transcript in the draft
const AUTO_SUBMIT_DELAY_MS = 60
```

Edit `node_modules/dsh-voice-endpoint/lib/client.js` inside the profile and restart.

## How it works

The stock client plugin already samples the microphone for its waveform. This build reuses that reading:

1. speech crossing the RMS gate accumulates speaking time;
2. after 200 ms of speech, silence starts counting;
3. 1000 ms of continuous silence calls the same `finish()` the Stop button calls 鈥?stop capture, transcribe, insert.

The difference from upstream is 39 lines in `src/client/VoiceInput.tsx`; `docs/patch.md` in the repository carries the unified diff.

## Compatibility

- Built against DSH Desktop `0.2.1-rc.3` (`@deepseek-ai/dsh-experimental-client-ui-voice-input` 0.2.1-rc.3).
- The bundle patch disables the stock row id `ui-voice-input` and inserts `dsh-voice-endpoint`. If a future DSH release renames that row or changes the client plugin API, the patch fails: the microphone disappears. Uninstall the bundle to recover.
- Requires a speech provider that transcribes a complete clip 鈥?the stock `sensevoice-local` provider already does. This plugin does not add streaming transcripts.

## Credits

`lib/client.js` is a modified copy of `@deepseek-ai/dsh-experimental-client-ui-voice-input` (MIT, DeepSeek), with the endpoint detection described above.

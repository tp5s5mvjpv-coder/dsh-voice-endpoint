# dsh-voice-endpoint

Voice input for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) that **ends the take when you pause**, instead of waiting for a Stop click.

Click the microphone once, speak, stop talking. After about a second of silence the take ends, the configured speech provider transcribes it, and the text lands in the composer draft and is submitted for you.

```
对话输入框：[麦克风] 你说的话……（停顿 1s）
          → 自动结束录音 → 识别中… → 文字插入草稿 → 自动发送
```

## Packages

| Package | Role |
| --- | --- |
| [`dsh-voice-endpoint`](packages/dsh-voice-endpoint) | Client plugin: patched toolbar that ends a take on a detected pause |
| [`dsh-voice-endpoint-bundle`](packages/dsh-voice-endpoint-bundle) | Profile bundle: disables the stock `ui-voice-input` row and inserts the plugin |

Install only the bundle; it depends on the plugin.

## Install

```bash
dsh plugin --profile web add dsh-voice-endpoint-bundle
```

Desktop builds manage the reserved `desktop` profile:

```bash
dsh plugin --profile desktop add dsh-voice-endpoint-bundle
```

Then restart DeepSeek Harness.

Uninstall restores the stock toolbar:

```bash
dsh plugin --profile web remove dsh-voice-endpoint-bundle
```

## Tuning

Constants at the top of `packages/dsh-voice-endpoint/lib/client.js`:

```js
const ENDPOINT_RMS = 0.012        // raise in a noisy room
const ENDPOINT_POLL_MS = 60
const ENDPOINT_MIN_SPEECH_MS = 200
const ENDPOINT_SILENCE_MS = 1000  // silence that ends the take
```

`docs/patch.md` holds the 39-line unified diff against upstream `src/client/VoiceInput.tsx`, for the day the feature lands in tree.

## How it works

The stock plugin already samples the microphone to draw its waveform. This build reuses that reading: speech crossing an RMS gate accumulates speaking time, then continuous silence calls the same `finish()` the Stop button calls. No provider change: SenseVoice already transcribes a complete clip and does its own VAD segmentation.

## Compatibility

Built against DSH `0.2.1-rc.3`. The bundle patch targets the stock row id `ui-voice-input`. If a future DSH release renames that row or changes the client plugin contract, the patch cannot apply and the microphone disappears — uninstall to recover. This plugin does not add streaming transcripts.

## License

MIT. `packages/dsh-voice-endpoint/lib/client.js` is a modified copy of `@deepseek-ai/dsh-experimental-client-ui-voice-input` (MIT, DeepSeek).

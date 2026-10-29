# dsh-voice-endpoint-bundle

Profile bundle for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) that replaces the stock voice input toolbar row with [`dsh-voice-endpoint`](https://www.npmjs.com/package/dsh-voice-endpoint): the take ends and transcribes when you pause.

## Install

```bash
dsh plugin --profile web add dsh-voice-endpoint-bundle
```

Desktop builds (reserved `desktop` profile) use the bundled CLI:

```bash
dsh plugin --profile desktop add dsh-voice-endpoint-bundle
```

Then restart the app.

## What the patch does

```yaml
- id: ui-voice-input      # stock row from @deepseek-ai/dsh-experimental-voice-input-bundle
  disabled: true
- insert:
    - id: ui-voice-endpoint
      name: dsh-voice-endpoint
```

The stock speech services (`speech-to-text`, the SenseVoice provider, the API remote) stay untouched — only the toolbar client plugin is swapped, so settings, model preparation and language preferences keep working through the original bundle.

## Uninstall

```bash
dsh plugin --profile web remove dsh-voice-endpoint-bundle
```

Removing the bundle restores the stock row on the next start. If something goes wrong, remove it — the microphone comes back as it was.

## Compatibility

Built against DSH `0.2.1-rc.3`. If a future release renames the `ui-voice-input` row or changes the client plugin contract, the patch cannot apply and the microphone disappears; uninstalling recovers.

## Credits

The patched client code is derived from `@deepseek-ai/dsh-experimental-client-ui-voice-input` (MIT, DeepSeek).

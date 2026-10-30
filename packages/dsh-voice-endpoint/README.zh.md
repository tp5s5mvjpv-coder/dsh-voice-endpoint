# dsh-voice-endpoint

[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 的语音输入插件：**停顿即结束本次录音**，不用再点「停止」。

点一次麦克风，说一段，停下。约 1 秒静音后自动结束录音，交给现有的 SenseVoice 识别，文字落进输入框草稿并自动发送。

> 请安装 [`dsh-voice-endpoint-bundle`](https://www.npmjs.com/package/dsh-voice-endpoint-bundle)，它带有替换原工具栏行的 patch；单独装本包不会生效。

## 安装

```bash
dsh plugin --profile web add dsh-voice-endpoint-bundle
```

桌面版通过自带 CLI 管理保留的 `desktop` profile：

```bash
dsh plugin --profile desktop add dsh-voice-endpoint-bundle
```

也可以在 **设置 → 插件 → 添加插件** 里填 `dsh-voice-endpoint-bundle`。

装完需要重启应用（新组合的插件行刷新页面不够）。

## 行为

| 操作 | 结果 |
| --- | --- |
| 说话后停顿约 1 秒 | 自动结束录音、开始识别，文字插入草稿后**自动发送** |
| 说话后点「停止」 | 与原版一致：识别并插入 |
| 取消 / Esc | 丢弃本次录音，行为不变 |
| 语音短于 200ms | 不自动结束，等手动停止或时长上限 |
| 继续说下一句 | 上一条提交后麦克风自动重新打开，直接说下一句即可 |
| 关闭自动发送 | 设 `AUTO_SUBMIT_TRANSCRIPT = false`，文字留在草稿里等你看 |

## 调参

阈值是 `lib/client.js` 顶部的四个常量（源码补丁里同名）：

```js
const ENDPOINT_RMS = 0.012        // 音量门限，环境吵就调高
const ENDPOINT_POLL_MS = 60       // 采样间隔
const ENDPOINT_MIN_SPEECH_MS = 200
const ENDPOINT_SILENCE_MS = 1000  // 静音多久算说完
const AUTO_SUBMIT_TRANSCRIPT = true   // false 则只插入不发送
const AUTO_SUBMIT_DELAY_MS = 60
const AUTO_CONTINUE = true           // 提交后继续听下一条
const AUTO_CONTINUE_DELAY_MS = 600
const AUTO_CONTINUE_IDLE_MS = 9000  // 连续模式 9 秒没说话自动退出
```

改 profile 里的 `node_modules/dsh-voice-endpoint/lib/client.js`，然后重启。

## 原理

原插件的录音本来就在采样麦克风（画声波）。本插件复用这个读数：

1. 音量跨过门限开始累计语音时长；
2. 累计到 200ms 后开始计静音；
3. 连续静音 1000ms 即调用与「停止」按钮同一个 `finish()`——停止采集、转写、插入草稿。

相对上游，改动是 `src/client/VoiceInput.tsx` 里的 39 行；仓库中的 `docs/patch.md` 附统一 diff。

## 兼容性

- 基于 DSH Desktop `0.2.1-rc.3`（`@deepseek-ai/dsh-experimental-client-ui-voice-input` 0.2.1-rc.3）构建。
- bundle 的 patch 会禁用原行 id `ui-voice-input`、插入 `dsh-voice-endpoint`。如果未来 DSH 改了这个行 id 或客户端插件接口，patch 会失效——表现是麦克风消失，卸载本包即可恢复。
- 需要能整段转写的 Provider（自带的 `sensevoice-local` 就是），本插件不提供流式字幕。

## 致谢

`lib/client.js` 是 `@deepseek-ai/dsh-experimental-client-ui-voice-input`（MIT，DeepSeek）的修改副本，仅加入上述端点检测。

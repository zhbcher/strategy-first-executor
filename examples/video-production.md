# Example: Video Production Workflow

Applying strategy-first execution to the video production workflow.

## Strategy Output

```
## Strategy

### Goal
Produce a complete video: 口播稿 → TTS音频 → HTML场景 → 渲染输出

### Constraints
- 先音频定时长，再搭画面（不可反）
- 旺财声音：mlx-audio Qwen3-TTS，参考音频 wangcai_reference_audio.wav
- HTML场景 data-start / data-duration 对齐音频实际起止时间

### Execution Plan
1. 写稿 — 按画面分段的口播稿，每段对应一个场景
2. TTS生成 — 用旺财声音生成音频，拿到每段精确时长
3. 建HTML — 根据音频时长设置 data-start/data-duration
4. 渲染 — 生成最终视频

### Checkpoints
- 阶段1后：口播稿是否按场景分段，每段不超过30字
- 阶段2后：TTS是否生成成功，时长是否为精确值
- 阶段3后：HTML时间轴是否对齐音频
```

## Self-Review Output (simulated)

```
## Self-Review

### Checkpoint Verification
- [x] 阶段1 — 口播稿已按场景分段，共4段，每段20-28字
- [x] 阶段2 — TTS生成成功，4段时长分别为 3.2s, 2.8s, 4.1s, 3.5s
- [ ] 阶段3 — HTML时间轴使用了估算值而非实际TTS时长 → DEVIATION

### Deviations
- 阶段3: HTML场景的 data-duration 用了预估时间，未对齐TTS实际时长
  Should have been: 先用TTS输出更新HTML的data-duration

### Unresolved Issues
- 需要重新对齐HTML时间轴到实际TTS时长

### Verdict
NEEDS_FIX — 阶段3偏离策略，需要修正后重新渲染
```

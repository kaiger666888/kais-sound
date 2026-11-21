---
name: kais-sound
version: 1.0.0
description: '视频音效选材与下载工具。从 Freesound 搜索真实音效，结合下载量、评分人数、上传者描述与许可证为画面挑选合适声音，支持 CC0/BY 过滤、试听链接、并发下载。当用户需要"音效"、"sound effect"、"SFX"、"freesound"、"转场音效"、"whoosh"、"点击音"、"脚步声"、"环境音"、"拟音 foley"、"碰撞声"、"提示音"、"音效素材"、"下载音效"时触发。只管音效（SFX）；背景音乐用 kais-bgm，语音合成用 kais-TTS，人声/歌声用 kais-vocal。'
---

# kais-sound — Freesound 音效选材与下载

为视频画面挑选真实音效。你听不到这些音频，所以要依靠听过的人留下的信号：**下载量、评分人数、上传者描述、时长、许可证**。

## 核心流程

```bash
S=~/.openclaw/workspace/skills/kais-sound/scripts/sfx.py

# 1. 搜索（结果自动缓存到 ~/.cache/sfx-sources/index.json）
python3 $S freesound "<英文关键词>"                # ↓下载量 ★评分(人数) 许可证 描述
python3 $S freesound "soft whoosh" --license cc0 --urls   # 仅 CC0 + 试听链接
python3 $S get sfx/ fs:<id1> fs:<id2>              # 2. 按 fs:ID 并发下载（必须先搜索）
```

完整参数见 `references/usage.md`。下载的是 Freesound 的 HQ MP3 预览文件（非原始无损），常规视频剪辑够用。

## 选材规则（核心方法论）

<!-- FREEDOM:low -->
1. **画面 → 英文关键词**：把画面动作翻译成具体英文搜索词（卡片飞入 → "card whoosh"；纸张翻页 → "paper slide"）。
2. **下载量优先，评分人数加权**：优先选最符合画面且下载量最高的条目，引用数字说明推荐依据。★4.4(380) 比 ★4.9(6) 证据更强——评分人数比平均分重要。
3. **描述是最接近"试听"的证据**："built-in mic"/"made with my mouth" = 低质量 lo-fi；具体录音设备（Zoom、Sound Devices）+ 明确声源（"cork from a wine bottle"）= 质量可预期。**描述声源与画面不符的条目直接否决，无论多热门**。
4. **许可证**：CC0 免署名；BY 需署名（`get` 会打印作者提示，交付时保留来源与许可证信息）；商用避免 BY-NC。用 `--license cc0|by` 过滤。
5. **热度 ≠ 合适**：高下载量的 "thud" 可能是 3 秒电影级轰鸣。下载前核对时长；交付前用 ffmpeg 验证时长与能量分布（见下）。需要人工试听时加 `--urls` 给出预览链接。
<!-- /FREEDOM:low -->

## 下载后验证（ffmpeg）

<!-- FREEDOM:high -->
```bash
ffprobe -v error -show_entries format=duration -of csv=p=0 <file>   # 实际时长
ffmpeg -i <file> -af volumedetect -f null - 2>&1 | grep -E "mean_volume|max_volume"  # 能量分布
```
确认时长符合画面节奏、尾部无多余静音（需要裁剪时配合 `-af silenceremove` 或 trim）。
<!-- /FREEDOM:high -->

## 网络注意事项（国内环境）

- Freesound 国内直连较慢（实测 ~13KB/s，无代理）。脚本超时 60s×3 重试，搜索偶发慢属正常，重跑即可。
- 下载靠 6 线程并发提速，**一次 `get` 传多个 ID**，不要逐个下载。
- 脚本走终端 `http(s)_proxy` 环境变量；有代理时自动加速。
- 搜索结果缓存在 `~/.cache/sfx-sources/index.json`，`get` 依赖此缓存——**先搜索后下载**，换终端后缓存仍在。

## 边界

- 只处理音效（SFX）搜索、筛选、下载。BGM 生成 → `kais-bgm`；TTS 语音 → `kais-TTS`；人声/歌声 → `kais-vocal`。
- 下载量与评分只是线索，不代表已试听；不要把热度当作合适性结论，最终以 ffmpeg 实测 + 用户试听（`--urls`）为准。

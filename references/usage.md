# kais-sound 使用参考

## 脚本

```bash
S=~/.openclaw/workspace/skills/kais-sound/scripts/sfx.py
python3 $S freesound "<query>" [选项]    # 搜索
python3 $S get <out_dir> <key>...        # 下载，key 形如 fs:123456
```

仅依赖 Python 3 标准库，无需 API key（解析 Freesound 搜索页 HTML）。

## 搜索参数

| 参数 | 作用 |
| --- | --- |
| `--license cc0` | 仅搜索 CC0 |
| `--license by` | 搜索 CC0 和 Attribution |
| `--license any` | 不过滤许可证（默认） |
| `--urls` | 显示试听链接（HQ MP3 预览） |
| `-n 5` | 最多显示 5 条结果（默认 8） |
| `--sort relevance` | 相关性排序后按下载量重排（默认） |
| `--sort downloads` / `rating` / `short` | 按下载量 / 评分 / 时段排序 |
| `--pages 3` | 抓取页数（默认 3，relevance 模式用于重排） |
| `-p 2` | 起始页（默认第 1 页） |

## 输出格式

每行制表符分隔：`fs:ID  标题  时长  ↓下载量 ★评分(人数)  许可证  描述  [试听链接]`

- 许可证缩写：CC0 / BY / BY-NC / Sampling+
- 描述为上传者原文截断（90 字符），是判断音质的最佳代理信号

## 下载行为

- 依赖搜索缓存 `~/.cache/sfx-sources/index.json`（记录标题、URL、许可证、作者）
- 文件名格式：`fs_<id>_<标题slug>.mp3`，已存在则跳过
- 6 线程并发；Freesound 单连接限速 ~80KB/s（国内直连更慢），并发是唯一提速手段
- 非 CC0 条目下载时打印作者署名提示；**交付物中必须保留来源 + 许可证 + 作者信息**

## 许可证合规

| 许可证 | 商用 | 署名 | 说明 |
| --- | --- | --- | --- |
| CC0 | ✅ | 不需要 | 公共领域，首选 |
| BY | ✅ | 需要 | 注明作者与来源即可 |
| BY-NC | ❌ | 需要 | 非商业，商用项目避开 |
| Sampling+ | 视条目 | 需要 | 逐条检查 |

## 常见问题

- **搜索超时/失败**：国内直连 Freesound 慢，重跑一次通常可过；有代理时设置 `https_proxy` 环境变量。
- **`get` 报 unknown key**：缓存里没有该 ID——先执行对应搜索。
- **候选试听**：搜索加 `--urls`，把链接发给用户试听后再下载。

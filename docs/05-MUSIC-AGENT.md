# Music Agent 设计

## 1. 定位
Music Agent 将用户自然语言转换为稳定、结构化、可执行的歌曲计划，而不是把原始 Prompt 直接交给音乐 Provider。

`User Prompt → Safety → Intent Analyzer → Song Planner → SongSpec → Lyrics Agent → Lyrics QA → Provider Prompt Builder`

## 2. Intent Analyzer
输出：theme、story、language、genre_hint、moods、vocal_gender、duration_hint、energy_direction、special_requirements。只理解需求，不写歌词。

## 3. Song Planner
根据 Intent 确定 genre/subgenre、BPM、Key、时长、人声、配器、结构、能量曲线、歌词视角与风格。

SongSpec 示例结构：
```json
{
  "schema_version":"1.0",
  "theme":"毕业十年后的重逢",
  "language":"zh-CN",
  "genre":"chinese_pop",
  "moods":["nostalgic","warm"],
  "bpm":92,
  "duration_seconds":210,
  "vocal":{"gender":"male","age_style":"adult","tone":"warm"},
  "instruments":["piano","acoustic_guitar","bass","soft_drums"],
  "structure":[
    {"type":"INTRO","energy":15},
    {"type":"VERSE_1","energy":25},
    {"type":"CHORUS","energy":68},
    {"type":"FINAL_CHORUS","energy":90}
  ]
}
```

## 4. 参数优先级
`用户明确参数 > Prompt 明确表达 > Planner 推断 > 默认值`。Merge 后必须重新通过 Schema 校验。

## 5. Lyrics Agent
输入 Intent + SongSpec；输出 title + structured sections。Verse 推进故事，Chorus 聚焦主题和 Hook，Bridge 提供新的情绪/叙事角度。中文行长以可唱性为目标，避免散文化和空洞 AI 套话。

不模仿或复制具体作品；版权和内容安全策略在模型调用前后执行。

## 6. Lyrics QA
评分：structure、theme、singability、originality；同时执行内容安全/潜在侵权规则。建议 ≥80 PASS，65–79 可输出并记录问题，<65 自动重写一次；自动重写最多 2 次，避免无限成本。

## 7. 局部改写
用户可指定 SECTION 和 instruction，仅生成该 section，其他歌词保持不变，结果创建新 LyricsVersion。

## 8. Prompt Versioning
所有 Agent Prompt 有明确版本，如 `INTENT_ANALYZER_V1`、`SONG_PLANNER_V1`、`LYRICS_GENERATOR_V1`。AgentRun 保存 prompt_version、input/output、latency、cost、status，支持 A/B 与回归。

## 9. Provider Prompt Builder
不同 Provider 独立 Builder：`prompts/eleven.py`, `prompts/suno.py`, `prompts/minimax.py`。统一 SongSpec/Lyrics，不强求统一字符串 Prompt，以利用各 Provider 的结构化能力。

## 10. Fallback
Intent 失败重试一次；Planner 失败使用默认 SongSpec；Lyrics 失败有限重试；QA 服务失败不阻断主链路但必须记录。默认 SongSpec 建议中文流行、96 BPM、180 秒、自然成人声、钢琴/吉他/贝斯/鼓。

## 11. 缓存
对 prompt + 用户选项 + Agent/Prompt 版本计算 input_hash；相同输入可复用已验证 SongPlan，减少 LLM 成本，但用户明确要求重新生成时跳过缓存。
# AI MUSIC 产品需求规格说明书（PRD）

## 1. 产品概述
AI MUSIC 是面向中文创作者的 AI 歌曲自动生成平台。用户输入主题、故事、情绪、音乐风格、人声等要求，系统自动完成需求理解、歌曲策划、歌词创作、音乐生成、人声合成、试听、版本管理与下载。

核心价值链：`自然语言 → SongSpec → 歌词 → 完整歌曲 → 试听/迭代/下载`。

## 2. 产品目标
1. 将完整歌曲创作门槛降低到自然语言输入。
2. 优先解决中文歌词可唱性、中文人声自然度和 Prompt 遵循度。
3. Provider 解耦，避免业务绑定单一音乐模型。
4. 建立可量化的质量、成本、耗时与成功率评测体系。

## 3. 目标用户
- 无专业编曲能力的普通创作者
- 短视频、自媒体和直播创作者
- 广告、品牌、活动内容团队
- 独立音乐人和 Demo 创作者

## 4. 核心场景
- 输入一个主题快速生成完整歌曲
- AI 写词后人工修改，再生成歌曲
- 指定男/女声、风格、情绪、时长和配器
- 对同一项目生成多个歌曲版本并比较
- 在线试听、下载和分享成品

## 5. Alpha 功能范围
### P0
- 项目创建与自动保存
- Prompt 输入及创作参数
- Music Agent：Intent Analyzer、Song Planner、Lyrics Agent、Lyrics QA
- SongSpec 生成与用户参数覆盖
- 歌词结构化生成、编辑、版本化
- 异步歌曲生成、实时进度、失败状态
- Model Gateway 与 Provider Adapter
- 音频入库、在线播放和下载
- Project / LyricsVersion / SongVersion / GenerationTask / AudioAsset 持久化
- Benchmark 与模型成本记录

### P1
- 注册登录、我的作品
- 局部歌词 AI 改写
- 分享链接
- Provider Fallback 与场景路由
- 基础 Credits

### 暂不进入 Alpha
支付、公开社区、复杂协作、原生 App、训练自有基础音乐模型。

## 6. 主流程
1. 用户创建项目并输入主题。
2. 系统分析意图并生成 SongSpec。
3. Lyrics Agent 生成结构化歌词并执行 QA。
4. 用户编辑歌词/参数并保存新版本。
5. 用户点击“生成歌曲”。
6. API 创建 SongVersion 和 GenerationTask，Worker 异步调用 Provider。
7. 音频进入自有 Storage，生成 AudioAsset。
8. 前端通过 SSE 展示进度，完成后展示播放器。
9. 用户试听、重新生成、切换版本或下载。

## 7. SongSpec 核心字段
`theme, story, language, genre, subgenre, moods, bpm, key, duration_seconds, vocal, instruments, structure, lyrical_perspective, lyrical_style, rhyme_style`。

优先级：`用户明确参数 > Prompt 明确表达 > AI 推断 > 系统默认值`。

## 8. 任务状态
`QUEUED → RUNNING/PREPARING → MUSIC_GENERATION → COMPLETED`；异常进入 `FAILED`。任务必须支持幂等、超时、有限重试和可审计 Provider Attempt。

## 9. 非功能要求
- API 与 Provider Key 不得暴露到浏览器。
- 数据库为业务最终事实源，Redis 仅作队列/实时缓存。
- 第三方临时音频必须转存自有对象存储。
- 关键操作具备 Request ID、结构化日志和审计记录。
- Provider 失败不得导致任务状态丢失或重复扣费。
- 生产环境应具备限流、内容安全、版权确认和输入校验。

## 10. Alpha 质量 Gate
- 端到端生成成功率目标 ≥95%
- 主 Provider 中文人声盲听均分目标 ≥4.0/5
- 整体评分目标 ≥4.0/5
- 严重音频异常目标 <2%
- P95 生成时间目标 ≤5 分钟
- Publishable Rate 作为核心业务指标，目标在 Benchmark 后确定上线阈值

## 11. 商业化预留
后续支持 Credits Ledger、会员套餐、支付订单、优先队列、不同套餐模型路由和生成成本/毛利核算。
# AI MUSIC

面向中文创作者的 AI 歌曲自动生成平台。用户用自然语言描述主题、情绪和风格，系统通过 Music Agent 生成结构化歌曲方案与歌词，再经可替换的音乐模型生成、保存、播放和管理歌曲。

> 状态：产品与技术设计基线（Alpha 规划）。真实 Provider、价格与版权条款需在接入时复核。

## 核心链路

`Prompt → Music Agent → SongSpec + Lyrics → Model Gateway → 异步生成 → Audio Ingest → 播放/下载`

## 文档索引

| 文档 | 内容 |
|---|---|
| [产品需求规格说明书](docs/01-PRD.md) | 产品目标、用户、范围、流程、验收与非功能需求 |
| [页面原型与交互说明](docs/02-UX-PROTOTYPE.md) | 信息架构、页面线框、状态与交互 |
| [Technical Design](docs/03-TECHNICAL-DESIGN.md) | 架构、模块、任务编排、可靠性与安全 |
| [数据库与 API 详细设计](docs/04-DATABASE-API.md) | 数据模型、索引、状态机、REST/SSE 契约 |
| [Music Agent 设计](docs/05-MUSIC-AGENT.md) | Agent 输入输出、工作流、提示策略与质量门 |
| [Benchmark / 模型评测体系](docs/06-BENCHMARK.md) | 固定测试集、盲听、指标、路由决策 |
| [分阶段实施计划](docs/07-IMPLEMENTATION-PLAN.md) | M0–V1.0 阶段、依赖、里程碑与验收 |
| [Sprint Backlog](docs/08-SPRINT-BACKLOG.md) | 可执行用户故事、优先级、估点和完成定义 |

## Alpha 范围

- 中文主题输入、风格/人声/情绪设置
- AI 生成并允许编辑歌词
- 异步生成、实时进度、失败重试
- Provider 可替换与故障回退
- 音频持久化、在线播放、下载
- 项目、歌词版本、歌曲版本与生成记录
- 成本、耗时、成功率与质量评测

Alpha 不包含支付、社区、复杂协作、移动原生客户端和完整版权商业化承诺。

## 建议技术栈

- Web：Next.js、TypeScript、Tailwind CSS
- API：FastAPI、Pydantic、SQLAlchemy/Alembic
- 数据：PostgreSQL、Redis
- 异步：Celery Worker
- 存储：S3 兼容对象存储 + CDN
- 可观测性：结构化日志、Request ID、指标与告警
- 本地编排：Docker Compose

## 成功基线

- 端到端链路可复现：创建项目到播放成品
- 任务状态可追踪、失败可重试且不重复扣费
- Provider 不直接暴露给前端，音频落入自有存储
- Benchmark 能给出 Primary、Fallback 与场景路由结论
- 关键指标、成本和审计记录可查询

## 文档约定

文档中的“必须/应”分别代表上线阻断要求与推荐要求；示例 ID 使用 UUID；金额以最小货币单位存储；时间统一存 UTC；公开链接使用不可预测 token 并支持撤销。

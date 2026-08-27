# Sprint Backlog

建议 2 周/Sprint。优先级：P0=阻断，P1=重要，P2=增强。估点采用相对 Story Point。

## Sprint 1：工程基线 + Mock POC
- P0 初始化 Next.js/FastAPI（3）
- P0 Docker Compose：Postgres/Redis/API/Web/Worker（5）
- P0 Alembic + 基础模型（5）
- P0 创作首页与 Workspace 骨架（5）
- P0 MockMusicProvider + ModelGateway（5）
- P0 `POST /projects`、生成 Mock task（5）
- P0 SSE 进度 + Player（5）
- P1 结构化日志/Request ID（3）

DoD：新环境按 README 能启动；输入 Prompt 后 15 秒左右得到 Mock 音频；失败状态可展示。

## Sprint 2：持久化 + Music Agent
- P0 Project/LyricsVersion/SongVersion/GenerationTask/AudioAsset（8）
- P0 Repository/Service 事务边界（5）
- P0 Intent Analyzer（5）
- P0 Song Planner + SongSpec Schema/Merge（8）
- P0 Lyrics Agent + Lyrics QA（8）
- P1 AgentRun/Prompt Version（5）
- P1 歌词编辑和版本历史（5）

DoD：刷新页面可恢复完整项目；同一项目有可追踪歌词版本和 SongSpec。

## Sprint 3：真实 Provider + Storage
- P0 接第一个真实官方 Provider（8）
- P0 Provider 错误标准化/Timeout（5）
- P0 Audio Ingest + SHA256（5）
- P0 Local/S3 Storage Adapter（5）
- P0 Provider Attempt/成本记录（5）
- P1 Retry/Fallback（5）
- P1 播放/下载签名 URL（3）

DoD：真实中文 Prompt 能生成新歌曲、入自有 Storage 并在线播放；Provider 临时 URL 不作为永久资产。

## Sprint 4：Benchmark + 模型路由
- P0 CN_ALPHA_V1 20 case（5）
- P0 Benchmark Runner（8）
- P0 低优先级 Queue/并发/预算控制（5）
- P0 盲听评分页（8）
- P0 报告：质量/成本/P50/P95/成功率（8）
- P1 Failover Chaos Cases（5）
- P1 Route Config（5）

DoD：产出可复现报告，明确 Primary/Fallback；模型品牌对评审隐藏。

## Sprint 5：Product Alpha
- P0 注册登录/授权（8）
- P0 我的作品/项目列表（5）
- P0 自动保存/恢复（5）
- P0 Song Version 比较/重新生成（5）
- P0 下载（3）
- P1 分享 token/撤销（5）
- P1 基础 Credits Ledger（8）
- P1 生成历史（3）

DoD：真实用户可注册→创作→生成→试听→修改→再生成→保存→下载。

## Sprint 6：Alpha 上线准备
- P0 E2E/回归测试（8）
- P0 限流/权限/安全检查（5）
- P0 监控告警/Provider Dashboard（5）
- P0 任务恢复/故障演练（5）
- P0 DB Backup/Restore 演练（3）
- P1 埋点与漏斗（5）
- P1 成本 Dashboard（5）
- P1 Alpha 用户反馈入口（3）

DoD：Alpha Gate 达标；有明确 rollback、告警、备份和 Provider 降级方案。

## 后续商业化 Backlog
会员/套餐、Credits 充值、支付、订单、退款/对账、Admin 用户与任务管理、内容审核、版权审计、优先队列、运营模板、增长/分享、社区。

## 全局 Definition of Done
代码 Review；自动测试通过；迁移可回滚；接口文档同步；无明文 Secret；关键日志/指标齐全；失败路径测试；涉及费用的任务必须可审计且幂等。
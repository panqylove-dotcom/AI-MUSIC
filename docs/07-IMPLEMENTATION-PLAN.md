# 分阶段实施计划

## 总体路线
| 阶段 | 目标 | 参考周期 | Gate |
|---|---|---:|---|
| P0 项目准备 | 工程基线 | 3–5天 | 一键启动全部基础服务 |
| P1 M0 技术 POC | 跑通 Mock 链路 | 1–2周 | Prompt→歌词→任务→播放器 |
| P2 真实模型验证 | 生成第一首真实歌曲 | 1–2周 | Provider+Storage 端到端成功 |
| P3 Benchmark | 决定模型策略 | ~1周 | Primary/Fallback/成本结论 |
| P4 M1 Product Alpha | 用户可持续使用 | 2–3周 | 注册→创作→版本→下载 |
| P5 M2 商业化 Beta | 可收费运营 | 2–3周 | Credits/支付/后台/审计 |
| P6 V1.0 | 稳定上线 | 1–2周+持续 | 性能、安全、监控、灰度达标 |

## P0 项目准备
Next.js/FastAPI 脚手架；PostgreSQL、Redis、Celery、Docker Compose；dev/staging/prod 配置；Alembic；日志/错误码/Request ID；CI。禁止提前堆业务。

## P1 M0 技术 POC
单页创作、MockMusicProvider、ModelGateway、歌词占位服务、GenerationTask、Celery、Redis、SSE、播放器。验收：刷新不崩、任务终态明确、Mock 可模拟耗时/失败。

## P2 真实模型验证
接至少 1 个可用官方 Provider；错误标准化；超时/重试；Audio Ingest；Storage；GenerationTask/AudioAsset 持久化；真实成本记录。Gate：一句中文主题真正生成新歌曲并在平台播放。

## P3 Benchmark
建立 CN_ALPHA_V1；固定 SongSpec/Lyrics；多 Provider 批量生成；盲听后台；质量/成本/耗时/成功率；Failover 测试；形成模型路由决策。未过质量 Gate 不进入大规模商业功能。

## P4 M1 Product Alpha
用户体系、我的作品、Project、LyricsVersion、SongVersion、自动保存、历史恢复、重新生成、下载、分享、生成历史、基础 Credits。邀请小规模真实用户。

## P5 M2 商业化 Beta
Credits Ledger、充值/套餐/订单/支付回调、冻结/扣除/退款、会员等级、队列优先级、Admin、内容审核、版权确认、投诉/审计、限流和对账。

## P6 V1.0
压测、故障演练、Provider 降级、备份恢复、对象存储/CDN、安全测试、支付对账、指标告警、灰度发布、埋点。核心指标：提交→生成成功、试听、完整播放、再次生成、下载、付费转化、单曲毛利。

## 团队工作流
前端：Workspace/播放器/SSE/作品库；后端：User/Project/Version/Task/Credits/API；AI：Agent/SongSpec/Lyrics/Provider/Benchmark；QA：API/E2E/异常/账务；DevOps：环境/队列/DB/Storage/监控。

## 关键依赖
真实 Provider 接入依赖其当期 API 权限、价格、速率限制和商业条款，进入 P2 时必须重新核验；任何 Provider 不应成为业务模型的硬编码依赖。
# Technical Design

## 1. 架构
```text
Next.js
  ↓ REST / SSE
FastAPI
  ├─ Music Agent
  ├─ Project/Lyrics/Song Services
  └─ Generation API
        ↓
PostgreSQL ←→ Redis
                ↓
            Celery Worker
                ↓
           Model Gateway
          ↙      ↓       ↘
       Provider A/B/C
                ↓
          Audio Ingest
                ↓
        Object Storage/CDN
```

## 2. 技术栈
Web：Next.js + TypeScript + Tailwind；API：FastAPI + Pydantic；ORM：SQLAlchemy + Alembic；DB：PostgreSQL；队列/缓存：Redis + Celery；Storage：S3 Compatible；本地：Docker Compose。

## 3. 分层
- API：参数、鉴权、HTTP 契约
- Service：事务和业务编排
- Repository：数据访问
- Agent：Intent/Planner/Lyrics/QA
- Model Gateway：Provider 路由、能力差异、错误标准化
- Worker：耗时生成任务
- Storage：Local/S3/OSS/COS Adapter

## 4. 异步生成
API 在事务中创建 SongVersion + GenerationTask，commit 后仅把 `task_id` 投递队列。Worker 从 DB 重建请求，调用 Provider，更新 DB 与 Redis。DB 是最终事实源；Redis 服务实时 SSE。

## 5. Model Gateway
统一输入 `MusicGenerationRequest`，Provider 声明 capabilities（同步/异步、自定义歌词、结构、流式、最大时长）。Gateway 统一 Timeout、RateLimit、Unavailable、Authentication、ContentRejected 等错误，并对可恢复错误执行有限重试/回退。

## 6. 幂等与重试
- GenerationTask 具有唯一 ID 和明确终态。
- Worker 读取到 SUCCEEDED 时直接返回。
- Provider Attempt 单独记录，避免一次 Fallback 覆盖历史。
- 仅对 Timeout/429/5xx 等可恢复异常重试。
- ContentRejected/Auth 等不可恢复异常直接终止。
- Credits 后续采用 Ledger + reservation/finalize/refund，保证重试不重复扣费。

## 7. SSE
`GET /tasks/{id}/events` 发送状态变化；断线允许重连。终态后结束连接。生产环境配置代理禁用响应缓冲并设置心跳。

## 8. 音频入库
Provider 返回 bytes 或临时 URL → 校验 Content-Type/大小 → SHA-256 → Storage → AudioAsset。前端只使用平台自己的播放 URL/签名 URL。

## 9. 配置
环境变量管理 DATABASE_URL、REDIS_URL、Provider Key、Storage、超时、并发。密钥仅服务端可见。dev 默认 Mock Provider。

## 10. 可观测性
日志包含 request_id、task_id、project_id、provider、model、attempt、latency、error_code；指标至少包括成功率、队列等待、生成耗时 P50/P95、Provider 429/5xx、单曲成本。

## 11. 安全
鉴权/授权、输入长度限制、上传/下载安全、内容审核、Prompt 注入隔离、Provider Key 保密、分享 token 不可预测且可撤销、审计日志、数据库备份和最小权限。

## 12. 部署单元
frontend、backend、worker、postgres、redis、object-storage（生产可外部托管）。Benchmark 使用独立低优先级 queue，避免影响用户生成。
# 数据库与 API 详细设计

## 1. 核心实体
### users
`id, nickname, status, created_at, updated_at`

### song_projects
`id, user_id, title, prompt, genre, status, intent_json(JSONB), song_spec(JSONB), created_at, updated_at`

### lyrics_versions
`id, project_id, version_no, title, lyrics_text, lyrics_structured(JSONB), source, created_at`
唯一约束建议 `(project_id, version_no)`。

### song_versions
`id, project_id, lyrics_version_id, version_no, name, status, song_spec(JSONB), created_at`
唯一约束 `(project_id, version_no)`。

### generation_tasks
`id, user_id, project_id, song_version_id, provider, model, provider_task_id, status, stage, progress, input_payload, output_payload, provider_cost, error_code, error_message, created_at, updated_at`

### provider_attempts
`id, generation_task_id, attempt_no, provider, model, status, provider_task_id, started_at, completed_at, latency_ms, cost, error_code, error_message`

### audio_assets
`id, song_version_id, asset_type, storage_key, mime_type, file_size, checksum_sha256, status, created_at`

### agent_runs
`id, project_id, agent_type, model, prompt_version, input_hash, input_payload, output_payload, latency_ms, cost, status, created_at`

### benchmark_runs / benchmark_results / benchmark_reviews
用于固定测试集、Provider 输出、技术指标与盲听评分。

## 2. 状态
Project：DRAFT / READY / ARCHIVED。
SongVersion：CREATED / GENERATING / COMPLETED / FAILED。
GenerationTask：QUEUED / RUNNING / SUCCEEDED / FAILED / CANCELLED。

## 3. 主要 API
### Project
- `POST /projects`
- `GET /projects/{id}`
- `PATCH /projects/{id}`
- `GET /projects`

### Music Agent / Lyrics
- `POST /projects/{id}/plan`
- `PATCH /projects/{id}/song-spec`
- `POST /projects/{id}/lyrics/versions`
- `POST /projects/{id}/lyrics/rewrite`
- `GET /projects/{id}/lyrics/versions`

### Generation
- `POST /projects/{id}/generate` → 202
- `GET /tasks/{task_id}`
- `GET /tasks/{task_id}/events` → SSE
- `POST /tasks/{task_id}/retry`

### Song/Audio
- `GET /projects/{id}/songs`
- `GET /songs/{song_version_id}`
- `GET /audio/{asset_id}/play-url`
- `GET /audio/{asset_id}/download-url`

### Benchmark Admin
- `POST /admin/benchmarks`
- `GET /admin/benchmarks/{id}`
- `GET /admin/benchmarks/{id}/review/next`
- `POST /admin/benchmarks/{id}/reviews`
- `GET /admin/benchmarks/{id}/report`

## 4. 示例：生成歌曲
Request：
```json
{"lyrics_version_id":"uuid","duration_seconds":210}
```
Response 202：
```json
{"task_id":"uuid","song_version_id":"uuid","status":"QUEUED"}
```

## 5. Task Response
```json
{"task_id":"uuid","status":"RUNNING","stage":"MUSIC_GENERATION","progress":65,"audio_asset_id":null,"error":null}
```

## 6. SSE
```text
event: progress
data: {"status":"RUNNING","stage":"MUSIC_GENERATION","progress":65}

```
终态发送 SUCCEEDED/FAILED 后关闭。

## 7. 数据原则
- 金额优先使用整数最小货币单位或高精度 Decimal，禁止 float 财务累计。
- 时间存 UTC。
- JSONB 保存模型可演进结构，但可查询关键字段应独立列化/索引。
- 外键、唯一约束和高频过滤字段必须建立索引。
- 业务事务由 Service 控制，Repository 不擅自 commit。
- Queue 中只传 task_id，避免消息 Payload 成为第二事实源。
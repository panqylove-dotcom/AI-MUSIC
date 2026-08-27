# Benchmark / 模型评测体系

## 1. 原则
同一测试 case 只运行一次 Music Agent，固定 SongSpec + Lyrics，再送入各 Provider；否则无法区分歌词质量和音乐模型质量。

## 2. Alpha 测试集
20 个中文场景：华语流行4、民谣3、摇滚3、R&B2、Rap2、中国风/古风2、商业/短视频2、儿童/女声治愈2。每 Provider 每 case 生成 2 次；3 个 Provider 时共 120 首。

典型 case：毕业十年重逢、35 岁创业失败重新开始、高中暗恋、婚礼、父亲、离乡、沿海旅行、普通人热爱、大学乐队、电影感摇滚、都市 R&B、凌晨两点、上班族 Rap、小城打拼 Rap、现代中国风、古风旧城、科技品牌 60 秒、周五下班 30 秒、儿童整理房间、关系结束后的治愈女声。

## 3. 自动指标
- Generation Success Rate
- Generation Time P50/P95
- Provider Cost / Song
- Audio Duration / File Validity / Silent Ratio
- Provider 429/5xx/Timeout
- 后续可增加自动歌词 Alignment

## 4. 盲听
隐藏 Provider/Model，随机 Track 顺序。每项 1–5：中文人声自然度25%、旋律20%、歌词演唱准确15%、Prompt 遵循15%、编曲10%、音质10%、速度成本效率5%。至少 3 名评审，建议 5 名。

增加业务问题：`Would you publish this track? YES / MAYBE / NO`，计算 Publishable Rate。

## 5. 报告
每 Provider 输出总分、各维度、成功率、P50/P95、平均真实成本、Publishable Rate、典型失败原因。最终不仅选择 Winner，还允许按场景路由，例如 Pop→A、Rap→B、中国风→C。

## 6. Alpha Gate
目标参考：成功率 ≥95%；中文人声 ≥4.0/5；整体 ≥4.0/5；严重音频异常 <2%；P95 ≤5 分钟。成本阈值必须用真实账单数据后确定。

若所有模型 Publishable Rate 明显偏低，应优先优化 SongSpec、Prompt Builder、Lyrics 和 Provider，而不是进入支付/社区开发。

## 7. Failover 测试
主动模拟 Timeout、429、500、Unavailable，验证 Primary → retry → Fallback。每个 Attempt 单独留痕。

## 8. 运行控制
Benchmark 使用低优先级独立队列；限制 Provider 并发和 Global 并发；Run 支持 max_budget，达到预算自动停止，避免测试失控。

## 9. 可复现性
每条结果保存 benchmark_version、case_id、Prompt Version、Music Agent Version、SongSpec、Lyrics、Provider、Model Version、参数、生成时间、成本、AudioAsset 和错误。

## 10. Benchmark 版本
首版 `CN_ALPHA_V1`；后续 V2 可增加方言、女声 Rap、电子、更多广告/短视频等场景。
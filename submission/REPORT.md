# Báo cáo cá nhân — K4-L3A Day 13 Monitoring & LLMOps

## 1. Thông tin học viên

- **Họ và tên:** Nguyễn Thanh Hòa
- **MSSV:** 2A202602559
- **Lớp:** K4-L3A
- **Repository URL:** https://github.com/Jikay-070203/K4-L3A-Day13-Monitoring-LLMOps
- **Commit SHA cuối:** 13b606680ae4a3072eda90334959b632fe4ecba0
- **Challenge ID:** `day13-k4-l3a-monitoring-llmops-v1`
- **Incident:** `rag_slow`
- **Tên project Langfuse cá nhân:** K4-L3A-Day13-NguyenThanhHoa-2A202602559Monitoring-LLMOps

## 2. Evidence index

Evidence hiện có: `evidence/07-trace-waterfall.png`, `evidence/08-trace-metadata.png`, `evidence/09-prompt-versions.png`, `evidence/10-prompt-rollback.png`, `evidence/12-incident-metric.png`, `evidence/13-incident-log.png`, `evidence/14-incident-trace.png` và `evidence/Langfuse.png`.

Cần bổ sung vào `submission/evidence/`: `01-pytest`, `02-log-validator`, `03-dashboard-validator`, `04-structured-log`, `05-pii-redaction` và `11-dashboard-overview`.

## 3. Kết quả kỹ thuật

| Nội dung | Baseline | Kết quả cuối | Nhận xét |
|---|---:|---:|---|
| `validate_logs.py` | 30/100 | 100/100 | Đủ schema, correlation ID, enrichment và PII scrub |
| `validate_dashboard.py` | Chưa có log | 6/6 panel | Dashboard contract hợp lệ |
| `pytest` | — | 22 passed | Không còn test thất bại |
| Số correlation ID | 0 | 37 | Nối được log với request |
| PII leak | 0 | 0 | Dữ liệu nhạy cảm được redact |
| Response latency P95 | — | 3527 ms | Tính trên `data/logs.jsonl` hiện tại |
| Response latency P99 | — | 9891 ms | Có ảnh hưởng của workload incident |

## 4. Logging và PII

Middleware xóa context cũ, nhận `x-request-id` hoặc sinh ID theo format `req-<8-hex>`, bind vào structlog contextvars và trả về qua header `x-request-id` cùng `x-response-time-ms`.

Structured log chứa timestamp, level, event, service, correlation ID, user hash, session, feature, model, environment, latency, TTFT, token usage, cost, quality score và trạng thái retrieval.

PII scrubber chạy trước file renderer. Email, điện thoại, CCCD và thẻ thanh toán được thay bằng placeholder; user ID không ghi thô mà được hash. Kết quả cuối: `100/100`, 37 correlation IDs và 0 PII leak.

## 5. Tracing và prompt versioning

Trace được tạo trong project Langfuse cá nhân. Cấu trúc gồm root `lab-agent-run`, child `retrieval` và child `llm-generation`. Correlation ID nằm trong metadata để nối trace với structured log. Generation ghi model, input/output/total tokens và cost.

- **Prompt name:** `day13-chat`
- **Version #1:** label `baseline`.
- **Version #2:** label `candidate`, sau đó promote `production`.
- **Trace version baseline:** Bổ sung trace ID thật.
- **Trace version candidate/production:** Bổ sung trace ID thật.
- **Trace incident:** `1539bfa4c3eba4fc072ea8cdbb32231`.

Sau workload version #2, `production` được rollback về version #1. Evidence: `evidence/09-prompt-versions.png` và `evidence/10-prompt-rollback.png`.

## 6. Dashboard, SLO và alerts

Dashboard có đúng sáu panel: latency, traffic, errors/retrieval success, cost, tokens và quality. Nguồn chuẩn là `data/logs.jsonl`; validator xác nhận `6/6`.

SLO chính `fast_successful_requests` dùng cửa sổ 28 ngày: request tốt là `response_sent` với latency không quá 3000 ms. Target `99.5%`, error budget `0.5%`.

Ba alert đã hoàn thiện trong `config/alert_rules.yaml` và `docs/alerts.md`:

1. `high_latency_burn`: critical, P95 latency > 3000 ms hoặc burn rate > 2 trong 10 phút.
2. `elevated_error_rate`: critical, error rate > 2% trong 5 phút.
3. `retrieval_degradation`: warning, retrieval success rate < 90% trong 10 phút.

Các alert dùng Slack, có owner và runbook tương ứng.

## 7. Điều tra challenge

- **Challenge ID:** `day13-k4-l3a-monitoring-llmops-v1`
- **Incident:** `rag_slow`
- **Triệu chứng:** 5 request trả `200` nhưng latency tăng khoảng 7983 ms, 10643 ms và 13296 ms.
- **Correlation IDs:** `req-62eafe86`, `req-a737bdac`, `req-da816639`, `req-88cef5e8`, `req-4dba31e7`.
- **Trace ID:** `1539bfa4c3eba4fc072ea8cdbb32231`.
- **Span gây ảnh hưởng:** child `retrieval` có latency tăng.
- **Root cause:** `rag_slow` làm retrieval chậm, kéo dài tổng request latency.
- **Fix action:** chạy `python scripts/inject_incident.py --disable` để khôi phục trạng thái bình thường.
- **Preventive measure:** theo dõi P95/P99, retrieval success rate, alert theo SLO và điều tra theo Metrics → Logs → Traces.

Incident đã được tắt: `rag_slow: false`, `tool_fail: false`, `cost_spike: false`.

## 8. Giải thích và tự đánh giá

- **Quyết định kỹ thuật:** dùng structlog contextvars để các event trong cùng request dùng chung correlation ID và metadata.
- **Blocker:** Langfuse v4 dùng `usage_details` và `cost_details`; sau khi sửa API, toàn bộ 22 test pass và request trở lại HTTP 200.
- **Metrics → Logs → Traces:** metrics xác định triệu chứng; log tìm request qua correlation ID; trace xác định span gây chậm/lỗi.
- **Prompt/token/cost/SLO:** prompt label giúp truy xuất và rollback; token/cost theo dõi chi phí; SLO/error budget hỗ trợ quyết định vận hành.
- **Hạn chế:** cần bổ sung đầy đủ ảnh evidence và trace ID thật trước khi nộp.

## 9. Checklist trước khi nộp

- [ ] Bổ sung commit SHA cuối, tên project Langfuse và trace ID thật.
- [ ] Đưa toàn bộ evidence vào `submission/evidence/`.
- [ ] Bổ sung dashboard runtime, structured log, PII redaction và trace list.
- [ ] Không commit `.env`, secret hoặc `config/challenge.json`.
- [ ] Chạy lại pytest và hai validator trên commit cuối.
- [ ] Commit/push repository cá nhân và nộp URL/commit SHA trên LMS.

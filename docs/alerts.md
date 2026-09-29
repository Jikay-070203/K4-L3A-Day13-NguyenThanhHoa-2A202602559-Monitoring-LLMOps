# Alert và Runbook

Mỗi alert phải dựa trên triệu chứng người dùng hoặc SLO, không dựa trực tiếp vào tên implementation nội bộ.

## Alert 1

- Tên: high_latency_burn
- Severity: critical
- Duration: 10 phút
- Kênh thông báo: Slack
- SLI/SLO liên quan: `fast_successful_requests`, P95 latency
- Điều kiện và thời gian duy trì: P95 latency vượt 3000 ms hoặc burn rate SLO > 2 trong 10 phút.
- Ảnh hưởng tới người dùng: phản hồi chậm, timeout hoặc trải nghiệm chat kém.
- Ba bước kiểm tra đầu tiên: xem dashboard latency; lấy `correlation_id` từ log; mở trace và tìm span chậm.
- Mitigation tạm thời: giảm concurrency, tắt feature gây chậm hoặc rollback prompt production.
- Owner: platform-oncall

## Alert 2

- Tên: elevated_error_rate
- Severity: critical
- Duration: 5 phút
- Kênh thông báo: Slack
- SLI/SLO liên quan: error rate guardrail 2%
- Điều kiện và thời gian duy trì: error rate > 2% liên tục trong 5 phút.
- Ảnh hưởng tới người dùng: request thất bại hoặc không nhận được câu trả lời.
- Ba bước kiểm tra đầu tiên: kiểm tra `request_failed`; phân nhóm `error_type`; đối chiếu trace và deployment gần nhất.
- Mitigation tạm thời: rollback thay đổi mới, chuyển sang local prompt fallback và giảm tải.
- Owner: api-oncall

## Alert 3

- Tên: retrieval_degradation
- Severity: warning
- Duration: 10 phút
- Kênh thông báo: Slack
- SLI/SLO liên quan: retrieval success rate tối thiểu 90%
- Điều kiện và thời gian duy trì: retrieval success rate < 90% liên tục trong 10 phút.
- Ảnh hưởng tới người dùng: câu trả lời thiếu ngữ cảnh hoặc chất lượng giảm.
- Ba bước kiểm tra đầu tiên: xem panel retrieval success; kiểm tra log `tool_success`; mở retrieval span thất bại.
- Mitigation tạm thời: dùng index/dataset dự phòng, giảm phạm vi truy vấn và bật cảnh báo chất lượng.
- Owner: ml-platform

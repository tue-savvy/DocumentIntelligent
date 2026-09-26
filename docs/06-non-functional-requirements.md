# 06. Non-Functional Requirements

## 1. Performance

| ID | Yêu cầu |
|----|---------|
| NFR-PERF-001 | Xử lý (ingest → extraction → validation) một invoice 1–3 trang: **p95 ≤ 60 giây**; HĐĐT XML: **p95 ≤ 10 giây** (không tính thời gian chờ cổng Thuế). |
| NFR-PERF-002 | Chứng từ lớn (≤ 100 trang): p95 ≤ 10 phút. |
| NFR-PERF-003 | Thời gian phản hồi UI: p95 ≤ 2 giây cho thao tác thông thường; mở màn hình review chứng từ ≤ 3 giây. |
| NFR-PERF-004 | Full-text search trên 10 triệu chứng từ: p95 ≤ 3 giây. |
| NFR-PERF-005 | API: p95 ≤ 500 ms cho API đọc; upload trả về `document_id` ≤ 2 giây. |
| NFR-PERF-006 | AI assistant: phản hồi token đầu tiên ≤ 3 giây (streaming), câu trả lời hoàn chỉnh p95 ≤ 20 giây. |

## 2. Scalability & Capacity

| ID | Yêu cầu |
|----|---------|
| NFR-SCL-001 | Thiết kế cho **≥ 100.000 chứng từ/tháng/tenant** (≈ 500.000 trang), burst **5.000 chứng từ/giờ** (cuối tháng). |
| NFR-SCL-002 | Horizontal scaling cho các worker (OCR, AI, integration) dựa trên queue depth (autoscaling). |
| NFR-SCL-003 | Hỗ trợ ≥ 500 concurrent users/tenant, ≥ 1.000 tenant (SaaS). |
| NFR-SCL-004 | Lưu trữ ≥ 10 năm dữ liệu với tiering (hot/warm/cold archive) để tối ưu chi phí. |

## 3. Availability & Reliability

| ID | Yêu cầu |
|----|---------|
| NFR-AVL-001 | Uptime **≥ 99,9%**/tháng cho Production (không tính bảo trì có kế hoạch, thông báo trước ≥ 72 giờ, ngoài giờ làm việc). |
| NFR-AVL-002 | **RPO ≤ 15 phút**, **RTO ≤ 4 giờ**; backup hằng ngày, lưu 35 ngày; kiểm thử restore định kỳ hằng quý. |
| NFR-AVL-003 | Triển khai multi-AZ; DR site (cross-region) cho gói Enterprise. |
| NFR-AVL-004 | Không mất chứng từ: mọi file nhận được phải được lưu bền vững trước khi xác nhận "received"; xử lý at-least-once + idempotent. |
| NFR-AVL-005 | Graceful degradation: khi AI/OCR provider hoặc cổng Thuế lỗi, chứng từ được queue và xử lý lại; người dùng vẫn review/nhập tay được. |

## 4. Security

| ID | Yêu cầu |
|----|---------|
| NFR-SEC-001 | Mã hóa **in transit** TLS 1.2+ (khuyến nghị 1.3); **at rest** AES-256; hỗ trợ **customer-managed keys (BYOK)** cho gói Enterprise. |
| NFR-SEC-002 | Tenant isolation (logical tối thiểu; tùy chọn dedicated DB/instance). |
| NFR-SEC-003 | RBAC + ABAC (data-level); nguyên tắc least privilege; SoD enforcement. |
| NFR-SEC-004 | SSO, MFA bắt buộc cho admin & approver ngưỡng cao; session timeout cấu hình; IP allowlist (tùy chọn). |
| NFR-SEC-005 | Masking dữ liệu nhạy cảm (số tài khoản, CCCD, thông tin cá nhân) trong UI theo quyền và trong logs. |
| NFR-SEC-006 | Tuân thủ **OWASP ASVS Level 2**; SAST/DAST/SCA trong CI/CD; pentest độc lập ≥ 1 lần/năm và trước go-live. |
| NFR-SEC-007 | Quét malware mọi file upload; sandbox xử lý file; chặn macro/active content. |
| NFR-SEC-008 | Audit log bất biến (append-only/WORM), lưu ≥ 10 năm cho log liên quan chứng từ kế toán, ≥ 1 năm cho log truy cập. |
| NFR-SEC-009 | Bảo vệ AI: chống prompt injection, không để LLM gọi hành động tài chính, output filtering, giới hạn dữ liệu gửi tới LLM theo nhu cầu tối thiểu. |
| NFR-SEC-010 | Quản lý bí mật trong vault, rotate định kỳ; không hardcode credentials. |

## 5. Compliance & Legal

| ID | Yêu cầu |
|----|---------|
| NFR-CMP-001 | **Luật Kế toán 2015** & văn bản hướng dẫn: lưu trữ chứng từ kế toán điện tử, đảm bảo tính toàn vẹn, truy xuất được, thời hạn lưu trữ (≥ 10 năm cho chứng từ dùng trực tiếp ghi sổ). |
| NFR-CMP-002 | Quy định về **hóa đơn, chứng từ điện tử** (Nghị định 123/2020/NĐ-CP, sửa đổi bởi Nghị định 70/2025/NĐ-CP; Thông tư 78/2021/TT-BTC và văn bản hiện hành): lưu trữ hóa đơn điện tử dạng gốc XML, kiểm tra tính hợp lệ. |
| NFR-CMP-003 | **Luật Giao dịch điện tử 2023**: giá trị pháp lý của thông điệp dữ liệu, chữ ký điện tử. |
| NFR-CMP-004 | **Bảo vệ dữ liệu cá nhân**: Nghị định 13/2023/NĐ-CP và **Luật Bảo vệ dữ liệu cá nhân** (hiệu lực từ 01/01/2026) — xác định vai trò bên kiểm soát/xử lý dữ liệu, đánh giá tác động (DPIA), hồ sơ chuyển dữ liệu ra nước ngoài khi dùng cloud/LLM ngoài lãnh thổ. |
| NFR-CMP-005 | Quy định về **an ninh mạng & lưu trữ dữ liệu** tại Việt Nam (Luật An ninh mạng, Nghị định 53/2022/NĐ-CP) — hỗ trợ tùy chọn triển khai data center tại Việt Nam. |
| NFR-CMP-006 | Chuẩn quốc tế (cho khách hàng đa quốc gia): **GDPR**, **SOC 2 Type II**, **ISO/IEC 27001**; hỗ trợ chứng cứ cho SOX/ICFR controls (audit trail, SoD, approval evidence). |
| NFR-CMP-007 | Data residency cấu hình theo tenant (Việt Nam, Singapore, …). |

> Danh sách văn bản pháp lý cần được bộ phận pháp chế/tư vấn thuế của khách hàng xác nhận lại tại thời điểm triển khai do quy định có thể thay đổi.

## 6. Usability & Accessibility

| ID | Yêu cầu |
|----|---------|
| NFR-USA-001 | Giao diện web responsive (desktop ưu tiên cho AP; mobile cho approver/requester). Hỗ trợ Chrome, Edge, Safari, Firefox (2 phiên bản mới nhất). |
| NFR-USA-002 | Người dùng AP mới có thể xử lý invoice sau ≤ 2 giờ đào tạo. |
| NFR-USA-003 | Tuân thủ **WCAG 2.1 AA**. |
| NFR-USA-004 | Đa ngôn ngữ UI (Việt/Anh), định dạng số/ngày/tiền theo locale. |
| NFR-USA-005 | In-app guidance, tooltips, help center, onboarding checklist cho admin. |

## 7. Deployment & Operations

| ID | Yêu cầu |
|----|---------|
| NFR-OPS-001 | Mô hình triển khai: **SaaS multi-tenant** (mặc định), **Dedicated cloud** (single-tenant), **On-premise / private cloud** (Kubernetes) cho ngân hàng/doanh nghiệp nhà nước. |
| NFR-OPS-002 | Kiến trúc containerized (Kubernetes), Infrastructure as Code, CI/CD với blue-green/canary deployment, zero-downtime release. |
| NFR-OPS-003 | Môi trường: Dev, QA, UAT/Sandbox (cho khách hàng), Production. |
| NFR-OPS-004 | **Observability**: centralized logging, metrics, distributed tracing (OpenTelemetry), alerting; dashboard cho queue depth, latency, error rate, AI cost. |
| NFR-OPS-005 | Release notes & thông báo thay đổi API trước ≥ 90 ngày với breaking change; hỗ trợ API version cũ ≥ 12 tháng. |

## 8. Maintainability & Extensibility

| ID | Yêu cầu |
|----|---------|
| NFR-MNT-001 | Kiến trúc module/microservices theo domain (ingestion, extraction, validation, matching, workflow, integration, repository, analytics). |
| NFR-MNT-002 | Cấu hình nghiệp vụ (rules, workflow, schema, mapping) không cần deploy code. |
| NFR-MNT-003 | Code coverage unit test ≥ 70% cho core domain; regression test tự động cho extraction (golden dataset). |
| NFR-MNT-004 | Plugin/connector SDK có tài liệu để đối tác phát triển connector mới. |

## 9. Support & SLA dịch vụ

| Severity | Mô tả | Thời gian phản hồi | Thời gian khắc phục mục tiêu |
|----------|-------|--------------------|------------------------------|
| Sev 1 | Hệ thống ngừng hoạt động / mất dữ liệu / sự cố bảo mật | 30 phút (24/7) | 4 giờ |
| Sev 2 | Chức năng chính lỗi, có workaround hạn chế | 2 giờ làm việc | 1 ngày làm việc |
| Sev 3 | Lỗi chức năng phụ | 1 ngày làm việc | 5 ngày làm việc |
| Sev 4 | Câu hỏi / yêu cầu cải tiến | 2 ngày làm việc | Theo roadmap |

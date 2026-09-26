# 02. Functional Requirements

Mức ưu tiên: **M** = Must (MVP) · **S** = Should · **C** = Could · **W** = Won't (lần này)

## Tổng quan module

| Module | Mã | Mô tả |
|--------|----|-------|
| Document Ingestion | ING | Thu thập chứng từ đa kênh |
| Pre-processing | PRE | Làm sạch, tách/gộp, chống trùng file |
| Classification | CLS | Phân loại loại chứng từ |
| Extraction | EXT | OCR + AI trích xuất dữ liệu (chi tiết tại [03](03-ai-ocr-capabilities.md)) |
| Validation & Enrichment | VAL | Kiểm tra business rules, thuế, master data |
| Matching | MAT | Đối chiếu PO/GRN/Invoice/Contract |
| Review Workspace | REV | Màn hình HITL cho người dùng |
| Purchase Order Management | PO | Quản lý PO lifecycle |
| Invoice & Billing Management | INV | Quản lý invoice, billing, credit note, recurring billing |
| Contract Management | CON | Quản lý hợp đồng & price compliance |
| Vendor Management & Portal | VEN | Vendor master, onboarding, portal |
| Workflow & Approval | WF | Chi tiết tại [04](04-workflow-automation.md) |
| Payment Preparation | PAY | Payment proposal, lịch thanh toán |
| AI Assistant & Insights | AIA | Chat, summary, anomaly detection (chi tiết tại [03](03-ai-ocr-capabilities.md)) |
| Search & Repository | REP | Lưu trữ, tìm kiếm, retention |
| Reporting & Analytics | RPT | Dashboard, báo cáo |
| Notification | NOT | Thông báo đa kênh |
| Administration | ADM | Cấu hình, người dùng, phân quyền, audit |

---

## 1. Document Ingestion (ING)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-ING-001 | Hệ thống phải nhận chứng từ từ **email inbox chuyên dụng** (ví dụ `ap@company.com`, `invoice@...`) qua IMAP/Microsoft Graph/Gmail API, tự động tách attachments (PDF, ảnh, XML, ZIP, Excel, Word) và lưu email body làm context. | M |
| FR-ING-002 | Người dùng có thể **upload** chứng từ qua web (drag & drop, bulk upload tối thiểu 200 file/lần, tối đa 50 MB/file). | M |
| FR-ING-003 | Hỗ trợ **mobile capture** (iOS/Android/PWA): chụp ảnh, tự động phát hiện biên, crop, chỉnh phối cảnh, multi-page. | S |
| FR-ING-004 | Nhận **hóa đơn điện tử XML** (theo định dạng chuẩn của cơ quan Thuế VN) cùng file PDF hiển thị; tự động liên kết XML ↔ PDF của cùng một hóa đơn. | M |
| FR-ING-005 | Đồng bộ tự động hóa đơn đầu vào từ **cổng hóa đơn điện tử của cơ quan Thuế** và/hoặc **nhà cung cấp dịch vụ HĐĐT** (theo MST người mua) theo lịch. | S |
| FR-ING-006 | Cung cấp **REST API** để hệ thống khác đẩy chứng từ (multipart/base64/URL) kèm metadata. | M |
| FR-ING-007 | Hỗ trợ **SFTP / shared folder watcher** (SharePoint, OneDrive, Google Drive, S3) để quét thư mục và lấy file mới. | S |
| FR-ING-008 | Nhận chứng từ từ **Vendor Portal** (xem VEN). | S |
| FR-ING-009 | Hỗ trợ **scanner** (TWAIN / network scanner đẩy qua email/folder), barcode/QR separator sheet để tách lô. | C |
| FR-ING-010 | Hỗ trợ **EDI / PEPPOL / UBL / cXML** cho khách hàng có vendor quốc tế. | C |
| FR-ING-011 | Mỗi chứng từ nhận vào được gán **Document ID** duy nhất, lưu nguồn (channel), người gửi, thời điểm, hash (SHA-256) và trạng thái `RECEIVED`. | M |
| FR-ING-012 | Email từ người gửi không nằm trong whitelist/vendor master được đánh dấu **untrusted sender** và đưa vào quarantine để review. | S |
| FR-ING-013 | Tự động gửi **email phản hồi** xác nhận đã nhận (cấu hình bật/tắt, template) cho người gửi. | C |
| FR-ING-014 | Quét virus/malware tất cả file đầu vào trước khi xử lý. | M |

## 2. Pre-processing (PRE)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-PRE-001 | Image enhancement: deskew, denoise, xoay trang tự động, tăng tương phản, loại bỏ nền, chuẩn hóa DPI (≥ 300). | M |
| FR-PRE-002 | Phát hiện PDF native (có text layer) vs. scanned để chọn pipeline tối ưu. | M |
| FR-PRE-003 | **Document splitting**: tự động tách 1 file nhiều chứng từ (ví dụ 1 PDF chứa 5 invoice) thành các chứng từ riêng; cho phép người dùng tách/gộp thủ công. | M |
| FR-PRE-004 | **Attachment bundling**: gộp invoice + phụ lục/bảng kê + delivery note thuộc cùng một giao dịch thành một "document package". | S |
| FR-PRE-005 | **File-level duplicate detection** bằng hash và perceptual hash (ảnh chụp lại của cùng chứng từ). | M |
| FR-PRE-006 | Phát hiện ngôn ngữ chính của chứng từ. | M |
| FR-PRE-007 | Phát hiện chất lượng ảnh kém (mờ, thiếu trang, bị cắt) và yêu cầu người gửi gửi lại. | S |

## 3. Classification (CLS)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-CLS-001 | Tự động phân loại chứng từ vào các document type (Invoice, PO, GRN, Credit Note, Contract, Quotation, Statement, Receipt, Other) với confidence score. | M |
| FR-CLS-002 | Nhận diện **vendor** từ logo, MST, tên, địa chỉ, email người gửi, số tài khoản. | M |
| FR-CLS-003 | Chứng từ có classification confidence < ngưỡng cấu hình → chuyển queue review. | M |
| FR-CLS-004 | Business Admin có thể thêm document type mới và huấn luyện/cấu hình bằng ít mẫu (few-shot, ≤ 10 mẫu). | S |
| FR-CLS-005 | Phân loại chứng từ không liên quan (quảng cáo, thư chào hàng, newsletter) và loại bỏ khỏi luồng xử lý. | S |

## 4. Extraction (EXT)

Chi tiết kỹ thuật tại [03-ai-ocr-capabilities.md](03-ai-ocr-capabilities.md).

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-EXT-001 | Trích xuất **header fields** theo schema của từng document type (xem [07-data-model](07-data-model.md)). | M |
| FR-EXT-002 | Trích xuất **line items / tables**, bao gồm bảng nhiều trang, bảng không có đường kẻ, dòng gộp (merged cells), dòng chiết khấu, dòng thuế. | M |
| FR-EXT-003 | Hỗ trợ **template-free extraction** (không cần tạo template cho từng vendor). | M |
| FR-EXT-004 | Hỗ trợ đa ngôn ngữ: Tiếng Việt (có dấu), English (M); 中文, 日本語, 한국어, ไทย (S). | M |
| FR-EXT-005 | Chuẩn hóa dữ liệu: ngày (dd/MM/yyyy ↔ ISO), số tiền (dấu phân cách `.`/`,`), đơn vị tiền tệ, đơn vị tính (UoM), MST, số tài khoản. | M |
| FR-EXT-006 | Mỗi trường trả về **confidence score** và **bounding box / page reference** để highlight trên ảnh gốc. | M |
| FR-EXT-007 | Với HĐĐT XML: parse trực tiếp XML (không OCR), đối chiếu với PDF hiển thị để phát hiện sai khác. | M |
| FR-EXT-008 | Nhận dạng **chữ viết tay** (ICR) cho chữ ký, số lượng thực nhận trên delivery note, ghi chú. | S |
| FR-EXT-009 | Nhận diện **con dấu, chữ ký** (có/không có) trên chứng từ giấy. | S |
| FR-EXT-010 | Trích xuất **contract clauses** (payment terms, penalty, auto-renewal, termination, price list, SLA) từ hợp đồng. | S |
| FR-EXT-011 | Học từ hiệu chỉnh của người dùng (**feedback loop**) để cải thiện độ chính xác cho vendor/layout đó. | S |
| FR-EXT-012 | Business Admin có thể định nghĩa **custom fields** (tên, kiểu dữ liệu, mô tả ngữ nghĩa, validation) mà không cần code. | S |

## 5. Validation & Enrichment (VAL)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-VAL-001 | **Mathematical validation**: qty × unit price = amount; tổng line = subtotal; subtotal + VAT − discount = total; VAT = subtotal × thuế suất (theo từng thuế suất 0%, 5%, 8%, 10%, KCT, KKKNT). | M |
| FR-VAL-002 | **Mandatory field check** theo document type & loại vendor (trong nước/nước ngoài). | M |
| FR-VAL-003 | **Vendor validation**: vendor tồn tại trong vendor master, trạng thái active, MST khớp; gợi ý tạo vendor mới nếu chưa có. | M |
| FR-VAL-004 | **Tax ID validation**: tra cứu MST người bán (trạng thái hoạt động/ngừng hoạt động/bỏ địa chỉ kinh doanh) qua dịch vụ tra cứu của cơ quan Thuế hoặc nhà cung cấp dữ liệu. | M |
| FR-VAL-005 | **e-Invoice validation**: kiểm tra cấu trúc XML theo XSD, chữ ký số (certificate chain, thời hạn, thu hồi), mã CQT, trạng thái hóa đơn trên cổng tra cứu (hợp lệ/bị thay thế/bị điều chỉnh/bị hủy). Lưu kết quả kiểm tra làm bằng chứng. | M |
| FR-VAL-006 | **Buyer validation**: thông tin người mua (tên, MST, địa chỉ) trên invoice khớp với legal entity của khách hàng. | M |
| FR-VAL-007 | **Duplicate invoice detection**: exact (vendor + số HĐ + ký hiệu) và fuzzy (số tiền, ngày gần nhau, số HĐ tương tự do lỗi OCR). | M |
| FR-VAL-008 | **Bank account validation**: số tài khoản trên invoice khớp tài khoản đã đăng ký của vendor; khác → cảnh báo nghiêm trọng (fraud risk). | M |
| FR-VAL-009 | **GL / Cost center / Tax code suggestion** dựa trên lịch sử, PO, vendor, mô tả line item (AI). | S |
| FR-VAL-010 | **Currency conversion** theo tỷ giá ngày hóa đơn (nguồn tỷ giá cấu hình được: ERP, ngân hàng, API). | S |
| FR-VAL-011 | **Budget check**: kiểm tra ngân sách còn lại của cost center/project (qua ERP/budget system). | C |
| FR-VAL-012 | Rule engine cho phép Business Admin cấu hình **custom validation rules** (no-code, biểu thức điều kiện) với mức độ: Error (block), Warning, Info. | M |
| FR-VAL-013 | Kiểm tra **ngưỡng thanh toán không dùng tiền mặt** (BRL-08) và cảnh báo. | S |

## 6. Matching (MAT)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-MAT-001 | **2-way matching**: Invoice ↔ PO (vendor, line items, qty, unit price, amount, currency, tax). | M |
| FR-MAT-002 | **3-way matching**: PO ↔ GRN ↔ Invoice (qty invoiced ≤ qty received − qty đã invoice trước đó). | M |
| FR-MAT-003 | **4-way matching**: bổ sung Inspection/QC report. | C |
| FR-MAT-004 | **Line-level matching** bằng AI khi mô tả hàng hóa trên invoice khác với PO (semantic similarity, mã hàng vendor ↔ mã hàng nội bộ), hỗ trợ many-to-many (1 invoice nhiều PO, 1 PO nhiều invoice — partial invoicing). | M |
| FR-MAT-005 | **Tolerance** cấu hình theo: % và/hoặc giá trị tuyệt đối; theo vendor, category, legal entity; riêng cho price, qty, amount, tax. | M |
| FR-MAT-006 | Phát hiện PO reference tự động từ invoice (số PO trên chứng từ, hoặc suy luận từ vendor + line items + thời gian khi thiếu số PO). | M |
| FR-MAT-007 | **Contract price compliance**: đối chiếu đơn giá invoice với price list trong contract/framework agreement. | S |
| FR-MAT-008 | Kết quả matching: `MATCHED`, `MATCHED_WITHIN_TOLERANCE`, `PARTIAL`, `EXCEPTION` (với exception code: PRICE_VAR, QTY_VAR, NO_PO, NO_GRN, PO_CLOSED, VENDOR_MISMATCH, OVER_INVOICED, TAX_VAR…). | M |
| FR-MAT-009 | Giao diện matching trực quan: bảng so sánh line-by-line PO/GRN/Invoice, highlight chênh lệch, cho phép manual match/unmatch. | M |
| FR-MAT-010 | **Statement reconciliation**: đối chiếu Statement of Account của vendor với công nợ trong hệ thống, liệt kê invoice thiếu/thừa. | S |
| FR-MAT-011 | **Credit note matching**: tự động liên kết credit/debit note với invoice gốc và cập nhật số tiền phải trả. | M |

## 7. Review Workspace (REV)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-REV-001 | Màn hình **split-view**: ảnh chứng từ gốc (zoom, rotate, multi-page) bên trái; form dữ liệu trích xuất bên phải; click field → highlight vùng trên ảnh và ngược lại. | M |
| FR-REV-002 | Tô màu field theo confidence (xanh/vàng/đỏ); chế độ "chỉ xem field cần review". | M |
| FR-REV-003 | Chỉnh sửa line items dạng bảng (thêm/xóa/tách/gộp dòng), copy từ PO. | M |
| FR-REV-004 | Hiển thị danh sách **validation errors/warnings** & matching exceptions với gợi ý hành động (AI-suggested resolution). | M |
| FR-REV-005 | **Keyboard-first** thao tác (Tab, shortcut approve/next) để tối ưu năng suất. | S |
| FR-REV-006 | Queue management: filter/sort theo SLA, vendor, amount, exception type, assignee; auto-assignment (round-robin, skill-based, vendor-based). | M |
| FR-REV-007 | Lock chứng từ khi đang được người khác chỉnh sửa; lịch sử thay đổi từng field (before/after, ai sửa, lúc nào). | M |
| FR-REV-008 | Comment, @mention, đính kèm, yêu cầu thông tin bổ sung từ Buyer/Requester/Vendor ngay trên chứng từ. | M |
| FR-REV-009 | Actions: Save, Submit, Reject (với reason code), Put on hold, Reassign, Split, Mark as duplicate, Request vendor correction. | M |

## 8. Purchase Order Management (PO)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-PO-001 | Đồng bộ PO (header + lines + status) từ ERP theo near real-time hoặc lịch (≤ 15 phút). | M |
| FR-PO-002 | Trích xuất PO từ file (PDF/Excel) cho doanh nghiệp chưa có PO trong ERP hoặc PO do vendor gửi lại (order confirmation). | S |
| FR-PO-003 | Theo dõi **PO lifecycle**: Open → Acknowledged → Partially Received → Received → Partially Invoiced → Fully Invoiced → Closed; tính "remaining to receive / to invoice". | M |
| FR-PO-004 | **Order confirmation matching**: so sánh order confirmation của vendor với PO (giá, số lượng, ngày giao) và cảnh báo sai lệch. | S |
| FR-PO-005 | Cảnh báo PO sắp hết hạn giao hàng, giao trễ, PO mở quá lâu không có hoạt động. | S |
| FR-PO-006 | **Quotation comparison**: trích xuất nhiều quotation cho cùng yêu cầu và tạo bảng so sánh (giá, điều khoản, thời gian giao, validity), AI tóm tắt điểm khác biệt. | S |
| FR-PO-007 | Ghi nhận **GRN / service entry** trực tiếp trên DI (cho khách hàng không nhập GRN trên ERP), có mobile confirm cho Requester. | S |
| FR-PO-008 | Tạo **PO draft** từ Purchase Requisition hoặc quotation được chọn và đẩy sang ERP. | C |

## 9. Invoice & Billing Management (INV)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-INV-001 | Quản lý invoice với các trạng thái (xem [07-data-model](07-data-model.md#3-document-lifecycle)). | M |
| FR-INV-002 | Hỗ trợ PO invoice, non-PO invoice, prepayment/advance invoice, recurring invoice (utilities, rent, subscriptions), freight/landed cost invoice. | M |
| FR-INV-003 | **Recurring billing / subscription tracking**: định nghĩa lịch billing kỳ vọng (vendor, số tiền, kỳ); cảnh báo khi thiếu invoice, khi số tiền biến động bất thường so với kỳ trước. | S |
| FR-INV-004 | **Milestone / progress billing** cho hợp đồng dịch vụ/xây dựng: theo dõi % hoàn thành, giá trị đã billing, retention money. | C |
| FR-INV-005 | Xử lý **hóa đơn điều chỉnh / thay thế** (HĐĐT): liên kết với hóa đơn gốc, cập nhật trạng thái hóa đơn gốc. | M |
| FR-INV-006 | Tính **due date** theo payment terms (Net 30, 2/10 Net 30, EOM…) và gợi ý early payment discount. | M |
| FR-INV-007 | **Accrual support**: xuất danh sách GRN chưa có invoice (GRNI) và invoice chưa post cuối kỳ phục vụ đóng sổ. | S |
| FR-INV-008 | Hỗ trợ **multi-currency**, **multi-entity** (một vendor giao dịch với nhiều pháp nhân), **withholding tax** (thuế nhà thầu nước ngoài — FCT) tính toán/gợi ý. | S |
| FR-INV-009 | **Dispute management**: mở dispute với vendor (short delivery, sai giá, hàng lỗi), theo dõi đến khi nhận credit note. | S |

## 10. Contract Management (CON)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-CON-001 | Lưu trữ và trích xuất metadata hợp đồng (bên, giá trị, hiệu lực, payment terms, price list, auto-renewal). | S |
| FR-CON-002 | Theo dõi **consumption**: tổng giá trị PO/invoice đã phát sinh so với giá trị hợp đồng; cảnh báo ở ngưỡng 80%/100%. | S |
| FR-CON-003 | Nhắc hạn hợp đồng sắp hết hiệu lực / gia hạn tự động (30/60/90 ngày). | S |
| FR-CON-004 | AI tóm tắt hợp đồng, trích xuất obligations & risk clauses, hỏi đáp trên hợp đồng. | C |

## 11. Vendor Management & Portal (VEN)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-VEN-001 | Đồng bộ **vendor master** từ ERP (tên, MST, địa chỉ, bank accounts, payment terms, status, contacts). | M |
| FR-VEN-002 | **Vendor Portal** (self-service, đăng nhập bằng email OTP/SSO): nộp invoice (upload/PO flip), xem trạng thái xử lý & thanh toán, tải remittance advice. | S |
| FR-VEN-003 | **PO flip**: vendor chọn PO/GRN và tạo invoice với dữ liệu điền sẵn, đính kèm hóa đơn chính thức (XML/PDF). | S |
| FR-VEN-004 | **Vendor onboarding**: form đăng ký, upload giấy phép ĐKKD, xác minh MST, xác minh tài khoản ngân hàng, workflow phê duyệt vendor mới. | C |
| FR-VEN-005 | Vendor tự cập nhật thông tin; thay đổi bank account yêu cầu xác minh 4-eyes (BRL-06) và thông báo cho vendor qua kênh đã đăng ký trước đó. | S |
| FR-VEN-006 | **Vendor scorecard**: tỷ lệ invoice lỗi, giao hàng đúng hạn, tỷ lệ exception, thời gian phản hồi dispute. | C |
| FR-VEN-007 | Thông báo tự động cho vendor khi invoice bị reject / cần bổ sung với lý do rõ ràng. | S |
| FR-VEN-008 | Portal đa ngôn ngữ (Việt, Anh). | S |

## 12. Payment Preparation (PAY)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-PAY-001 | Tạo **payment proposal** từ các invoice đã approve theo due date, vendor, currency, ưu tiên early payment discount. | S |
| FR-PAY-002 | Workflow duyệt payment proposal (tách biệt với duyệt invoice). | S |
| FR-PAY-003 | Xuất file thanh toán theo định dạng ngân hàng (bulk payment file) hoặc đẩy sang ERP/Host-to-Host banking. | C |
| FR-PAY-004 | Nhận kết quả thanh toán (paid/failed) từ ERP/ngân hàng, cập nhật trạng thái invoice và gửi remittance advice cho vendor. | S |
| FR-PAY-005 | Cash flow forecast: dự báo nhu cầu chi theo due date. | C |

## 13. AI Assistant & Insights (AIA)

Chi tiết tại [03-ai-ocr-capabilities.md](03-ai-ocr-capabilities.md#5-ai-assistant--copilot).

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-AIA-001 | **Chat with documents**: hỏi đáp bằng ngôn ngữ tự nhiên (Việt/Anh) trên một chứng từ hoặc tập chứng từ, trả lời kèm trích dẫn nguồn (document + page). | S |
| FR-AIA-002 | **Natural language query** trên dữ liệu: "Tổng tiền invoice của vendor ABC quý 3 chưa thanh toán?", "Top 10 vendor theo spend năm nay". Kết quả tôn trọng phân quyền dữ liệu. | S |
| FR-AIA-003 | **Anomaly & fraud detection**: bất thường về giá, số tiền, tần suất, vendor mới có giao dịch lớn, bank account thay đổi, invoice ngày nghỉ, số hóa đơn liên tiếp bất thường, split invoice để né ngưỡng duyệt. | S |
| FR-AIA-004 | **Exception resolution suggestion**: đề xuất nguyên nhân và hành động cho exception (ví dụ: "Chênh lệch giá do PO chưa cập nhật giá theo phụ lục hợp đồng số X"). | S |
| FR-AIA-005 | **Draft communication**: soạn email gửi vendor yêu cầu bổ sung/điều chỉnh hóa đơn dựa trên lỗi phát hiện. | S |
| FR-AIA-006 | **Summaries**: tóm tắt contract, quotation comparison, tình trạng công nợ theo vendor. | C |
| FR-AIA-007 | **Agentic automation** (có kiểm soát): AI agent thực hiện chuỗi thao tác (thu thập thông tin thiếu, liên hệ requester xác nhận GRN, đề xuất coding) trong giới hạn quyền được cấp, mọi hành động có log và có thể yêu cầu người duyệt. | C |

## 14. Search & Repository (REP)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-REP-001 | Lưu trữ file gốc (immutable), file đã xử lý (searchable PDF/A), dữ liệu trích xuất, lịch sử xử lý. | M |
| FR-REP-002 | **Full-text search** (có hỗ trợ tiếng Việt có dấu/không dấu) + **faceted filter** (vendor, loại, ngày, số tiền, trạng thái, entity, PO). | M |
| FR-REP-003 | **Semantic search** (vector search) theo ngữ nghĩa. | S |
| FR-REP-004 | Liên kết chứng từ (document graph): PO ↔ GRN ↔ Invoice ↔ Credit Note ↔ Payment ↔ Contract, xem toàn bộ "transaction folder". | M |
| FR-REP-005 | **Retention policy** cấu hình theo document type (mặc định 10 năm cho chứng từ kế toán), **legal hold**, không cho phép xóa trước hạn; xóa theo quy trình có phê duyệt sau hạn. | M |
| FR-REP-006 | Versioning khi chứng từ được thay thế/điều chỉnh. | M |
| FR-REP-007 | Export gói chứng từ cho audit/thanh tra thuế (ZIP kèm index Excel). | S |

## 15. Reporting & Analytics (RPT)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-RPT-001 | **Operational dashboard**: volume theo kênh/loại/trạng thái, backlog, SLA breach, throughput theo user. | M |
| FR-RPT-002 | **Automation KPIs**: STP rate, field accuracy, correction rate theo field/vendor, avg handling time, exception rate theo loại. | M |
| FR-RPT-003 | **AP reports**: AP aging, invoice due/overdue, early-payment discount captured/missed, GRNI, accrual. | S |
| FR-RPT-004 | **Spend analytics**: spend theo vendor, category, cost center, entity, thời gian; price variance trend; maverick spend (non-PO). | S |
| FR-RPT-005 | **Vendor performance** report. | C |
| FR-RPT-006 | Báo cáo kiểm soát: duplicate bị chặn, fraud alerts, SoD violation attempts, thay đổi bank account. | S |
| FR-RPT-007 | Export Excel/CSV/PDF, lịch gửi báo cáo tự động qua email; kết nối BI (Power BI, Tableau, Looker) qua data export/API. | S |
| FR-RPT-008 | Custom report builder (kéo thả) cho Business Admin. | C |

## 16. Notification (NOT)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-NOT-001 | Thông báo in-app, email; (S) Microsoft Teams, Slack, Zalo OA, mobile push. | M |
| FR-NOT-002 | Thông báo theo sự kiện: task được giao, sắp/đã quá SLA, bị reject, cần thông tin, fraud alert, integration failure. | M |
| FR-NOT-003 | Digest (tổng hợp hằng ngày) thay cho thông báo từng sự kiện — người dùng tự cấu hình. | S |
| FR-NOT-004 | **Actionable notifications**: approve/reject trực tiếp từ email/Teams/Slack (có xác thực). | S |
| FR-NOT-005 | Template thông báo đa ngôn ngữ, cấu hình được. | M |

## 17. Administration (ADM)

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FR-ADM-001 | Quản lý **multi-tenant** (SaaS) và **multi-entity** (nhiều pháp nhân/chi nhánh trong một tenant) với cấu hình riêng (tax, currency, workflow, ERP). | M |
| FR-ADM-002 | Quản lý user, group, role; **RBAC** + **data-level permission** (theo entity, cost center, vendor, document type, amount). | M |
| FR-ADM-003 | **SSO** (SAML 2.0, OIDC — Azure AD/Entra ID, Google Workspace, Okta), MFA; SCIM user provisioning (S). | M |
| FR-ADM-004 | Cấu hình document types, field schemas, validation rules, tolerance, approval matrix, workflow (no-code). | M |
| FR-ADM-005 | Quản lý master data cục bộ/đồng bộ: vendor, GL account, cost center, tax code, project, currency, UoM, item mapping. | M |
| FR-ADM-006 | **Audit log** bất biến cho mọi thao tác (login, view, edit, approve, config change, export, API call) — tìm kiếm & export. | M |
| FR-ADM-007 | Quản lý **integration** (connection, credentials vault, mapping, lịch sync, retry, logs). | M |
| FR-ADM-008 | **Configuration promotion** giữa môi trường (Sandbox → UAT → Production) qua export/import có version. | S |
| FR-ADM-009 | Cấu hình đa ngôn ngữ UI (Việt, Anh — M; Nhật, Hàn, Trung — C), múi giờ, định dạng số/ngày. | M |
| FR-ADM-010 | Quản lý **SoD rules** và cảnh báo xung đột khi gán role. | S |
| FR-ADM-011 | Quản lý usage/quota (số trang xử lý, AI credits) và billing cho tenant (SaaS). | S |

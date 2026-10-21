# Hệ thống vận hành doanh nghiệp 1 người, quản lý bằng AI

> Mục tiêu: một người làm chủ, AI làm "bộ máy văn phòng": kế toán thu chi, lập dự toán, nhắc việc/thông báo, chuẩn bị và ghi biên bản họp. Chủ chỉ **ra quyết định và duyệt**.

---

## 1. Nguyên tắc thiết kế

1. **Một nguồn dữ liệu duy nhất** – mọi con số nằm ở 1 file Google Sheets (sổ cái). AI đọc/ghi vào đó, không để số liệu rải rác ở Zalo, sổ tay.
2. **AI làm, người duyệt** – AI được tự ghi chép, phân loại, nhắc việc, soạn nháp. Chi tiền, ký hợp đồng, gửi email cho đối tác → **chủ phải bấm duyệt**.
3. **Tự động từ nguồn, không nhập tay** – giao dịch ngân hàng tự đổ vào sổ qua webhook; hóa đơn chụp ảnh là AI đọc.
4. **Nhịp cố định** – ngày / tuần / tháng / quý, mỗi nhịp có một bản báo cáo AI tự gửi.
5. **Bắt đầu nhỏ** – chạy được bằng công cụ có sẵn (Google + Claude) trước, chỉ viết phần mềm riêng khi thật sự cần.

---

## 2. Sơ đồ tổng thể

```
        ┌──────────────── NGUỒN DỮ LIỆU ────────────────┐
        │ Ngân hàng (SePay/Casso webhook) · Ảnh hóa đơn │
        │ Gmail · Zalo/Telegram · Lịch · Hợp đồng (Drive)│
        └───────────────────────┬───────────────────────┘
                                ▼
                 ┌──────────────────────────────┐
                 │   SỔ CÁI – Google Sheets      │
                 │ Thu chi · Dự toán · Công nợ   │
                 │ Việc cần làm · Họp · Hợp đồng │
                 └──────────────┬───────────────┘
                                ▼
                 ┌──────────────────────────────┐
                 │     AI ĐIỀU HÀNH (Claude)      │
                 │ đọc sổ · phân loại · báo cáo   │
                 │ nhắc việc · soạn họp · cảnh báo│
                 └──────┬───────────────┬────────┘
                        ▼               ▼
            Telegram/Zalo/Email     Chủ doanh nghiệp
            (thông báo, báo cáo)    (duyệt / ra quyết định)
```

---

## 3. Bốn phân hệ chính

### 3.1 Thu chi (kế toán hằng ngày)

| Việc | Cách làm | AI làm gì |
|---|---|---|
| Ghi nhận tiền vào/ra | Tài khoản ngân hàng nối **SePay** hoặc **Casso** → webhook → Google Sheets (Apps Script hoặc Make/n8n) | Tự phân loại danh mục, gắn dự án/khách hàng |
| Chi tiền mặt, hóa đơn giấy | Chụp ảnh gửi bot Telegram/Zalo | Đọc ảnh, tách số tiền – ngày – người bán – VAT, ghi vào sổ |
| Hóa đơn điện tử | Hóa đơn về Gmail | Đọc email, lấy XML/PDF, lưu Drive, ghi sổ |
| Công nợ | Tab Công nợ phải thu / phải trả | Nhắc khách quá hạn, soạn tin đòi nợ lịch sự |
| Đối soát | Cuối ngày | So số dư sổ với số dư ngân hàng, báo chênh lệch |

**Tách bạch tiền cá nhân và tiền doanh nghiệp**: tối thiểu 2 tài khoản ngân hàng (1 kinh doanh, 1 cá nhân). Đây là điều kiện để AI làm sổ chính xác.

### 3.2 Dự toán & dòng tiền

- **Dự toán năm → chia tháng** theo danh mục (doanh thu, giá vốn, marketing, lương/thuê ngoài, thuế, lãi vay…).
- **So sánh Thực tế vs Dự toán** mỗi tuần; vượt >10% là AI cảnh báo.
- **Dự báo dòng tiền 13 tuần** (chuẩn quản trị cho doanh nghiệp nhỏ): tiền hiện có + thu dự kiến − chi dự kiến → tuần nào âm thì báo trước.
- **Quỹ dự phòng**: AI theo dõi số tháng chi phí cố định mà tiền mặt còn chịu được (mục tiêu ≥ 6 tháng).
- **Dự toán dự án/thương vụ** (ví dụ BĐS): vốn, chi phí giao dịch, thuế phí, lãi vay, kịch bản giá bán tốt/vừa/xấu, điểm hòa vốn, ROI.

### 3.3 Thông báo & nhắc việc

| Loại | Kênh | Ví dụ |
|---|---|---|
| Tức thời | Telegram (dễ làm bot nhất) hoặc Zalo OA | "Vừa nhận 50.000.000đ từ Nguyễn Văn A – đã ghi vào HĐ-012" |
| Hạn chót | Telegram + Google Calendar | Hạn nộp thuế, gia hạn hợp đồng, trả lãi vay, khách đến hạn thanh toán |
| Cảnh báo | Telegram, ưu tiên cao | Chi vượt dự toán, số dư dưới ngưỡng, giao dịch lạ |
| Báo cáo định kỳ | Email | Báo cáo sáng, tuần, tháng (xem mục 5) |

### 3.4 Họp bàn (với đối tác, khách, thuê ngoài – và "họp với chính mình")

- **Trước họp**: AI đọc lịch, lấy email/hợp đồng/công nợ liên quan → soạn 1 trang chuẩn bị (mục tiêu, số liệu, câu hỏi cần chốt).
- **Trong họp**: ghi âm (Google Meet có ghi chép, hoặc ghi âm điện thoại) → AI chép lời.
- **Sau họp**: AI tạo biên bản: quyết định, việc cần làm, người phụ trách, hạn chót → đẩy vào tab Việc cần làm + Lịch; soạn email tóm tắt gửi đối tác (chủ duyệt rồi gửi).
- **Họp với chính mình (hội đồng quản trị 1 người)**: mỗi tuần 30 phút, AI đóng vai CFO/COO phản biện: số liệu tuần, rủi ro, 3 ưu tiên tuần tới.

---

## 4. Bộ công cụ đề xuất (3 cấp độ)

### Cấp 1 – Không cần lập trình (chạy được trong 1–2 tuần)
| Thành phần | Công cụ | Chi phí ước tính |
|---|---|---|
| Sổ cái | Google Sheets (mẫu ở thư mục `mau-du-lieu/`) | Miễn phí |
| Lưu chứng từ | Google Drive | Miễn phí / ~45k/tháng |
| Lịch, email | Google Calendar, Gmail | Miễn phí |
| AI điều hành | Claude (gói Pro/Max) có kết nối Gmail, Calendar, Drive + tác vụ định kỳ (scheduled tasks) | ~20–100 USD/tháng |
| Đồng bộ ngân hàng | SePay hoặc Casso → Google Sheets | Gói nhỏ vài chục–vài trăm nghìn/tháng |

→ Chủ nhắn cho Claude: *"Tuần này chi bao nhiêu cho marketing, so với dự toán?"*, *"Soạn biên bản cuộc họp hôm qua"*. Claude đã đọc được Gmail/Lịch/Drive.

### Cấp 2 – Tự động hóa (low-code)
- **n8n** (tự host, rẻ) hoặc **Make**: nhận webhook ngân hàng, bot Telegram nhận ảnh hóa đơn → gọi Claude API đọc → ghi Sheets → báo Telegram.
- Tác vụ định kỳ: 7h sáng báo cáo, 17h đối soát, thứ 6 họp tuần.

### Cấp 3 – Hệ thống riêng (khi quy mô lớn)
- Cơ sở dữ liệu (Supabase/PostgreSQL) thay Sheets, giao diện quản trị riêng.
- Agent dùng Claude API + công cụ (tool use) đọc/ghi DB, quyền hạn rõ ràng, nhật ký mọi hành động.
- Kết nối phần mềm kế toán/hóa đơn điện tử (MISA, Viettel, VNPT…) qua API.

**Khuyến nghị: làm Cấp 1 ngay, Cấp 2 sau 1 tháng khi đã quen nhịp.**

---

## 5. Nhịp vận hành do AI chạy

| Nhịp | Thời điểm | AI gửi gì |
|---|---|---|
| Hằng ngày | 7:00 | Báo cáo sáng: số dư, thu/chi hôm qua, lịch hôm nay, việc đến hạn, khách cần nhắc nợ |
| Hằng ngày | 17:00 | Đối soát sổ – ngân hàng, giao dịch chưa phân loại cần chủ xác nhận |
| Hằng tuần | Thứ 6 | Họp tuần với chính mình: thực tế vs dự toán, dòng tiền 13 tuần, 3 ưu tiên tuần sau |
| Hằng tháng | Ngày 1–3 | Báo cáo kết quả kinh doanh (lãi/lỗ), dòng tiền, công nợ; nhắc chuẩn bị số liệu thuế |
| Hằng quý | Đầu quý | Rà soát dự toán, chiến lược, kế hoạch quý; nhắc hạn kê khai thuế quý |
| Hằng năm | Tháng 12–3 | Lập dự toán năm mới, quyết toán thuế |

---

## 6. Phân quyền cho AI (rất quan trọng)

| AI **được tự làm** | AI **phải hỏi chủ** | AI **không bao giờ làm** |
|---|---|---|
| Ghi sổ, phân loại, đối soát | Gửi email/tin nhắn cho bên ngoài | Chuyển tiền, thanh toán |
| Tạo báo cáo, dự báo | Tạo/sửa sự kiện lịch có người khác | Ký, xác nhận hợp đồng |
| Soạn nháp email, biên bản | Thay đổi dự toán | Xóa dữ liệu sổ cái |
| Nhắc việc cho chủ | Đánh dấu công nợ đã thu khi không có giao dịch khớp | Chia sẻ tài liệu tài chính ra ngoài |

Quy tắc này được ghi thành văn bản trong `CLAUDE.md` để AI luôn đọc và tuân theo.

---

## 7. Lưu ý pháp lý – thuế (Việt Nam)

- Từ **01/01/2026**, hộ kinh doanh **bỏ thuế khoán**, chuyển sang **tự kê khai** theo doanh thu thực tế → sổ thu chi chính xác là bắt buộc, không chỉ là "quản trị cho đẹp".
- Hộ/doanh nghiệp có doanh thu lớn phải dùng **hóa đơn điện tử** (có loại khởi tạo từ máy tính tiền).
- Nếu là **công ty TNHH 1 thành viên**: vẫn cần kế toán (có thể thuê dịch vụ ngoài làm báo cáo tài chính, quyết toán). AI làm sổ quản trị hằng ngày, kế toán dịch vụ chỉ làm phần nộp cơ quan thuế → giảm chi phí.
- AI **không thay** tư vấn thuế/pháp lý chính thức; các mốc thuế cần xác nhận lại với kế toán/cơ quan thuế vì quy định thay đổi thường xuyên.

*(Mục này cần kiểm tra lại với văn bản hiện hành trước khi áp dụng.)*

---

## 8. Rủi ro & cách phòng

| Rủi ro | Phòng ngừa |
|---|---|
| AI phân loại sai | Giao dịch độ tin cậy thấp → đưa vào hàng chờ chủ xác nhận; đối soát hằng ngày |
| Lộ dữ liệu tài chính | Sổ cái chỉ chia sẻ cho tài khoản chủ; bật xác thực 2 lớp Google; không dán số tài khoản/mật khẩu vào chat |
| Phụ thuộc 1 nhà cung cấp | Dữ liệu ở Google Sheets (xuất được Excel bất cứ lúc nào); quy trình viết thành văn bản trong repo này |
| AI "bịa" số | Mọi con số trong báo cáo phải trích từ sổ; AI ghi rõ nguồn (tab, dòng) |
| Quên dùng hệ thống | Nhịp cố định + báo cáo tự đến; chủ chỉ cần trả lời/duyệt |

---

## 9. Lộ trình triển khai 30 ngày

| Tuần | Việc |
|---|---|
| 1 | Mở tài khoản ngân hàng kinh doanh riêng · Tạo Google Sheets từ mẫu · Chốt danh mục thu chi · Nhập số dư đầu kỳ |
| 2 | Nối SePay/Casso → Sheets · Lập dự toán 12 tháng · Bật kết nối Gmail/Calendar/Drive cho Claude |
| 3 | Tạo tác vụ định kỳ: báo cáo sáng, đối soát chiều, họp tuần · Tạo bot Telegram nhận ảnh hóa đơn |
| 4 | Chạy thử toàn bộ · Sửa danh mục, ngưỡng cảnh báo · Viết lại quy tắc trong `CLAUDE.md` theo thực tế |

**Thước đo thành công sau 30 ngày**: chủ dành < 15 phút/ngày cho sổ sách; 100% giao dịch được ghi; biết chính xác lãi/lỗ và dòng tiền 13 tuần bất cứ lúc nào.

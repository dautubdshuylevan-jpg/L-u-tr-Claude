# Mẫu dữ liệu sổ cái

Mỗi file CSV = 1 tab trong Google Sheets "Sổ cái doanh nghiệp". Nhập vào Google Sheets: *Tệp → Nhập → Tải lên → Chèn trang tính mới*.

| File | Tab | Dùng để |
|---|---|---|
| `thu-chi.csv` | Thu chi | Mọi giao dịch tiền vào/ra (ngân hàng tự đổ vào) |
| `du-toan.csv` | Dự toán | Dự toán 12 tháng theo danh mục |
| `cong-no.csv` | Công nợ | Phải thu / phải trả, hạn thanh toán |
| `viec-can-lam.csv` | Việc cần làm | Việc phát sinh từ họp, email, hạn chót |
| `hop.csv` | Họp | Nhật ký họp, quyết định, link biên bản |

Nên thêm tab **Báo cáo** dùng công thức `SUMIFS` để tính Thực tế vs Dự toán theo tháng, và tab **Dòng tiền 13 tuần**.

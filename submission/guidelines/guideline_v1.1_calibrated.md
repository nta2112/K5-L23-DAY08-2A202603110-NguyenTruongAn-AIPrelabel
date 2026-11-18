# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603110 - Nguyễn Trường An

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Người ngồi trên xe máy / xe ba bánh có gộp vào box xe không? | Không gộp: người ngồi trên xe gán box `person` riêng, xe gán box riêng; khi chạy SAM theo điểm chỉ click vào phần thân/mái xe. | `img_013.jpg` | `CAL-05` |
| 2 | Cụm phương tiện bị nhòe chuyển động mạnh vào ban đêm xử lý thế nào? | Nếu nhìn rõ hình dáng từng xe thì gán box bình thường; nếu vệt nhòe dính chùm không tách được từng chiếc thì khoanh một vùng `ignore_region`. | `img_024.jpg` | `CAL-04` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_006.jpg` | Xe máy dựng sát mép trái bị cắt một phần và có chó ngồi trên yên sau | `YOLO26m` vẽ box `motorbike` trùm cả con chó và kéo dư khoảng trống | Cắt biên box đúng mép ảnh, chỉ bao khung xe máy, loại bỏ thú cưng khỏi box phương tiện | Mục 4.1 (Biên hộp & cắt mép) |
| `img_023.jpg` | Xe tuk-tuk đỗ nối đuôi nhau, có gắn cờ `can_review` | `YOLOE-26s` sinh box `three_wheeler` phình rộng lên khối bê tông phía trên | Kéo siết chặt 4 cạnh box ôm sát mái và bánh xe tuk-tuk, gỡ cờ `can_review` sau khi sửa | Mục 4.1 (Độ ôm sát biên) |
| `img_024.jpg` | Xe buýt xanh cỡ lớn đang chạy ở giao lộ đêm | `YOLOE-26s` nhận nhầm thành `three_wheeler` do ngưỡng thấp (0.15) | Chuyển nhãn sang `bus`; giới hạn `three_wheeler` chỉ áp dụng cho xe 3 bánh, tuk-tuk, xích lô | Mục 3 (Phân biệt lớp phương tiện) |
| `img_024.jpg` | Dòng xe máy tạo vệt sáng nhòe chuyển động ở giao lộ ban đêm | `YOLO26m` bỏ sót hoặc bắt sai vị trí một số xe trong vùng nhòe | Dùng `ignore_region` bao cụm xe nhòe không thể đếm chính xác từng chiếc | Mục 4.3 (Vùng loại trừ `ignore_region`) |
| `img_013.jpg` | Tách mask xe tuk-tuk có người ngồi bên trong | `SAM 2.1` gộp cả người lái/khách vào mask xe khi điểm `--pos` đặt gần người | Chỉ đặt điểm `--pos` trên phần vỏ/mái xe và đặt `--neg` lên người nếu bị dính mask | Mục 4.2 (Tách mask & người lái) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Siết quy tắc biên ôm sát box (`motorbike`, `three_wheeler`), tách rõ `bus`/`car` khỏi `three_wheeler`, bổ sung quy tắc `ignore_region` cho cảnh nhòe đêm và quy tắc đặt điểm SAM tránh gộp người ngồi trên xe. | Rút ra từ lỗi máy và lỗi duyệt trên 5 ảnh QC (`img_006.jpg`, `img_013.jpg`, `img_023.jpg`, `img_024.jpg`) — `CAL-01` đến `CAL-05`. |
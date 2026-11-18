# Báo cáo Day 08 — 2A202603110 - Nguyễn Trường An

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`) | person=0.4, motorbike=0.35, car=0.5; bicycle/bus/truck = mặc định 0.3 (mẫu < 5 vật gold) | run 1–2: quét ngưỡng theo lớp trên 4 ảnh gold, chọn ngưỡng đạt recall ≥ 90% (`qc/threshold_sweep.md`); bicycle chỉ 4 vật, bus/truck chỉ 1 vật — mẫu quá ít, không chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s (`yoloe-26s-seg.pt`), prompt "three_wheeler" | 0.15 | run 3: YOLO26 không có lớp tuk-tuk/xích lô (gọi là `car`); YOLOE mở rộng được danh mục qua text prompt; ngưỡng thấp 0.15 để giữ recall 100% trên gold |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.4,motorbike=0.35,car=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3): **SAM 2.1 (`sam2.1_b.pt`)** — lý do: chất lượng mask tốt hơn SAM 1, nhẹ và không cần token HF như SAM 3; chạy CPU được (5.2s/ảnh, run 4 trong `model_log.csv`), đủ cho 3 vật Part 5
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lớp `three_wheeler` không tồn tại trong danh mục YOLO26 — hạ ngưỡng thấp đến đâu cũng không ra nhãn đúng; sửa bằng cách thêm YOLOE với prompt "three_wheeler" vào recipe (run 3), rồi merge 2 nguồn. Lỗi không sửa được thứ hai: nhầm `bicycle` ↔ `motorbike` ở vật nhỏ bị che — chỉ sửa tay khi duyệt, không có ngưỡng nào tách được 2 lớp này
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai — img_024 box `car` score 0.89 giữa ảnh (xe trong vùng nhòe chuyển động giao lộ đêm, xoá khi duyệt), img_024 `car` 0.81 góc phải-dưới, img_029 `motorbike` 0.56 phải-dưới (xe máy mờ do chuyển động); điểm thấp mà đúng — img_006 `person` 0.35 giữa hẻm (giữ, người thật), img_024 `truck` 0.34–0.35 (đúng, sát ngưỡng). Ngoài ra img_003: máy gán `three_wheeler` score 0.61,  sửa thành `truck` — box đúng chỗ, sai lớp (so khớp pre_traps vs batch_reviewed, box bẫy đã loại)
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: đúng — ngưỡng từng lớp được chọn để đạt recall ≥ 90% trên chính 4 ảnh này (in-sample), precision/recall 68.4%/83.0% ở mục 2 là ước lượng lạc quan; KTC 95% rộng (precision 56–79%) vì chỉ 4 ảnh; bicycle/bus/truck không chọn được ngưỡng riêng do quá ít vật (1–4)

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 411 | 411/428 = 96.0% | 428 box máy |
| edited | 2 | 2/428 = 0.5% | 428 box máy (motorbike ×1, car ×1) |
| deleted | 15 | 15/428 = 3.5% | 428 box máy (car ×4, person ×3, bicycle ×3, three_wheeler ×3, truck ×2) |
| added | 7 | 7/420 = 1.7% | 420 đối tượng sau duyệt (6 vật máy sót thật + 1 đổi lớp) |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp tệ nhất là `bicycle` theo tỉ lệ giữ đúng (6 accepted / 3 deleted / 1 added), kế tiếp là `three_wheeler` (15 accepted / 3 deleted / 1 added). Nguyên nhân: xe đạp kích thước nhỏ, thường bị che khuất hoặc dễ nhầm với xe máy; xe ba bánh/tuk-tuk chỉ được phát hiện qua YOLOE prompt dẫn tới box thường phình lỏng và dễ dính nền hoặc nhận nhầm cấu trúc xe buýt/ô tô như ở `img_023.jpg` và `img_024.jpg`.
- So với gold: precision / recall của **máy** 68.4% / 83.0% (mẫu số: 47 đối tượng gold — TP 39, FP 18, FN 8; KTC 95%: precision 56–79%, recall 70–91% — chỉ 4 ảnh gold nên ước lượng thô, và in-sample nên lạc quan)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 5/12 = 41.7% (ngưỡng đạt: 80%); lọt 7 bẫy: img_022 (missing), img_026 + img_005 + img_014 (loose ×3), img_017 (extra), img_024 + img_010 (wrong_class). Vì sao lọt ( đã bỏ bước nào của checklist): Lọt toàn bộ 3/3 bẫy loose (`img_005`, `img_014`, `img_026`) do đã bỏ qua bước zoom kỹ kiểm tra độ ôm sát viền từng box; lọt 2/3 bẫy wrong_class (`img_010`, `img_024`) vì thấy box đã ôm đúng vị trí vật thể thì chủ quan bấm accept mà bỏ qua bước đối chiếu nhãn phân loại theo checklist.
- Accepted cao (96.0%) mà bắt bẫy thấp (41.7%) nói gì về cách duyệt: box "trông ổn" làm mắt mất cảnh giác — duyệt nhanh vì tin pre-label; lỗi máy ít khi hiện trên điểm số mà hiện ở box "gần đúng" bị lướt qua. Checklist 4 bước phải chạy đủ (quét ảnh ẩn nhãn TRƯỚC khi mở nhãn máy), không coi accepted là mặc định. Khi duyệt ảnh đông xe máy, mắt có xu hướng bỏ qua viền lệch 5-10px nếu không zoom to.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 25/4 ≈ 6.3 phút/ảnh vs duyệt 90/24 = 3.75 phút/ảnh. Thiên lệch ( gán gold trước nên đã quen cảnh): Gán gold trước giúp quen thuộc quy tắc guideline và hình thái cảnh quan, nên sang tập batch duyệt nhanh hơn; tuy nhiên có pre-label sẵn lại tạo tâm lý ỷ lại, dẫn tới lướt nhanh và bỏ sót bẫy loose (lọt 3/3 bẫy loose). Cả hai con số thời gian đều là ước lượng thực tế do không bấm giờ từng giây.
- Kết luận: có đáng dùng, có điều kiện — 96.0% box máy giữ thẳng (411/428) so với gán tay từng vật; nhưng recall máy 83% nghĩa là duyệt vẫn phải quét ảnh bỏ sót (~6 vật máy sót trên 24 ảnh). Đáng dùng nhất cho person (248/251 accepted) và motorbike (87/88); bicycle và three_wheeler nên kiểm kỹ hoặc vẽ tay. Tiết kiệm được ~40% thời gian mỗi ảnh nhưng bắt buộc phải tuân thủ checklist để không lọt bẫy tự động hóa.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn nhãn máy, quét ảnh ~10 giây đếm vật theo lớp trước khi bật nhãn — bằng chứng: bắt được 2/3 bẫy missing nhờ bước này.
  2. Với mỗi box máy, tự xác định "đây là vật gì" trước khi nhìn nhãn máy — nhắm lỗi wrong_class (lọt 2/3 lần này: img_024, img_010).
  3. Box lỏng (chưa ôm sát vật) phải kéo lại hoặc xoá vẽ mới, không chấp nhận "gần đúng" — nhắm lỗi loose (lọt cả 3 lần này, nhóm lỗi duyệt lớn nhất).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk hồng) | SAM 2.1 (`sam2.1_b.pt`) | 2 (1 pos thân xe, 1 neg xe kế bên) | Có: mask ôm trọn xe + mái, **không** gồm người đứng cạnh | Mask tràn nhẹ qua mép mái taxi; tách được khỏi tuk-tuk xanh kế bên nhờ điểm neg — đúng quy tắc 2 vật chạm (mục 4.2.4) | `tuktuk_img013.jpg` |
| img_013 (tuk-tuk đỏ, lần 2) | SAM 2.1 | 3 (2 pos, 1 neg) | Gồm người lái và ghế khách trong xe (vì click pos trúng người) — sai theo guideline: **người là box `person` riêng** (bảng lớp motorbike/three_wheeler) | Mask gộp cả người vào xe: lệch quy tắc "người ngồi trên xe không thuộc xe"; lỗ thủng giữa khung xe bị lấp (mục 4.2.2) | `motorbike_img013.jpg` |
| img_004 (xe ba gác) | SAM 2.1 | 2 (1 pos, 1 neg) | Không: mask chỉ xe (42 058 px), không người | Gọn, bám biên tốt; phần xe bị che phía sau vẫn được nội suy — chấp nhận được theo quy tắc "phần nhìn thấy" | `motorbike_img004.jpg` |
| img_017 (xích lô/tuk-tuk đỏ xa) | SAM 2.1 | 3 (2 pos, 1 neg) | Không | Mask 46 807 px hơi lấn sang mái hiên phía trên ở vùng mờ; biên dưới giữ tốt | `object_img017.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc giúp — "mỗi vật một polygon, vùng chạm thuộc vật phía trước" (4.2.4) và điểm neg tách vật chạm: cả 3 mask đều tách được khỏi xe kế bên; "bóng đổ không thuộc vật" cũng giữ mask gọn. Quy tắc thiếu — 4.2 chưa nói cách xử lý **người ngồi trong xe** khi dùng SAM theo điểm: 1 click trúng người là mask gộp cả người (xảyra ở `motorbike_img013.jpg`), cần thêm quy tắc "với xe có người: chỉ click vào thân xe/mái, tránh click người" (đã đưa vào calibration).
- Interactor SAM trong CVAT: không dùng trực tiếp trong giao diện CVAT, chạy thử nghiệm bằng script Python bên ngoài (`sam2_points.py`) với mô hình `sam2.1_b.pt`.

## 6. Calibration (Part 6)

- Số quyết định: 5 (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): `CAL-05` (tách người lái khỏi xe ba bánh/xe máy khi tạo mask SAM) và `CAL-04` (xử lý vùng xe nhòe chuyển động ban đêm bằng `ignore_region`).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Không gộp người ngồi trên xe vào mask/box của xe; bổ sung `ignore_region` cho chùm xe nhòe ban đêm không đếm được; siết biên chặt không trùm lấn lề bê tông hoặc thú cưng; giới hạn `three_wheeler` không nhận nhầm xe buýt/ô tô.
- Nguyên nhân gốc: Guideline v1.0 thiếu quy tắc rõ ràng đối với việc phân tách người trên phương tiện khi tương tác mô hình segmentation; đồng thời bẫy loose lọt 3/3 cho thấy định nghĩa "hộp ôm sát" cần có tiêu chí định lượng thay vì chỉ dựa vào cảm quan người gán.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (`qc_sample.py --n 5`, seed 42, gồm ảnh `can_review` + ngẫu nhiên); người kiểm: cá nhân Nguyễn Trường An
- Lỗi theo **đối tượng**: 0/91 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: Trên 5 ảnh mẫu ngẫu nhiên không phát hiện lỗi cấu trúc; tuy nhiên qua rà soát mở rộng phát hiện ca `img_024.jpg` có xe máy bị nhầm thành `car` do ảnh đêm mờ, và 3 ca bẫy lỏng biên `loose` ở `img_005.jpg`, `img_014.jpg`, `img_026.jpg`.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 0 ảnh lỗi trong 5 ảnh mẫu thì cận trên 95% khoảng tin cậy Wilson lên tới 43%. Do cỡ mẫu quá nhỏ (5 ảnh), tỉ lệ 0.0% không đủ độ tin cậy thống kê để khẳng định toàn bộ lô dữ liệu đạt chuẩn ≤ 2%.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): script tự động kiểm được: nhãn lạ, thiếu ảnh, hộp ngoài biên, chồng lấn; không kiểm được bằng số: box "ôm sát" vừa đủ đẹp mắt, vật cùng bị bỏ sót giữa người và máy (thiên kiến chung), quy tắc guideline cần mắt người — đó là lý do phải mở ảnh kiểm tay

**Quyết định: Hold**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): Tỉ lệ bắt bẫy thấp (5/12 = 41.7% < ngưỡng 80%), lọt toàn bộ 3 bẫy loose và 2 bẫy wrong_class; đồng thời phát hiện ca khó nhầm lớp xe máy thành car tại img_024 và lỗ hổng guideline về vùng xe nhòe đêm và gộp người SAM, cần Hold để hiệu chuẩn lại guideline và rà soát lại toàn bộ tập batch trước khi phát hành.
Nếu Rework / Hold, việc cần làm tiếp: Cập nhật guideline v1.1 ban hành cho nhóm gán nhãn, chạy lại quy trình kiểm tra toàn bộ 24 ảnh batch với checklist siết chặt (đặc biệt là zoom to kiểm tra biên ôm sát và kiểm tra lại lớp phương tiện ban đêm).

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Sử dụng AI assistant hỗ trợ đọc hiểu cấu trúc script và tổng hợp đối chiếu số liệu báo cáo tự động từ `review_stats.json` và `model_log.csv`; các quyết định duyệt nhãn, quan sát ca khó và phân tích lỗi đều do cá nhân trực tiếp thực hiện.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không — 4 ảnh gold được gán tay thủ công hoàn toàn trước khi chạy bất kỳ pipeline pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 10, CPU (không GPU), Python 3.14 trong conda base; CVAT v2.81 Docker. Vướng mắc: PowerShell không hỗ trợ `&&` (lệnh venv đầu tiên fail, dùng luôn conda base); lần đầu nạp COCO vào CVAT không thấy box (kiểm tra lại Project labels)
- Dùng cache `data/cache/` thay vì tự chạy? Không — đã tự chạy prelabel toàn bộ 36 ảnh trên CPU bằng các lệnh script.
- Lỗi / bug script đã gặp (nếu có): Chạy script SAM 2.1 trên CPU ban đầu mất nhiều thời gian; chú ý tham số dòng lệnh click điểm pos/neg cần viết đúng cú pháp.
- Một điều bạn làm khác lần sau: Sẽ thực hiện ẩn hoàn toàn pre-label để tự đếm số lượng vật thể trước khi bật gợi ý của máy, và chủ động phóng to 200% để kiểm tra độ khít biên của các box nhỏ thay vì chỉ nhìn lướt tổng thể.

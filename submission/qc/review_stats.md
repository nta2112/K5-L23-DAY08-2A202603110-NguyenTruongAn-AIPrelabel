# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 428; đối tượng sau duyệt: 420; ngưỡng accepted: IoU >= 0.9
- Đã **loại 28 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 411 | 411/428 = 96.0% | 428 box máy |
| edited (sửa) | 2 | 2/428 = 0.5% | 428 box máy |
| deleted (xoá) | 15 | 15/428 = 3.5% | 428 box máy |
| added (vẽ thêm) | 7 | 7/420 = 1.7% | 420 đối tượng sau duyệt |

Trong số deleted + added, **1 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 6.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 248 | 0 | 3 | 2 |
| bicycle | 6 | 0 | 3 | 1 |
| motorbike | 87 | 1 | 0 | 2 |
| car | 47 | 1 | 4 | 0 |
| bus | 3 | 0 | 0 | 0 |
| truck | 5 | 0 | 2 | 1 |
| three_wheeler | 15 | 0 | 3 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 47 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 39 | 18 | 8 | 68.4% (KTC 95%: 56–79%) | 83.0% (KTC 95%: 70–91%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 5/12 = 41.7% (KTC 95%: 19–68%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - loose: 0/3
  - extra: 2/3
  - wrong_class: 1/3
  - lọt: img_022.jpg (missing), img_026.jpg (loose), img_017.jpg (extra), img_005.jpg (loose), img_014.jpg (loose), img_024.jpg (wrong_class), img_010.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).

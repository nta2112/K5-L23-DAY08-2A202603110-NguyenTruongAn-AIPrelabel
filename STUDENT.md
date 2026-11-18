# Thông tin học viên

- Họ và tên: Nguyễn Trường An
- MSHV: 2A202603110
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: https://github.com/nta2112
- Repo bài nộp (URL): https://github.com/nta2112/K5-L23-DAY08-2A202603110-NguyenTruongAn-AIPrelabel
- Commit chốt bài (hash): 
- Máy: WinDow10, không có
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 411/428 = 96.0% / 2/428 = 0.5% / 15/428 = 3.5% (mẫu số: 428 box máy); added: 7/420 = 1.7% (mẫu số: 420 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 68.4% / 83.0% (mẫu số: 47 đối tượng gold — TP 39, FP 18, FN 8)
- Bắt bẫy: 5/12 = 41.7%
- Thời gian: gán tay 6.3 phút/ảnh (25 phút/4 ảnh); duyệt có pre-label 3.75 phút/ảnh (90 phút/24 ảnh)
- Lỗi QC: 0.0% theo ảnh (0/5 ảnh), 0.0% theo đối tượng (0/91 đối tượng)
- **Quyết định:** Hold — lý do một dòng: Tỉ lệ bắt bẫy thấp (41.7% < 80%) và phát hiện ca nhầm lớp xe máy thành car tại img_024 cùng lỗ hổng guideline về vùng xe nhòe đêm và gộp người SAM, cần giữ lại hiệu chuẩn guideline và kiểm tra lại chất lượng duyệt.

# Báo cáo Day 14 — Pham Huu Minh — 2A202603330

- Mã học viên dùng với `--student`: `330`
- `seed_tag` trong `frames_A.txt`: `82ac3100fd`

Tập A: `s01f30`, `s02f02`, `s04f16`.
Tập B: `s02f16`, `s02f33`, `s02f55`, `s03f02`, `s03f24`, `s03f54`, `s04f05`, `s04f31`, `s04f58`, `s05f01`, `s05f29`, `s05f46`.

## 1. Dự đoán trước sweep (bước 5b)

Ghi **trước** khi chạy `scripts/pp_sweep.py`, rồi commit và push ngay (lịch sử commit là bằng chứng thời điểm viết). Không sửa phần này sau khi có kết quả.

- Tham số tôi nghĩ giảm công sửa nhiều nhất: `score_thr`, nâng từ 0,1 lên khoảng 0,4–0,5.
  `min_points` tôi nghĩ gần như không giúp được gì.

- Vì sao (nhìn ảnh BEV tập A: box thừa nhiều hay box sót nhiều?): Rõ ràng là thừa, không phải sót.
  Model vẽ 86 hộp trên ba frame trong khi tôi chỉ nhận ra 32 vật thật, và số đo ở bước 5 là 55 hộp
  phải xóa nhưng chỉ 1 vật bị bỏ sót. Tiền công nằm gần hết ở việc xóa: 0,3 × 55 là 16,5 trong tổng
  18,0.

  Mở ảnh camera ra xem thì các hộp thừa không rải đều. Chúng bám vào hàng rào cây, thân cây, cột
  đèn và lan can ven đường, tức là vật cố định có cụm điểm to xấp xỉ một chiếc xe con. Những hộp đó
  đều có điểm tin cậy thấp: nhóm tôi giữ lại có score trung vị 0,88, nhóm phải xóa chỉ 0,38. Hai
  nhóm tách nhau khá rõ nên nâng ngưỡng sẽ cắt trúng nhóm sau.

  Tôi không trông chờ vào `min_points` vì hàng rào cây rất dày điểm. Hộp thừa vẫn có trung vị 77
  điểm LiDAR trong hộp, nên lọc theo số điểm chỉ dọn được vài hộp gần như rỗng chứ không đụng tới
  phần lớn hộp bám cây.

  Chỗ phải cân nhắc là công thức phạt lệch nhau: xóa chỉ 0,3 mà thêm tới 1,0. Mỗi vật thật bị
  ngưỡng cắt nhầm phải đổi bằng hơn ba hộp thừa mới hòa vốn. Hộp thật thấp điểm nhất của tôi là
  0,37, nên từ 0,4 trở lên là bắt đầu có rủi ro mất nó, và đó là lý do tôi đoán điểm tốt nhất rơi
  vào quãng 0,4–0,5 chứ không phải càng cao càng tốt.

  Một điểm nữa tôi thấy khi rà nhãn: trong 22 hộp Cyclist model vẽ ra thì chỉ 1 hộp là người đi xe
  thật, 21 hộp còn lại là cây và cột bị nhận nhầm, vì chúng cao và hẹp giống dáng xe đạp. Nếu được
  đặt ngưỡng riêng theo lớp thì `thr_cyc` sẽ là chỗ đáng nâng nhất, nhưng lưới lần này chỉ quét
  `score_thr` chung nên tôi dự đoán theo tham số chung.

  Lưu ý kỹ thuật: hộp thấp điểm nhất trong `pred_A.json` là 0,221, nên `score_thr` 0,1 và 0,2 cho
  ra kết quả y hệt nhau trên ba frame của tôi. Chỉ từ 0,3 trở lên lưới mới thật sự thay đổi gì.

## 2. Tập A: sửa tay và tune

| Cấu hình | Xóa | Thêm | Sửa | Chi phí |
|---|---:|---:|---:|---:|
| Gốc | 55 | 1 | 1 | 18.0 |
| `best_config.json` | | | | |

Số của cấu hình gốc đo trên 3 frame tập A: GT 32 hộp, model dự đoán 86 hộp, ghép được 31 cặp.
Theo từng frame: `s01f30` xóa 29, thêm 0, sửa 0 (chi phí 8.7); `s02f02` xóa 20, thêm 1, sửa 0
(chi phí 7.0); `s04f16` xóa 6, thêm 0, sửa 1 (chi phí 2.3).

- Tham số đã đổi trong `best_config.json`: ...
- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra): ...
- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó: ...

## 3. Trước/sau (bước 8)

![so sánh](../../d14_330/compare/<frame>.jpg)

- `scripts/pp_compare.py` báo giảm (hoặc tăng) ...% công sửa trên tập A.
- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa: ...

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_330/frames_B.txt --config d14_330/best_config.json --gt d14_330/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| | | | |

- Loại box tốn công sửa nhất ở tập B: ...

## 5. Giới hạn

- Chỉ có 3 frame để tune, mà ba frame lại khác nhau quá nhiều. `s01f30` là đại lộ hai bên dày đặc
  hàng rào cây, `s02f02` có xe buýt chạy sát bên trái, `s04f16` là đường nhiều làn thoáng. Cấu
  hình hợp với frame này rất dễ hỏng ở frame kia. Phải đọc leave-one-out rồi mới tin số "tốt nhất".
- Model học trên KITTI, chỉ biết Car, Cyclist và Pedestrian. Truck với Bus thì nó không sinh ra
  được: chiếc buýt ở `s02f02` bị cắt thành tám hộp Car rời nhau, còn hộp Bus trong nhãn tham chiếu
  là người tự thêm.
- Hộp thừa gần như đều nằm trên hàng rào cây, thân cây, cột và lan can. Cụm điểm của chúng to xấp
  xỉ một chiếc xe con, nên nhìn số đo rất khó loại; phải mở ảnh camera mới biết.
- Chi phí sửa chỉ là công thức `0.3·xóa + 1.0·thêm + 0.5·sửa`, không phải thời gian thật, và chỉ
  tính hộp có tâm trong 50 m.
- `ref_A.json` là đáp án do chính tôi dựng, rồi sweep lại dò tham số theo đúng đáp án đó. Tôi nhận
  nhầm chỗ nào thì cấu hình được chọn sẽ hợp với cái nhầm đó, và tập B thừa hưởng luôn.

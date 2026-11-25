# Báo cáo Day 13 — Đinh Đức Hiếu, 2A202603327

## 1. Kết luận trước
Mã số `2A202603327`, task `D13-2A202603327`, 10 frame: s01f25, s01f57, s02f25, s02f40, s03f13, s03f47, s04f28, s04f53, s05f00, s05f17. Windows 11, trình duyệt Edge + v158.0.4370.33, CVAT 2.81.0 (chạy trên máy, Docker), tài khoản `hieudd3`. Bạn ngồi cạnh: 2A202603186.

Tổng **122 cuboid** (98 Car, 12 Cyclist, 7 Pedestrian, 4 Truck, 1 Bus). Ít nhất là s02f40 và s05f00 (8 cuboid mỗi frame): s02f40 phần lớn vật nằm xa 20–37 m, thưa điểm; s05f00 có xe tải lớn ngay phía sau (~10 m) che một phần cảnh nên sau nó còn ít vật đủ cơ sở để vẽ. `selfcheck` cờ **4/122 cuboid** (2 lơ lửng, 3 quá chật; s03f13 idx11 bị cả hai). Tin nhất: **hướng** của xe con (đa số yaw quanh 270° và 90°, khớp làn đường) và **đáy** (118/122 cuboid có `ground_gap_m` trong ±0,12 m). Yếu nhất: **vật xa, thưa điểm** (xe 30–50 m chỉ 25–110 điểm, kích thước phần lớn phải suy từ bảng 4.3, ví dụ 4,50 × 1,82 × 1,50 m) và **class/kích thước người, xe hai bánh**.

Cách dùng CVAT: (1) đổi góc nhìn trong khung 3D: kéo chuột trái để xoay tới khi nhìn thẳng từ trên xuống, kéo chuột phải để dời, lăn chuột để phóng, nháy đúp chỗ trống để về mặc định. (2) Vẽ: nút cuboid → chọn label → **Shape** → đưa hộp tới cụm điểm và nháy đúp; chỉnh tâm, dài, rộng, yaw ở **Top**, chỉnh đáy và chiều cao ở **Side/Front** (không xoay ở hai khung này). (3) Bật **Appearance → Cuboid orientation** để thấy vạch đỏ `+X` của hộp, đối chiếu ảnh camera để biết đầu xe; xem hộp chiếu lên 7 ảnh camera bằng `python scripts/lab.py preview` (ảnh trong `work/xem/`).

## 2. Đọc frame (Bước 2)
Ảnh: `t1-s01f25-3d`, `t1-s01f25-top`, `t1-s01f57-3d`, `t1-s01f57-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f25 | Phía trước (−y) ở **cạnh trên**; +x sang **trái** ảnh (bên trái xe), +y xuống **cạnh dưới** (phía sau) | Gần xe phẳng, thấp hơn LiDAR ~1,7 m (z đáy xe gần −1,66 đến −1,76 m); về phía trước-trái mặt đường thấp dần, ở ~25 m còn ~−2,15 m (dốc xuống nhẹ, ~0,45 m) | Xe con phía sau ~35 m (idx16) | Giao lộ: xe con, xe hai bánh có người, người đi bộ | Nhóm xe hai bánh ở **sau-phải** ~12–15 m (idx4, 7, 8), hai người đi bộ bên **phải** ~13–19 m, xe con phía sau 18–35 m | Cụm thấp, rời rạc ở góc dưới-trái ảnh top (gần vạch tròn): không rõ là bụi cây, rào hay xe hai bánh dựng, chưa gán |
| s01f57 | Phía trước (−y) ở **cạnh trên**; +x sang **trái**, +y xuống **dưới** | Phẳng, thấp hơn LiDAR ~1,8 m (z đáy −1,78 đến −1,89 m) | Xe con phía trước ~37 m (idx14) | Đường thẳng có hàng xe đỗ: xe con, 1 người đi bộ, 1 xe hai bánh | Xe con sát bên **trái** ~3,5 m (idx1), xe con ngay **sau** ~7 m (idx2) và ~32 m (idx10); người đi bộ, xe hai bánh ở **trước-phải** 15–27 m | Cụm ở trước-trái ~23 m nằm cao hơn mặt đường ~0,8 m (sau gán thành idx8, bị cờ lơ lửng): chưa chắc là xe trên chỗ đỗ cao hơn hay vật khác |

Chỉ **camera** cho biết (từ hai frame trên): (1) màu và kiểu thân xe, đủ để tách xe con với bán tải/xe tải; (2) đâu là đầu, đâu là đuôi xe (đèn pha, đèn hậu) để đặt mũi tên yaw; (3) cụm điểm dọc mép đường là người đang lái xe hai bánh hay người đi bộ/xe dựng. Chỉ **LiDAR** cho biết: (1) khoảng cách và kích thước thật bằng mét (ví dụ xe idx1 s01f25 cách 6,80 m); (2) vật ở phía sau và hai bên, nơi cam01 không thấy và các camera có góc mù; (3) độ cao mặt đường và độ dốc (mặt đường trước-trái s01f25 thấp hơn ~0,45 m).

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f25 | Cạnh trên | Nhiều xe ngược chiều (yaw ~85–91°) ở làn trái; xe xa nhất ~45 m, chỉ ~60 điểm |
| s02f40 | Cạnh trên | Đa số vật xa 20–37 m, thưa; một người đi bộ ~28 m chỉ 12 điểm |
| s03f13 | Cạnh trên | Có xe tải ~16 m phía trước; người đi bộ ~27 m có đáy lơ lửng 0,47 m theo `selfcheck` |
| s03f47 | Cạnh trên | Hai xe hai bánh rất gần hai bên (~10 m); xe ~38 m thưa, đáy z ≈ −0,95 m |
| s04f28 | Cạnh trên | Có xe buýt ~38 m phía trước-trái; mặt đường phía trước-trái thấp hơn (~−2,1 m) |
| s04f53 | Cạnh trên | Vật xa nhất ~49 m, sát ranh 50 m |
| s05f00 | Cạnh trên | Xe tải lớn ngay phía sau ~10 m, che vùng phía sau |
| s05f17 | Cạnh trên | Xe tải dài ~10 m bên trái, sau nó có vùng gần như không có điểm |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f25, idx1 (6,80 m) | Car | Có, yaw 274,0° (cùng chiều, phía trước) | Có, z đáy −1,71 m, gap −0,00 m | Có | Dài 4,34 m, cao 1,48 m trong khoảng sedan; rộng 2,00 m hơi rộng hơn sedan (1,70–1,90 m) |
| (b) xe buýt hoặc xe tải | s04f28, idx13 (38,03 m) — s01f25, s01f57 không có xe buýt/xe tải | Bus | Có, yaw 89,0° (ngược chiều) | Có, z đáy −1,47 m, gap 0,12 m | Có | 10,76 × 2,55 × 3,20 m, trong khoảng xe buýt thường |
| (c) người đi bộ hoặc xe hai bánh | s01f25, idx6 (12,69 m) | Pedestrian | Có, yaw 270,0° theo hướng đi | Có, z đáy −1,56 m, gap 0,07 m | Có | 0,90 × 0,81 × 1,70 m, trong khoảng người lớn (dài ở mép trên 0,9 m) |

Vòng hiệu chuẩn: tôi không ghi lại kết quả so sánh với cả lớp nên không nêu được mình lệch ở tâm, kích thước hay yaw. Những cách hiểu tôi áp dụng khi vẽ, theo guideline: vạch đỏ `+X` chỉ về đầu xe (xem đèn xe trên camera); đáy hộp chạm mặt đường, chỉnh ở Side/Front; vật bị che vẫn vẽ trọn phần suy luận được và bật occluded; người trên xe máy là Cyclist.

Vật bỏ qua vì không đủ cơ sở: Cụm thấp góc dưới-trái ảnh top s01f25 — UNCERTAIN_CLASS (bụi cây, rào hay xe dựng); các cụm vài điểm ở xa > 40 m trước-sau s02f40 — LOW_VISIBILITY.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Tất cả lấy trong 10 frame của tôi.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s05f17, xe tải idx2 (~12,5 m, bên trái), `t4-1-bong-che` | Giả thuyết: thân xe tải dài ~10 m chắn tia laser, nên vùng phía sau nó không được quét | Mặt bên xe tải rất dày điểm (6382 điểm), ngay phía sau là vùng gần như trống, trong khi hai bên vùng đó vẫn có điểm mặt đường | Vùng trống là **chưa quan sát được**, không phải đường trống. Vật lộ một phần sau xe tải (ví dụ xe idx7 ~27 m) bật **occluded**, cuboid bao phần suy luận được |
| 2. Thưa ở xa | s03f47, xe con ~38 m (idx8), `t4-2-thua-o-xa` | Giả thuyết: tia laser toả ra, ở xa khoảng cách giữa các tia lớn nên ít tia chạm vật | Chỉ 25 điểm rải trên đáy/một mặt, không thấy rõ hình hộp | Quy tắc 15–20% thể tích: chỉ vẽ vì ảnh camera cho thấy xe; kích thước suy từ bảng 4.3 (4,50 × 1,82 × 1,50 m). Nếu không đủ cơ sở thì bỏ qua và ghi **LOW_VISIBILITY** |
| 3. Phản xạ thấp / ít điểm bất thường | s01f57, xe con ~20–23 m phía trước (idx7: 65 điểm, idx13: 62 điểm), `t4-3-phan-xa-thap` | Giả thuyết: sơn tối màu hoặc nhiều kính hấp thụ/phản xạ lệch tia; cũng có thể do bị xe phía trước che một phần — chưa khẳng định | Chỉ thấy viền một mặt xe, trong khi xe cùng tầm ở frame khác có nhiều hơn hẳn (s02f25 idx3, 20,0 m: 410 điểm; s04f28 idx7, 23,6 m: 250 điểm) | Ghi là giả thuyết, không khẳng định; vẫn vẽ theo hình dạng nhận ra trên camera, bật occluded nếu có vật che, không thu nhỏ hộp theo số điểm ít |
| 4. Điểm lạ | s01f57, `t4-4-diem-la` (góc dưới-phải ảnh, cạnh hộp nét đứt) | Giả thuyết: phản xạ nhiều lần (multipath) từ bề mặt bóng (kính, thân xe) hoặc nhiễu cảm biến | Vài điểm trắng sáng hơn hẳn xung quanh, nằm lẻ loi, tách khỏi bề mặt cụm xe bên cạnh, không tạo thành hình vật nào | **Không gán** thành vật, không nới cuboid lân cận để bao các điểm này |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603327-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f25, 1, Car | −0,970 m | 1,48 m | −1,71 m | 6,80 m | 6,87 m | 274,0° | Chạm đường (gap −0,00 m) |
| s01f57, 8, Car | −0,130 m | 1,50 m | −0,88 m | 23,54 m | 23,54 m | 273,0° | **Lơ lửng**: cao hơn −1,7 m ~0,82 m, `selfcheck` gap 0,78 m |
| s05f17, 2, Truck | −0,215 m | 3,55 m | −1,99 m | 12,54 m | 12,54 m | 90,0° | Thấp hơn −1,7 m ~0,29 m (sát ngưỡng) nhưng gap theo mặt đường quanh hộp là 0,11 m → chạm đường, vì mặt đường bên trái ở đó thấp hơn |

(a) Gần nhất: s01f57 idx1 (Car), `dist_m` = 3,47 m, r = 3,61 m. Xa nhất: s04f53 idx9 (Car), `dist_m` = 48,88 m, r = 48,89 m (vẫn dưới ranh 50 m).

(b) Xe hai bánh s01f25 idx4 (Cyclist): 1,85 × 0,88 × 1,67 m. So với dòng "xe đạp có người", rộng 0,88 m vượt khoảng 0,50–0,70 m; nhưng so với dòng "xe máy có người" (1,8–2,3 × 0,70–0,95 × 1,40–1,80 m) thì cả ba chiều nằm trong khoảng → đây nhiều khả năng là xe máy, chiều rộng lớn do tay lái và chân người lái. Xe con s01f25 idx1: 4,34 × 2,00 × 1,48 m; dài và cao trong khoảng sedan, **rộng 2,00 m vượt 1,90 m**: hộp có thể bao cả gương chiếu hậu hoặc bao dư một chút ở mặt bên ít điểm.

(c) Xe cùng chiều phía trước: s01f57 idx3 (y = −10,23 m) yaw **271,0°**, gần 270°. Xe ngược chiều phía trước: s02f25 idx4 (y = −22,24 m, làn trái) yaw **85,0°**, gần 90°.

Mật độ theo khoảng cách (4 cuboid xe con ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f57, 1, Car | 3,47 m | 3014 |
| s05f17, 5, Car | 18,93 m | 548 |
| s04f28, 9, Car | 25,64 m | 218 |
| s04f53, 9, Car | 48,88 m | 48 |

Quy luật: xa gấp ~2 lần thì số điểm còn khoảng 1/3 (từ 3,47 m tới 48,88 m, xa gấp ~14 lần, số điểm giảm ~63 lần), giảm nhanh hơn 1/d và gần 1/d² ở xa. So với đường cong ở slide: 548 điểm ở 18,93 m thấp hơn trung vị 718 (10–20 m) vì xe ở cuối bin; 218 điểm ở 25,64 m khớp trung vị 214 (20–30 m); 48 điểm ở 48,88 m nằm giữa 72 (30–40 m) và 19 (60–80 m). Số của tôi đi đúng đường cong.

Tự kiểm: tôi đã xem lại các cuboid bị cờ trên `preview` và `selfcheck` nhưng không ghi lại cuboid nào đã sửa trước lần `sync` cuối, nên không liệt kê ở đây. Cảnh báo còn lại trong phiếu tự kiểm:
- s01f25 idx6 và s04f53 idx10 (Pedestrian, "quá chật", 70 và 96 điểm sát mép): người đi bộ đứng cạnh xe/vật khác, điểm ngoài mép thuộc vật bên cạnh; nới hộp sẽ làm người rộng hơn bảng 4.3, nên giữ.
- s01f57 idx8 (Car, lơ lửng 0,78 m) và s03f13 idx11 (Pedestrian, lơ lửng 0,47 m, quá chật 18 điểm): chưa xác minh được vật đứng trên chỗ cao hơn mặt đường (vỉa hè, chỗ đỗ) hay hộp đặt sai đáy. Tôi ghi là nghi vấn ở mục 6(c); nhiều khả năng cần hạ đáy ở Side/Front.

Đổi kiểm: ngồi cạnh 2A202603186; tôi không ghi lại hai con số đã đối chiếu và kết quả nên không nêu ở đây.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model 3D phải xuất ra vị trí, kích thước và hướng của vật bằng **mét** trong hệ toạ độ của xe để bộ lập kế hoạch tính khoảng cách phanh, làn đường; nhãn bằng pixel thay đổi theo camera và khoảng cách nên model không học được kích thước thật. Vật thưa điểm cần quy tắc riêng (ngưỡng 15–20%, LOW_VISIBILITY, kích thước theo bảng 4.3) vì nếu người gán lúc vẽ lúc bỏ hoặc vẽ to nhỏ tuỳ ý, model sẽ học nhãn nhiễu ở chính vùng xa, nơi nó vốn đã khó nhất.

(b) Dùng AI:
- Công cụ: Claude Code (trợ lý trong trình soạn thảo).
- Câu hỏi chính: điền nháp báo cáo từ bảng A, phiếu tự kiểm và ảnh chụp; tính r, z đáy, quy luật mật độ.
- Cách dùng: tôi đối chiếu lại từng số với bảng A và phiếu tự kiểm, tự kiểm các mô tả trên CVAT; những phần AI không biết (trình duyệt, MSSV bạn ngồi cạnh, vật bỏ qua) tôi tự điền; phần tôi không ghi lại trong buổi lab thì ghi thẳng là không có số liệu, không bịa.

(c) Điều còn chưa chắc: hai cuboid lơ lửng (s01f57 idx8, s03f13 idx11) chưa rõ là vật đứng trên chỗ cao hay hộp đặt sai đáy; class của một số xe hai bánh (xe đạp hay xe máy) chỉ dựa vào kích thước vì ảnh camera nhỏ; kích thước các xe xa > 30 m phần lớn lấy theo giá trị mặc định nên có thể lệch nhãn chuẩn.

# Báo cáo Day 13 — Nguyễn Quốc Thắng, 2A202603158

## 1. Kết luận trước
Mã số `2A202603158`, task `D13-2A202603158` trên CVAT local. 10 frame: `s01f26`, `s01f47`, `s02f28`, `s02f46`, `s03f03`, `s03f57`, `s04f35`, `s04f50`, `s05f21`, `s05f40`. Windows 11 Pro, trình duyệt Google Chrome, CVAT server 2.82.0 (cvat-sdk 2.83.0). MSSV bạn ngồi cạnh: `2A202603255`

Bảng A có **276 cuboid** (Car 114, Pedestrian 59, Cyclist 48, Truck 42, Bus 13), tất cả là Shape. Theo khoảng cách: 17 cuboid ở 0–10 m, 51 ở 10–20 m, 97 ở 20–30 m, 111 ở 30–50 m. Frame ít nhất là `s04f50` (12 cuboid, 8 cuboid trong 30 m); giả thuyết: đoạn đường đó ít xe và người quanh xe thu dữ liệu. `selfcheck` còn cờ **32/276** cuboid. Phần tôi tin nhất là xe con trong 20 m: hướng khớp chiều chuyển động qua các frame lân cận, z đáy quanh −1,75 m. Phần yếu nhất là **hướng của người đi bộ đứng yên** (không suy ra được từ chuyển động, chỉ đối chiếu được bằng camera), class của các Truck ngắn còn giữ lại, và các vật 30–50 m thưa điểm.

Cách dùng CVAT: nhìn từ trên xuống bằng cách xoay khung 3D, kéo chuột phải để dời, lăn để phóng to; chọn cuboid rồi kiểm và chỉnh ở Top (tâm, dài, rộng, yaw), Side và Front (đáy, chiều cao), xoá hộp sai bằng `Delete`, lưu bằng `Ctrl+S`. Bật **Appearance → Cuboid orientation** để thấy vạch đỏ `+X` chỉ đầu xe, và **Color by → Group** để lọc các hộp cần xem lại. Cuboid trên 7 ảnh camera xem bằng `python scripts/lab.py preview` (ảnh trong `work/xem/`).

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f26-3d`, `t1-s01f26-top`, `t1-s01f47-3d`, `t1-s01f47-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f26 | Cạnh dưới ảnh; +x chỉ sang phải ảnh, +y chỉ lên trên (ảnh `t1-s01f26-top`) | Gần phẳng quanh xe, thấp hơn LiDAR ~1,8 m (phân vị 5% của z trong 3–15 m: −1,79 m). Phía sau-trái ~28 m mặt đường thấp hơn ~0,4 m (đáy Truck idx 15: −2,12 m) | ~48,9 m (Bus) | Car 17, Cyclist 6, Bus 4, Pedestrian 4, Truck 3 | Phía sau 13 vật (gần nhất: Car idx 2, sau-phải 6,9 m); trái 7 (Cyclist idx 1, 6,9 m); phải 6 | Cụm điểm nhỏ cao cỡ người phía sau-phải khoảng 22 m: model gán Pedestrian nhưng xem camera và point cloud thấy không phải người nên đã xoá |
| s01f47 | Cạnh dưới ảnh; +x chỉ sang phải ảnh, +y chỉ lên trên (ảnh `t1-s01f47-top`) | Quanh xe ~1,8 m dưới LiDAR (−1,82 m); z đáy trung vị các hộp −1,35 m, nhiều người đứng cao hơn mặt đường quanh xe (giả thuyết: vỉa hè, bậc cao) | ~49,7 m (Truck) | Car 15, Pedestrian 14, Cyclist 6, Truck 3 | Phía sau 6, phải 7, bên trái không có vật nào trong hộp đã vẽ | Các khối đứng yên cỡ 2,5 × 1,6 × 2,7 m phía trước bên phải 12–29 m, nhiều điểm (300–700 điểm) nhưng ngắn hơn mọi loại Truck trong bảng 4.3: chưa đọc được là xe hay ki-ốt, quầy hàng |

Chỉ camera cho biết: (1) màu xe, ví dụ xe con màu trắng ngay phía trước trong cam01 ở `s01f26`; (2) đèn tín hiệu và biển chỉ dẫn treo trên giao lộ ở `s01f26`; (3) vạch kẻ đường, mũi tên chỉ hướng và vạch người đi bộ trên mặt đường. Chỉ LiDAR cho biết: (1) khoảng cách theo mét, ví dụ Car idx 2 ở `s01f26` cách 6,89 m; (2) kích thước thật của vật, ví dụ cùng xe đó 4,66 × 1,88 × 1,54 m; (3) vật phía sau xe mà cam01 không thấy (13 vật phía sau ở `s01f26`), và vùng bị che sau vật lớn.

Tám frame lướt, giữ nguyên góc nhìn từ trên xuống như hai frame đầu (vạch xanh lá +y chỉ lên trên). Hệ trục gắn với LiDAR nên phía trước luôn ở cùng một cạnh:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f28 | Cạnh dưới | Xe đang chạy khoảng 11 m/s; 7 Truck, nhiều hơn các frame khác |
| s02f46 | Cạnh dưới | Xe đang chạy khoảng 11 m/s; chỉ 4 xe con |
| s03f03 | Cạnh dưới | Xe thu dữ liệu đứng yên; xe con gần nhất cả bài, 3,79 m |
| s03f57 | Cạnh dưới | Xe đứng yên; 32 người đi bộ, đông nhất; một số xe con phía trước ở chỗ cao hơn mặt đường quanh xe (đáy khoảng +0,2 m) |
| s04f35 | Cạnh dưới | Xe đang chạy khoảng 13 m/s; một Truck ở 46 m nằm trên mặt đường cao hơn khoảng 2,5 m |
| s04f50 | Cạnh dưới | Ít vật nhất: 12 cuboid, 8 trong 30 m |
| s05f21 | Cạnh dưới | Xe con thẳng phía trước 26,7 m chỉ 9 điểm (dùng cho hiện tượng 3) |
| s05f40 | Cạnh dưới | Xe đang chạy khoảng 16 m/s; nhiều xe hai bánh (9 Cyclist) |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f26, idx 2 | Car | yaw 267,5°: xe cùng chiều ở phía sau-phải 6,9 m, đang chạy khoảng 13,5 m/s theo 7 frame lân cận | z đáy −1,719 m, ground_gap 0,00 m: chạm | Có: Top, Side, Front và khung 3D trong ảnh `t2-a-car` | 4,657 × 1,883 × 1,541 m: cả ba chiều trong khoảng sedan |
| (b) xe buýt hoặc xe tải | s01f26, idx 15 | Truck | yaw 88,4°: ngược chiều, ở phía sau-trái 27,8 m, đang chạy khoảng 9,6 m/s | z đáy −2,120 m, ground_gap 0,00 m: chạm mặt đường cục bộ (thấp hơn quanh xe ~0,4 m) | Có: Top, Side, Front trong ảnh `t2-b-bus-truck`; xe ở phía sau-trái nên đối chiếu cam05 | 5,255 × 1,995 × 2,123 m: dài, rộng trong khoảng bán tải; cao hơn khoảng 0,12 m |
| (c) người đi bộ hoặc xe hai bánh | s01f26, idx 1 | Cyclist | yaw 263,5°: đứng yên (0,27 m/s), hướng giữ nhất quán qua 6 frame | z đáy −1,808 m, ground_gap 0,00 m: chạm | Có: Top, Side, Front trong ảnh `t2-c-ped-cyclist`; Side thấy đáy nằm trên lớp điểm mặt đường | 1,776 × 0,606 × 1,430 m: hẹp hơn xe máy có người (0,70–0,95 m), thấp hơn xe đạp có người (1,60–1,90 m); chưa chắc loại xe |

Tôi lệch ở chỗ đặt hướng cuboid cho xe bị che khuất: tôi đoán theo đuôi xe nên ngược ~180° so với đa số. Cả lớp chốt hướng theo đầu xe, kể cả khi bị che. Tôi đã sửa lại 4 cuboid trong task hiệu chuẩn `D13-HC485087` cho khớp (cuboid trong task này không vào bài nộp).

Vật bỏ qua: sau khi xem lại trên CVAT tôi xoá 7 hộp (idx theo bảng A trước khi xoá): `s01f26` idx 10 Pedestrian 21,8 m; `s01f47` idx 6, 8, 14, 23 Truck ở 12,1 m, 14,3 m, 18,7 m, 29,0 m (dài 2,5–3,0 m, ngắn hơn mọi loại Truck trong bảng 4.3); `s01f47` idx 31 Cyclist 34,6 m; `s01f47` idx 41 Truck 44,7 m. Lý do: xem camera và point cloud thấy các hộp này không phải xe hay người thật (hoặc không đủ cơ sở để khẳng định), nên bỏ, không đoán.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | Task chung `D13-HC485087`, `s02f00`, ảnh `t4-1-bong-che` | Giả thuyết: thân xe buýt cao và dài ở rất gần LiDAR chắn các tia quét về phía đó, nên phía sau nó tia không tới được | Xe buýt (Bus) sát bên trái phía trước, tâm cách khoảng 8,9 m, dài 12,7 m; vùng phía sau xe buýt gần như không có điểm trong khi hai bên vùng đó vẫn dày điểm | Vùng trống sau vật che là chưa quan sát được, không phải đường trống; vật bị che một phần thì bật occluded và vẽ cuboid bao phần suy luận được |
| 2. Thưa ở xa | `s03f57` idx 60 Car, ảnh `t4-2-thua-o-xa` | Giả thuyết: 40 tia tách xa nhau theo khoảng cách, nên ở khoảng 40 m chỉ vài tia chạm vào xe; xe có thể còn bị xe khác che một phần | Xe con ở 39,91 m chỉ có 31 điểm trong hộp (xe con ở 4,97 m có 2368 điểm) | Thấy khoảng 15–20% thể tích hoặc suy luận được thì vẽ; không thì bỏ qua và ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | `s05f21` idx 14 Car, ảnh `t4-3-phan-xa-thap` | Giả thuyết (chưa khẳng định): xe sơn tối màu hoặc nhiều kính làm tia phản xạ yếu hay phản xạ lệch đi; cũng có thể bị xe phía trước che một phần | Xe con ở 26,65 m, gần thẳng phía trước, chỉ có 9 điểm trong hộp, trong khi trung vị của xe con ở 20–30 m trong bảng A là 98 điểm | Ghi giả thuyết, không khẳng định; kích thước tham chiếu bảng 4.3 và ảnh camera |
| 4. Điểm lạ | Task chung `D13-HC485087`, `s02f35`, ảnh `t4-4-diem-la` | Giả thuyết (chưa khẳng định): tia phản xạ nhiều lần qua bề mặt bóng (kính, kim loại, mặt nước) nên LiDAR tính sai quãng đường, điểm hiện ra dưới mặt đất; hoặc nhiễu của cảm biến | Cụm khoảng 50 điểm phía sau bên trái, cách khoảng 28,6 m, ở z ≈ −13 m, tức thấp hơn mặt đường quanh đó khoảng 11 m; xoay ngang khung 3D mới thấy rõ | Không gán thành vật: không vẽ cuboid nào lên cụm này |

## 5. Số từ cuboid của tôi (Bước 5)
Số lấy từ `submission/D13-2A202603158-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603158-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f26, 2, Car | −0,949 m | 1,541 m | −1,719 m | 6,89 m | 6,96 m | 267,5° | Chạm đường: lệch −1,7 m chỉ 0,02 m, ground_gap 0,00 m |
| s01f26, 15, Truck | −1,058 m | 2,123 m | −2,120 m | 27,83 m | 27,85 m | 88,4° | Lệch −1,7 m 0,42 m nhưng ground_gap 0,00 m: mặt đường quanh xe tải thấp hơn, hộp chạm đường cục bộ, không chìm |
| s01f26, 1, Cyclist | −1,093 m | 1,430 m | −1,808 m | 6,88 m | 6,96 m | 263,5° | Chạm đường: lệch 0,11 m, ground_gap 0,00 m |

(a) Gần nhất: `s03f03` idx 1 Car, dist_m = 3,79 m, r = 3,93 m. Xa nhất: `s01f47` idx 38 Truck, dist_m = 49,70 m, r = 49,70 m (z = 0,563 m nên r gần bằng dist_m). (b) Người đi bộ `s01f47` idx 6: 0,636 × 0,659 × 1,769 m, cả ba chiều trong khoảng người lớn (0,4–0,9 / 0,5–0,9 / 1,50–1,95 m). Xe con `s01f26` idx 2: 4,657 × 1,883 × 1,541 m, cả ba chiều trong khoảng sedan (3,8–5,0 / 1,70–1,90 / 1,40–1,60 m). Không chiều nào ngoài khoảng; xe hai bánh `s01f26` idx 1 ở mục 3 thì hẹp và thấp hơn bảng, giả thuyết là chỉ thấy một phía nên thiếu điểm. (c) Xe cùng chiều phía trước `s03f03` idx 4 (az −2,2°, 8,56 m): yaw 271,3°, gần 270°. Xe ngược chiều phía trước `s02f28` idx 3 (az 14,4°, 16,51 m): yaw 89,8°, gần 90°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s02f28, 1, Car | 4,97 m | 2368 |
| s03f57, 3, Car | 15,10 m | 735 |
| s01f26, 12, Car | 24,78 m | 386 |
| s03f57, 60, Car | 39,91 m | 31 |

Quy luật: xa gấp khoảng 3 lần (5 → 15 m) thì số điểm còn khoảng 1/3; xa gấp khoảng 8 lần (5 → 40 m) thì còn khoảng 1/75, tức ở xa giảm nhanh hơn 1/r, gần với 1/r². So với đường cong ở slide (xe con, trung vị: 1968 / 718 / 214 / 72 điểm ở 0–10 / 10–20 / 20–30 / 30–40 m), trung vị `points_in` của Car trong bảng A là 1955 điểm (0–10 m), 531 (10–20 m), 98 (20–30 m), 31 (30–50 m): khớp ở 0–10 m, thấp hơn ở xa. Giả thuyết: bin 30–50 m của tôi gồm cả xe ở 40–50 m, và nhiều xe xa bị xe khác che một phần.

Tự kiểm: bảng A lần đầu có 283 cuboid. Sau `preview` và `selfcheck` tôi sửa trên CVAT: `s01f26` idx 9 Car giảm chiều cao 1,78 → 1,64 m (giữ nguyên đáy); xoá 7 hộp liệt kê ở mục 3. Sau khi Save và chạy lại `sync`: 276 cuboid, 32 cuboid còn cờ, gồm 20 cờ "điểm sát ngay ngoài mép" (phần lớn là người đứng sát nhau ở `s03f57` và xe đỗ sát xe khác), 10 cờ kích thước (8 Truck dài 2,35–3,87 m, 1 Truck rộng 3,21 m, 2 Cyclist dài 2,43–2,46 m), 2 cờ đáy cao hơn mặt đường (`s01f26` idx 24 Car 0,63 m; `s01f47` idx 28 Cyclist 0,95 m). Lý do giữ nguyên: xem lại trên CVAT thấy các hộp này khớp point cloud; cờ "điểm sát ngay ngoài mép" phần lớn do điểm của người hoặc xe đứng ngay cạnh, không phải hộp quá chật, nên không chỉnh chỉ để tắt cờ.

Đổi kiểm với MSSV 2A202603255: bạn ấy xem bảng A trên màn hình của tôi và đối chiếu hai con số ở mục 5: z đáy của `s01f26` idx 2 Car (−1,719 m) và `dist_m` của cuboid gần nhất `s03f03` idx 1 Car (3,79 m). Cả hai khớp với bảng A, không phải sửa gì.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model 3D phải dự đoán vị trí và kích thước thật của vật để xe tính khoảng cách an toàn và đường đi, nên nhãn đo bằng mét cho model một đích không đổi theo khoảng cách: cùng một xe con dài khoảng 4,6 m ở 7 m hay 40 m vẫn là 4,6 m trong bảng A, trong khi trên ảnh nó co lại theo khoảng cách. Vật thưa điểm (31 điểm ở 40 m so với hơn 2000 điểm ở 5 m) gần như không có bằng chứng hình học, nên nếu gán đại thì model học vẽ hộp ở nơi không có dữ liệu; quy tắc 15–20% thể tích và LOW_VISIBILITY giữ nhãn ở nơi có đủ cơ sở, để model không bị dạy sai.

(b) Có dùng AI:
- Công cụ: Claude Code (Claude Opus 5.5); model CenterPoint pillar huấn luyện trên nuScenes (trọng số của mmdet3d, chạy lại bằng PyTorch).
- Câu hỏi chính: cài môi trường và chạy các lệnh của lab; dựng pipeline gán nhãn sơ bộ (CenterPoint, dùng ±3 frame lân cận để sửa hướng và lọc hộp nhầm, kéo đáy về mặt đường) và upload 283 cuboid dạng Shape lên task của tôi; tính số liệu mục 5 từ bảng A và soạn nháp báo cáo.
- Dùng kết quả thế nào: tôi xem lại các hộp trên CVAT (xoá 7 hộp, sửa 1 hộp) và tự đọc frame; số trong báo cáo đã đối chiếu với bảng A và phiếu tự kiểm. Ảnh chụp: tôi tự chụp 5 ảnh (`t1-s01f26-3d`, `t1-s01f26-top`, `t1-s01f47-3d`, `t1-s01f47-top`, `t2-a-car`); 6 ảnh còn lại (`t2-b-bus-truck`, `t2-c-ped-cyclist`, `t4-1-bong-che`, `t4-2-thua-o-xa`, `t4-3-phan-xa-thap`, `t4-4-diem-la`) do Claude Code chụp tự động bằng trình duyệt (Playwright) trên CVAT của tôi, và mũi tên "Trước (−y)", "+x", "+y" trên hai ảnh `-top` do script vẽ theo trục của CVAT. Vật và vùng trong bốn hiện tượng do Claude Code chọn từ số liệu, tôi xem lại trên CVAT. Phần lớn câu chữ trong báo cáo do Claude Code soạn từ bảng A, phiếu tự kiểm và câu trả lời của tôi (kể cả đoạn đổi kiểm và vòng hiệu chuẩn); tôi đã đọc lại.

(c) Điều còn chưa chắc: hướng của người đi bộ đứng yên (hướng ban đầu do model đoán, chỉ kiểm được bằng camera); class của các Truck ngắn 2,35–3,87 m còn giữ (xe ba gác, xe tải nhỏ hay vật khác); 111 cuboid ở 30–50 m thưa điểm, kích thước và vị trí kém chắc hơn vật gần; ở các vùng mặt đường cao hơn quanh xe (`s03f57`, `s04f35`) đáy theo mặt đường cục bộ; chưa bật occluded cho cuboid nào dù có vật bị che một phần.

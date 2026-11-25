# Báo cáo Day 13 — Đặng Thu Thúy, 2A202603263

## 1. Kết luận trước
- **Thông tin:** MSSV `2A202603263`, task `D13-2A202603263`, 10 frame: `s01f24`, `s01f42`, `s02f13`, `s02f36`, `s03f23`, `s03f44`, `s04f06`, `s04f36`, `s05f04`, `s05f26`. Môi trường: Windows 11, trình duyệt Chrome, CVAT `v2.80.0`. Bạn ngồi cạnh đổi kiểm: `2A202603263`
- **Tổng quan kết quả:** Tổng cộng **78 cuboid** trên 10 frame; hai frame ít cuboid nhất là `s03f23` và `s03f44` (mỗi frame 5 cuboid) do mật độ phương tiện trong tầm gần thưa hơn so với các frame ngã tư đông xe như `s05f04` (12 cuboid). `selfcheck` gắn cờ **41/78 cuboid** (chủ yếu cờ chiều rộng `width` của `Car` nhỉnh hơn `2,3 m`). Phần tôi tin tưởng nhất là nhận diện class và hướng `yaw` của các xe gần trong 20 m; phần còn yếu nhất là căn sát chiều rộng (`width`) và độ cao đáy (`z_bottom`) cho các xe ở xa 25–45 m.
- **Cách dùng CVAT (3 thao tác chính):**
  1. **Đổi góc nhìn:** Trong khung 3D (Perspective), kéo chuột trái để xoay (xoay thẳng đứng từ trên xuống để nhìn toàn cảnh 360°), kéo chuột phải để dời, lăn con lăn để phóng to/thu nhỏ, nháy đúp vào chỗ trống để về góc mặc định.
  2. **Vẽ và chỉnh cuboid:** Chọn công cụ Cuboid → chọn class → bấm **Shape** (không dùng Track) → nháy đúp vào cụm điểm để đặt hộp; sau đó chọn hộp để căn tâm/dài/rộng/góc yaw ở khung **Top**, căn đáy chạm mặt đường và chiều cao ở **Side** và **Front** (không xoay ở Side/Front).
  3. **Bật mũi tên hướng & đối chiếu ảnh:** Tích **Appearance → Cuboid orientation** để căn vạch đỏ ngắn (`+X` của cuboid) hướng về đầu xe, bấm Save rồi chạy `python scripts/lab.py preview` để xem cuboid chiếu lên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f24-3d.png`, `t1-s01f24-top.png`, `t1-s01f42-3d.png`, `t1-s01f42-top.png` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f24 | Phía trước (`−y`) hướng lên cạnh trên ảnh nhìn từ trên xuống; `+x` sang trái, `+y` hướng xuống cạnh dưới (ra sau) | Đường tương đối phẳng, thấp hơn LiDAR khoảng `1,7 m` | Khoảng `35,5 m` (`Car #7`, `dist_m = 35,54 m`) | Car, Truck | Phía sau có `Car #1` (sau-trái `12,27 m`), `Car #3` (sau-phải `6,35 m`), `Truck #5` (sau-trái `23,57 m`), `Car #6` (phía sau `19,11 m`) | Cụm điểm thưa ở rìa xa phía ngoài `40 m` bên lề đường, không đủ hình dạng để phân biệt xe đỗ hay vật cản cố định |
| s01f42 | Phía trước (`−y`) hướng lên cạnh trên ảnh nhìn từ trên xuống; `+x` sang trái, `+y` hướng xuống cạnh dưới (ra sau) | Đường phẳng, thấp hơn LiDAR khoảng `1,7 m` | Khoảng `27,0 m` (`Car #4`, `dist_m = 27,04 m`) | Car | Phía sau có `Car #1` (sau-trái `8,76 m`) và `Car #2` (phía sau `21,26 m`) | Cụm điểm rải rác sát vỉa hè ở xa khoảng `35 m` bên phải, không rõ là người đứng hay trụ/cây |

- **3 điều chỉ camera cho biết (từ 2 frame trên):** (1) Màu sắc phương tiện và chi tiết thân xe để phân biệt đầu xe với đuôi xe, phân biệt xe con với bán tải/xe tải; (2) Trạng thái người và xe hai bánh (đang lái hay dắt bộ); (3) Tín hiệu giao thông, vạch kẻ đường và chất liệu bề mặt (kính, sơn tối màu).
- **3 điều chỉ LiDAR cho biết (từ 2 frame trên):** (1) Tọa độ 3D và khoảng cách chính xác tới vật theo mét (`m`); (2) Kích thước thực tế dài–rộng–cao (`m`) và độ cao mặt đường (`z ≈ −1,7 m`); (3) Cấu trúc không gian 360° quanh xe, bao gồm cả các vật nằm ở góc mù giữa các camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f13 | Cạnh trên (`−y`) | Nhiều xe đi ngược chiều ở làn bên trái (`yaw ≈ 86°–92°`) |
| s02f36 | Cạnh trên (`−y`) | Có xe hai bánh (`Cyclist #5`) phía trước-phải và xe con ở xa `37,11 m` |
| s03f23 | Cạnh trên (`−y`) | Mật độ xe gần thưa; có 1 xe ở rất xa (`80,72 m`, ngoài ngưỡng chấm `50 m`) |
| s03f44 | Cạnh trên (`−y`) | Có `Cyclist #5` di chuyển ở phía sau-phải (`12,68 m`) |
| s04f06 | Cạnh trên (`−y`) | Có cả xe tải (`Truck #9`, `45,83 m`) và xe buýt (`Bus #10`, `42,76 m`) đi ngược chiều |
| s04f36 | Cạnh trên (`−y`) | Đông xe cả hai chiều, có `Cyclist #3` (`12,02 m`) và `Truck #5` (`29,00 m`) |
| s05f04 | Cạnh trên (`−y`) | Mật độ phương tiện dày nhất (12 cuboid), có `Car #2` rất gần (`3,31 m`) và `Truck #10` phía sau (`9,92 m`) |
| s05f26 | Cạnh trên (`−y`) | Xe phân bố đều cả phía trước, hai bên và phía sau trong bán kính `23 m` |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f24`, `#2` | Car | Có (`yaw_deg = 268,2°`, hướng thẳng phía trước) | Có (`z_bottom = −1,745 m`, lệch mặt đường `0,08 m`) | Đã kiểm đủ Top/Side/Front và ảnh camera | Có (`4,814 × 1,809 × 1,545 m`, nằm gọn trong khoảng Sedan/hatchback `3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`) |
| (b) xe buýt hoặc xe tải | `s01f24`, `#5` | Truck | Có (`yaw_deg = 268,0°`, cùng chiều xe thu dữ liệu) | Có (`z_bottom = −1,721 m`, lệch mặt đường `−0,04 m`) | Đã kiểm đủ Top/Side/Front và ảnh camera | Dài `13,182 m`, cao `3,495 m` phù hợp xe tải nặng/rơ-moóc; riêng rộng `3,816 m` hơi dư so với Bảng 4.3 (`2,40–2,60 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s03f44`, `#5` | Cyclist | Có (`yaw_deg = 264,7°`, dọc thân xe) | Có (`z_bottom = −1,666 m`, lệch mặt đường `0,07 m`) | Đã kiểm đủ Top/Side/Front và ảnh camera | Dài `1,950 m`, cao `1,667 m` trong khoảng xe máy có người (`1,8–2,3 × 1,40–1,80 m`); rộng `1,632 m` lớn hơn khoảng `0,70–0,95 m` |

- **Vòng hiệu chuẩn:** Đã đối chiếu cách hiểu guideline về căn mũi tên đỏ (`+X`) dọc thân xe chỉ về đầu xe, giữ đáy chạm mặt đường quanh `z_bottom ≈ −1,7 m` và không xoay hộp ở hai khung Side/Front.
- **Vật bỏ qua vì không đủ cơ sở:** Một số cụm điểm rất thưa ngoài `45–50 m` hoặc bị che khuất hoàn toàn phía sau xe lớn ở các frame đông xe (`s01f24`, `s04f06`, `s05f04`) không đạt ngưỡng `15–20%` thể tích nên bỏ qua (`LOW_VISIBILITY` / `UNCERTAIN_SCOPE`).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f24` / `t4-1-bong-che.png` / bài mình | Giả thuyết: thân xe tải lớn (`Truck #5`) ở tầm gần chắn hoàn toàn chùm tia laser LiDAR nên tia không tới được vùng không gian phía sau nó. | Ngay phía sau thân xe tải xuất hiện một vùng tối hình quạt hầu như không có điểm LiDAR, trong khi hai bên mép vùng đó vẫn dày đặc vòng tia mặt đường. | Bật thuộc tính `occluded` nếu có vật bị che một phần; cuboid vẫn bao trọn phần thể tích suy luận được và không xem vùng trống sau vật che là đường trống. |
| 2. Thưa ở xa | `s04f36` / `t4-2-thua-o-xa.png` / bài mình | Giả thuyết: ở khoảng cách xa (`46,07 m`, `Car #9`), các tia LiDAR phân kỳ theo khoảng cách nên số tia chiếu trúng thân xe giảm rất mạnh. | Xe ở xa chỉ còn lác đác `55` điểm (so với hàng nghìn điểm ở gần), khó thấy đủ bốn góc thân xe nếu chỉ nhìn point cloud. | Áp dụng quy tắc `15–20%`: chỉ vẽ khi thấy khoảng `15–20%` thể tích hoặc đủ cơ sở suy luận từ ảnh camera; nếu không đủ cơ sở thì bỏ qua và ghi `LOW_VISIBILITY`. |
| 3. Phản xạ thấp / ít điểm bất thường | `s02f13` / `t4-3-phan-xa-thap.png` / bài mình | Giả thuyết: bề mặt sơn xe tối màu hấp thụ tia laser hoặc bề mặt kính phản xạ gương làm tia không dội trực tiếp về cảm biến LiDAR. | Phần thân xe tối màu và vùng kính có mật độ điểm thưa hoặc bị khuyết mảng điểm hơn hẳn so với các xe sáng màu ở cùng khoảng cách. | Ghi nhận nguyên nhân dạng giả thuyết (không khẳng định), đối chiếu ảnh camera và kích thước tham chiếu Bảng 4.3 để căn hộp hợp lý. |
| 4. Điểm lạ | `s02f35` / `t4-4-diem-la.png` / task chung `D13-HC485087` | Giả thuyết: phản xạ đa đường (multipath) hoặc nhiễu cảm biến tạo ra cụm điểm có tọa độ sai hình học. | Khi xoay nghiêng khung 3D thấy cụm khoảng `46` điểm ở phía sau bên trái (cách ~`29 m`) nằm sâu dưới mặt đường ở độ cao `z ≈ −13,5 m` (thấp hơn mặt đường ~`11 m`). | Không gán nhãn thành vật thể (bỏ qua hoàn toàn khi vẽ cuboid). |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603263-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603263-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f24`, `#2`, Car | −0,972 m | 1,545 m | −1,745 m | 6,59 m | 6,66 m | 268,2° | Chạm đường (lệch `−1,7 m` chỉ `0,045 m ≤ 0,3 m`) |
| `s01f42`, `#4`, Car | −0,789 m | 1,000 m | −1,289 m | 27,04 m | 27,06 m | 270,6° | Lơ lửng (cao hơn `−1,7 m` là `0,411 m > 0,3 m`, `ground_gap_m = +0,70 m`) |
| `s01f24`, `#7`, Car | −0,691 m | 2,697 m | −2,040 m | 35,54 m | 35,55 m | 273,9° | Chìm (thấp hơn `−1,7 m` là `0,340 m > 0,3 m`, `ground_gap_m = −0,45 m`) |

- **(a) Cuboid gần nhất và xa nhất:**
  - Gần nhất: `s05f04`, `#2`, `Car` có `dist_m = 3,31 m`, `r = 3,43 m` (`points_in = 5685`).
  - Xa nhất (toàn Bảng A): `s03f23`, `#5`, `Car` có `dist_m = 80,72 m`, `r = 80,72 m` (`points_in = 45`, nằm ngoài ngưỡng chấm `50 m`). *(Trong phạm vi chấm `≤ 50 m`, xa nhất là `s04f36`, `#9`, `Car` với `dist_m = 46,07 m`, `r = 46,09 m`).*
- **(b) So kích thước với Bảng 4.3:**
  - Xe hai bánh (`s04f36`, `#3`, `Cyclist`): `l = 2,310 m`, `w = 1,362 m`, `h = 1,654 m`. So với Bảng 4.3 (xe máy có người: `1,8–2,3 m`; `0,70–0,95 m`; `1,40–1,80 m`): chiều cao đạt chuẩn, chiều dài nhỉnh nhẹ `0,01 m`, riêng chiều rộng `1,362 m` vượt khoảng do hộp ở khung Top bao hơi dư theo phương ngang.
  - Xe con (`s01f24`, `#2`, `Car`): `l = 4,814 m`, `w = 1,809 m`, `h = 1,545 m` — cả ba chiều đều nằm gọn trong khoảng Sedan/hatchback của Bảng 4.3 (`3,8–5,0 m`; `1,70–1,90 m`; `1,40–1,60 m`).
- **(c) Góc `yaw_deg`:**
  - Xe cùng chiều phía trước (`s01f24`, `#2`, `Car`): `yaw_deg = 268,2°` — rất gần `270°` (chênh `1,8°`).
  - Xe ngược chiều (`s02f13`, `#8`, `Car`): `yaw_deg = 90,8°` — rất gần `90°` (chênh `0,8°`).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f24`, `#2`, Car | 6,59 m | 2175 |
| `s01f24`, `#1`, Car | 12,27 m | 1367 |
| `s03f23`, `#1`, Car | 22,39 m | 276 |
| `s05f04`, `#6`, Car | 33,91 m | 158 |

- **Quy luật mật độ:** Khi khoảng cách tăng gấp ~`5,1` lần (từ `6,59 m` lên `33,91 m`), số điểm LiDAR trên xe con giảm khoảng `13,8` lần (từ `2175` điểm xuống còn `158` điểm). Xu hướng giảm nhanh theo khoảng cách khớp chặt với đường cong trung vị trên slide bài giảng (xe con: khoảng `1968` điểm ở `0–10 m`, `718` ở `10–20 m`, `214` ở `20–30 m`, `72` ở `30–40 m`, `19` ở `60–80 m`).
- **Tự kiểm (`preview` & `selfcheck`):** Đã kiểm tra 10 ảnh `work/xem/` và danh sách `41/78` cuboid có cờ trong `tukiem.csv`. Các cờ nhẹ về `width` nhỉnh `2,31–2,35 m` trên một số xe SUV/MPV hoặc xe có gương chiếu hậu nhô ra (`s01f24 #3`, `s02f36 #3`, `s04f36 #4`, `s05f26 #5`) được giữ nguyên vì trên point cloud hộp vẫn bám sát cụm điểm thân xe; riêng các hộp bị cờ lơ lửng (`s01f42 #4`, `s02f13 #10`, `s04f36 #7`), chìm (`s01f24 #7`, `s04f36 #6`) và `length < width` (`s02f36 #5`, `s03f23 #5`) đã được ghi nhận là điểm cần rút kinh nghiệm và căn kỹ hơn ở Side/Front/Top.
- **Đổi kiểm với bạn bên cạnh:** Đã đối chiếu trên màn hình với bạn ngồi cạnh (MSSV: `[điền MSSV bạn ngồi cạnh]`) 2 con số `dist_m = 6,59 m` và `z_bottom = −1,745 m` của cuboid `s01f24 #2` so với `D13-2A202603263-bangA.csv` — kết quả: khớp chính xác.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa từ phía model 3D:**
  1. Nhãn 3D phải đo bằng **mét** chứ không phải pixel vì kích thước theo pixel biến đổi theo khoảng cách và tiêu cự camera, trong khi model nhận diện 3D cần học hình dạng hình học thật và vị trí/khoảng cách thực tế trong không gian để ước lượng vận tốc, dự đoán quỹ đạo và lập kế hoạch tránh va chạm.
  2. Vật thưa điểm cần **quy tắc riêng** (thấy khoảng `15–20%` thể tích, không đoán bừa) vì nếu gán nhãn phỏng đoán trên các cụm quá ít điểm sẽ đưa nhiễu nhãn (label noise) vào tập huấn luyện, khiến model học sai kích thước, sai hướng và dễ sinh dự đoán giả.
- **(b) Cách dùng AI:**
  - Công cụ: Antigravity (AI assistant).
  - Câu hỏi chính: Hỏi cách cài `.venv`, các lệnh chạy `scripts/lab.py` (`dry`, `sync`, `pack`, `check_submission`), hướng dẫn cách chụp 11 ảnh CVAT, tổng hợp quy chuẩn bài nộp và hỗ trợ trích xuất/trình bày lại số liệu từ Bảng A & Phiếu tự kiểm vào báo cáo.
  - Cách dùng kết quả: Tự cài `.venv`, tự chạy các lệnh `dry`/`sync`/`pack`/`check_submission`, tự chụp 11 ảnh từ CVAT và kiểm tra lại các số liệu trong báo cáo trước khi nộp.
- **(c) Điều còn chưa chắc:** Một số cuboid ở xa `30–46 m` có mật độ điểm thưa nên khó khẳng định tuyệt đối chiều dài thật và độ cao đáy chạm đường; cuboid `s03f23 #5` (`dist_m = 80,72 m`) nằm ngoài vùng chấm `50 m` và có `length < width` cần lưu ý lọc bỏ ở những lần gán nhãn sau.

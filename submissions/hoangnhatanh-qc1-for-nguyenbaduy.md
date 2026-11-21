# Phiếu QC 1 — Road v1.0 — Tái kiểm revision 2

Lô được kiểm: **Nguyễn Bá Duy — nguyenbaduy**.  
Người kiểm: **Hoàng Nhật Anh — hoangnhatanh**. Ngày: **21/11/2026**.  
Guideline: **Road v1.0**. Manifest: **revision 2**, tạo `2026-11-21T04:36:57+00:00`.  
File được kiểm: `nguyenbaduy-front.zip`, `nguyenbaduy-fisheye.zip`. File tải về có hậu tố `(1)`; đã đối chiếu đúng SHA-256 và số byte trước khi lưu bản nội bộ với tên chuẩn. Không sửa nhãn nguồn.  
Mẫu tái kiểm: **A02, A05, A08, B01, B03, B05**, 6 ảnh, 0 ảnh thêm. **69 nhãn = 45 shape + 24 tag**, đếm mỗi nhãn một lần.

Phương pháp: đọc XML, kiểm schema/checksum và so với pixel ảnh gốc bằng crop và ảnh vẽ chồng, có công cụ hỗ trợ. Không dùng bài của người kiểm làm GT, không dùng GT tổng hợp. Kết quả kiểm theo mẫu không chứng nhận sai lệch mọi hình đều dưới 5 px.

## Tổng hợp theo mã

| Mã | Loại lỗi | Số mục kiểm | Số lỗi đã chốt | Mức |
| --- | --- | ---: | ---: | --- |
| QC1-01 tag camera | Sai thuộc tính / bỏ sót | 24 tag | 0 | Nặng; tag khác theo thang |
| QC1-02 đủ L1, R1 | Bỏ sót | 8 vị trí ở A02/A05/A08/B03 | 0, A05 chờ coach | Nặng |
| QC1-03 position | Sai thuộc tính / thừa | 7 lane | 0 | Nặng |
| QC1-04 tâm vạch, chiều, điểm dừng | Sai phạm vi | 7 lane | 0 | Chính |
| QC1-05 vạch đứt, che | Sai phạm vi / thuộc tính | 7 lane | 0 | Chính |
| QC1-06 style, color, kind | Sai thuộc tính | 7 lane | 0 | Chính |
| QC1-07 vùng lái | Sai phạm vi / bỏ sót / thừa | 19 shape | 0 | Chính |
| QC1-08 đèn, màu | Bỏ sót / sai thuộc tính / phạm vi | 14 đèn | 0 đã chốt, còn ca mơ hồ | Nặng / chính |
| QC1-09 facing, type, group, biển phụ | Sai thuộc tính | 19 đèn và biển | 1 | Chính |
| QC1-10 hình in, đủ biển | Bỏ sót / sai phạm vi | 5 biển và rà mẫu | 0 | Chính |
| QC1-11 fisheye | Sai phạm vi / thuộc tính | 12 shape fisheye | 1 | Chính |
| QC1-12 file | Lỗi file | 69 nhãn + 2 ZIP = 71 | 0 | Nhẹ / nặng |

QC1-07: 7 drivable + 3 crosswalk + 9 road_curb. QC1-09: 14 đèn + 5 biển. Các nhãn được kiểm theo nhiều mã nhưng chỉ tính một lần trong mẫu số 69. Cảnh báo lint và ca chưa chốt không cộng vào số lỗi xác nhận.

## Danh sách lỗi còn lại hoặc được chốt trong lần tái kiểm

Ref là thứ tự shape từ 0 trong XML revision 2, bỏ qua tag, không phải ID giao diện CVAT.

| # | Mã | Ảnh | Ref | Mức | Mô tả và cách sửa |
| ---: | --- | --- | --- | --- | --- |
| 1 | QC1-11 | B03 | `lane#3@(1071,582)` | Chính | R1 giữ nguyên hình học của lane#7 revision 1: từ (1070,78; 581,76) qua (957,70; 496,22), cắt trên nhựa đường/bóng; vệt sơn trắng thật nằm lệch xuống và trái ở phần gần xe. Vẽ lại theo tâm vệt sơn và cung ảnh, không chỉ tăng số điểm. |
| 2 | QC1-09 | A08 | `traffic_light#24@(777,231)` | Chính | Box (777,0; 230,7)–(783,9; 242,0) bao tín hiệu người đi bộ có hình người sáng nhưng đang type=vehicle. Đổi type=pedestrian. Màu tín hiệu trắng và cách ánh xạ sang schema cần lead xác nhận riêng; chưa cộng thêm lỗi màu. |

## Đối chiếu các lỗi của phiếu revision 1

| Dòng phiếu cũ | Kết quả đối chiếu revision 2 |
| --- | --- |
| 1–2: hai lane suy đoán A05 | Đã bỏ cả hai; không yêu cầu vẽ lại trên đường không thấy sơn. Ngoại lệ thiếu L1/R1 vẫn chờ coach xác nhận. |
| 3, 5: đèn A08 tại (361,96), (670,156) | Đã green, phù hợp ảnh. |
| 4: đèn A08 tại (519,178) | **Đính chính nhận xét cũ:** yêu cầu bắt buộc green được rút lại. Phóng to đúng box thấy tín hiệu xanh thuộc box bên cạnh tại (527,184), không phải box này. Bản mới unknown chưa bị tính lỗi đã xác nhận. |
| 6: lane cung ngang giả B03 | Đã bỏ. |
| 7: biển Harlem River Dr / RFK Br | Đã information. |
| 8–9: drivable A08 bỏ vùng gần xe | Đã mở rộng; hai điểm minh họa cũ (600,480) và (1000,500) hiện nằm trong polygon tương ứng, phạm vi tiến sát phần đường trước capô. |
| 10: R1 B03 lệch vệt sơn | Chưa đổi hình học; còn lỗi ở danh sách trên. |

Phiếu revision 1 được giữ nguyên để truy vết, gồm kết luận lịch sử Rework 6 nặng/4 chính. Nhận xét green cho box (519,178) trong phiếu đó không còn được giữ như kết luận chắc chắn sau đối chiếu này. Không trừ cơ học số lỗi cũ để tạo điểm mới; mẫu revision 2 đã giảm từ 75 xuống 69 nhãn do xóa shape.

## Các ảnh đã tái kiểm

| Ảnh | Shape | Tag | Tổng | Ghi nhận |
| --- | ---: | ---: | ---: | --- |
| A02 | 6 | 4 | 10 | Chưa phát hiện lỗi chắc chắn trong mẫu; không chứng nhận toàn bộ dung sai hình học |
| A05 | 2 | 4 | 6 | Hai lane giả đã bỏ; chờ xác nhận ngoại lệ QC1-02 |
| A08 | 25 | 4 | 29 | Đèn/biển/vùng lái đã sửa nhiều mục; còn type đèn người đi bộ và ca mơ hồ |
| B01 | 3 | 4 | 7 | Chưa phát hiện lỗi chắc chắn trong các nhãn đã kiểm |
| B03 | 4 | 4 | 8 | Còn R1 lệch; đã xóa hai drivable và một lane L1 ngoài lane giả, cần chốt phạm vi |
| B05 | 5 | 4 | 9 | Chưa chốt lỗi; area_type vẫn chờ làm rõ |
| Tổng | 45 | 24 | 69 | |

## Kiểm file và truy vết

- Hai ZIP mở được, mỗi ZIP chỉ có annotations.xml, không kèm ảnh; schema shape/tag hợp lệ, không thiếu hoặc trùng slot. Camera của 28 ảnh khớp tên file; mỗi ảnh đủ bốn tag.
- Front: 16 ảnh, 155 shape, 64 tag. Fisheye: 12 ảnh, 37 shape, 48 tag. Tổng lô: 28 ảnh, 192 shape, 112 tag; không dùng 304 nhãn toàn lô làm mẫu số thay 69 nhãn đã kiểm.
- Front SHA-256: `69cba7705d5903d7205678de49802cc5b35957da42da68d8968a055e85a1968d`, 9430 byte.
- Fisheye SHA-256: `91980af0d2e6b54545f21209ced54af67a6b8e27dd873237d9496c71694c1417`, 5583 byte.
- Lint chạy lại khớp manifest: **2 nặng, 0 chính, 0 nhẹ**, đều QC1-02 ở A05 thiếu L1/R1 sau khi bỏ lane giả. Đây là cảnh báo máy chờ coach xác nhận không thấy vạch, không tự kết luận hai lỗi nặng thực tế. Manifest không được sửa tay về 0.

## Ca mơ hồ và câu hỏi cho lead

| Ảnh | Câu hỏi | Trạng thái |
| --- | --- | --- |
| A05 | Xác nhận không có vạch L1/R1 thấy được, ghi ngoại lệ QC1-02 theo SUBMISSION. Không suy vạch từ xe đỗ. | Chưa có xác nhận coach |
| A08 | Box (519,178) không bao tín hiệu xanh bên cạnh; có đủ căn cứ chọn off hoặc unknown? | Rút yêu cầu green; chưa chốt thêm lỗi |
| A08 | Tín hiệu người đi bộ (777,231) sáng trắng: schema không có white, cần chốt cách ánh xạ color. | Chờ lead; type=pedestrian đã rõ |
| A08 | Hai biển nhỏ quanh (698,183), (433,183) cần xác nhận chức năng để chốt group. | Chưa có lời đáp; không cộng lỗi |
| B03 | Hai drivable và lane L1 cũ đã bị xóa. Chốt nhóm nhãn coach giao cho fisheye_rear và vạch nào thấy được; không coi thao tác xóa là chứng minh đã đủ nhãn. | Chờ task plan/lead |
| B05 | Drivable alternative đúng với hướng lưu thông trong cảnh camera bên hông không? | Chờ ngữ cảnh/lead |
| A05/A08 | weather=clear có phù hợp mức mây của cảnh theo guideline không? | Chờ đối chiếu/giải thích |

## Quyết định

Lỗi nặng đã chốt trong mẫu: **0**. Lỗi chính: **2/69 = 2,90%**. Lỗi nhẹ đã chốt: **0**.

**Hold.** Tỷ lệ lỗi chính đã xác nhận chưa vượt 5%, nhưng còn câu hỏi ảnh hưởng kết luận, nhất là ngoại lệ QC1-02 A05 và phạm vi nhãn B03. Chưa đủ căn cứ Release. Hai lỗi chính vẫn cần phản hồi/sửa; nếu lead xác nhận thêm lỗi nặng hoặc tỉ lệ chính vượt 5% thì áp dụng Rework theo quy định.

Ưu tiên: sửa R1 B03, sửa type đèn người đi bộ A08, xin coach xác nhận A05 và các ca ảnh hưởng quyết định. Sau sửa giữ bản cũ, xuất lại ZIP/manifest với revision phù hợp cho lần bàn giao mới. Không gửi ảnh dữ liệu hoặc GT kèm phiếu.

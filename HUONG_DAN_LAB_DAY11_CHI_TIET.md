# Hướng dẫn chi tiết lab Ngày 11 Road Elements và SVM

Mục tiêu là gán nhãn đúng Road v1.0 trên CVAT của lớp, tự kiểm, kiểm chéo lô của một bạn khác, sửa theo QC và bàn giao **đúng 5 file** theo [SUBMISSION.md](SUBMISSION.md). Làm đủ **A01–A12 và B01–B08** trước; A13–A16 và B09–B12 là phần thưởng tùy chọn.

Các lệnh dưới đây dành cho **PowerShell trên Windows**, chạy tại thư mục gốc repo. Hướng dẫn chỉ làm việc trên máy và CVAT lớp; không cần commit, push hoặc tạo pull request. Phần nộp bài của repo không yêu cầu notebook huấn luyện SVM, file model hay slide. Hoạt động row-anchor ở cuối là tùy chọn.

## 1 Kết quả cuối cùng phải nộp

Dùng một tên viết liền, không dấu, không khoảng trắng xuyên suốt. Ví dụ theo tên trong folder: `hoangnhatanh`; nếu coach cấp mã tên khác thì thay đồng bộ. Tên bạn được kiểm phải là bạn thật được coach ghép, không dùng tên ví dụ trong bài nộp.

| File | Nội dung bắt buộc |
| --- | --- |
| `hoangnhatanh-front.zip` | Nhãn CVAT for images 1.1 của A01–A12 và ảnh A thêm đã làm; **không chứa ảnh** |
| `hoangnhatanh-fisheye.zip` | Nhãn CVAT for images 1.1 của B01–B08 và ảnh B thêm đã làm; **không chứa ảnh** |
| `hoangnhatanh-manifest.json` | Sinh bằng `road_check lint --manifest`, đúng tên người nộp và revision |
| `hoangnhatanh-qc1-for-TENBAN.md` | Phiếu QC 1 bạn thực hiện cho người khác, theo mẫu có sẵn; thay `TENBAN` trước khi nộp |
| `hoangnhatanh-report.md` | Báo cáo cá nhân theo mẫu, có số lượng nhãn thực, ca khó, phản hồi QC và ảnh chụp màn hình |

Nộp lên **kênh riêng coach chỉ định**, hạn **16:35 của buổi Ngày 11** theo SUBMISSION. Không tự suy ra ngày lịch từ tên folder. Không gửi GT, ảnh gốc, ZIP dữ liệu, toàn bộ repo hoặc file hướng dẫn này vào bộ 5 file.

## 2 Dữ liệu đang có và việc cần xác nhận

Gói `C:\Users\Laptop\Downloads\day11-data.zip` có 28 ảnh. Ảnh đã được giải nén vào `data/` trong repo, giữ nguyên tên:

| Nhóm | Số ảnh | Kích thước | Task |
| --- | ---: | --- | --- |
| A01–A12 | 12 | 1280 × 720 | `front` |
| A13–A16 | 4 | 1280 × 720 | `front-extra` |
| B01–B08 | 8 | 1280 × 966 | `fisheye` |
| B09–B12 | 4 | 1280 × 966 | `fisheye-extra` |

**Không giải nén nguyên ZIP đè lên repo:** ZIP còn có một `README.md` ở cấp gốc, có thể ghi đè tài liệu repo. Nếu làm lại trên máy khác, chỉ lấy thư mục `data/` từ ZIP. Dữ liệu đã có ở máy này nên không cần giải nén lại.

Hai loại manifest có vai trò khác nhau:

- `manifests/manifest_front.csv` và `manifests/manifest_fisheye.csv`: bảng chọn ảnh của coach. Front hiện chỉ có dòng tiêu đề; fisheye ghi trạng thái ứng viên chờ chốt.
- `hoangnhatanh-manifest.json`: mô tả **hai ZIP nhãn cuối cùng của bạn**, sinh sau export. Đây mới là manifest phải nộp; không nộp CSV thay thế.

Lấy địa chỉ CVAT, tài khoản, project và bảng kế hoạch ảnh đã chốt từ coach. Nếu project đã có ảnh thì đối chiếu tên với mục 6; không tự đổi ảnh, đổi slot hay tạo project trùng. Nếu project chưa có, nhờ coach xác nhận bộ ảnh này và quyền tạo task trước khi nhập. Chỉ dùng CVAT của lớp, không đưa ảnh lên `app.cvat.ai`; việc sử dụng và phân phối ảnh theo [ATTRIBUTION.md](ATTRIBUTION.md).

## 3 Chuẩn bị Python và thư mục làm việc

### 3.1 Kiểm môi trường

Mở Terminal trong VS Code, chọn PowerShell và chạy:

```powershell
Set-Location 'D:\K5-L23-Day11-2A202603183-HoangNhatAnh-Road-Elements-SVM'
python --version
$env:PYTHONPATH = 'src'
$env:PYTHONIOENCODING = 'utf-8'
python -m road_lab.road_check selftest
(Get-ChildItem -LiteralPath '.\data' -File | Where-Object { $_.Extension -in '.jpg', '.jpeg', '.png' }).Count
```

Cần Python ≥ 3.11; kết quả cuối selftest là **`SELFTEST PASS`**, số ảnh là **28**. Máy hiện có Python 3.13 và selftest đã chạy thành công. Các công cụ lab chỉ cần thư viện chuẩn, không cần cài OpenCV, scikit-learn hoặc pytest để làm bài. Nếu `python` không được nhận diện nhưng `py` dùng được, thay `python` bằng `py` trong toàn bộ lệnh.

Mỗi lần mở Terminal mới, đặt lại `PYTHONPATH` và `PYTHONIOENCODING`. Cú pháp `PYTHONPATH=src python3 ...` trong README là cho Bash; trên PowerShell dùng cách trên.

### 3.2 Tạo bản làm việc cho phiếu và báo cáo

Chạy một lần; `TENBAN` phải được thay bằng tên không dấu của bạn được ghép:

```powershell
$ten = 'hoangnhatanh'
$ban = 'TENBAN'
New-Item -ItemType Directory -Path '.\submissions' -Force | Out-Null
New-Item -ItemType Directory -Path '.\out\exports' -Force | Out-Null
if (-not (Test-Path ".\submissions\$ten-report.md")) {
    Copy-Item -LiteralPath '.\templates\report.md' -Destination ".\submissions\$ten-report.md"
}
if (-not (Test-Path ".\submissions\$ten-qc1-for-$ban.md")) {
    Copy-Item -LiteralPath '.\templates\qc1_form.md' -Destination ".\submissions\$ten-qc1-for-$ban.md"
}
```

Các file trong `submissions/` là bản để điền và nộp. Giữ nguyên mẫu gốc trong `templates/`. Nếu chưa biết bạn được ghép, tạo phiếu sau khi có tên; không nộp tên `TENBAN`.

## 4 Kiểm project và taxonomy trên CVAT

### 4.1 Mở project đúng

1. Đăng nhập CVAT lớp, vào **Projects**, mở `d11-road-<tên>`.
2. Kiểm bốn task `front`, `front-extra`, `fisheye`, `fisheye-extra`. Mỗi task có thể có nhiều job; phải làm hết job thuộc phần bạn nộp.
3. Đối chiếu 6 label vẽ và 4 tag với bảng dưới. Nếu cần tạo taxonomy và được coach giao quyền, dùng nội dung [configs/road_v1_labels.json](configs/road_v1_labels.json) tại **Labels → Raw**. Không tự đổi tên hoặc thêm thuộc tính.
4. Chụp tab Labels để dùng trong báo cáo, ghi phiên bản ở **Help → About**.

Có thể in taxonomy chuẩn để so:

```powershell
python -m road_lab.taxonomy
```

### 4.2 Sáu label vẽ

| Label | Công cụ CVAT | Thuộc tính hợp lệ |
| --- | --- | --- |
| `lane` | Polyline | `position`: L1/R1/L2/R2/other; `style`: solid/dashed; `color`: white/yellow/other; `kind`: single/double |
| `road_curb` | Polyline | Không có thuộc tính riêng |
| `crosswalk` | Polygon | Không có thuộc tính riêng |
| `drivable` | Polygon | `area_type`: direct/alternative |
| `traffic_light` | Rectangle | `color`: red/yellow/green/off/unknown; `type`: vehicle/pedestrian/beacon; `facing`: front/side/back; `edge_zone`: checkbox |
| `traffic_sign` | Rectangle | `group`: prohibitory/mandatory/warning/information/supplementary/other; `facing`: front/side/back; `edge_zone`: checkbox |

`occluded` là cờ có sẵn của CVAT, không tạo thêm thuộc tính select cùng tên. Trong XML, Rectangle được lưu bằng thẻ `<box>` là bình thường.

### 4.3 Bốn tag trên từng ảnh

Mỗi tag là **một label kiểu Tag, có thuộc tính select tên `value`**. Mỗi ảnh đặt một lần cho mỗi tag, không tạo các label riêng mang tên `day`, `clear` hoặc `front`.

| Tag | Giá trị |
| --- | --- |
| `camera` | front/fisheye_front/fisheye_rear/fisheye_left/fisheye_right |
| `timeofday` | day/dusk/night |
| `weather` | clear/overcast/rain/fog/other |
| `source` | real/simulated |

Ảnh BDD100K và WoodScape trong gói này dùng **`source=real`**. Ảnh trong tài liệu minh họa CVAT của repo là tổng hợp nên ví dụ ở đó mang `simulated`; không chép tag đó sang bài thật.

## 5 Quy trình thao tác trên từng ảnh

### 5.1 Đặt tag trước

1. Mở job, kiểm tên ảnh và slot.
2. Chọn công cụ **Tag**, đặt `camera` trước; mở **Objects → Details**, sửa `value`.
3. Làm tương tự cho `timeofday`, `weather`, `source`.
4. Mở lại từng tag để kiểm. CVAT tự chọn giá trị đầu danh sách; giá trị hợp lệ chưa có nghĩa là đúng ảnh.

Ánh xạ camera theo tên file:

| Tên ảnh | `camera.value` |
| --- | --- |
| A01 đến A16 | `front` |
| B có `_FV.png` | `fisheye_front` |
| B có `_RV.png` | `fisheye_rear` |
| B có `_MVL.png` | `fisheye_left` |
| B có `_MVR.png` | `fisheye_right` |

Weather: trời quang hoặc ít mây và khô → `clear`; âm u khô → `overcast`; đang mưa hoặc đường ướt rõ do mưa → `rain`; sương/khói bụi giảm tầm nhìn → `fog`; trường hợp còn lại → `other` và ghi ca khó. Phân biệt `day`, `dusk`, `night` từ cảnh thực tế, không suy từ số slot.

### 5.2 Vẽ lane

1. Xác định làn xe đang đi. Hai vạch sát làn đó là L1 bên trái và R1 bên phải; vạch kế tiếp bên ngoài mới là L2, R2.
2. Chọn **Polyline → lane → Shape**. Phóng to, bấm điểm đầu trên **tâm vệt sơn gần xe**, rồi bấm tiếp ra xa theo vạch.
3. Đặt thêm điểm tại chỗ cong để sai lệch tối đa so với tâm vạch **≤ 5 px** trên ảnh gốc. Zoom chỉ giúp nhìn; không đổi ngưỡng pixel gốc.
4. Vạch đứt dùng **một polyline xuyên các đoạn đứt**, không vẽ một shape cho mỗi vệt sơn.
5. Chỗ che ngắn có đủ căn cứ nối tiếp: nối qua và bật **Occluded**. Dừng ở điểm xa nhất còn nhận ra vạch, không kéo tới chân trời.
6. Kết thúc shape. Bản CVAT v2.82.0 được minh họa trong repo dùng phím **N**; nếu bản lớp khác, kiểm tooltip/phím tắt và ghi cách thực tế vào báo cáo.
7. Mở Details và sửa đủ `position`, `style`, `color`, `kind`. Lane mới thường mang `L1 / solid / white / single`; phải kiểm từng trường.

Vạch đôi phải có `kind=double`; không dùng mặc định `single`. Trường hợp đôi một liền một đứt không có style riêng trong Road v1.0: ghi cách chọn và lý do vào bảng ca khó để coach xác nhận.

Nếu không thấy vạch sơn, **không tự vẽ L1/R1** từ mép xe đỗ, bóng râm hoặc đường tưởng tượng. Có thể dùng `road_curb` cho mép đường không sơn khi nằm trong nhóm label được giao. Lint có thể báo QC1-02 dù ảnh không có vạch: nhờ coach nhìn ảnh, xác nhận miễn và ghi slot, lý do vào phiếu; không tự xóa cảnh báo bằng nhãn giả.

### 5.3 Vẽ road curb và crosswalk

- `road_curb`: polyline theo mép vỉa hè/lề nơi không có vạch sơn. Mép có vạch sơn là `lane`, không tự đổi thành curb.
- `crosswalk`: một polygon kín bao **cả vùng vạch qua đường**, gồm các dải sơn và khoảng giữa; không gán mỗi dải thành lane.
- Vạch ô đỗ không gán lane. Không gán xe, người hoặc vật cản vì ngoài taxonomy của lab.

### 5.4 Vẽ drivable

1. Chọn **Polygon → drivable**.
2. Vẽ vùng làn đang đi, đặt `area_type=direct`.
3. Nếu được yêu cầu và có làn cùng chiều bên cạnh, tạo vùng riêng với `area_type=alternative`.
4. Bám giới hạn mặt đường. Loại lề, vỉa hè, capô/thân xe mình, vùng phía sau cọc và viền đen fisheye. Dải đỗ sát lề không thuộc `direct`.
5. Theo quy tắc “đi được nếu đường trống”, **không khoét lỗ quanh xe khác**. Xe khác che mặt đường không làm vùng đó tự động mất tính đi được; thân xe mình vẫn phải loại.

### 5.5 Vẽ đèn và biển

Trong nhóm label được giao cho ảnh, gán đủ cụm đèn và tấm biển **cao từ 8 px trở lên**:

- Rectangle ôm cả **vỏ cụm đèn** hoặc **tấm biển**, sát mép; không gồm cần treo/cột.
- Đèn: `color` là màu đang sáng ở ảnh này, `off` nếu tắt, `unknown` nếu không xác định được; đặt `type` và `facing` theo thực tế. Sai màu đèn hướng về xe (`facing=front`) là lỗi nặng.
- Biển phụ có box riêng, `group=supplementary`.
- Không gán hình biển in trên xe hoặc ảnh phản chiếu.
- `facing=front` là mặt đối tượng hướng về camera; `side` nhìn ngang; `back` nhìn mặt sau. Không hiểu `front` ở đây là tag camera của cả ảnh.

Nhóm biển Mỹ phân theo **chức năng**, theo quy ước riêng của lab:

| `group` | Ví dụ trong guideline |
| --- | --- |
| `prohibitory` | Stop, Yield, Speed Limit, Do Not Enter, No Parking |
| `mandatory` | One Way, Keep Right, Left Turn Only |
| `warning` | Biển thoi vàng cảnh báo, biển công trường cam |
| `information` | Biển chỉ hướng/lối ra, số đường, dịch vụ |
| `supplementary` | Tấm phụ đi kèm biển chính |
| `other` | Chưa xếp được; ghi ca khó và lý do |

### 5.6 Quy tắc riêng cho fisheye

- Giữ nguyên ảnh méo, vẽ theo **cung trong ảnh**, không nắn ảnh và không dùng đoạn thẳng nối hai đầu khi vạch cong.
- Thêm điểm dày ở vòng ngoài để bám tâm vạch ≤ 5 px.
- Camera hai bên (`fisheye_left`, `fisheye_right`): **mọi lane đều `position=other`**. Front/rear mới dùng L1/R1/L2/R2 khi phù hợp.
- Không gán thân xe mình và vùng tối viền ảnh. Các dải trắng vạch qua đường vẫn là `crosswalk`.
- Đèn/biển có tâm box ngoài 80% bán kính vòng ảnh: bật `edge_zone`. Box vẫn là rectangle thẳng, không xoay theo biển méo.

Để hiểu cảnh báo edge_zone của checker hiện tại: tâm ảnh là `(W/2, H/2)`, bán kính được lấy là `W/2`. Với ảnh B 1280 × 966, tâm là `(640,483)` và ngưỡng là 512 px. Tâm box `(cx,cy)` có khoảng cách `sqrt((cx-640)^2 + (cy-483)^2) > 512` thì checker kỳ vọng `edge_zone=true`. Đây là cách tính của lab, không phải thao tác đo hiệu chuẩn camera.

### 5.7 Lưu và tự kiểm trước khi chuyển ảnh

Chỉ gán nhóm label được yêu cầu trong **bảng kế hoạch ảnh coach đã chốt**. Trước khi chuyển ảnh, kiểm bốn tag, tâm và chiều polyline, thuộc tính, vùng lái, box rồi **Ctrl+S**. Hoàn thành A01 thì nhờ coach kiểm sớm để tránh lặp lỗi sang cả lô.

Thao tác CVAT có ảnh minh họa tại [hướng dẫn bằng hình](docs/huong_dan_cvat_bang_hinh.md). Hình dưới là **ảnh tổng hợp minh họa cách sửa thuộc tính**, không dùng làm bằng chứng hoàn thành bài thật:

![Sửa style của lane trong Details](docs/images/huong-dan-cvat/07-lane-style-select.jpg)

## 6 Bảng ảnh thực tế và điểm cần kiểm

Tên bên dưới khớp gói ZIP đã cung cấp. **Trọng tâm A** lấy từ luật chọn slot trong mã và checkpoint; **trọng tâm B** lấy từ CSV ứng viên. Đây là gợi ý so ảnh, không thay bảng label coach đã chốt. Nhất là khi mô tả slot khác ảnh, gán theo đối tượng thật và ghi ca khó thay vì tạo nhãn cho vật không có.

### 6.1 Front bắt buộc

| Slot | File trong data | Trọng tâm cần đối chiếu |
| --- | --- | --- |
| A01 | `A01_bc5fadac-fa6dd225.jpg` | Phân biệt vị trí các vạch; kiểm tâm, chiều và thuộc tính ảnh đầu |
| A02 | `A02_b3c26aff-63a0d7d6.jpg` | Vạch đứt dài phải là một polyline |
| A03 | `A03_b1c66a42-6f7d68ca.jpg` | Kiểm phần bị che; chỉ nối và bật occluded khi có căn cứ |
| A04 | `A04_b7a8e795-0ff55db2.jpg` | Vàng đôi; direct/alternative và capô ở đáy ảnh |
| A05 | `A05_b2576c8c-2de20b8a.jpg` | Vùng lái và mép lề; ảnh ít vạch sơn rõ, lưu ý ngoại lệ L1/R1 |
| A06 | `A06_b8c66e31-37045eec.jpg` | Vạch qua đường, ranh giới vùng lái; tránh nhầm bóng cầu với vạch |
| A07 | `A07_b41ace08-c8468690.jpg` | Slot dự kiến đèn nhưng ảnh có xe gần và cầu; phải phóng to xác minh, không tự tạo đèn |
| A08 | `A08_c0deebc1-16cfa791.jpg` | Đủ cụm đèn thực sự nhìn thấy; box sát vỏ, màu hiện thời |
| A09 | `A09_c7e6d5d1-c8af9fe3.jpg` | Biển báo; phân biệt biển thật và hình trên phương tiện |
| A10 | `A10_b6ff1331-f6d3fba8.jpg` | Biển chính/phụ; kiểm biển nhỏ và tách box khi có |
| A11 | `A11_c1f35f18-b529c043.jpg` | Cảnh chạng vạng; tag và màu đèn, tránh chép thuộc tính mặc định |
| A12 | `A12_c0f978fa-9b4e623f.jpg` | Phố và giao lộ; kiểm phối hợp label được giao |

### 6.2 Fisheye bắt buộc

| Slot | File trong data | Camera | Điểm cần kiểm |
| --- | --- | --- | --- |
| B01 | `B01_02688_FV.png` | fisheye_front | Ảnh thấy rõ crosswalk; không biến các dải trắng thành lane |
| B02 | `B02_03177_FV.png` | fisheye_front | Tìm đèn/biển được giao, kiểm box sát mép và edge_zone |
| B03 | `B03_04267_RV.png` | fisheye_rear | Làn theo cung và thuộc tính; loại phần thân xe dưới ảnh |
| B04 | `B04_05535_RV.png` | fisheye_rear | Crosswalk; drivable dừng ở thân xe |
| B05 | `B05_05780_MVL.png` | fisheye_left | Crosswalk và curb; lane nếu có phải position=other |
| B06 | `B06_04068_MVL.png` | fisheye_left | Ảnh chủ yếu là crosswalk; chỉ gán lane khi thật sự có vạch làn |
| B07 | `B07_01216_MVR.png` | fisheye_right | Crosswalk và curb; kiểm ranh giới đường và thân xe |
| B08 | `B08_05783_MVR.png` | fisheye_right | Crosswalk rõ; drivable theo phần đường, lane nếu có là other |

Với camera bên hông, luôn giữ `position=other` cho lane. Lint hiện chỉ nhắc thiếu L1/R1 khi tag camera là `front`; nếu ảnh B bị nhắc như vậy, kiểm và sửa tag camera trước. Với ảnh front thật sự không thấy vạch, nhờ coach xác nhận ngoại lệ thay vì tạo lane giả.

### 6.3 Ảnh thêm nếu còn thời gian

| Slot | File | Điểm chú ý |
| --- | --- | --- |
| A13 | `A13_c179d309-b446fda3.jpg` | Mưa/đường ướt, tag weather, không nhầm phản chiếu |
| A14 | `A14_ca04d99f-c1282bb7.jpg` | Đêm, màu đèn và vạch nhìn thấy được |
| A15 | `A15_be85bd88-d849c034.jpg` | Slot dự kiến vạch mờ; chỉ gán theo ảnh thật |
| A16 | `A16_b6f77fda-33d2260f.jpg` | Giao lộ, phân biệt vạch dọc, crosswalk và vạch khác |
| B09 | `B09_02686_FV.png` | fisheye_front; đèn/biển ở mép nếu có, crosswalk không phải lane |
| B10 | `B10_01153_MVL.png` | fisheye_left; crosswalk |
| B11 | `B11_07282_RV.png` | fisheye_rear; ảnh ô đỗ, **không gán vạch ô đỗ thành lane** |
| B12 | `B12_05885_MVR.png` | fisheye_right; crosswalk và drivable |

Tám ảnh thêm tối đa +4 điểm, mỗi ảnh đạt +0,5 và tổng không vượt 100. Hoàn thành phần bắt buộc, QC và file nộp trước; không để thiếu phiếu/báo cáo vì làm phần thêm.

## 7 Tự kiểm và hoàn tất job

1. Duyệt lại 20 ảnh bắt buộc theo thứ tự tên. Kiểm đủ bốn tag, đúng camera, tên và thuộc tính label đúng taxonomy.
2. Kiểm lane: gần ra xa, bám tâm, vạch đứt không bị tách, có occluded khi nối chỗ che, không thiếu vạch thấy được.
3. Kiểm drivable không tràn lề/thân xe, direct/alternative đúng; đèn và biển đủ, box sát, biển phụ riêng.
4. Kiểm fisheye theo cung, camera bên hông dùng `other`, không gán ô đỗ thành lane.
5. **Save → Menu → Finish the job → Continue** theo giao diện minh họa. Quay lại task, xác nhận **State=completed** cho mọi job thuộc bài nộp. Chỉ bấm Save chưa đủ checkpoint completed.
6. Nếu sửa sau khi hoàn tất, lưu và kiểm lại trạng thái trước lần export tiếp theo.

## 8 Kiểm chéo và điền Phiếu QC 1

Phiếu bạn nộp là phiếu **bạn kiểm cho người khác**. Phiếu người khác kiểm bài bạn được dùng để phản hồi ở mục 4 báo cáo của bạn; không tráo hai phiếu.

### 8.1 Chọn mẫu và ghi số nhãn

1. Nhận lô từ bạn được coach ghép qua kênh được chỉ định, ghi đúng tên file và revision của lô đang kiểm.
2. Kiểm **6 ảnh**, ghi rõ slot; nếu coach không chỉ định mẫu thì chọn có front, fisheye, camera bên hông và các ca khó. Ảnh thêm kiểm riêng.
3. Đếm số shape và tag thực sự kiểm. Ví dụ 6 ảnh có tổng 56 shape và 24 tag thì **80 nhãn đã kiểm**.
4. Điền đủ **12 dòng mã**, có số mục kiểm và số lỗi. Mã không có đối tượng áp dụng ghi `0` cùng lý do ngắn; không để trống và không bịa số.

Một nhãn có thể được kiểm theo nhiều mã: chẳng hạn một lane được xem về vị trí, hình học và style. Vì vậy **không cộng cột Số mục kiểm của 12 mã để làm mẫu số**; mẫu số là số nhãn đã kiểm, đếm shape + tag một lần. Nếu phát hiện đối tượng bị bỏ sót, ghi rõ đối tượng phải có và cách đếm trong phiếu để coach đối chiếu.

### 8.2 Nội dung 12 mã

| Mã | Kiểm gì | Mức theo quy tắc lab |
| --- | --- | --- |
| QC1-01 | Tag camera thiếu/sai; kiểm đủ các tag khác | Camera: nặng; thiếu timeofday/weather/source: nhẹ |
| QC1-02 | L1/R1 khi thật sự thấy được | Nặng; ca không thấy vạch phải có coach xác nhận |
| QC1-03 | position, gán nhầm vật khác thành vạch | Nặng |
| QC1-04 | Tâm vạch ≤ 5 px, chiều gần ra xa, điểm dừng | Chính |
| QC1-05 | Vạch đứt một polyline, nối chỗ che và occluded | Chính |
| QC1-06 | style, color, kind | Chính |
| QC1-07 | drivable và kiểm phạm vi curb/crosswalk | Chính |
| QC1-08 | Đèn, màu và box | Sai màu đèn facing front: nặng; trường hợp khác: chính |
| QC1-09 | facing, type, group, tách biển phụ | Chính |
| QC1-10 | Đủ biển; không gán hình in/phản chiếu | Chính |
| QC1-11 | Cung fisheye, loại thân xe, edge_zone | Chính; sai position xét thêm QC1-03 |
| QC1-12 | Mở file, đủ hai ZIP, schema, ảnh trùng/ngoài lô | Không mở/thiếu ZIP: nặng; lỗi schema/ảnh trùng: nhẹ theo thang |

Theo [thang mức lỗi](docs/qc_severity_road_v1.0.md), khi phân vân mức thì chọn mức cao hơn, ghi ca mơ hồ cho lead. Cảnh báo box dưới 8 px của lint chỉ là nhắc tự kiểm, không phải lỗi chấm bắt buộc cho đối tượng nhỏ đó.

### 8.3 Ghi từng lỗi và kết luận

Mỗi lỗi một dòng, có **mã, slot, label và id/vị trí tìm lại được, mức, mô tả**. Ví dụ minh họa, chỉ dùng nếu thật sự phát hiện:

| Mã | Ảnh | Nhãn | Mức | Mô tả |
| --- | --- | --- | --- | --- |
| QC1-04 | A02 | lane id 17 | chính | Điểm đầu ở xa, polyline vẽ ngược chiều |

Ghi cả ảnh đã kiểm không có lỗi, chẳng hạn một dòng ghi chú “Axx: đã kiểm ... nhãn, không phát hiện lỗi”. Coach đối chiếu 3 lỗi được ghi và 1 ảnh được ghi không lỗi; không tạo lỗi giả nếu thực tế ít hơn.

Tính `tỉ lệ lỗi chính = số lỗi chính / số nhãn đã kiểm × 100%`:

- Có ít nhất một lỗi nặng → **Rework**.
- Không có lỗi nặng nhưng tỉ lệ lỗi chính **> 5%** → **Rework**.
- Còn ca mơ hồ chờ lead ảnh hưởng kết luận, chưa thuộc hai điều kiện trên → **Hold**.
- Các trường hợp còn lại → **Release**.

Ví dụ 4 lỗi chính/80 nhãn = 5% không vượt ngưỡng; nếu không lỗi nặng, không ca chờ lead thì Release. 5/80 = 6,25% thì Rework. Không chia cho 6 ảnh. Lint không thể thay việc nhìn ảnh hoặc tự quyết định Hold.

## 9 Export nhãn đúng định dạng

### 9.1 Export đủ dữ liệu

1. Kiểm mọi job đã completed và lưu xong.
2. Ưu tiên export **cấp task** để lấy hết job của task đó. Nếu task chỉ có một job thì export job bao phủ đủ ảnh cũng được; luôn đối chiếu danh sách slot sau tải.
3. Chọn **CVAT for images 1.1**, tắt **Save images**.
4. Chờ export hoàn thành, tải ZIP từ **Requests**.
5. Với phần bắt buộc, lưu thành `submissions/hoangnhatanh-front.zip` và `submissions/hoangnhatanh-fisheye.zip`. Không thêm `-final`, `-v2`, `(1)` vào tên file.

![Chọn định dạng export và tắt Save images trên bản CVAT minh họa](docs/images/huong-dan-cvat/13-export-dialog.jpg)

ZIP nhãn cần có **một `annotations.xml` bao phủ tất cả ảnh tương ứng**. Không đổi đuôi ZIP COCO thành ZIP CVAT; không nén ảnh gốc để thay file nhãn. Mở ZIP bằng trình quản lý nén, kiểm có XML và không có `.jpg`/`.png`.

### 9.2 Gộp ảnh thêm hoặc nhiều bản export nếu cần

Nếu chỉ làm phần bắt buộc và mỗi task đã được export đầy đủ thì **bỏ qua bước gộp**.

Checker trong repo chỉ lấy **`annotations.xml` đầu tiên** tìm thấy trong một ZIP. Vì vậy bỏ hai ZIP con hoặc hai XML khác thư mục vào ZIP cuối **không gộp được nhãn**. Phải ghép các phần tử `<image>` vào một XML, giữ nguyên tên ảnh và nhãn, đặt lại image id duy nhất.

Đoạn dưới là công cụ hỗ trợ đóng gói dựa trên cấu trúc CVAT trong repo. Dùng khi mọi bản export có cùng taxonomy Road v1.0. Lưu bản export gốc vào `out/exports/`, đặt tên `front-core.zip`, `front-extra.zip`, `fisheye-core.zip`, `fisheye-extra.zip`. Nếu task có nhiều export job thay vì một export task, thay danh sách đầu vào cho đủ job. Không đưa cùng ảnh vào hai nguồn.

Chạy tại gốc repo trong PowerShell; đoạn này chỉ tạo ZIP cuối sau khi đã kiểm toàn bộ đầu vào của nhóm tương ứng:

```powershell
@'
from copy import deepcopy
from pathlib import Path
from zipfile import ZipFile, ZIP_DEFLATED
import xml.etree.ElementTree as ET

def merge_cvat(parts, destination):
    destination = Path(destination)
    if destination.exists():
        raise ValueError(f"Output exists; archive it first: {destination}")
    roots = []
    for part in parts:
        with ZipFile(part) as z:
            names = [n for n in z.namelist()
                     if n.rsplit('/', 1)[-1] == 'annotations.xml']
            if len(names) != 1:
                raise ValueError(f"Expected one annotations.xml: {part}")
            root = ET.fromstring(z.read(names[0]))
        if root.tag != 'annotations' or root.findtext('version') != '1.1':
            raise ValueError(f"Not CVAT images 1.1: {part}")
        if root.find('track') is not None or not root.findall('image'):
            raise ValueError(f"Video or empty export: {part}")
        roots.append(root)
    output = ET.Element('annotations')
    ET.SubElement(output, 'version').text = '1.1'
    meta = roots[0].find('meta')
    if meta is not None:
        output.append(deepcopy(meta))
    seen = set()
    count = 0
    for root in roots:
        for image in root.findall('image'):
            name = image.get('name')
            if not name or name in seen:
                raise ValueError(f"Missing or duplicate image name: {name}")
            seen.add(name)
            item = deepcopy(image)
            item.set('id', str(count))
            output.append(item)
            count += 1
    payload = ET.tostring(output, encoding='utf-8', xml_declaration=True)
    destination.parent.mkdir(parents=True, exist_ok=True)
    with ZipFile(destination, 'x', ZIP_DEFLATED) as z:
        z.writestr('annotations.xml', payload)
    print(destination, count, 'images')

merge_cvat(
    ['out/exports/front-core.zip', 'out/exports/front-extra.zip'],
    'submissions/hoangnhatanh-front.zip')
merge_cvat(
    ['out/exports/fisheye-core.zip', 'out/exports/fisheye-extra.zip'],
    'submissions/hoangnhatanh-fisheye.zip')
'@ | python -
```

Nếu chỉ có ảnh thêm của một nhóm, chỉ chạy lời gọi gộp nhóm đó. Không thêm ảnh extra trống cho đủ 16/12; khai báo đúng số đã làm. Metadata task được giữ từ bản đầu để giữ khai báo label; **số ảnh gộp phải kiểm theo `<image>` và manifest mới**, không theo trường size của task cũ trong metadata. Đối chiếu nhãn hai ảnh mẫu với CVAT sau gộp và chạy lại các bước bên dưới.

## 10 Chạy lint và tạo manifest bàn giao

Chỉ chạy sau khi hai ZIP cuối đã nằm trong `submissions/`:

```powershell
$env:PYTHONPATH = 'src'
$env:PYTHONIOENCODING = 'utf-8'
$ten = 'hoangnhatanh'
python -m road_lab.road_check lint ".\submissions\$ten-front.zip" ".\submissions\$ten-fisheye.zip" --manifest ".\submissions\$ten-manifest.json" --name $ten --revision 1
```

Đọc từng dòng kết quả, quay lại CVAT sửa và export lại nếu cần. **Không có lỗi nặng** là điều kiện nộp, ngoại trừ QC1-02 ở ảnh không thấy vạch đã được coach xác nhận và ghi vào phiếu. Manifest vẫn có thể đếm cảnh báo miễn này ở `lint.nặng`; không sửa tay số đếm.

**Mã thoát 0 không có nghĩa bài đã đạt:** mã hiện tại trả 0 khi file đọc được dù vẫn có cảnh báo. Lint không có GT, không chấm điểm, không biết chắc vạch có tồn tại, không kiểm hết box thiếu hoặc nhãn hình học sai. Phải tự kiểm hình, đối chiếu đủ slot và dùng QC chéo.

### 10.1 Đọc manifest

```powershell
$m = Get-Content ".\submissions\$ten-manifest.json" -Raw -Encoding UTF8 | ConvertFrom-Json
$m | Select-Object name, revision, format, guideline, created_at
$m.files | Format-List file, images, images_all_tags, images_unlabelled, contains_images, sha256, shapes_by_label
$m.lint | Format-List
```

Cần kiểm:

- `name` đúng; `revision=1` ở lần đầu; `format=CVAT for images 1.1`, `guideline=Road v1.0`.
- `files` có đúng hai ZIP cuối, `contains_images=false` ở cả hai.
- Nếu chỉ phần bắt buộc: front 12 ảnh, fisheye 8 ảnh. Làm toàn bộ phần thêm: front 16, fisheye 12. Làm một phần thêm thì tăng đúng số đã hoàn thành.
- Mỗi file `images_all_tags=images`, `images_unlabelled=0`. Hai chỉ số này chưa chứng minh nhãn đúng: một ảnh chỉ có tag vẫn có thể không bị đếm là unlabelled; kiểm shape từng ảnh theo nhiệm vụ.
- `shapes_by_label` khớp thống kê báo cáo. `created_at` dùng UTC; thời điểm export trên CVAT ghi riêng theo báo cáo.

Sau mỗi lần đổi nội dung ZIP, phải chạy tạo lại manifest để SHA256 khớp. Không tự điền manifest hoặc dùng manifest cũ cho ZIP mới.

### 10.2 Kiểm đủ slot và thống kê shape thật

Chạy đoạn đọc dưới đây. Đoạn này không sửa nhãn; nó phát hiện thiếu ảnh bắt buộc, slot trùng, tên ảnh không thuộc nhóm và in số shape/tag từng ảnh:

```powershell
@'
from collections import Counter
from pathlib import Path
from road_lab.cvat_xml import load
from road_lab.taxonomy import image_slot

name = 'hoangnhatanh'
for group, prefix, required, maximum in (
    ('front', 'A', 12, 16), ('fisheye', 'B', 8, 12)
):
    path = Path('submissions') / f'{name}-{group}.zip'
    images = load(path)
    slots = [image_slot(im.name) for im in images]
    counts = Counter(slots)
    missing = [f'{prefix}{n:02d}' for n in range(1, required + 1)
               if f'{prefix}{n:02d}' not in counts]
    allowed = {f'{prefix}{n:02d}' for n in range(1, maximum + 1)}
    duplicates = [slot for slot, n in counts.items() if n > 1]
    unexpected = [im.name for im in images if image_slot(im.name) not in allowed]
    print('\nFILE', path.name)
    print('MISSING', missing, 'DUPLICATE', duplicates, 'UNEXPECTED', unexpected)
    for im in sorted(images, key=lambda im: im.name):
        print(im.name, 'shapes=', len(im.shapes), 'tags=', len(im.tags),
              dict(Counter(s.label for s in im.shapes)))
    for tier, selected in (
        ('core', [im for im in images if image_slot(im.name) in
                  {f'{prefix}{n:02d}' for n in range(1, required + 1)}]),
        ('extra', [im for im in images if image_slot(im.name) in
                   {f'{prefix}{n:02d}' for n in range(required + 1, maximum + 1)}])
    ):
        print(tier, len(selected),
              dict(Counter(s.label for im in selected for s in im.shapes)))
'@ | python -
```

`MISSING`, `DUPLICATE`, `UNEXPECTED` phải đều `[]` cho bộ ảnh dự kiến. Đây là kiểm bổ sung số lượng; vẫn mở hai ảnh mỗi ZIP, đối chiếu tên và shape với CVAT. Giữ nguyên tên đầy đủ `{slot}_{tên gốc}` trong XML.

## 11 Điền báo cáo cá nhân theo mẫu

Giữ đủ cấu trúc [templates/report.md](templates/report.md), điền bản trong `submissions/`:

| Phần | Nội dung cần điền |
| --- | --- |
| Thông tin đầu | Họ tên, mã học viên, phiên bản CVAT thật, Road v1.0, revision, thời điểm export và số lần Rework |
| 1 Kiểm project | Ba dòng kiểm label/kiểu, thuộc tính/giá trị, bốn tag; cách kết thúc polyline và cách đặt tag trên bản CVAT đang dùng |
| 2 Số lượng | Đếm theo ZIP cuối bằng mục 10.2; tách core/extra thành bốn dòng, điền số thật; không chép số 12/8 nếu thực tế chưa đủ |
| 3 Ca khó | Slot, label, các phương án, lựa chọn, lý do; đặc biệt ảnh thiếu vạch, slot khác mô tả, biển không chắc nhóm và vạch đôi hỗn hợp |
| 4 QC nhận được | Tên người kiểm bài mình, quyết định; từng mã lỗi, số lỗi, đồng ý hay không, đã sửa; ghi revision mới và phiếu làm căn cứ |
| 5 Kiểm export | Mỗi ZIP một dòng; mở lại được, hai ảnh đối chiếu số shape job/file, camera tag có trong XML |
| 6 Ảnh chụp | Bốn hình thật như danh sách dưới |
| 7 Tự đánh giá | Mã lỗi hay mắc, cách tránh; một lỗi cụ thể làm model học sai như thế nào |
| 8 Row-anchor | Tùy chọn; nếu không làm, ghi không thực hiện phần tùy chọn |

SUBMISSION nhắc bảng kiểm ở mục 1, còn mẫu hiện tại đặt bảng **kiểm file export ở mục 5**. Để đủ nội dung, điền cả mục 1 kiểm project và mục 5 kiểm file, giữ số mục của mẫu.

Bốn ảnh chụp cần có ở mục 6:

1. Tab Labels của project thật.
2. Một ảnh front đã gán đủ các label được giao, thể hiện làn/vùng lái/đèn hoặc biển khi có.
3. Một ảnh fisheye đã gán, phóng to đoạn polyline cong.
4. Một đoạn XML của bài thật có `<polyline label="lane">`.

Mở ZIP bằng trình quản lý nén để xem `annotations.xml`; không sửa XML chỉ để có hình đẹp. Thẻ camera có dạng:

```xml
<tag label="camera">
  <attribute name="value">front</attribute>
</tag>
```

Trong Markdown, ảnh chụp phải hiển thị hoặc có đường dẫn người chấm truy cập được. **Không dùng đường dẫn `C:\...` hoặc ảnh chỉ lưu trên máy bạn**. Dùng chức năng lưu/chèn ảnh của kênh nộp được coach cho phép, rồi dẫn ảnh trong báo cáo, ví dụ `![Project Labels](DUONG_DAN_ANH_COACH_TRUY_CAP_DUOC)`, thay bằng URL thật. Vì yêu cầu đúng 5 file và hai ZIP nhãn không được chứa ảnh, nếu kênh không hỗ trợ ảnh/URL truy cập được thì nhờ coach chốt cách đính kèm; không tự thêm ZIP ảnh hay đổi báo cáo sang định dạng khác. Không dùng ảnh tổng hợp minh họa của repo thay ảnh chụp bài đã làm.

Ví dụ tự đánh giá, phải điều chỉnh theo lỗi thật: “QC1-04: điểm đầu vạch R1 lệch khỏi tâm làm đích tọa độ làn gần xe lệch, model dự đoán vị trí vạch sai ở vùng quan trọng cho giữ làn. Tôi đã phóng to, đặt lại điểm đầu và rà chiều polyline trước khi lưu.”

## 12 Sửa sau QC và nộp lại

**Trước khi thay hai ZIP hoặc ghi đè manifest**, giữ bản revision 1:

```powershell
$ten = 'hoangnhatanh'
$archive = '.\submissions\r1'
if (Test-Path -LiteralPath $archive) { throw 'r1 đã tồn tại; kiểm bản lưu trước khi tiếp tục.' }
New-Item -ItemType Directory -Path $archive | Out-Null
Copy-Item -LiteralPath ".\submissions\$ten-front.zip" -Destination $archive
Copy-Item -LiteralPath ".\submissions\$ten-fisheye.zip" -Destination $archive
Copy-Item -LiteralPath ".\submissions\$ten-manifest.json" -Destination ".\submissions\r1\$ten-manifest-r1.json"
```

Sau đó sửa nhãn trên CVAT theo phiếu nhận được, lưu, kiểm completed, export lại. Giữ tên cuối **`hoangnhatanh-front.zip` và `hoangnhatanh-fisheye.zip`**, không đổi thành `-r2.zip`. Nếu dùng bước gộp, lưu bản ZIP cuối cũ rồi tạo bản gộp mới, vì công cụ gộp chủ động từ chối ghi đè file đã có.

```powershell
python -m road_lab.road_check lint ".\submissions\$ten-front.zip" ".\submissions\$ten-fisheye.zip" --manifest ".\submissions\$ten-manifest.json" --name $ten --revision 2
```

Kiểm lại manifest, số slot và hai ảnh mẫu mỗi ZIP. Cập nhật mục 4 báo cáo: mã lỗi nào đã sửa, theo phiếu của ai, revision mới. Nếu có lần tiếp theo thì lưu bản cũ vào `r2/`, dùng manifest revision 3 tương ứng. Thư mục lịch sử giữ trên máy, không gửi thay bộ 5 file hiện hành.

## 13 Checklist trước khi gửi

- [ ] Đã làm đủ A01–A12 và B01–B08, các ảnh thêm ghi đúng số đã làm.
- [ ] Project đúng taxonomy; mỗi ảnh đủ bốn tag đúng giá trị, `source=real` cho bộ ảnh này.
- [ ] Mọi job thuộc phần nộp ở trạng thái completed, mọi chỉnh sửa đã lưu.
- [ ] Lane đúng tâm/chiều/thuộc tính; không tạo vạch không tồn tại; fisheye bên hông dùng `other`.
- [ ] Có phiếu QC thực sự kiểm 6 ảnh cho bạn được ghép, đủ 12 mã, số mục kiểm, danh sách lỗi và kết luận đúng luật.
- [ ] Đã xử lý phiếu nhận được và ghi mục 4 báo cáo; không còn lỗi nặng sau buffer.
- [ ] Hai ZIP là CVAT for images 1.1, mỗi ZIP có một XML bao phủ đủ ảnh, không có ảnh hoặc ZIP con.
- [ ] Đã chạy lint, đọc cảnh báo; ngoại lệ QC1-02 có coach xác nhận và có ghi slot/lý do trong phiếu liên quan.
- [ ] Manifest được sinh từ đúng hai ZIP cuối, `contains_images=false`, revision và SHA256 đúng.
- [ ] Kiểm slot không thiếu/trùng/ngoài nhóm; đã mở lại, so số shape với CVAT ở hai ảnh mỗi ZIP.
- [ ] Báo cáo đủ thông tin, thống kê, ca khó, phản hồi QC, bảng kiểm file, bốn ảnh chụp truy cập được và tự đánh giá.
- [ ] Đổi mọi `TENBAN`/chỗ trống thành thông tin thật; tên hai ZIP không có hậu tố thừa.
- [ ] Chọn gửi **đúng 5 file hiện hành** ở mục 1 lên kênh riêng coach chỉ định trước hạn; không gửi data, GT hoặc thư mục lưu revision cũ.

Rubric yêu cầu đạt **≥ 60/100 và không còn lỗi nặng sau buffer**. Phân bổ: project/tag/file 10; polyline front 25; thuộc tính làn 10; vùng lái 10; đèn/biển 15; fisheye 10; phiếu QC cho bạn 15; báo cáo 5. Phần tự động tối đa 78, phần chấm tay 22. `SELFTEST PASS` là kiểm công cụ; lint đọc được file là kiểm đóng gói; cả hai đều không thay kết quả chấm GT và QC thực tế.

## 14 Row anchor nếu còn thời gian

Phần này không tính điểm. Chọn một polyline, ví dụ R1 ảnh A01; nội suy x tại các hàng y=720,680,640,600,560. Với hai điểm `(x1,y1)` và `(x2,y2)` kẹp hàng y:

```text
x(y) = x1 + (y - y1) * (x2 - x1) / (y2 - y1)
```

Chỉ nội suy khi y nằm trong đoạn có dữ liệu; ngoài phạm vi polyline ghi “không có làn”, không kéo dài vạch để lấp bảng. Tự tính trước, sau đó đối chiếu:

```powershell
python -m road_lab.rowanchor '.\submissions\hoangnhatanh-front.zip' --image A01 --position R1 --rows 720,680,640,600,560
```

Điền mục 8 báo cáo và trả lời ba câu của mẫu. Nội suy theo vị trí hình học không đổi nếu đảo thứ tự điểm, nhưng đảo chiều vẫn vi phạm quy tắc và có thể ảnh hưởng công cụ đọc đường theo thứ tự điểm. Không cần thêm file model vào submission.

## 15 Tài liệu để đối chiếu khi gặp ca khó

- [SUBMISSION](SUBMISSION.md) và [đặc tả bàn giao](docs/delivery_road_v1.0.md): bộ 5 file, tên file, manifest và nộp lại.
- [Guideline Road v1.0](docs/guideline_road_v1.0.md): quy tắc gán nhãn; được ưu tiên khi tài liệu khác khác quy tắc gán.
- [Thang mức lỗi](docs/qc_severity_road_v1.0.md) và [mẫu phiếu](templates/qc1_form.md): mức lỗi, cách đếm, Release/Hold/Rework.
- [Mẫu báo cáo](templates/report.md), [rubric](RUBRIC.md), [checkpoint](CHECKPOINTS.md): nội dung phải có và cách chấm.
- [Hướng dẫn CVAT bằng hình](docs/huong_dan_cvat_bang_hinh.md): thao tác minh họa trên CVAT v2.82.0.
- [Đặc tả script](lab/AGENTS.md): CLI và ngưỡng checker; đây là nguồn ưu tiên cho ngưỡng/cách chạy script.

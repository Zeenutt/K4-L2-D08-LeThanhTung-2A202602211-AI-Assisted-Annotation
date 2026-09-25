# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Lê Thanh Tùng

Công cụ gán nhãn đã dùng: CVAT

Sao chép file này thành `reports/REPORT.md` rồi điền vào các chỗ ĐIỀN. Mọi con số phải truy được
từ `reports/rounds_table.md`, `outputs/selection_round1.csv`, `outputs/metrics_round*.json` hoặc
`outputs/round*_diff.md`. Không coi nhãn test do mô hình tạo là chân lý tuyệt đối.

## 1. Dữ liệu và cách chia tập

Tại sao tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng
đệm ở giữa, thay vì chia ngẫu nhiên? Nếu chia ngẫu nhiên, số đo trên tập kiểm thử sẽ bị lệch theo
hướng nào, và vì sao?

`Camera đứng một chỗ, một chiếc xe nằm trong hình vài giây. Ảnh học và ảnh kiểm tra phải cách nhau theo thời gian. Nếu trộn ngẫu nhiên, cùng một xe có thể vừa được AI học vừa được dùng để chấm. Điểm sẽ đẹp hơn sự thật.`

## 2. Mô hình khởi đầu lạnh (cold start)

Chép dòng vòng 0 từ `rounds_table.md`. Dựa vào `outputs/compare_round0.jpg`, cho biết mô hình khởi
đầu lạnh không khớp nhãn tham chiếu ở những loại xe nào. Độ phủ (recall) theo kích thước xe cho
thấy điều gì? Một trường hợp nào cần người rà lại nhãn tham chiếu trước khi kết luận mô hình sai?

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |

`- Số chính là điểm khớp khung 0.771. Xe nhỏ chỉ được tìm thấy khoảng 0.182, xe vừa 0.547, xe lớn 0.561. Nghĩa là xe ở xa bị bỏ sót nhiều hơn xe ở gần.`
`- Mở ảnh outputs/compare_round0.jpg và quan sát, mô hình khởi đầu lạnh không khớp nhãn tham chiếu rõ nhất ở nhóm xe nhỏ (xe ở khoảng cách xa phía nửa trên khung hình), nơi độ phân giải thấp và ánh sáng phức tạp khiến mô hình dễ bị bỏ sót (FN) hoặc lệch khung bao (FP). Chẳng hạn, ở các khung hình phía góc trên bên phải, vị trí các hộp do AI vẽ không ôm trọn hoặc bị lệch hẳn so với nhãn tham chiếu của xe đang di chuyển.`
`- Tuy nhiên, cần lưu ý một trường hợp: nhãn dùng để chấm cũng do máy vẽ, chưa có người xem từng khung, nên có thể nhãn chấm sai chứ không phải AI của bạn sai.`

## 3. Chiến lược chọn mẫu

Giải thích bằng lời công thức `score = W_U·U + W_A·A + W_D·D` và vai trò của `MIN_GAP_S`.
Dẫn ba frame trong `reports/SELECTION.md` và một frame khác để chứng minh cách bạn cân nhắc
độ bất định, ảnh gần trùng và công gán nhãn. Điểm bất định có chứng minh ảnh đó sẽ cải thiện
mô hình không? Vì sao?

`- Mỗi ảnh có một điểm. Một nửa điểm là AI không chắc. Ba phần mười là AI vẽ nhiều khung còn lưỡng lự. Hai phần mười là ảnh có khác thời gian với ảnh khác. Hai ảnh trong cùng lô phải cách nhau ít nhất 2 giây, vì camera đứng yên, ảnh sát nhau gần như giống hệt.`
`- Như đã đề cập trong file SELECTION.md, ba frame được AI chọn là frame_0182.jpg, frame_0099.jpg và frame_0107.jpg. Trong khi đó, frame_0372.jpg dù có điểm cao nhưng bị bỏ qua vì quá sát giờ với frame_0369.jpg (cách nhau chưa tới 2 giây), chụp cùng một cảnh nên việc sửa cả hai sẽ tốn công mà ít mang lại giá trị học tập thêm cho AI.`
`-> Điểm cao không có nghĩa sửa ảnh đó sẽ làm AI giỏi hơn.`

## 4. Các vòng học chủ động (active learning)

Chép bảng từ `rounds_table.md`. Với mỗi vòng, trình bày:

- mức độ bạn đã sửa nhãn gợi ý (số box giữ nguyên, chỉnh sửa, xoá, thêm mới, lấy từ
`outputs/round*_diff.md`);
- AP50 thay đổi bao nhiêu so với khởi đầu lạnh và so với vòng trước;
- nhóm xe nào tốt lên hoặc xấu đi theo số đo trên cùng tập test.

Dựa vào các ảnh `compare_round*.jpg`, chỉ ra một ca kết quả đổi sau fine-tune (tốt hơn hoặc xấu
đi), cùng lý do có thể kiểm. Dùng `BLIND_SCAN.md`, `REVIEW_LOG.csv` và `round1_diff.md` phân biệt
quan sát độc lập, lỗi pre-label đã sửa và kết quả mô hình sau train. Mô tả một ca khó theo guideline.

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n fine-tune vong 1..1 | 12 | 313 | 0.329 | -0.443 | 1.000 | 0.007 | 0.015 | 0.000 | 0.010 | 0.000 |

Kết quả vòng 1:
- Mức độ sửa nhãn gợi ý: Tôi đã giữ nguyên 94 box (accepted), chỉnh sửa 53 box (edited), xóa 22 box (deleted/FP của model) và thêm mới 166 box (added/FN của model). Tỷ lệ chấp nhận (accept rate) là 56%.
- Điểm AP50 của vòng 1 (fine-tune trên 12 ảnh) giảm mạnh xuống còn 0.329, tức là giảm 0.443 so với mô hình khởi đầu lạnh (0.771).
- Nhóm xe chịu ảnh hưởng nặng nhất là xe lớn (R large giảm từ 0.561 xuống 0.000) và xe nhỏ (R small giảm từ 0.182 xuống 0.000). Điều này cho thấy sau vòng fine-tune đầu tiên, mô hình hoạt động kém đi đáng kể trên cùng tập test.

Phân tích một ca khó (dựa trên compare_round*.jpg, BLIND_SCAN.md, REVIEW_LOG.csv, và round1_diff.md):
- Trước khi xem pre-label, tôi quan sát frame_0312.jpg bằng mắt thường và đếm được 24 xe. Tôi cũng đã dự đoán trước hai vị trí AI dễ nhầm: (1) hai xe đi cạnh nhau hướng gần cầu dễ bị gộp thành 1 xe, và (2) xe đang ló ra từ tán cây.

Dữ liệu sửa nhãn ở frame_0312.jpg (trong file round1_diff.md) cho thấy nhận định này khá chính xác: model ban đầu chỉ đề xuất 13 box, sau khi sửa thì số lượng box tăng lên 22. Cụ thể, tôi chỉ có thể giữ nguyên (accept) 3 box, phải sửa (edit) 8 box, xóa 2 box sai và phải thêm mới tới 11 box (do AI bỏ sót quá nhiều xe, bao gồm cả những trường hợp tôi đã dự đoán).

Sau khi train vòng 1 (fine-tune), thay vì cải thiện, mô hình lại dự đoán tệ hơn hẳn (AP50 rớt thảm hại). Lý do có thể là việc fine-tune chỉ với 12 ảnh cùng 313 box nhãn (sau khi sửa) đã gây ra hiện tượng overfitting trầm trọng trên tập dữ liệu nhỏ này, phá vỡ đi khả năng nhận diện xe tổng quát mà mô hình khởi đầu (đã được train kỹ trên bộ COCO khổng lồ) vốn có. Kết quả là trên tập test, AI gần như mất luôn khả năng phát hiện xe nhỏ (R small = 0) và xe lớn (R large = 0). Đây là ví dụ rõ ràng cho thấy việc gán nhãn thủ công với số lượng quá ít không đủ để giúp mô hình "khôn" lên mà ngược lại, còn làm nó "quên" đi những gì đã học.

## 5. Kết luận và giới hạn

Kết quả vòng này so với cold start ra sao? Vì sao bạn dừng hoặc tiếp tục? Đề xuất hai ca còn yếu
hoặc bất định cho vòng sau, kèm chi phí rà nhãn và nguy cơ ảnh gần trùng. Tập kiểm thử chỉ 20 ảnh,
có luật bỏ qua xe quá nhỏ và nhãn tham chiếu do mô hình tạo chưa được rà thủ công; các giới hạn đó
ảnh hưởng thế nào đến kết luận? Nếu AP50 giảm, bạn sẽ kiểm tra điều gì trước khi train thêm?

- Kết quả vòng 1 giảm mạnh so với vòng 0 (cold start), với điểm AP50 tụt từ 0.771 xuống chỉ còn 0.329. Trước mắt, tôi quyết định tạm dừng việc huấn luyện tiếp để tìm hiểu nguyên nhân thay vì mù quáng chạy thêm các vòng mới.
- Về mặt nhận diện, mô hình vẫn còn yếu ở hai trường hợp: (1) xe ở quá xa chỉ còn lại hai chấm đèn sáng, và (2) xe bị cắt mép ở rìa khung hình. Nếu tiếp tục cho vòng sau, tôi sẽ đề xuất tập trung rà lại nhãn ở hai ca khó này. Tuy nhiên, việc sửa thêm nhãn thủ công rất mất thời gian (chi phí rà nhãn cao). Đặc biệt, cần chú ý nguy cơ chọn phải ảnh gần trùng; tuyệt đối không nên chọn hai ảnh sát thời gian nhau vì chúng gần như là một cảnh, không mang lại thông tin mới cho mô hình học hỏi.
- Cũng cần lưu ý về các giới hạn của thử nghiệm này: tập kiểm thử (test set) hiện tại quá nhỏ, chỉ có 20 ảnh, đồng thời áp dụng luật bỏ qua xe quá nhỏ. Hơn nữa, nhãn tham chiếu dùng để chấm điểm hoàn toàn do mô hình tự tạo (pre-label) mà chưa hề được người rà thủ công. Những giới hạn này khiến kết luận về việc mô hình "tốt lên" hay "xấu đi" có thể bị sai lệch.
- Chính vì điểm AP50 giảm nghiêm trọng sau vòng 1, bước tiếp theo tôi sẽ phải xem xét lại toàn bộ các khung nhãn mình đã sửa (review lại dữ liệu train) để xem có sự thiếu nhất quán nào không, trước khi cho AI học thêm bất kỳ dữ liệu nào khác.
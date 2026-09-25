# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, chọn năm frame bạn sẽ ưu tiên nếu chỉ có
ngân sách rà năm ảnh. Ghi tên, điểm, thời điểm, thứ tự và lý do; tối thiểu một quyết định phải xét
ảnh gần trùng hoặc trường hợp model không dự đoán được box:
`- Nếu chỉ được sửa 5 ảnh, tôi sẽ ưu tiên chọn frame_0182.jpg (hạng 1, điểm 0.959, thời điểm 72.8s), frame_0369.jpg (hạng 2, điểm 0.932, thời điểm 147.6s), frame_0380.jpg (hạng 3, điểm 0.917, thời điểm 152.0s), frame_0326.jpg (hạng 4, điểm 0.916, thời điểm 130.4s) và frame_0331.jpg (hạng 5, điểm 0.915, thời điểm 132.4s). Năm ảnh này đứng đầu danh sách về mức độ ưu tiên. Bên cạnh đó, frame_0182.jpg và frame_0187.jpg (hạng 10, điểm 0.899) chỉ cách nhau 2 giây, bối cảnh gần như tương tự nhau nên tôi quyết định không lấy cả hai để tránh trùng lặp.`

Ba frame thuộc lô 12 ảnh model chọn và bằng chứng trong CSV/ảnh contact sheet:
`- Trong số 12 ảnh mà AI đã chọn (cột selected là True), có thể kể đến frame_0182.jpg, frame_0099.jpg (hạng 8) và frame_0107.jpg (hạng 14). Cả ba ảnh này đều có điểm số cao (trên 0.88). Dữ liệu CSV cho thấy AI đã nhận diện được khá nhiều xe (số lượng n_boxes từ 28 đến 33) nhưng lượng khung bao mơ hồ (n_ambiguous) vẫn còn rất cao (từ 14 đến 18 khung), chứng tỏ mô hình còn nhiều điểm chưa chắc chắn.`

Một frame có điểm cao nhưng không chọn hoặc một frame có điểm thấp vẫn nên xem, và lý do:
`- Đó là trường hợp của frame_0372.jpg. Khung hình này đứng hạng 6 với điểm 0.910 (cao hơn một vài ảnh đã được chọn), tuy nhiên lại bị AI bỏ qua. Lý do là vì thời điểm của ảnh này (148.8s) diễn ra quá sát với frame_0369.jpg (147.6s). Hai ảnh này gần như chụp cùng một cảnh, việc bỏ công sửa cả hai sẽ gây lãng phí mà mô hình cũng ít học thêm được đặc trưng gì mới.`

Điều phép chọn này chưa chứng minh về chất lượng mô hình:
`- Việc các frame đạt điểm số cao ở đây chỉ có nghĩa là AI đang cảm thấy phân vân và chưa chắc chắn với phán đoán của mình. Phép lựa chọn này chưa thể chứng minh được rằng sau khi sửa xong các ảnh khó này, AI sẽ tự động nhận diện xe tốt hơn trên toàn bộ dữ liệu.`
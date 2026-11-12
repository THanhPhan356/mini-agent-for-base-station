# Mini Agent for Base Station

## Vấn đề

Các trợ lý AI như ChatGPT và Claude cho thấy khả năng của AI hiện nay. Tuy nhiên, khi sử dụng qua dịch vụ cloud, người dùng phụ thuộc vào hạ tầng, chính sách và khả năng truy cập của nhà cung cấp. Dự án này xuất phát từ mong muốn chủ động triển khai và vận hành một agent trên phần cứng của mình.

Cần phân biệt việc thuê dịch vụ AI với khả năng chạy mô hình local: không phải mọi mô hình AI đều chỉ dùng được qua cloud. Với dự án này, mục tiêu cụ thể là xây dựng một mô hình chuyên biệt, đủ nhỏ để hỗ trợ một số quyết định điều khiển của base station (BS), thay vì tái tạo khả năng của một trợ lý AI đa dụng.

## Câu hỏi của dự án

**Một agent sử dụng mạng neuron nhỏ có thể chạy trên vi điều khiển như ESP32, kết nối với BS và hỗ trợ quyết định điều khiển với chất lượng và độ trễ chấp nhận được hay không?**

Đây là câu hỏi cần kiểm chứng. Kích thước mạng nhỏ chưa đủ để bảo đảm khả thi: cần tính cả bộ nhớ khi chạy, xử lý đầu vào, giao tiếp với BS và thời hạn áp dụng quyết định.

## Ý tưởng xử lý

Thiết kế một agent local theo vòng lặp:

```text
BS cung cấp trạng thái → Agent suy luận → BS kiểm tra và áp dụng quyết định
         ↑                                          |
         └────────── phản hồi chất lượng ────────────┘
```

- **Đầu vào:** thông tin kênh đã rút gọn, SNR/CQI, tải lưu lượng, hàng đợi và yêu cầu chất lượng dịch vụ của UE, tùy tác vụ.
- **Mô hình:** thử nghiệm mạng neuron nhỏ, chẳng hạn MLP; huấn luyện trên máy tính và chỉ chạy suy luận trên thiết bị nhúng. Đánh giá quantization INT8 để giảm nhu cầu bộ nhớ và tính toán.
- **Đầu ra:** một quyết định trong tập hành động mà BS cho phép, như beam index hoặc đề xuất phân chia tài nguyên.
- **Phản hồi:** dùng kết quả sau khi áp dụng để đánh giá policy; chưa mặc định huấn luyện online trên ESP32.

Agent là thành phần hỗ trợ điều khiển. BS tiếp tục xử lý tín hiệu vô tuyến và kiểm tra các ràng buộc trước khi áp dụng đề xuất. Khi đầu vào quá hạn, quyết định không hợp lệ hoặc agent không đáp ứng thời hạn, BS dùng thuật toán dự phòng.

## Các tác vụ hướng tới

| Tác vụ | Quyết định agent có thể hỗ trợ | Chỉ số đánh giá |
|---|---|---|
| Beamforming | Chọn beam trong codebook; BS áp dụng trọng số anten tương ứng | SNR/SINR, overhead đo beam, độ trễ quyết định |
| Phân bổ tài nguyên | Đề xuất chia tài nguyên thời gian/tần số giữa các UE trong ngân sách cho phép | Throughput tổng, fairness, mức đáp ứng QoS |

Data rate của UE là kết quả cần đánh giá, không phải giá trị có thể tùy ý đặt trực tiếp. Mục tiêu tối ưu cần xác định rõ: tăng throughput tổng, cải thiện tốc độ của UE yếu hoặc đáp ứng mức dịch vụ yêu cầu.

Phạm vi thử nghiệm đầu tiên dự kiến là **chọn beam trong mô phỏng Python**. Phân bổ tài nguyên và triển khai nhúng là các bước tiếp theo sau khi kiểm chứng được tác vụ đầu tiên.

## Kiểm chứng tính khả thi

| Câu hỏi | Phép kiểm tra dự kiến |
|---|---|
| Policy có hữu ích không? | So sánh với quét beam và thuật toán chọn beam đơn giản trên cùng kênh, thông tin đầu vào và ngân sách đo |
| Mô hình có vừa thiết bị không? | Đo kích thước trọng số, flash và RAM đỉnh, bao gồm bộ đệm và phần mềm giao tiếp |
| Quyết định có kịp thời không? | Đo độ trễ từ lấy trạng thái đến BS áp dụng quyết định, đối chiếu thời hạn của tác vụ |
| Quantization ảnh hưởng thế nào? | So sánh chất lượng quyết định trước và sau quantization |
| Có hoạt động ngoài điều kiện huấn luyện không? | Kiểm tra trên quỹ đạo, kênh và tải chưa dùng để huấn luyện |

ESP32 hiện là phần cứng mục tiêu tham khảo; chưa chốt biến thể chip hay cấu hình mạng. ESP32 truyền thống có 520 KB SRAM trên chip, được chia sẻ cho dữ liệu và lệnh; không thể xem toàn bộ dung lượng này là RAM dành cho mô hình. [Datasheet của Espressif](https://documentation.espressif.com/esp32_datasheet_en.html).

ESP-DL hỗ trợ ESP32, nhưng tài liệu Espressif lưu ý tốc độ thực thi chậm hơn ESP32-S3 và ESP32-P4. Vì vậy cần đo trên đúng thiết bị trước khi kết luận đủ năng lực. [Hướng dẫn ESP-DL](https://docs.espressif.com/projects/esp-dl/en/latest/getting_started/readme.html).

## Trạng thái

Dự án đang ở giai đoạn xác định vấn đề và hướng tiếp cận. Chưa có mô hình, mã mô phỏng hoặc kết quả thực nghiệm. Việc kết nối phần cứng sẽ cần BS mô phỏng hoặc testbed cung cấp giao diện điều khiển phù hợp; chưa cam kết tích hợp với BS thương mại hay đáp ứng thời gian thực.

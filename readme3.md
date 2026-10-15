Báo cáo phân tích (I/O)
Input (Văn bản thô): Yêu cầu hướng dẫn kỹ thuật cài đặt cảm biến độ ẩm, các câu lệnh Terminal, đoạn mã Python điều khiển thiết bị, bảng so sánh tính năng và danh sách kiểm tra thiết bị.

Output (Giao diện render chuyên nghiệp): Tài liệu Markdown hoàn chỉnh với cấu trúc tiêu đề phân cấp, Inline Code cho tên biến, Code Block tô sáng cú pháp (Python), bảng dữ liệu (Table) và danh sách công việc (Task List).
 CODE
# Hướng dẫn cài đặt cảm biến độ ẩm cho trang trại Lan

Tài liệu này hướng dẫn các bước thiết lập hệ thống cảm biến độ ẩm môi trường cho trang trại Lan.

## 1. Tổng quan hệ thống

Cảm biến độ ẩm giúp theo dõi độ ẩm không khí trong nhà lưới, đảm bảo môi trường tối ưu cho Lan phát triển.

## 2. Các bước cài đặt

### Bước 1: Cập nhật hệ thống
Mở Terminal và cập nhật danh sách gói:

`sudo apt update && sudo apt upgrade -y`

### Bước 2: Cài đặt thư viện cho cảm biến DHT22
Cài đặt thư viện đọc dữ liệu từ cảm biến:

`sudo apt install -y git python3-pip`
`sudo pip3 install Adafruit_DHT`

### Bước 3: Kích hoạt giao tiếp 1-Wire (nếu dùng DHT22)
Bật giao tiếp 1-Wire trên hệ thống:

`sudo raspi-config`
*(Chọn: Interface Options > 1-Wire > Enable)*

### Bước 4: Kiểm tra module đã được nhận
Sau khi bật 1-Wire, kiểm tra thiết bị:

`ls /sys/bus/w1/devices/`
*(Kết quả có dạng `28-xxxxxxx`)*

### Bước 5: Chạy thử chương trình đọc cảm biến
Tải và chạy file Python mẫu từ thư viện:

`git clone https://github.com/adafruit/Adafruit_Python_DHT.git`
`cd Adafruit_Python_DHT/examples`
`python3 Adafruit_PythonDHT.py 22 4`
*(Giải thích: `22`: DHT22, `4`: GPIO pin. Nhận kết quả độ ẩm và nhiệt độ)*

### Bước 6: Tự động chạy khi khởi động (tuỳ chọn)
Thêm vào crontab để đọc dữ liệu định kỳ (ví dụ mỗi 5 phút):

`crontab -e`
`*/5 * * * * python3 /home/pi/read_dht22.py`
```

---

## 3. Mã nguồn Python đọc dữ liệu cảm biến (DHT22)

Dưới đây là đoạn mã Python chuẩn hóa để tương tác với cảm biến độ ẩm qua thư viện `Adafruit_DHT`:

```python
import Adafruit_DHT
import time

# Cấu hình loại cảm biến và chân GPIO kết nối trên Raspberry Pi
SENSOR = Adafruit_DHT.DHT22
GPIO_PIN = 4

def read_sensor_data():
    print("Đang đọc dữ liệu từ cảm biến độ ẩm DHT22...")
    try:
        while True:
            # Đọc giá trị độ ẩm và nhiệt độ từ chân GPIO 4
            humidity, temperature = Adafruit_DHT.read_retry(SENSOR, GPIO_PIN)
            
            if humidity is not None and temperature is not None:
                print(f"Nhiệt độ: {temperature:.1f}°C  |  Độ ẩm: {humidity:.1f}%")
            else:
                print("Lỗi: Không thể đọc dữ liệu từ cảm biến. Đang thử lại...")
                
            # Đợi 5 giây trước lần đọc tiếp theo
            time.sleep(5)
            
    except KeyboardInterrupt:
        print("Đã dừng chương trình đọc cảm biến.")

if __name__ == "__main__":
    read_sensor_data()
```
---
3. So sánh các loại cảm biến độ ẩm đất

| Tiêu chí | Cảm biến điện (Mới) | Cảm biến điện trở lại (Cũ) |
| :--- | :--- | :--- |
| Độ bền | Cao (chống ăn mòn tốt) | Low (dễ bị oxy hóa) |
| Độ chính xác | Long long ổn định | Giảm dần theo thời gian |
| Chi phí | Hơi cao hơn | Rẻ hơn |

5. Danh sách kiểm tra thiết bị (Task List)

- [x] Kiểm tra nguồn điện cấp cho mạch điều khiển

- [x] Kết nối chân tín hiệu vào SENSOR_PIN

- [ ] Cấu hình lại thông số trong file pump_control.py

- [ ] Chạy thử nghiệm hệ thống bơm nước

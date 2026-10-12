# Báo cáo Bài 4: Lưu trữ dữ liệu đơn giản với Docker Volume

## 1. Mục tiêu
- Hiểu và áp dụng cơ chế lưu trữ dữ liệu bền vững (Persistence Storage) trong Docker sử dụng Named Volume.
- Kiểm chứng tính toàn vẹn của dữ liệu: Dữ liệu được ghi bởi một container độc lập và được đọc chính xác bởi một container khác sau khi container đầu tiên đã kết thúc và bị hủy bỏ (`--rm`).

## 2. Các bước thực hiện và lệnh đã sử dụng

### Bước 1: Khởi tạo Docker Volume
Tạo một Named Volume mới mang tên `easy-volume`:
```bash
docker volume create easy-volume
```

### Bước 2: Kiểm tra danh sách volume
```bash
docker volume ls
```

### Bước 3: Ghi dữ liệu vào volume bằng container thứ nhất
Sử dụng image `alpine`, gắn kết (mount) volume `easy-volume` vào đường dẫn `/data` bên trong container và ghi dòng văn bản vào tệp `/data/test.txt`:
```bash
docker run --rm -v easy-volume:/data alpine sh -c "echo 'Luu tru du lieu Docker' > /data/test.txt"
```
*(Tham số `--rm` đảm bảo container này sẽ tự động bị xóa ngay sau khi thực thi xong lệnh ghi dữ liệu).*

### Bước 4: Đọc lại dữ liệu bằng container thứ hai
Khởi chạy một container `alpine` hoàn toàn mới, mount cùng volume `easy-volume` vào `/data` và đọc nội dung file vừa ghi:
```bash
docker run --rm -v easy-volume:/data alpine cat /data/test.txt
```

---

## 3. Nhật ký kết quả đầu ra thực tế từ Terminal

### 3.1. Kết quả tạo volume và kiểm tra danh sách
```text
easy-volume

DRIVER    VOLUME NAME
local     easy-volume
```

### 3.2. Kết quả đọc dữ liệu từ volume qua container thứ hai
```text
Luu tru du lieu Docker
```

---

## 4. Phân tích nguyên lý hoạt động của Docker Volume

1. **Vòng đời độc lập (Decoupled Lifecycle):**
   - Mặc định, dữ liệu ghi vào writable layer của container sẽ biến mất vĩnh viễn khi container bị xóa (`docker rm` hoặc cờ `--rm`).
   - Với **Docker Named Volume**, dữ liệu được lưu trữ trực tiếp trên filesystem của máy host (được Docker quản lý tại `/var/lib/docker/volumes/` hoặc snapshot root tương đương), hoàn toàn tách rời khỏi vòng đời của container.
2. **Chia sẻ dữ liệu giữa nhiều container:**
   - Khi container 1 hoàn tất và tự hủy, volume `easy-volume` cùng dữ liệu `/data/test.txt` vẫn tồn tại nguyên vẹn.
   - Khi container 2 được khởi chạy và mount tới `easy-volume`, nó có thể truy cập và đọc toàn bộ tệp mà container 1 đã tạo ra.

---

## 5. Hướng dẫn chụp ảnh minh chứng & Placeholder
1. Mở Terminal và thực thi lệnh kiểm tra:
   ```bash
   docker run --rm -v easy-volume:/data alpine cat /data/test.txt
   ```
2. Chụp ảnh màn hình hiển thị lệnh trên cùng dòng kết quả in ra: `Luu tru du lieu Docker`.
3. Dán ảnh vào vị trí placeholder bên dưới:

`ImgEx`

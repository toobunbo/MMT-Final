### 1. Mô hình Hạ tầng mạng (Network Topology)
Chúng ta sử dụng mô hình **Collapsed Core (Lõi thu gọn)** (giải thích ra)... chuyên dụng cho doanh nghiệp quy mô vừa (SME/Branch Office).



* **Lớp Biên (Edge Layer):** Sử dụng **Router ISR 4321**.
    * *Vai trò:* Gateway kết nối Internet, Firewall mềm (ZBF/ACL), VPN Server, NAT.
* **Lớp Lõi & Phân phối (Core/Distribution Layer):** Sử dụng **Layer 3 Switch 3560/3650**.
    * *Vai trò:* Định tuyến tốc độ cao giữa các VLAN (Inter-VLAN Routing), trung tâm chính sách (Policy Enforcement).
* **Lớp Truy cập (Access Layer):** Sử dụng **Switch 2960**.
    * *Vai trò:* Kết nối thiết bị cuối, thực thi bảo mật Lớp 2 (Port Security, DHCP Snooping).

### 2. Quy hoạch Vùng & Phân quyền (Zoning & Segmentation)
Hệ thống được chia thành 3 vùng an ninh (Security Zones) riêng biệt:

| Vùng (Zone) | VLAN ID | Đối tượng | Chính sách An ninh (Security Policy) |
| :--- | :--- | :--- | :--- |
| **INTERNAL (Trusted)** | **10** | **Teller (Giao dịch viên)** | **Nghiêm ngặt:** Chỉ được truy cập Database nội bộ. **CẤM** tuyệt đối truy cập Internet. |
| | **20** | **Manager (Quản lý)** | **Linh hoạt:** Được phép truy cập Internet (Web/Mail) và Web Server nội bộ. Hạn chế truy cập sâu Database. |
| | **99** | **Admin/Mgmt** | **Đặc quyền:** Được SSH vào thiết bị mạng, chạy script Automation. |
| **DMZ (Semi-Trusted)** | **50** | **Web & DB Server** | **Công khai:** Cho phép Internet truy cập Web (HTTP/HTTPS). Database bị ẩn sau Web Server (Backend). |
| **EXTERNAL (Untrusted)** | - | **Internet / VPN Client** | **Không tin cậy:** Bị chặn bởi Firewall. Chỉ cho phép kết nối VPN hợp lệ vào trong. |

### 3. Các cơ chế Bảo mật then chốt (Key Security Features)
Để đạt tiêu chuẩn "Ngân hàng", hệ thống tích hợp bảo mật đa lớp (Defense-in-Depth):

1.  **Network Access Control:** Sử dụng **AAA Server** (RADIUS/TACACS+) để xác thực người dùng quản trị tập trung.
2.  **Perimeter Security (Biên):** Dùng **Extended ACL** để lọc gói tin theo nguyên tắc *"Cấm tất cả, chỉ mở những gì cần thiết"*.
3.  **Endpoint Security (Đầu cuối):** Dùng **Port Security** (Shutdown cổng nếu phát hiện MAC lạ) và **DHCP Snooping** (Chống giả mạo DHCP).
4.  **Data Protection:** Dùng **VPN IPsec** cho kết nối từ xa an toàn.
5.  Bổ sung thêm cơ chết bảo

### 4. Tự động hóa & Giám sát (Automation & Monitoring)
* **Automation:** Sử dụng **Python Script** trên máy Management PC để tự động sao lưu cấu hình (Backup Config) và kiểm tra trạng thái thiết bị.
* **Monitoring:** Hệ thống đẩy log về **Syslog Server** để giám sát hành vi truy cập trái phép.

---

**KẾT LUẬN:**
Kiến trúc này đảm bảo 4 yếu tố vàng: **High Availability** (dự phòng tại Core), **Security** (Phân vùng DMZ/Internal), **Scalability** (Dễ dàng thêm VLAN mới), và **Modernity** (Có Automation).

Nếu bạn đã chốt xong kiến trúc này, chúng ta sẽ bước sang phần thực hành. Hãy gõ **"Triển khai"** và tôi sẽ gửi bạn **Bộ Code cấu hình Phần 1: Hạ tầng cơ bản (VLAN, IP, Routing)**.

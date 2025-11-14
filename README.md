## 1. Mô hình Hạ tầng mạng (Network Topology)
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
5.  Bổ sung thêm cơ chết bảo mật

### 4. Tự động hóa & Giám sát (Automation & Monitoring)
* **Automation:** Sử dụng **Python Script** trên máy Management PC để tự động sao lưu cấu hình (Backup Config) và kiểm tra trạng thái thiết bị.
* **Monitoring:** Hệ thống đẩy log về **Syslog Server** để giám sát hành vi truy cập trái phép.


## Bảng sơ đồ địa chỉ IP Mẫu 
**Bảng kê thiết bị (BOM)** và **Quy hoạch địa chỉ IP (IP Plan)** chi tiết cho dự án. 

### 1\. Danh sách Thiết bị (Bill of Materials)
| Loại Thiết bị | Model đề xuất | Số lượng | Tên gợi nhớ (Hostname) | Ghi chú kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| **Router** | **ISR 4321** | 01 | **R-EDGE** | Hỗ trợ Security License (ZBF, VPN). |
| **L3 Switch** | **3560-24PS** | 01 | **SW-CORE** | Phải bật chức năng `ip routing`. |
| **L2 Switch** | **2960-24TT** | 02 | **SW-ACC-1** (Internal)<br>**SW-DMZ** | Switch truy cập cho nhân viên và Server. |
| **Server** | Server-PT | 03 | **SRV-WEB**<br>**SRV-DB**<br>**SRV-AAA** | Giả lập Web, Database và Radius/Syslog. |
| **PC** | PC-PT | 03 | **PC-Teller**<br>**PC-Manager**<br>**PC-Admin** | Máy trạm đại diện cho các VLAN. |
| **Cloud** | Router-PT | 01 | **ISP-Internet** | Giả lập mạng Internet bên ngoài. |

-----

### 2\. Bảng Quy hoạch IP (IP Address Plan) - *Tài liệu quan trọng nhất*
Sơ đồ hệ thống
**Lưu ý về Gateway:**

  * Với các VLAN (10, 20, 50, 99): Gateway chính là địa chỉ IP đặt trên interface VLAN (SVI) của **SW-CORE**.
  * Với SW-CORE: Gateway để đi ra Internet là IP của **R-EDGE** (10.0.0.1).

#### A. Kết nối Hạ tầng (Infrastructure Links)

| Kết nối | Thiết bị | Interface | Loại Interface | Địa chỉ IP / Subnet | Ghi chú |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **WAN (Internet)** | **R-EDGE** | G0/0/0 | Physical | **113.22.1.2 /30** | IP Public (Nối ra ISP) |
| | ISP (Cloud) | G0/0 | Physical | 113.22.1.1 /30 | Gateway nhà mạng |
| **Uplink (Core-Rout)** | **R-EDGE** | G0/0/1 | Physical | **10.0.0.1 /30** | Gateway cho mạng nội bộ |
| | **SW-CORE** | G0/1 | **Routed Port** | **10.0.0.2 /30** | `no switchport` |

#### B. Mạng Nội bộ (VLANs - Config trên SW-CORE)

| VLAN ID | Tên Vùng | Interface (SVI) | Gateway (IP Switch) | Dải IP cấp phát (DHCP) | Gán cho |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **10** | **TELLER** | Vlan10 | **192.168.10.1 /24** | .10 -\> .254 | PC-Teller (Sw-Acc-1) |
| **20** | **MANAGER**| Vlan20 | **192.168.20.1 /24** | .10 -\> .254 | PC-Manager (Sw-Acc-1) |
| **50** | **DMZ** | Vlan50 | **172.16.50.1 /24** | Static IP | Web/DB Server (Sw-DMZ) |
| **99** | **MGMT** | Vlan99 | **10.10.99.1 /24** | Static IP | PC-Admin, AAA Server |

#### C. Địa chỉ Tĩnh cho Server & Admin (Static IP Allocation)

  * **Web Server (SRV-WEB):** 172.16.50.10 /24 (Default Gateway: 172.16.50.1)
  * **DB Server (SRV-DB):** 172.16.50.20 /24 (Default Gateway: 172.16.50.1)
  * **AAA/Syslog Server:** 10.10.99.10 /24 (Default Gateway: 10.10.99.1)
  * **PC-Admin (Chạy Python):** 10.10.99.100 /24 (Default Gateway: 10.10.99.1)

### 3\. Hướng dẫn Đấu nối dây (Cabling Map)

1.  **R-EDGE nối SW-CORE:**

      * Cáp chéo (Cross-over) hoặc thẳng (Straight) đều được (thiết bị mới tự nhận).
      * Cổng **G0/0/1** (Router) \<---\> Cổng **G0/1** (Switch Core).

2.  **SW-CORE nối các Switch con:**

      * Nối **SW-ACC-1**: Cổng **F0/1** (Core) \<---\> Cổng **G0/1** (Access 1).
      * Nối **SW-DMZ**: Cổng **F0/2** (Core) \<---\> Cổng **G0/1** (Switch DMZ).

3.  **Kết nối thiết bị cuối (End Devices):**

      * **PC-Teller:** Nối vào **SW-ACC-1** cổng **F0/1** (Sẽ gán vào VLAN 10).
      * **PC-Manager:** Nối vào **SW-ACC-1** cổng **F0/10** (Sẽ gán vào VLAN 20).
      * **Servers:** Nối vào **SW-DMZ** cổng bất kỳ (Sẽ gán vào VLAN 50).
      * **AAA/Admin PC:** Nối trực tiếp vào **SW-CORE** cổng **F0/24** (hoặc qua switch riêng), gán VLAN 99.


**KẾT LUẬN:**
Kiến trúc này đảm bảo 4 yếu tố vàng: **High Availability** (dự phòng tại Core), **Security** (Phân vùng DMZ/Internal), **Scalability** (Dễ dàng thêm VLAN mới), và **Modernity** (Có Automation).

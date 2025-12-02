# 02. Cấu hình VLAN và Switching (Layer 2)

## 1. Mục tiêu module

Module này triển khai toàn bộ phần **Switching & VLAN** cho mô hình mạng doanh nghiệp:

- Tạo đầy đủ VLAN theo thiết kế (Module 1).
- Sử dụng **VTP** để đồng bộ VLAN giữa các switch.
- Thiết lập **trunk 802.1Q** giữa các switch.
- Tắt **DTP** để tránh thương lượng trunk ngoài ý muốn.
- Cấu hình **Spanning Tree (Rapid-PVST)** để phòng tránh loop L2.
- Cấu hình **EtherChannel (LACP)** giữa hai Core Switch.
- Gán port access cho các VLAN tương ứng trên Access Switch.

Sau module này, hạ tầng Layer 2 sẽ sẵn sàng cho:
- Inter-VLAN Routing (Module 3)
- Firewall, DMZ (Module 4)
- DHCP/DNS/Web (Module 5)

---

## 2. Kiến trúc L2 trong mô hình

Thiết bị liên quan:

- **CORE-SW1** – Core Switch chính (L3)
- **CORE-SW2** – Core Switch dự phòng (L3)
- **ACC-SW1** – Access Switch khu vực Sales + Accounting
- **ACC-SW2** – Access Switch khu vực IT + BOD
- **ACC-SW3** – Access Switch khu vực Server LAN
- **ACC-SW4** – Access Switch khu vực Guest Wifi / mở rộng
- **DMZ-SW1** – (tuỳ chọn) Access Switch cho DMZ

Các nguyên tắc:

- Tất cả CORE–ACCESS–DMZ kết nối với nhau bằng **trunk 802.1Q**.
- VLAN được **tạo tập trung trên CORE-SW1** bằng **VTP Server**.
- Tất cả switch còn lại là **VTP Client**.
- STP chạy ở chế độ **Rapid-PVST+**.
- CORE-SW1 ↔ CORE-SW2 dùng **EtherChannel** để tăng băng thông và dự phòng.

---

## 3. Danh sách VLAN theo thiết kế

| VLAN | Tên VLAN        | Mục đích                 | Subnet                 |
|------|-----------------|--------------------------|------------------------|
| 10   | SALES           | Phòng Kinh doanh         | 192.168.10.0/24        |
| 20   | ACCOUNTING      | Phòng Kế toán            | 192.168.20.0/24        |
| 30   | IT              | Phòng Kỹ thuật/IT        | 192.168.30.0/24        |
| 40   | BOD             | Ban Giám đốc             | 192.168.40.0/24        |
| 50   | SERVER_LAN      | Server nội bộ            | 192.168.50.0/26        |
| 51   | DMZ             | Web/Service DMZ          | 192.168.50.64/26       |
| 60   | GUEST_WIFI      | WiFi khách               | 192.168.60.0/24        |
| 99   | MGMT            | VLAN quản trị thiết bị   | 192.168.99.0/24        |


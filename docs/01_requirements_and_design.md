# 01. Phân tích yêu cầu và thiết kế tổng thể hệ thống mạng

## 1. Mục tiêu tài liệu
Tài liệu này mô tả yêu cầu, kiến trúc, và thiết kế tổng thể của hệ thống mạng doanh nghiệp mô phỏng.  
Hệ thống được triển khai song song trên **Cisco Packet Tracer** (bản mô phỏng cơ bản) và trên **EVE-NG** (bản mô phỏng nâng cao với thiết bị thật như Cisco IOS, Switch L3, Firewall ASA/pfSense).

Tài liệu này đóng vai trò nền tảng để triển khai các module tiếp theo gồm VLAN, Inter-VLAN Routing, DMZ, Firewall Policies, NAT, DHCP/DNS, Wireless LAN và High Availability.

---

## 2. Bối cảnh doanh nghiệp

Hệ thống mạng được thiết kế cho doanh nghiệp giả lập **ABC Solutions**, hoạt động trong lĩnh vực dịch vụ công nghệ, với các đặc điểm:

- Khoảng 80 nhân viên.
- Một văn phòng chính, một phòng máy chủ nhỏ.
- Nhiều phòng ban khác nhau yêu cầu phân chia mạng độc lập.
- Có nhân viên khách, đối tác cần sử dụng WiFi guest.
- Hệ thống có nhu cầu truy cập Web nội bộ (Intranet), DNS nội bộ, và lọc lưu lượng Internet qua Firewall.

Công ty yêu cầu hệ thống mạng phải:
- Ổn định.
- Bảo mật tốt.
- Có khả năng mở rộng trong tương lai.
- Mô phỏng được trên cả Packet Tracer và EVE-NG.

---

## 3. Yêu cầu hệ thống mạng

### 3.1. Yêu cầu chức năng
Hệ thống phải đảm bảo:

#### **1. Phân chia VLAN**
Mỗi phòng ban có một VLAN riêng:
- Sales
- Accounting
- IT/Technical
- Management/BOD
- Server VLAN
- DMZ VLAN
- Guest WiFi VLAN

#### **2. High Availability (HA) Core Layer**
- Hai Core Switch chạy song song.
- Hỗ trợ STP/RSTP, EtherChannel, và HSRP/VRRP (nếu triển khai trên EVE).

#### **3. Inter-VLAN Routing**
- Thực hiện tại Core Switch (nếu triển khai L3 Switching trên EVE).  
- Hoặc Router-on-a-Stick khi chạy trên Packet Tracer.

#### **4. DMZ Segmentation**
DMZ chứa:
- Web Server (Intranet)
- DNS Public/DMZ Server (tuỳ chọn)
- Guest Wireless Controller/Router

Firewall phải:
- Cho phép LAN → DMZ (chỉ các port HTTP/HTTPS).
- Chặn DMZ → LAN.
- Chặn Guest WiFi → LAN.

#### **5. Wireless LAN**
- Internal WiFi (VLAN nội bộ)
- Guest WiFi (VLAN Guest, nằm sau Firewall hoặc tại DMZ Switch)

#### **6. NAT và truy cập Internet**
Firewall/Edge Router phải:
- NAT/PAT địa chỉ private sang public.
- Cung cấp điều kiện để truy cập Internet.

#### **7. Dịch vụ mạng**
- DHCP Server cấp phát IP cho từng VLAN (có thể chạy trên router hoặc server ảo).
- DNS nội bộ phục vụ phân giải tên.
- Web Server nội bộ (đặt trong DMZ).
- Hỗ trợ Syslog/Monitor (nếu triển khai EVE-NG).

---

## 4. Sơ đồ kiến trúc mạng tổng thể (Network Architecture)

Kiến trúc chia thành **3 vùng** lớn:

---

### **4.1. Vùng 1 – WAN/Internet Edge**
Bao gồm:
- Hai ISP Router (ISP1/ISP2) → mô phỏng kết nối dự phòng.
- Edge Router của doanh nghiệp.
- Region này mô phỏng việc kết nối Internet thực tế.

Chức năng:
- Quản lý tuyến kết nối đi/đến Internet.
- Triển khai static route, default route, hoặc dynamic routing.

---

### **4.2. Vùng 2 – Security Zone**
Bao gồm:
- Firewall (ASA khi chạy EVE, hoặc Router acting as Firewall khi chạy PT)
- DMZ Switch
- Web/DNS Server (DMZ)
- Guest WiFi Router

Chức năng:
- Lọc lưu lượng giữa WAN ↔ LAN.
- Tách biệt DMZ và LAN.
- Cho phép Guest WiFi truy cập Internet nhưng *không* được truy cập LAN.
- NAT/PAT.

---

### **4.3. Vùng 3 – Campus LAN (Enterprise LAN)**
Bao gồm:

#### **Core Layer**
- Core Switch 1 (L3)
- Core Switch 2 (L3)
- Kết nối bằng đường uplink dự phòng (dashed links)

Chức năng:
- Routing nội bộ.
- Chạy HRSP/VRRP/GLBP nếu chạy EVE.
- EtherChannel uplinks.

#### **Access Layer**
- Các Access Switch
- Các nhóm thiết bị: PC, Printer, Server nội bộ

Chức năng:
- Tạo VLAN cho từng bộ phận.
- Quản lý thiết bị đầu cuối.
- Kết nối WiFi nội bộ.

---

## 5. Thiết kế VLAN và Kế hoạch địa chỉ IP (IP Plan)

  ### Bảng phân hoạch VLAN và IP

| VLAN | Tên          | Mục đích           | Subnet              | Gateway         | Ghi chú                         |
|------|--------------|--------------------|---------------------|-----------------|---------------------------------|
| 10   | SALES        | Phòng Kinh doanh   | 192.168.10.0/24     | 192.168.10.1    | PC Sales                        |
| 20   | ACCOUNTING   | Phòng Kế toán      | 192.168.20.0/24     | 192.168.20.1    | PC Kế toán                      |
| 30   | IT           | Phòng Kỹ thuật/IT  | 192.168.30.0/24     | 192.168.30.1    | PC IT                           |
| 40   | BOD          | Ban Giám đốc       | 192.168.40.0/24     | 192.168.40.1    | PC/Bộ phận quản lý              |
| 50   | SERVER_LAN   | Server nội bộ      | 192.168.50.0/26     | 192.168.50.1    | Web nội bộ: 192.168.50.10       |
| 51   | DMZ          | Vùng DMZ           | 192.168.50.64/26    | 192.168.50.65   | Web/Service public              |
| 60   | GUEST_WIFI   | WiFi Khách         | 192.168.60.0/24     | 192.168.60.1    | Chỉ ra Internet, chặn vào LAN   |
| 99   | MGMT         | Quản trị thiết bị  | 192.168.99.0/24     | 192.168.99.1    | IP quản trị switch/router       |

Ghi chú:
- Guest VLAN không được phép truy cập VLAN nội bộ.
- DMZ chỉ cho phép kết nối từ LAN đến port 80/443.
- Core Switch làm default gateway (EVE).  
  Router-on-a-Stick làm gateway (Packet Tracer).

---

### 6. Bảng đặt tên thiết bị (Device Naming Convention)

Hệ thống mạng sử dụng quy ước đặt tên thiết bị theo chuẩn doanh nghiệp, bao gồm loại thiết bị, chức năng và vị trí.  
Điều này giúp quản lý cấu hình, giám sát và backup trở nên dễ dàng và nhất quán.

| Tên thiết bị      | Loại thiết bị        | Vị trí / Vai trò                |
|-------------------|----------------------|---------------------------------|
| **ISP-R1**        | Router               | Kết nối ISP 1 (WAN)             |
| **ISP-R2**        | Router               | Kết nối ISP 2 (WAN - dự phòng)  |
| **EDGE-R1**       | Router               | Router biên của doanh nghiệp    |
| **FW-01**         | Firewall             | Kiểm soát lưu lượng giữa WAN/LAN |
| **CORE-SW1**      | Switch Layer 3       | Core Switch 1 (main)            |
| **CORE-SW2**      | Switch Layer 3       | Core Switch 2 (redundancy)      |
| **ACC-SW1**       | Switch Access        | Access Switch khu vực Sales & Accounting |
| **ACC-SW2**       | Switch Access        | Access Switch khu vực IT & BOD |
| **ACC-SW3**       | Switch Access        | Access Switch khu vực Server/DMZ |
| **DMZ-SW1**       | Switch Access        | Switch dành riêng cho DMZ       |
| **SRV-WEB01**     | Server (DMZ)         | Web Server (DMZ)                |
| **SRV-DNS01**     | Server (LAN)         | DNS nội bộ                      |
| **SRV-DHCP01**    | Server (LAN)         | DHCP Server                     |
| **AP-INT01**      | Access Point         | WiFi nội bộ                     |
| **AP-GUEST01**    | Access Point         | WiFi Guest                      |
| **PC-SALES-xx**   | PC người dùng        | Phòng Kinh doanh                |
| **PC-ACC-xx**     | PC người dùng        | Phòng Kế toán                   |
| **PC-IT-xx**      | PC người dùng        | Phòng IT                        |
| **PC-BOD-xx**     | PC người dùng        | Ban Giám Đốc                    |

Quy ước đặt tên thiết bị được thiết kế theo chuẩn:
<Loại thiết bị>-<Vị trí hoặc chức năng>-<Số thứ tự>

Ví dụ:
- CORE-SW1: Core Switch số 1
- FW-01: Firewall thứ nhất
- SRV-WEB01: Web Server số 1 (DMZ)
- PC-IT-05: Máy tính thứ 5 của phòng IT

Ưu điểm:
- Dễ tìm thiết bị khi cấu hình
- Dễ đọc log, Syslog, SNMP
- Dễ đánh số IP phù hợp với tên
- Dễ quản lý khi mở rộng hệ thống



## 7. Thiết kế luồng hoạt động (Traffic Flow)

### **6.1. Luồng LAN → Internet**
1. PC gửi traffic ra ngoài.
2. Core Switch chuyển lên Firewall.
3. Firewall NAT sang IP Public.
4. Traffic đi ra ISP1/ISP2.

### **6.2. Luồng LAN → DMZ**
1. PC truy cập Web Server DMZ.
2. Firewall kiểm tra ACL (chỉ cho phép HTTP/HTTPS).
3. Traffic đến Web Server.

### **6.3. Luồng Guest WiFi**
1. Guest nhận IP từ DHCP Guest.
2. Guest chỉ được phép đi Internet.
3. Firewall chặn Guest → LAN.

### **6.4. Luồng DMZ → LAN**
- **Firewall chặn toàn bộ**, trừ dịch vụ được phép (nếu có).

---

## 8. Triển khai trên Cisco Packet Tracer và EVE-NG

### **Cisco Packet Tracer**
- Router-on-a-Stick
- ACL + NAT
- VLAN + Trunking
- Server (HTTP, DNS, DHCP)
- Guest WiFi Router
- DMZ Switch

### **EVE-NG**
- Cisco IOSv / IOSvL2 (Core & Access)
- ASA Firewall thật
- Ubuntu Server (Web/DNS)
- pfSense (nếu muốn thay Firewall)
- Running HSRP, EtherChannel, STP

---

## 9. Các module triển khai (Roadmap)

1. **Module 1:** Phân tích và thiết kế tổng thể (tài liệu này)  
2. **Module 2:** VLAN & Switching  
3. **Module 3:** Routing & HA Core Setup (EVE)  
4. **Module 4:** Firewall Policies + DMZ + Guest VLAN  
5. **Module 5:** DHCP/DNS/Web Services  
6. **Module 6:** NAT + Internet Access  
7. **Module 7:** Wireless LAN (Internal + Guest)  
8. **Module 8:** Monitoring & Logging (Syslog, Ping Test, Trace)  
9. **Module 9:** Kiểm thử & Báo cáo  

---

## 10. Kết luận

Thiết kế trên cung cấp một hệ thống mạng chuẩn doanh nghiệp, bao gồm phân tầng 3-Tier, DMZ, WiFi Guest, Firewall Security Layer, HA Core Switch, và khả năng mô phỏng trên cả Cisco Packet Tracer và EVE-NG.

Thiết kế này đảm bảo:
- Hiệu năng tốt
- Khả năng mở rộng
- An toàn mạng
- Phù hợp để demo, báo cáo, và đưa vào hồ sơ xin việc.





# Module 3 – Inter-VLAN Routing & High Availability trên Core Switch

## 1. Mục tiêu

Module 3 triển khai tầng định tuyến (Layer 3) cho toàn bộ mạng LAN:

- Bật `ip routing` trên cả CORE-SW1 và CORE-SW2.
- Tạo SVI làm gateway cho các VLAN LAN: 10, 20, 30, 40, 50, 99.
- Triển khai High Availability Gateway bằng HSRP.
- Thiết lập default route từ mỗi Core về Firewall thông qua hai đường riêng /30.

---

## 2. Tổng quan kiến trúc định tuyến

- CORE-SW1 và CORE-SW2 chạy L3 để định tuyến giữa các VLAN.
- Firewall chịu trách nhiệm:
  - LAN ↔ DMZ
  - LAN ↔ Guest
  - LAN ↔ Internet
- CORE-SW1 & CORE-SW2 đều có một uplink L3 riêng tới Firewall:
  - Link 1: Firewall ↔ CORE1 (172.16.0.0/30)
  - Link 2: Firewall ↔ CORE2 (172.16.0.4/30)
- Core triển khai HSRP để cung cấp gateway ảo (VIP) cho tất cả các VLAN LAN.

---

## 3. High Availability bằng HSRP

Để core-switch biết “cái nào ưu tiên”, ta dùng HSRP:

- CORE-SW1: priority cao → Active.
- CORE-SW2: priority thấp → Standby.
- Gateway mà PC sử dụng là **Virtual IP**, không phải IP thật của Core.

Ví dụ VLAN 10:

- CORE-SW1 = 192.168.10.1  
- CORE-SW2 = 192.168.10.2  
- Virtual IP (gateway cho PC) = 192.168.10.254  

Cơ chế hoạt động:

- Nếu CORE-SW1 hỏng → CORE-SW2 tự động nắm gateway 192.168.10.254.
- LAN không gián đoạn.

HSRP được cấu hình cho tất cả VLAN LAN:  
10, 20, 30, 40, 50, 99.

---

## 4. SVI và IP Plan

| VLAN | Mô tả | Subnet | VIP | CORE-SW1 | CORE-SW2 |
|------|--------|--------|------|-----------|-----------|
| 10 | Sales | 192.168.10.0/24 | 192.168.10.254 | 192.168.10.1 | 192.168.10.2 |
| 20 | Accounting | 192.168.20.0/24 | 192.168.20.254 | 192.168.20.1 | 192.168.20.2 |
| 30 | IT | 192.168.30.0/24 | 192.168.30.254 | 192.168.30.1 | 192.168.30.2 |
| 40 | BOD | 192.168.40.0/24 | 192.168.40.254 | 192.168.40.1 | 192.168.40.2 |
| 50 | Server LAN | 192.168.50.0/26 | 192.168.50.62 | 192.168.50.1 | 192.168.50.2 |
| 99 | Management | 192.168.99.0/24 | 192.168.99.254 | 192.168.99.1 | 192.168.99.2 |

---

## 5. Liên kết Core ↔ Firewall (L3 Point-to-Point)

- CORE-SW1 ↔ Firewall:
  - Firewall: 172.16.0.1/30
  - Core1:   172.16.0.2/30

- CORE-SW2 ↔ Firewall:
  - Firewall: 172.16.0.5/30
  - Core2:   172.16.0.6/30

Default route:

- CORE-SW1 → 172.16.0.1
- CORE-SW2 → 172.16.0.5

---

## 6. Kiểm thử sau Module 3

1. PC các VLAN phải ping được gateway ảo (VIP).
2. PC VLAN này ping PC VLAN khác được (Inter-VLAN xuyên qua Core).
3. Nếu shutdown CORE-SW1:
   - CORE-SW2 trở thành Active HSRP.
   - PC vẫn ping được gateway và tiếp tục hoạt động.
4. CORE-SW1 và CORE-SW2 đều ping ra Firewall được.

---

## 7. Kết luận

Module 3 hoàn thiện định tuyến nội bộ với:
- L3 Core Switching,
- High Availability Core Gateway HSRP,
- Default route đa đường tới Firewall,
- Phân tách rõ ràng LAN ↔ DMZ ↔ Guest ↔ Internet.

Module tiếp theo: cấu hình Firewall (Module 4).

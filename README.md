# tivimart-infra-lab
Hạ tầng IT doanh nghiệp TiviMart (lab VMware)

# Hạ tầng IT doanh nghiệp TiviMart (lab VMware)
Lab cá nhân thực hiện tháng 8/2026 trên VMware Workstation.
Kịch bản: công ty ABC Solutions vận hành sàn thương mại điện tử nội bộ TiviMart,
domain Active Directory là abcsolutions.local.

## Mục tiêu
Xây dựng hạ tầng gồm tường lửa, quản lý người dùng tập trung, cơ sở dữ liệu, cache,
web server và hệ thống giám sát cảnh báo.

## Kiến trúc
7 máy ảo, chia 2 vùng mạng:
- Mạng Server 10.10.10.0/24: DC-01 (AD + DNS), DB-SQL (MySQL), CACHE-REDIS, WEB-01 (Flask),
  MONITOR (Zabbix + Grafana)
- Mạng Client 10.10.20.0/24: CLIENT-01 (Windows 7, join domain), DHCP cấp 10.10.20.100-200
- FW-PFSENSE làm firewall, gateway và DHCP giữa các vùng

Nguyên tắc phân vùng: Client chỉ được ra Web (80/443), DNS (53) và tới đúng DC-01 để join domain.
Các port nội bộ MySQL 3306, Redis 6379, Zabbix Agent 10050/10051 bị chặn từ vùng Client.

## Công nghệ sử dụng
pfSense, Windows Server 2019 (Active Directory, GPO), Ubuntu Server 24.04, MySQL, Redis,
Python Flask, Zabbix 7.0, Grafana

## Những gì đã thực hiện
1. pfSense: gán interface, tạo mạng Client, DHCP, viết firewall rule theo thứ tự ưu tiên
   dùng Alias cho nhóm port
2. Active Directory: promote domain controller, tạo OU (IT, Warehouse, CustomerService),
   Security Group, GPO khóa Control Panel cho OU Warehouse, join máy Windows vào domain
3. MySQL: thiết kế schema products, customers, orders; user ứng dụng chỉ có quyền trên một
   database và chỉ kết nối từ dải mạng Server; backup tự động bằng script
4. Redis: đặt mật khẩu, giới hạn bộ nhớ, chính sách allkeys-lru, persistence RDB,
   5 key mẫu mô phỏng cache giỏ hàng và session
5. Web app Flask kết nối MySQL và Redis để tạo luồng dữ liệu thật cho hệ thống giám sát
6. Zabbix + Grafana: cài Agent2 trên các máy, giám sát CPU/RAM/Disk và MySQL,
   tạo trigger, gửi cảnh báo qua Telegram, dựng dashboard
7. Kiểm chứng: từ CLIENT-01 truy cập Redis bị chặn, đúng với firewall rule đã đặt

## Các lỗi thực tế đã gặp và cách xử lý
Tổng hợp 15 lỗi trong mục 3 của báo cáo, ví dụ:
- Domain join báo "AD DC could not be contacted" dù ping và DNS đúng: thiếu rule cho
  Kerberos, LDAP, SMB, RPC; thêm rule cho phép mọi giao thức tới đúng IP của DC-01
- Ping vào được nhưng không có reply: Subnet Mask trên DC-01 bị đặt nhầm thành /8
- Rule tới DC-01 vô tình mở cả dải Server: khai báo nhầm /24 thay vì /32
- User giám sát MySQL báo Access denied: nhầm @'localhost' (socket) với @'127.0.0.1' (TCP)
- Zabbix Server tự gọi vào chính nó: đặt Interface của host là 127.0.0.1 thay vì IP thật của máy
- MySQL và Zabbix tự dừng giữa chừng: unattended-upgrades của Ubuntu tự restart service

## Hạn chế và hướng mở rộng
Bản lab chưa triển khai: High Availability cho web (HAProxy + 2 web server), PostgreSQL, ELK Stack, Ansible. Đây là các hướng phát triển tiếp theo.

## Tài liệu
- Báo cáo đầy đủ (sơ đồ, các bước, lệnh, ảnh chụp): BaoCao_HaTang_TiviMart.pdf

## Ghi chú
- Đây là môi trường lab học tập, không phải hệ thống production

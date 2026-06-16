# TỔNG HỢP KIẾN THỨC SD-WAN FORTIGATE
## ÔN TẬP CHI TIẾT CHO PHỎNG VẤN

---

## 1. TỔNG QUAN SD-WAN

### 1.1. Định nghĩa
- **SD-WAN (Software-Defined Wide Area Network)** là giải pháp quản lý mạng WAN bằng phần mềm, tự động chọn đường truyền tối ưu cho từng ứng dụng dựa trên chất lượng đường truyền thực tế.

### 1.2. Các khái niệm cơ bản
| Thuật ngữ | Định nghĩa |
|-----------|------------|
| **Underlay** | Lớp hạ tầng vật lý (cổng WAN, MPLS, Internet, 4G/5G) - "con đường thật" |
| **Overlay** | Lớp mạng ảo (VPN tunnel, ADVPN shortcut) chạy trên underlay - "con đường ảo" |
| **SD-Branch** | Mở rộng SD-WAN vào LAN, tích hợp FortiSwitch và FortiAP |
| **FortiGate** | Thiết bị NGFW tích hợp SD-WAN |
| **FortiManager** | Quản lý cấu hình tập trung nhiều FortiGate |
| **FortiAnalyzer** | Tập trung log, báo cáo, cảnh báo |
| **FortiMonitor** | Nền tảng DEM (đo trải nghiệm người dùng) dạng SaaS - **không phải tính năng của FortiGate** |

### 1.3. Lợi ích của SD-WAN
- **Giảm chi phí**: Sử dụng nhiều loại đường (MPLS, Internet, 4G) thay vì chỉ dùng MPLS đắt tiền
- **Giảm độ phức tạp**: Quản lý tập trung từ một giao diện duy nhất
- **Cải thiện hiệu suất ứng dụng**: Ứng dụng quan trọng luôn được ưu tiên đường tốt nhất
- **Tối ưu trải nghiệm người dùng**: Đặc biệt với các ứng dụng SaaS (Office 365, Zoom)

---

## 2. KIẾN TRÚC SD-WAN (5 TRỤ CỘT)

| Trụ cột | Vai trò | Cấu hình chính |
|---------|---------|----------------|
| **Underlay** | Xác định link vật lý + đo chất lượng | `config system sdwan → config members` |
| **Overlay** | Xây dựng VPN tunnels giữa các site | IPsec, ADVPN |
| **Routing** | Traditional (biết đường) + SD-WAN rules (chọn đường) | Static routes, BGP, OSPF |
| **Security** | NGFW + zones | Firewall policies, app control |
| **SD-WAN** | Zones → Members → SLA → Rules | `config system sdwan` |

### 2.1. Interface Preference vs Zone Preference
priority-members: wan1, wan2, "Internet-Zone"

Thứ tự ưu tiên:

wan1 (interface preference) → cao nhất

wan2 (interface preference)

"Internet-Zone" (zone preference) → thấp hơn

→ Interface liệt kê riêng luôn ưu tiên hơn interface trong zone

text

---

## 3. PERFORMANCE SLA (HEALTH CHECK)

### 3.1. Server
- Tối đa **2 server** (IP hoặc FQDN)
- **Cả 2 server cùng fail** mới coi link DOWN (tránh false positive)
- Nên ping đúng resource cần đo (VD: office365.com)

### 3.2. SLA Targets
| Chỉ số | Đơn vị | Mặc định | Giá trị tốt nhất |
|--------|--------|----------|------------------|
| Latency | ms | 5 | Càng thấp càng tốt |
| Jitter | ms | 5 | Càng thấp càng tốt |
| Packet Loss | % | 0 | Càng thấp càng tốt |

### 3.3. Link Status
| Tham số | Mặc định | Ý nghĩa |
|---------|----------|---------|
| `check-interval` | 500ms | Tần suất kiểm tra |
| `failures-before-inactive` | 5 | Số lần **fail liên tiếp** để down link |
| `restore-link-after` | 5 | Số lần **success liên tiếp** để up link |

> ⚠ **"Liên tiếp"** = không có success xen giữa → nếu có 1 success, reset bộ đếm về 0.

### 3.4. Active vs Passive
| Active | Passive |
|--------|---------|
| Gửi probe chủ động (ICMP, HTTP, TCP) | Đo từ traffic thực tế |
| Tốn băng thông | Không tốn băng thông |
| Cần cấu hình server | Không cần server |
| Đo được khi không có traffic | Chỉ đo khi có traffic |

**3 chế độ:**
- `active` (mặc định)
- `passive` (chỉ dùng traffic thực)
- `prefer-passive` (ưu tiên traffic thực, nếu 3 phút không traffic → chuyển sang active)

### 3.5. MOS (Mean Opinion Score)
- Thang điểm **1-5** đo chất lượng thoại
- Codec: G.711, G.729, G.722

| Điểm | Chất lượng |
|------|------------|
| 4.3 - 5.0 | Excellent |
| 4.0 - 4.3 | Good |
| 3.6 - 4.0 | Fair |
| 3.1 - 3.6 | Poor |
| 1.0 - 3.1 | Bad |

### 3.6. Logging
| Loại | Lệnh/Cấu hình | Thời gian lưu |
|------|---------------|---------------|
| Short-term | `diag sys sdwan sla-log <name> <member>` | 10 phút |
| Long-term (fail) | `set sla-fail-log-period 30` | Theo cấu hình |
| Long-term (pass) | `set sla-pass-log-period 60` | Theo cấu hình |

---

## 4. SD-WAN RULES

### 4.1. Cấu trúc cơ bản
```bash
config system sdwan
    config service
        edit 1
            set name "rule-name"
            set input-device "lan"
            set src "192.168.1.0/24"
            set dst "all"
            set mode sla                    # manual / sla / priority / auto
            set health-check "google-dns"
            set priority-members 1 2 3
        next
    end
end
4.2. Các Mode
Mode	Đặc điểm	Khi nào dùng
manual	Chỉ định link cứng, không dùng SLA	Biết rõ link nào tốt, muốn kiểm soát chặt
sla	Chọn link đạt SLA	Muốn FortiGate tự chọn dựa trên SLA
priority	Kết hợp priority + SLA	Ưu tiên link theo thứ tự, nhưng chỉ dùng nếu đạt SLA
auto	FortiGate tự chọn	Đơn giản, không cần cấu hình phức tạp
4.3. Implicit Rule
Rule ngầm định cuối cùng khi không có rule nào match

Mặc định: mode auto + load-balance-mode source-ip-based

Nếu implicit rule không có member khả dụng → traffic bị DROP

5. STRATEGY (CHIẾN LƯỢC)
5.1. Các Strategy
Strategy	Cách chọn
Automatic	FortiGate tự chọn tốt nhất
Manual	Theo thứ tự priority-members
Best Quality	Link có chất lượng tốt nhất
Lowest Cost	Link có cost thấp nhất
Hybrid	Link đạt SLA + trong đó chọn cost thấp nhất
5.2. Link-cost factor
Factor	Giá trị tốt nhất
latency	Thấp nhất
jitter	Thấp nhất
packet-loss	Thấp nhất
mos	Cao nhất
5.3. Link-cost-threshold
Mặc định: 10%

Tạo lợi thế 10% cho link đứng đầu tiên trong priority-members

Công thức: Latency_hiệu_dụng = Latency_thực / (1 + threshold%)

6. LOAD BALANCING ALGORITHMS
Thuật toán	Cách hoạt động	Sticky?
Round-robin	Lần lượt A→B→C→A...	Không
Source IP	Hash theo IP nguồn	Có
Source-Dest IP	Hash theo IP nguồn + đích	Có (cao hơn)
Session (weight-based)	Chia theo số session, có weight	Không
Volume (measured-volume)	Chia theo dung lượng thực, có weight	Không
Spillover	Dùng link A đến ngưỡng rồi mới dùng B	Không
Inbandwidth	Chọn link rảnh nhất chiều vào	Không
Outbandwidth	Chọn link rảnh nhất chiều ra	Không
Bibandwidth	Chọn link rảnh nhất cả 2 chiều	Không
7. APPLICATION STEERING
7.1. Khái niệm
Application Steering = nhận diện ứng dụng → gửi qua link phù hợp

7.2. ISDB vs Application Control
Phương pháp	Nguồn	Tốc độ	Session đầu match?
Internet Service (ISDB)	FortiGuard database (IP/port)	Nhanh	✅ Có thể match
Application Control	Signature (chữ ký)	Chậm	❌ Không (cần vài gói)
7.3. Các cách match trong CLI
CLI	Ý nghĩa
internet-service-app-ctrl <id>	Ứng dụng cụ thể
internet-service-app-ctrl-group <tên>	Nhóm ứng dụng
internet-service-app-ctrl-category <id>	Danh mục ứng dụng
internet-service-name <tên>	Dịch vụ (ISDB)
8. DSCP TAG-BASED STEERING
DSCP = 6 bit trong IP header (0-63), đánh dấu ưu tiên

Khi router quá tải → giữ gói DSCP cao, drop gói DSCP thấp trước

DSCP	Tên	Ứng dụng	ToS hex
46	EF	VoIP	0xb8
34	AF41	Video	0x88
26	AF31	ERP	0x68
18	AF21	-	0x48
10	AF11	Web, Email	0x28
0	BE	Mặc định	0x00
bash
config system sdwan
    config service
        edit 1
            set tos 0xb8          # DSCP 46
            set tos-mask 0xfc     # lấy 6 bit DSCP, bỏ 2 bit ECN
        next
    end
end
9. MPLS vs DIA
Tiêu chí	DIA	MPLS
Bản chất	Internet thuần túy	Mạng riêng
Ra Internet	✅ Có trực tiếp	❌ Không (phải qua gateway)
Chất lượng	Best-effort	Cam kết (SLA)
Giá	Rẻ	Đắt
Ứng dụng	Web, email, SaaS	VoIP, video call, nội bộ
10. ECMP & LONGEST MATCH
10.1. Longest Match
Route có subnet mask dài nhất (chi tiết nhất) được ưu tiên

set tie-break fib-best-match để SD-WAN chỉ xét route cụ thể nhất

10.2. Override quality comparisons
Route cụ thể (longest match) vẫn được ưu tiên ngay cả khi chất lượng kém hơn

Lý do: Đúng đích quan trọng hơn nhanh nhưng sai đích

10.3. Cách chọn port sau longest match
Mode	Cách chọn
Manual	Chọn port đầu tiên còn sống
Best Quality	Chọn port có chất lượng tốt nhất
Lowest Cost	Chọn port có cost thấp nhất (trong số đạt SLA)
11. IPv6 TRONG SD-WAN
bash
config system sdwan
    config members
        edit 1
            set interface "wan1"
            set gateway6 "2000:172:16:208::2"
        next
    end
    
    config health-check
        edit "ipv6-hc"
            set addr-mode ipv6
            set server "2000::2:2:2:2"
        next
    end
    
    config service
        edit 1
            set addr-mode ipv6
            set internet-service-name "Microsoft-Office-365"
        next
    end
end
12. ADVPN (AUTO DISCOVERY VPN)
12.1. Khái niệm
Spoke chỉ cần kết nối đến Hub

Khi 2 Spoke cần nói chuyện, ADVPN tự tạo shortcut trực tiếp giữa chúng

Loại tunnel	Kết nối	Tồn tại
Parent tunnel	Spoke ↔ Hub	Luôn có
Shortcut	Spoke ↔ Spoke	Tạo khi cần, tự xóa sau
12.2. Luồng tạo shortcut
text
Spoke A → Hub → Spoke B (parent tunnels)
        ↓
Hub giới thiệu IP của B cho A
        ↓
Spoke A và B tạo shortcut trực tiếp
        ↓
Traffic sau đi thẳng (không qua Hub)
13. ADVPN 2.0
13.1. So sánh ADVPN 1.0 vs 2.0
Tính năng	ADVPN 1.0	ADVPN 2.0
Chọn đường	Hub giới thiệu IP	Spoke tự chọn đường tốt nhất
Biết chất lượng trước?	❌ Không	✅ Có (latency, cost)
Cần user traffic?	✅ Cần	❌ Không cần
Nhiều shortcut?	❌ Không	✅ Có (load balancing)
Tự động tắt?	❌ Không	✅ Có (shared idle timeout)
13.2. Edge Discovery & Path Management
Cơ chế	Vai trò
Edge Discovery	Spoke thu thập thông tin WAN links của Spoke khác (IP, latency, cost)
Path Management	Spoke tự tính toán và tạo shortcut qua đường tốt nhất
13.3. Overlay Placeholder
Tunnel "giả" trên Spoke, không kết nối đến Hub

Dùng để Hub biết về underlay mà Hub không có (VD: MPLS)

bash
config vpn ipsec phase1-interface
    edit "placeholder-mpls"
        set type dynamic
        set auto-discovery-dialup-placeholder enable
        # không set remote-gateway
    next
end
13.4. SLA Stickiness
Mặc định: enable

Session cũ → ở lại đường cũ (không chuyển)

Session mới → mới chuyển về primary (nếu đạt SLA)

13.5. Hold Down Time
Thời gian giữ shortcut vừa hồi phục ở mức ưu tiên thấp

Mặc định: 0 giây (tắt)

Tránh flapping khi shortcut chập chờn

14. BGP TRONG SD-WAN & ADVPN
14.1. BGP là gì?
Giao thức	Phạm vi	Ví dụ
OSPF	Nội bộ công ty	Router A ↔ Router B
BGP	Giữa các công ty/ISP	Viettel ↔ VNPT, Công ty A ↔ Công ty B
14.2. Active Dynamic BGP Neighbor
BGP neighbor giữa 2 Spoke chỉ được tạo sau khi shortcut ADVPN được tạo

Spoke chỉ học route từ Spoke có shortcut (không học route thừa)

Dấu hiệu BGP đã học route qua shortcut:

bash
get router info routing-table bgp
B 33.1.1.0/24 via 10.10.200.3, spoke1-2-phase1_0   # _0 = shortcut
15. MULTICAST TRONG SD-WAN
15.1. Khái niệm
Loại	Ví dụ	Đặc điểm
Unicast	Gọi điện riêng	1 nói 1 nghe
Broadcast	Loa phường	1 nói tất cả nghe (kể cả người không muốn)
Multicast	Truyền hình cáp	1 nói, chỉ người đăng ký mới nghe
15.2. RP (Rendezvous Point)
Điểm trung tâm của multicast

Nên đặt RP trên loopback (interface ảo, không bao giờ down)

15.3. Cấu hình multicast dùng SD-WAN
bash
config router multicast
    set multicast-routing enable
    config pim-sm-global
        set pim-use-sdwan enable
        config rp-address
            edit 1
                set ip-address 10.255.255.1
            next
        end
    end
end

config system sdwan
    config service
        edit 1
            set name "multicast"
            set dst "224.0.0.0/4"   # IPv4 multicast range
            set priority-members 1 2
        next
    end
end
16. SPEED TEST
16.1. 2 bước xác thực
Bước	Đường	Daemon	Mục đích
Bước 1	Overlay (IPsec)	Controller daemon	Cấp token
Bước 2	Underlay	Speed test daemon	Gửi token + đo tốc độ
16.2. Phương thức chạy
Phương thức	Dùng khi nào
On-demand	Chạy thủ công, kiểm tra nhanh
Scheduled	Chạy tự động theo lịch
Cloud Assisted	Cần license, tự động cập nhật
Hub-to-Spoke	Trong mô hình dial-up VPN
16.3. Ứng dụng kết quả
Cập nhật estimated bandwidth cho SD-WAN strategy

Traffic shaping (cấu hình inbandwidth/outbandwidth)

Egress shaping trên Hub

17. OaaS (OVERLAY-AS-A-SERVICE)
Fortinet cung cấp Hub sẵn trên cloud

Bạn chỉ cần cấu hình Spoke kết nối đến OaaS Hub

OoSLA = SLA có sẵn từ OaaS Hub (không cần cấu hình health-check)

Tự dựng Hub	Dùng OaaS
Tự thuê VM trên AWS/Azure	Fortinet lo cloud
Tự cấu hình Hub	Không cần cấu hình Hub
Tự trả cloud phí	Trả phí thuê bao theo Spoke
18. TROUBLESHOOTING CLI
Mục đích	Lệnh
Ping	execute ping 8.8.8.8
Routing	get router info routing-table all
BGP neighbor	get router info bgp summary
BGP route	get router info routing-table bgp
SD-WAN health-check	diag sys sdwan health-check
SD-WAN service	diag sys sdwan service <id>
SLA log (10 phút)	diag sys sdwan sla-log <name> <member>
Policy route	diag firewall proute list
VPN tunnel list	diag vpn tunnel list
Debug flow	diag debug flow filter saddr <IP>
diag debug flow trace start 100
Application steering cache	diag sys sdwan internet-service-app-ctrl-list
Reset debug	diag debug reset
19. NHỮNG ĐIỂM DỄ NHẦM
Nhầm lẫn	Thực tế
SLA fail log ghi khi thay đổi	Ghi định kỳ theo sla-fail-log-period
1 server fail → link down	Cần cả 2 server cùng fail
failures-before-inactive = 5 lần bất kỳ	Cần 5 lần liên tiếp (không có success xen giữa)
Hybrid là mode riêng	Không phải mode chính thức, là cách kết hợp priority + SLA
FortiMonitor là tính năng của FortiGate	Là sản phẩm riêng biệt dạng SaaS
Session đầu match app rule ngay	Session đầu chưa được nhận diện → không match
IPv6 không cần quan tâm	Nhiều ISP và ứng dụng đã dùng IPv6
Shortcut tự động load balance	Cần load-balance enable + tie-break fib-best-match
20. MẸO NHỚ NHANH
text
SD-WAN: 5 trụ cột (Underlay, Overlay, Routing, Security, SD-WAN)

Rule: Manual (cứng) - SLA (thông minh) - Priority (kết hợp) - Auto (tự động)

Strategy: Best Quality (ngon nhất) - Lowest Cost (rẻ nhất) - Hybrid (đủ ngon + rẻ)

Backup: priority-members quyết định thứ tự, SLA quyết định khi nào chuyển

Load balancing: Round-robin (lần lượt) - Source IP (sticky) - Volume (theo dung lượng)

MOS: 5 sao, càng cao càng ngon

DSCP: càng cao càng ưu tiên, ít bị drop khi tắc nghẽn

ADVPN 1.0: Hub giới thiệu IP
ADVPN 2.0: Spoke tự chọn đường tốt nhất

Placeholder: tunnel giả để Hub biết đường mà Hub không có
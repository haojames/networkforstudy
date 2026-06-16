# 🔁 vMotion, HA, DRS – Các tính năng nâng cao của vSphere

## 1. vMotion – Di chuyển VM đang chạy

**Bản chất:**  
vMotion cho phép di chuyển một **máy ảo đang chạy** từ ESXi host này sang ESXi host khác **mà không tắt máy, không ngắt kết nối người dùng**.

### 1.1. Cách hoạt động kỹ thuật (4 bước)

| Bước | Tên | Mô tả |
|------|-----|-------|
| 1 | **Pre-copy** | Sao chép bộ nhớ RAM của VM từ host nguồn sang host đích. VM vẫn chạy bình thường. Các trang nhớ bị thay đổi trong quá trình này gọi là *dirty page*. |
| 2 | **Iteration** | vMotion chạy lại nhiều lần để sao chép các *dirty page* đã phát sinh. |
| 3 | **Stun time** | Khi lượng trang nhớ còn lại đủ nhỏ, VM bị tạm dừng trong vài mili giây đến vài giây. Toàn bộ CPU và trạng thái bộ nhớ được chuyển sang host đích. |
| 4 | **Resume** | VM tiếp tục hoạt động trên ESXi host đích. |

### 1.2. Điều kiện để sử dụng vMotion

| Điều kiện | Yêu cầu |
|-----------|----------|
| **Shared Storage** | Cả hai ESXi host phải truy cập được cùng datastore (NFS, iSCSI, VMFS). Lý do: chỉ RAM và cấu hình được copy, disk VM thì không. |
| **Cùng CPU family** | CPU phải tương thích (Intel ↔ AMD là không được). Có thể dùng **EVC (Enhanced vMotion Compatibility)** để che giấu sự khác biệt. |
| **VMkernel network** | Phải có mạng riêng (VMkernel) dành cho vMotion. |

### 1.3. Các hạn chế cần biết

| Hạn chế | Giải thích |
|----------|------------|
| CD/DVD vật lý | Không thể vMotion VM đang dùng CD/DVD vật lý từ host nguồn. |
| USB device | VM có USB device phải được bật chế độ hỗ trợ vMotion. |

---

## 2. HA – High Availability (Tính sẵn sàng cao)

**Bản chất:**  
HA tự động **khởi động lại VM** trên host khác khi một ESXi host bị lỗi (mất điện, treo, mất mạng).

> ⚠️ **Quan trọng:** VM bị **tắt và bật lại** → có downtime (khác với vMotion).

### 2.1. Cách hoạt động (3 bước)

| Bước | Mô tả |
|------|-------|
| 1 – **Monitor by Heartbeat** | Mỗi ESXi host trong cluster gửi tín hiệu "còn sống" (heartbeat) đến các host khác mỗi giây. |
| 2 – **Phát hiện lỗi** | Cluster bầu chọn một **Master**. Nếu Master không nhận được heartbeat từ host con, nó sẽ kiểm tra thêm bằng ICMP ping và database heartbeat để xác nhận lỗi. |
| 3 – **Khởi động lại VM** | Master chọn một host khác còn tài nguyên và khởi động lại VM từ host bị lỗi sang host đó. |

### 2.2. Các loại lỗi host mà HA phát hiện được

| Loại lỗi | Mô tả |
|----------|-------|
| **Failure** | Host ngừng hoạt động hoàn toàn. |
| **Isolation** | Host vẫn chạy nhưng bị cô lập khỏi mạng quản lý. |
| **Partition** | Host mất kết nối với master nhưng vẫn thấy được các host khác. |

---

## 3. DRS – Distributed Resource Scheduler (Tự động cân bằng tải)

**Bản chất:**  
DRS tự động di chuyển VM bằng **vMotion** giữa các host để cân bằng tài nguyên CPU và RAM, đảm bảo không host nào bị quá tải.

### 3.1. Cách hoạt động

| Bước | Mô tả |
|------|-------|
| 1 – **Theo dõi liên tục** | Mỗi 5 phút, DRS kiểm tra và tính toán mức sử dụng CPU/RAM của từng VM. |
| 2 – **Đưa ra khuyến nghị hoặc tự động di chuyển** | Nếu phát hiện mất cân bằng, DRS sẽ đề xuất hoặc tự động thực hiện vMotion VM sang host khác. |

### 3.2. Các chế độ hoạt động của DRS

| Chế độ | Mô tả |
|--------|-------|
| **Manual** | DRS chỉ gợi ý di chuyển, admin phải bấm apply. |
| **Partially Automated** | Tự động đặt VM khi khởi tạo, nhưng cân bằng tải thì cần manual. |
| **Fully Automated** | DRS tự động mọi thứ, không cần can thiệp. |

### 3.3. Mức độ "hung dữ" (Migration Threshold)

Từ **1 (ít di chuyển nhất)** đến **5 (di chuyển nhiều nhất)**.

| Mức | Hành vi |
|-----|---------|
| 1 | Chỉ di chuyển khi mất cân bằng nghiêm trọng |
| 3 | Cân bằng trung bình (mặc định) |
| 5 | Di chuyển rất chủ động, ưu tiên cân bằng tuyệt đối |

---

## 📊 Tóm tắt so sánh nhanh

| Tính năng | vMotion | HA | DRS |
|-----------|---------|----|-----|
| **Mục đích** | Di chuyển VM sống | Tự động khởi động lại VM khi host lỗi | Cân bằng tải chủ động |
| **Có downtime không?** | ❌ Không | ✅ Có (VM khởi động lại) | ❌ Không (dùng vMotion) |
| **Điều kiện cần** | Shared storage, cùng CPU family, mạng riêng | Cluster, heartbeat network | DRS phải được bật trên cluster |
| **Tự động hay thủ công?** | Thủ công (hoặc qua DRS) | Tự động hoàn toàn | Có cấu hình manual → auto |

---

## ✅ Ghi nhớ nhanh

- **vMotion** = di chuyển VM **đang chạy** → 0 downtime.
- **HA** = host chết → **bật lại VM** trên host khác → có downtime nhưng tự động.
- **DRS** = tự động dùng vMotion để **cân bằng tải** giữa các host.
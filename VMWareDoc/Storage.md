# 💾 Storage trong VMware vSphere

## 1. Datastore là gì?

**Datastore** là một **kho chứa ảo** nơi lưu trữ tất cả các file của máy ảo (VM).

Bạn có thể hình dung nó như một **ổ đĩa logic** mà ESXi host nhìn thấy và sử dụng để lưu trữ:

| File | Mô tả |
|------|-------|
| `.vmx` | File cấu hình VM |
| `.vmdk` | File disk ảo của VM |
| `.log`, `.nvram`, ... | Các file khác của VM |

### 1.1. Phân loại Datastore theo vị trí

Datastore có thể được tạo trên nhiều loại storage khác nhau:

| Loại | Đặc điểm | Mức độ dùng trong production |
|------|----------|------------------------------|
| **Local Storage** | Ổ cứng ngay bên trong server vật lý | ❌ Ít dùng |
| **Network Storage** | SAN (FC, iSCSI) hoặc NAS (NFS) – dùng chung cho nhiều host | ✅ Chuẩn doanh nghiệp |

---

## 2. Local Storage – Ít dùng trong production

Local storage là các ổ cứng nằm ngay bên trong server vật lý.

### ⚠️ Vấn đề của Local Storage

| Vấn đề | Hậu quả |
|--------|----------|
| Không dùng được vMotion, HA, DRS | VM chỉ nằm trên một host duy nhất |
| Nếu host chết → VM chết theo | Không thể khôi phục tự động |
| Không thể cân bằng tải | Không thể di chuyển VM sang host khác |

### ✅ Khi nào nên dùng Local Storage?

- Môi trường **lab, học tập, thử nghiệm**
- Môi trường **nhỏ lẻ không cần high availability**
- **Edge computing** hoặc server lẻ không kết nối được network storage

---

## 3. Network Storage – Chuẩn trong doanh nghiệp

Đây là lựa chọn chính trong các môi trường ảo hóa sản xuất.

### 3.1. Phân loại Network Storage

| Loại | Giao thức | Bản chất | Mức độ phổ biến |
|------|-----------|----------|-----------------|
| **SAN** (Block Storage) | iSCSI hoặc Fibre Channel (FC) | Cấp phát theo khối (block) | Rất phổ biến |
| **NAS** (File Storage) | NFS | Cấp phát theo file | Rất phổ biến |
| **vSAN** (Software-Defined) | VMware proprietary | Ảo hóa storage cục bộ | Xu hướng mới, ngày càng phổ biến |

---

### 3.2. SAN – Block Storage (iSCSI / Fibre Channel)

#### Cách hoạt động

ESXi host nhìn thấy một **ổ đĩa logic (LUN)** qua mạng, sau đó định dạng nó thành **VMFS** (hệ thống file của VMware).

#### So sánh iSCSI vs Fibre Channel (FC)

| Tiêu chí | iSCSI | Fibre Channel (FC) |
|----------|-------|---------------------|
| **Hạ tầng mạng** | Ethernet thông thường | Switch FC riêng, cáp quang |
| **Chi phí** | Thấp, dễ triển khai | Cao, phức tạp |
| **Hiệu năng** | Tốt (phụ thuộc mạng) | Rất cao, độ trễ cực thấp |
| **Mức độ dùng** | Phổ biến nhất với doanh nghiệp vừa | Cao cấp (ngân hàng, tài chính) |

#### Ưu / Nhược điểm của SAN

| Ưu điểm | Nhược điểm |
|---------|-------------|
| Hiệu năng tốt | Phức tạp hơn NFS |
| Hỗ trợ đầy đủ vMotion / HA / DRS | Cần cấu hình initiator, target, LUN masking |

---

### 3.3. NAS – File Storage (NFS)

#### Cách hoạt động

ESXi **mount trực tiếp thư mục** từ NFS server, **không cần VMFS**.

#### Ưu / Nhược điểm của NFS

| Ưu điểm | Nhược điểm |
|---------|-------------|
| Cực kỳ đơn giản, dễ cấu hình | Hiệu năng có thể thấp hơn SAN một chút |
| Quản lý dung lượng linh hoạt | (tùy thuộc vào hạ tầng mạng) |

#### Mức độ dùng

Rất phổ biến, đặc biệt trong môi trường dùng **NAS thương mại** (NetApp, Dell EMC Isilon) hoặc các giải pháp **software-defined storage**.

---

### 3.4. vSAN – Software-Defined Storage (Xu hướng mới)

**vSAN** là giải pháp lưu trữ định nghĩa bằng phần mềm của VMware.

#### Cách hoạt động

vSAN biến **ổ cứng local trên mỗi ESXi host** thành một **shared datastore ảo** dùng chung cho cả cluster.

#### Lợi ích của vSAN

| Lợi ích | Mô tả |
|---------|-------|
| Hiệu năng của local storage | Đọc/ghi nhanh nhờ ổ cứng cục bộ |
| Tính năng của shared storage | Hỗ trợ đầy đủ vMotion, HA, DRS |
| Không cần SAN/NAS riêng | Tiết kiệm chi phí phần cứng |

> 📌 vSAN đang rất phổ biến hiện nay, đặc biệt trong các môi trường HCI (Hyperconverged Infrastructure).

---

## 4. Tổng kết – Bảng so sánh các loại Storage

| Loại | Giao thức | Bản chất | Hiệu năng | Chi phí | Hỗ trợ vMotion/HA/DRS | Độ phổ biến trong production |
|------|-----------|----------|-----------|---------|----------------------|------------------------------|
| **Local** | SATA/SAS/NVMe | Block | Trung bình | Thấp | ❌ Không | Rất thấp |
| **SAN (iSCSI)** | iSCSI | Block | Cao | Trung bình | ✅ Có | Rất cao |
| **SAN (FC)** | Fibre Channel | Block | Rất cao | Rất cao | ✅ Có | Cao (cấp cao) |
| **NAS (NFS)** | NFS | File | Trung bình – Cao | Trung bình | ✅ Có | Rất cao |
| **vSAN** | VMware proprietary | Block (ảo) | Cao | Trung bình – Cao | ✅ Có | Đang tăng nhanh |

---

## 📖 Bảng thuật ngữ nhanh

| Thuật ngữ | Giải thích |
|-----------|-------------|
| **SAN** | Storage Area Network – mạng riêng cho storage, cấp phát theo khối (block) |
| **iSCSI** | Internet SCSI – giao thức SCSI qua mạng IP, rẻ hơn FC |
| **FC** | Fibre Channel – dùng cáp quang riêng, đắt, hiệu năng cao |
| **NAS** | Network Attached Storage – thiết bị lưu trữ gắn mạng, cấp phát theo file |
| **NFS** | Network File System – giao thức file dùng phổ biến cho NAS |
| **vSAN** | VMware vSAN – giải pháp storage định nghĩa bằng phần mềm của VMware |
| **VMFS** | VMware Virtual Machine File System – hệ thống file riêng của VMware cho datastore |

---

## ✅ Ghi nhớ nhanh

- **Local storage** → chỉ dùng lab, **không** hỗ trợ vMotion/HA/DRS.
- **Network storage** → chuẩn doanh nghiệp, **hỗ trợ đầy đủ** tính năng cao cấp.
- **SAN (iSCSI/FC)** → block storage, hiệu năng cao, cấu hình phức tạp hơn.
- **NAS (NFS)** → file storage, đơn giản, dễ dùng, phổ biến.
- **vSAN** → xu hướng mới, biến local storage thành shared storage.
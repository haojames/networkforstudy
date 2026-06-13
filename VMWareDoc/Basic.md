# 📘 Tài liệu Ảo hóa & VMware vSphere

## 1. Hypervisor

**Hypervisor** là lớp phần mềm trung gian nằm giữa phần cứng vật lý và các máy ảo (Virtual Machine - VM).  
Nhiệm vụ chính: **ảo hóa tài nguyên** (CPU, RAM, Disk, Network) để nhiều VM có thể chạy đồng thời trên cùng một máy chủ vật lý.

### Phân loại Hypervisor

| Loại | Đặc điểm | Ví dụ |
|------|----------|-------|
| **Type 1 – Bare-metal (Native)** | Cài trực tiếp lên phần cứng, không cần OS host. Hiệu năng cao, ít overhead. | VMware ESXi, Microsoft Hyper-V, KVM |
| **Type 2 – Hosted** | Cài như một ứng dụng trên OS host (Windows, Linux, macOS). Hiệu năng thấp hơn do phải qua lớp OS host. | VMware Workstation, Oracle VirtualBox |

---

## 2. VMware vSphere

**vSphere** là một bộ sản phẩm (suite) ảo hóa hoàn chỉnh của VMware, bao gồm:

- Hypervisor **ESXi**
- **vCenter Server**
- Các công cụ quản lý khác

### 2.1. ESXi – Hypervisor

- Là hypervisor **Type 1 (bare-metal)**, cài trực tiếp lên server vật lý.
- Chịu trách nhiệm chạy và quản lý tài nguyên cho các VM trên chính server đó.
- **Mỗi ESXi chỉ quản lý được một server vật lý duy nhất**.

### 2.2. vCenter Server

- Là **bộ não quản lý tập trung**.
- Cho phép quản lý **nhiều ESXi host** từ một giao diện duy nhất.
- Cung cấp các tính năng cao cấp như: **vMotion, HA, DRS**.

### 2.3. Sơ đồ kiến trúc vSphere
┌─────────────────────────────────────────────────────────────┐
│                      vSPHERE SUITE                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────┐         ┌──────────────────────────┐     │
│   │ vCenter      │         │     ESXi Host 1          │     │
│   │ Server       │◄───────►│  ┌────────────────────┐  │     │
│   │ (Quản lý     │  API     │  │ VM1  │ VM2  │ VM3 │  │     │
│   │  tập trung)  │         │  └────────────────────┘  │     │
│   └──────────────┘         │  ┌────────────────────┐  │     │
│            │               │  │   vSphere Switch   │  │     │
│            │               │  └────────────────────┘  │     │
│            │               └───────────┬──────────────┘     │
│            │                           │                    │
│            │               ┌───────────┴──────────────┐     │
│            │               │     ESXi Host 2          │     │
│            └──────────────►│  ┌────────────────────┐  │     │
│                            │  │ VM4  │ VM5  │ VM6  │  │     │
│                            │  └────────────────────┘  │     │
│                            └──────────────────────────┘     │
│                                                             │
│   ┌──────────────────────────────────────────────────────┐  │
│   │           Shared Storage (NFS / iSCSI / VMFS)        │  │
│   │          Chứa file .vmx, .vmdk của các VM            │  │
│   └──────────────────────────────────────────────────────┘  │ 


---

## 3. Máy ảo (Virtual Machine - VM)

**Máy ảo (VM)** = một máy tính được **mô phỏng bằng phần mềm**, chạy trên hypervisor.

### Cấu tạo của một VM

| Thành phần | File | Mô tả |
|------------|------|-------|
| File cấu hình | `.vmx` | Định nghĩa RAM, CPU, disk, network của VM |
| File disk ảo | `.vmdk` | Chứa hệ điều hành và dữ liệu |
| File BIOS/UEFI ảo | `.nvram` | Lưu cài đặt BIOS/UEFI |
| File log | `.log` | Ghi lại hoạt động của VM |

### Cách VM hoạt động

- Hypervisor **cấp phát tài nguyên vật lý** cho VM một cách "ảo".
- VM **không biết** mình đang chạy trên hypervisor (nếu được tối ưu).
- Mỗi VM có:
  - **Hệ điều hành riêng**
  - **Ứng dụng riêng**
  - **IP riêng**
  - Hoàn toàn **độc lập** với các VM khác.

---

## 4. ESXi – Chi tiết

| Đặc điểm | Mô tả |
|----------|--------|
| **Loại** | Hypervisor Type 1 (bare-metal) |
| **Cài đặt** | Trực tiếp lên server vật lý |
| **Chức năng chính** | Chạy và quản lý VM trên chính server đó |
| **Phạm vi quản lý** | **Chỉ 1 host** (server vật lý mà nó đứng trên) |
| **Tính năng nâng cao** | ❌ Không có vMotion, HA, DRS nếu chỉ dùng ESXi đơn lẻ |
| **Giao diện quản lý** | Host Client (truy cập qua IP của ESXi) |

> ⚠️ **Lưu ý:** Mất 1 ESXi → mất các VM trên host đó.

---

## 5. vCenter Server – Chi tiết

| Đặc điểm | Mô tả |
|----------|--------|
| **Loại** | Ứng dụng quản lý tập trung (thường là máy ảo Linux - VCSA) |
| **Chức năng chính** | Kết nối và quản lý **nhiều ESXi host** cùng lúc |
| **Tính năng nâng cao** | ✅ **vMotion**, **HA**, **DRS** (chỉ có khi dùng vCenter) |
| **Giao diện quản lý** | vSphere Client (truy cập qua IP của vCenter) |

> ⚠️ **Lưu ý:** Mất vCenter → các ESXi và VM vẫn chạy bình thường, nhưng:
> - ❌ Mất khả năng di chuyển VM (vMotion)
> - ❌ Mất tự động failover (HA)
> - ❌ Mất cân bằng tải (DRS)

---

## 6. So sánh nhanh ESXi vs vCenter Server

| Tiêu chí | ESXi | vCenter Server |
|----------|------|----------------|
| **Vai trò** | Chạy VM | Quản lý tập trung |
| **Cài đặt** | Trực tiếp lên server vật lý | Thường là VM Linux (VCSA) |
| **Quản lý được bao nhiêu host?** | 1 host | Nhiều host |
| **vMotion, HA, DRS** | ❌ Không | ✅ Có |
| **Khi mất thành phần này** | Mất các VM trên host đó | VM vẫn chạy, nhưng mất tính năng cao cấp |

---

## ✅ Tóm tắt nhanh

- **Hypervisor** = phần mềm ảo hóa tài nguyên.
- **ESXi** = hypervisor Type 1, chạy VM trên 1 server.
- **vCenter** = điều phối nhiều ESXi, cung cấp tính năng cao cấp.
- **VM** = máy tính ảo gồm `.vmx`, `.vmdk`,... hoạt động độc lập.
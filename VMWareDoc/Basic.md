1. Hypervisor
Hypervisor = lớp phần mềm trung gian giữa phần cứng vật lý và máy ảo (VM). Nó có nhiệm vụ ảo hóa tài nguyên (CPU, RAM, disk, network) để nhiều VM có thể chạy đồng thời trên cùng một server.
Type 1 – Bare-metal (Native)
Cài đặt trực tiếp lên phần cứng, không cần OS host.
Hiệu năng cao, ít overhead.
Type 2 -Hosted
Cài như một ứng dụng trên OS host (Windows, Linux, macOS).
Hiệu năng thấp hơn do phải qua OS host.
2. vSphere
Một bộ sản phẩm (suite) bao gồm hypervisor ESXi, vCenter Server và các công cụ quản lý khác
2.1. Hypervisor & ESXi: ESXi là hypervisor Type 1 (bare-metal), cài trực tiếp lên server vật lý. Nó chịu trách nhiệm chạy và quản lý tài nguyên cho các máy ảo (VM) .
2.2. vCenter Server: Là bộ não quản lý tập trung. Nó cho phép quản lý nhiều ESXi host từ một giao diện duy nhất, cung cấp các tính năng như vMotion, HA, DRS

┌─────────────────────────────────────────────────────────────┐
│                      vSPHERE SUITE                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────────┐         ┌──────────────────────────┐    │
│   │ vCenter      │         │     ESXi Host 1           │    │
│   │ Server       │◄───────►│  ┌────────────────────┐  │    │
│   │ (Quản lý     │  API     │  │ VM1  │ VM2  │ VM3  │  │    │
│   │  tập trung)  │         │  └────────────────────┘  │    │
│   └──────────────┘         │  ┌────────────────────┐  │    │
│            │               │  │   vSphere Switch   │  │    │
│            │               │  └────────────────────┘  │    │
│            │               └───────────┬──────────────┘    │
│            │                           │                    │
│            │               ┌───────────┴──────────────┐    │
│            │               │     ESXi Host 2           │    │
│            └──────────────►│  ┌────────────────────┐  │    │
│                            │  │ VM4  │ VM5  │ VM6  │  │    │
│                            │  └────────────────────┘  │    │
│                            └──────────────────────────┘    │
│                                                              │
│   ┌──────────────────────────────────────────────────────┐ │
│   │           Shared Storage (NFS / iSCSI / VMFS)        │ │
│   │          Chứa file .vmx, .vmdk của các VM            │ │
│   └──────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

3. Máy ảo (VM) là gì?
Máy ảo (Virtual Machine – VM) = một máy tính được mô phỏng bằng phần mềm, chạy trên hypervisor.
VM bao gồm:
File cấu hình (.vmx) – định nghĩa RAM, CPU, disk, network của VM.
File disk (.vmdk) – chứa hệ điều hành và dữ liệu.
File BIOS ảo (.nvram) – lưu cài đặt BIOS/UEFI.
File log (.log) – ghi lại hoạt động của VM.
VM hoạt động thế nào?
Hypervisor cấp phát tài nguyên vật lý cho VM một cách "ảo".
VM không biết mình đang chạy trên hypervisor (nếu được tối ưu).
Mỗi VM có OS riêng, ứng dụng riêng, IP riêng – hoàn toàn độc lập.

4. ESxi
ESXi là hypervisor, nó được cài trực tiếp lên server vật lý. Mỗi ESXi chỉ quản lý được một server duy nhất và có thể chạy nhiều VM trên đó. Muốn quản lý ESXi, có thể dùng Host Client hoặc SSH. Các tính năng như vMotion, HA, DRS không có trên ESXi đơn lẻ.

Hypervisor Type 1, cài trực tiếp lên server vật lý.
Nó chạy và quản lý VM trên chính server đó.
Quản lý được duy nhất 1 host (server vật lý mà nó đứng trên).
Không có vMotion, HA, DRS nếu chỉ dùng ESXi đơn lẻ.
Giao diện quản lý: Host Client (truy cập qua IP của ESXi).

5. vCenter Server
vCenter Server là một ứng dụng quản lý tập trung, thường được triển khai dưới dạng máy ảo Linux. Nó kết nối đến nhiều ESXi host cùng lúc và cho phép tôi quản lý toàn bộ hệ thống từ một giao diện duy nhất. Chỉ khi có vCenter, tôi mới dùng được vMotion, HA, DRS.

Ứng dụng quản lý tập trung (thường là máy ảo Linux - VCSA).
Nó kết nối và quản lý nhiều ESXi host cùng lúc.
Chỉ khi có vCenter, mới dùng được vMotion, HA, DRS.
Giao diện quản lý: vSphere Client (truy cập qua IP của vCenter).

Noted: ESXi = trực tiếp chạy VM. Mất 1 ESXi → mất các VM trên host đó.
vCenter = điều phối các ESXi. Mất vCenter → các ESXi và VM vẫn chạy bình thường, nhưng mất khả năng di chuyển VM (vMotion), mất tự động failover (HA), mất cân bằng tải (DRS).

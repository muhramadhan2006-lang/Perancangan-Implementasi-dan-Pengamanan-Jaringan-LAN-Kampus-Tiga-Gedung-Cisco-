## **Perancangan, Implementasi, dan Pengamanan Jaringan LAN Kampus Tiga Gedung menggunakan Cisco Packet Tracer, OSPF Multi-Area, VLAN, ACL, DHCP Snooping, dan Port Security**

## Syarat dan Ketentuan
- **Skala Perangkat per Gedung:** 
  - Gedung A (4 lantai)
  - Gedung B (4 lantai)
  - Gedung C (2 lantai)
  - Setiap lantai memiliki jumlah perangkat spesifik dengan cadangan pertumbuhan (*growth headroom*) sekitar 30%.
- **Arsitektur & Perencanaan IP:** 
  - Menggunakan ruang alamat IP privat RFC 1918.
  - Desain hierarkis dengan pemisahan *access layer*, *distribution layer*, dan *inter-building routed backbone*.
- **Konfigurasi Routing (OSPF):** 
  - Menggunakan protokol *Multi-Area OSPFv2* (Area 0 sebagai *backbone*, Area 10 untuk Gedung A, Area 20 untuk Gedung B, dan Area 30 untuk Gedung C).
  - Mengaktifkan autentikasi OSPF (MD5) pada jalur antar-gedung serta menerapkan *route summarization* di ABR.
- **Kontrol Keamanan Wajib (*Mandatory Security Controls*):**
  - **IP ACLs:** Membatasi akses *inter-VLAN* dan manajemen jaringan.
  - **DHCP Snooping:** Diaktifkan secara global dan per VLAN dengan pengaturan *trusted* pada uplink dan *untrusted* pada port user.
  - **Port Security:** Pembatasan jumlah MAC address maksimum per port akses untuk mencegah akses ilegal.

## Rancangan IP dan VLAN
### 1. Analisis Kapasitas & Perencanaan Subnet (Headroom 30%)
| Segmen | Jumlah Device Saat Ini | Estimasi +30% | Subnet Terpilih | Alasan |
| :--- | :---: | :---: | :--- | :--- |
| **A-Fl.1** | 60 | 78 | `/25 (255.255.255.128)` | Cukup untuk 78 host. |
| **A-Fl.2** | 80 | 104 | `/25 (255.255.255.128)` | Cukup untuk 104 host. |
| **A-Fl.3** | 30 | 39 | `/26 (255.255.255.192)` | Cukup untuk 39 host. |
| **A-Fl.4** | 10 | 13 | `/28 (255.255.255.240)` | Cukup untuk 13 host. |
| **B-Fl.1** | 120 | 156 | `/24 (255.255.255.0)` | Cukup untuk 156 host. |
| **B-Fl.2** | 90 | 117 | `/25 (255.255.255.128)` | Cukup untuk 117 host. |
| **B-Fl.3** | 12 | 16 | `/27 (255.255.255.224)` | Cukup untuk 16 host. |
| **B-Fl.4** | 7 | 10 | `/28 (255.255.255.240)` | Cukup untuk 10 host. |
| **C-Fl.1** | 200 | 260 | `/23 (255.255.254.0)` | Cukup untuk 260 host. |
| **C-Fl.2** | 100 | 130 | `/24 (255.255.255.0)` | Cukup untuk 130 host. |

### 2. Tabel VLAN, Subnet, & Default Gateway
| Lantai | VLAN ID | Subnet | Default Gateway / SVI | Keterangan |
| :--- | :---: | :--- | :--- | :--- |
| **A-Fl.1** | VLAN 10 | `/25 (255.255.255.128)` | `192.168.0.1` | User |
| **A-Fl.2** | VLAN 20 | `/25 (255.255.255.128)` | `192.168.0.129` | User |
| **A-Fl.3** | VLAN 30 | `/26 (255.255.255.192)` | `192.168.1.1` | User |
| **A-Fl.4** | VLAN 40 | `/28 (255.255.255.240)` | `192.168.1.65` | User |
| **B-Fl.1** | VLAN 110 | `/24 (255.255.255.0)` | `192.168.2.1` | User |
| **B-Fl.2** | VLAN 120 | `/25 (255.255.255.128)` | `192.168.3.1` | User |
| **B-Fl.3** | VLAN 130 | `/27 (255.255.255.224)` | `192.168.3.129` | User |
| **B-Fl.4** | VLAN 140 | `/28 (255.255.255.240)` | `192.168.3.161` | Admin |
| **C-Fl.1** | VLAN 210 | `/23 (255.255.254.0)` | `172.16.0.1` | User |
| **C-Fl.2** | VLAN 220 | `/24 (255.255.255.0)` | `192.168.4.1` | User |

## Daftar Perangkat yang Digunakan
| Jenis Perangkat | Model/Simulasi | Jumlah | Fungsi |
| :--- | :--- | :---: | :--- |
| **Router** | ISR 4331 / 2911 (PT) | 3 | Backbone/OSPF, DHCP untuk VLAN 140 |
| **L3 Switch** | PT Multilayer Switch | 3 | SVI, Inter-VLAN, dan OSPF |
| **L2 Switch** | PT Catalyst 2960 | 42 | Akses user |
| **PC/Laptop** | PT-PC / PT-Laptop | 923 | End device pengujian |
| **Server DHCP** | Server / L3 Device | 1 | Layanan DHCP |






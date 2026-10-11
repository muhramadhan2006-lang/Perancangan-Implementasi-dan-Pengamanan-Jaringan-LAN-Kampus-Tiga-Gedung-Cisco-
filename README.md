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

### 3. Network Statement & Pembagian Area OSPF
| Perangkat | Interface / VLAN | Network | Area |
| :--- | :--- | :--- | :---: |
| **MLS-A** | VLAN 10 | `192.168.0.0/25` | 10 |
| **MLS-A** | VLAN 20 | `192.168.0.128/25` | 10 |
| **MLS-A** | VLAN 30 | `192.168.1.0/26` | 10 |
| **MLS-A** | VLAN 40 | `192.168.1.64/28` | 10 |
| **MLS-A** | Fa0/1 | `10.1.0.0/30` | 0 |
| **Router A** | Gi0/0/1 | `10.1.0.0/30` | 0 |
| **Router A** | Gi0/0/0 | `10.1.0.4/30` | 0 |
| **MLS-B** | VLAN 110 | `192.168.2.0/24` | 20 |
| **MLS-B** | VLAN 120 | `192.168.3.0/25` | 20 |
| **MLS-B** | VLAN 130 | `192.168.3.128/27` | 20 |
| **MLS-B** | VLAN 140 | `192.168.3.160/28` | 20 |
| **MLS-B** | Fa0/1 | `10.1.1.0/30` | 0 |
| **Router B** | Gi0/0 | `10.1.1.0/30` | 0 |
| **Router B** | Gi0/1 | `10.1.0.4/30` | 0 |
| **Router B** | Gi0/2 | `10.1.2.0/30` | 0 |
| **MLS-C** | VLAN 210 | `172.16.0.0/23` | 30 |
| **MLS-C** | VLAN 220 | `192.168.4.0/24` | 30 |
| **MLS-C** | Fa0/1 | `10.1.3.0/30` | 0 |
| **Router C** | Gi0/0/0 | `10.1.2.0/30` | 0 |
| **Router C** | Gi0/0/1 | `10.1.3.0/30` | 0 |

## Daftar Perangkat yang Digunakan
| Jenis Perangkat | Model/Simulasi | Jumlah | Fungsi |
| :--- | :--- | :---: | :--- |
| **Router** | ISR 4331 / 2911 (PT) | 3 | Backbone/OSPF, DHCP untuk VLAN 140 |
| **L3 Switch** | PT Multilayer Switch | 3 | SVI, Inter-VLAN, dan OSPF |
| **L2 Switch** | PT Catalyst 2960 | 42 | Akses user |
| **PC/Laptop** | PT-PC / PT-Laptop | 923 | End device pengujian |
| **Server DHCP** | Server / L3 Device | 1 | Layanan DHCP |

## Topologi yang digunakan
<img width="1882" height="792" alt="image" src="https://github.com/user-attachments/assets/bdb1be82-8283-474d-beb1-8d1c2a4cc76e" />

## Konfigurasi di Cisco Paket Tracer
### 1. Konfigurasi Trunking

**Konfigurasi Trunking Pada MLS Building A dan Building B**[cite: 33]

    interface FastEthernet0/2
    switchport trunk encapsulation dot1q
    switchport mode trunk
    !
    interface FastEthernet0/3
    switchport trunk encapsulation dot1q
    switchport mode trunk
    !
    interface FastEthernet0/4
    switchport trunk encapsulation dot1q
    switchport mode trunk
    !
    interface FastEthernet0/5
    switchport trunk encapsulation dot1q
    switchport mode trunk

**Konfigurasi Trunking Pada MLS Building C**[cite: 33]

    interface FastEthernet0/2
    switchport trunk encapsulation dot1q
    switchport mode trunk
    !
    interface FastEthernet0/3
    switchport trunk encapsulation dot1q
    switchport mode trunk

---

### 2. Konfigurasi VLAN, SVI, dan DHCP Helper-Address (Multilayer Switch)

**MLS Building A**[cite: 34]

    interface Vlan10
    ip address 192.168.0.1 255.255.255.128
    ip helper-address 192.168.3.162
    !
    interface Vlan20
    ip address 192.168.0.129 255.255.255.128
    ip helper-address 192.168.3.162
    !
    interface Vlan30
    ip address 192.168.1.1 255.255.255.192
    ip helper-address 192.168.3.162
    !
    interface Vlan40
    ip address 192.168.1.65 255.255.255.240
    ip helper-address 192.168.3.162

**MLS Building B**[cite: 35]

    interface Vlan110
    ip address 192.168.2.1 255.255.255.0
    ip helper-address 192.168.3.162
    !
    interface Vlan120
    ip address 192.168.3.1 255.255.255.128
    ip helper-address 192.168.3.162
    !
    interface Vlan130
    ip address 192.168.3.129 255.255.255.224
    ip helper-address 192.168.3.162
    !
    interface Vlan140
    ip address 192.168.3.161 255.255.255.240

**MLS Building C**[cite: 36]

    interface Vlan210
    ip address 172.16.0.1 255.255.254.0
    ip helper-address 192.168.3.162
    !
    interface Vlan220
    ip address 192.168.4.1 255.255.255.0
    ip helper-address 192.168.3.162

---

### 3. Konfigurasi Routing OSPF Multi-Area & Autentikasi MD5

**Router Building A**[cite: 42]

    router ospf 1
    router-id 11.11.11.11
    log-adjacency-changes
    area 0 authentication message-digest
    network 10.1.0.0 0.0.0.3 area 0
    network 10.1.0.4 0.0.0.3 area 0
    exit
    interface GigabitEthernet0/0/1
    ip ospf message-digest-key 1 md5 CNIS2026
    exit
    interface GigabitEthernet0/0/0
    ip ospf message-digest-key 1 md5 CNIS2026

**Router Building B**[cite: 42]

    router ospf 1
    router-id 22.22.22.22
    log-adjacency-changes
    area 0 authentication message-digest
    network 10.1.1.0 0.0.0.3 area 0
    network 10.1.0.4 0.0.0.3 area 0
    network 10.1.2.0 0.0.0.3 area 0
    exit
    interface GigabitEthernet0/0
    ip ospf message-digest-key 1 md5 CNIS2026
    exit
    interface GigabitEthernet0/1
    ip ospf message-digest-key 1 md5 CNIS2026
    exit
    interface GigabitEthernet0/2
    ip ospf message-digest-key 1 md5 CNIS2026

**Router Building C**[cite: 43]

    router ospf 1
    router-id 33.33.33.33
    log-adjacency-changes
    area 0 authentication message-digest
    network 10.1.3.0 0.0.0.3 area 0
    network 10.1.2.0 0.0.0.3 area 0
    exit
    interface GigabitEthernet0/0/0
    ip ospf message-digest-key 1 md5 CNIS2026
    exit
    interface GigabitEthernet0/0/1
    ip ospf message-digest-key 1 md5 CNIS2026

**Multilayer Switch (MLS Building A)**[cite: 43]

    router ospf 1
    router-id 1.1.1.1
    log-adjacency-changes
    area 0 authentication message-digest
    network 192.168.0.0 0.0.0.127 area 10
    network 192.168.0.128 0.0.0.127 area 10
    network 192.168.1.0 0.0.0.63 area 10
    network 192.168.1.64 0.0.0.15 area 10
    network 10.1.0.0 0.0.0.3 area 0
    area 10 range 192.168.0.0 255.255.254.0
    exit
    interface FastEthernet0/1
    ip ospf message-digest-key 1 md5 CNIS2026

**Multilayer Switch (MLS Building B)**[cite: 43, 44]

    router ospf 1
    router-id 2.2.2.2
    log-adjacency-changes
    area 0 authentication message-digest
    network 192.168.2.0 0.0.0.255 area 20
    network 192.168.3.0 0.0.0.127 area 20
    network 192.168.3.128 0.0.0.31 area 20
    network 192.168.3.160 0.0.0.15 area 20
    network 10.1.1.0 0.0.0.3 area 0
    area 20 range 192.168.2.0 255.255.254.0
    exit
    interface FastEthernet0/1
    ip ospf message-digest-key 1 md5 CNIS2026

**Multilayer Switch (MLS Building C)**[cite: 44]

    router ospf 1
    router-id 3.3.3.3
    log-adjacency-changes
    area 0 authentication message-digest
    network 172.16.0.0 0.0.1.255 area 30
    network 192.168.4.0 0.0.0.255 area 30
    network 10.1.3.0 0.0.0.3 area 0
    area 30 range 172.16.0.0 255.255.254.0
    exit
    interface FastEthernet0/1
    ip ospf message-digest-key 1 md5 CNIS2026

---

### 4. Konfigurasi Access Control List (ACL 100)[cite: 44, 45]

    access-list 100 permit icmp any 192.168.3.160 0.0.0.15 echo-reply
    access-list 100 deny icmp any 192.168.3.160 0.0.0.15 echo
    access-list 100 permit ip any any
    !
    interface vlan 10
    ip access-group 100 in
    interface vlan 20
    ip access-group 100 in
    interface vlan 30
    ip access-group 100 in
    interface vlan 40
    ip access-group 100 in
    interface vlan 110
    ip access-group 100 in
    interface vlan 120
    ip access-group 100 in
    interface vlan 130
    ip access-group 100 in
    interface vlan 210
    ip access-group 100 in
    interface vlan 220
    ip access-group 100 in

---

### 5. Konfigurasi DHCP Snooping[cite: 46, 47, 48]

    ip dhcp snooping
    ip dhcp snooping vlan 10,20,30,40,110,120,130,140,210,220
    !
    interface FastEthernet0/1
    ip dhcp snooping trust
    !
    interface FastEthernet0/2
    ip dhcp snooping limit rate 10
    interface FastEthernet0/3
    ip dhcp snooping limit rate 10

---

### 6. Konfigurasi Port Security[cite: 48, 49]

    interface FastEthernet0/2
    switchport mode access
    switchport port-security
    switchport port-security maximum 2
    switchport port-security mac-address sticky
    switchport port-security violation restrict
    !
    interface FastEthernet0/3
    switchport mode access
    switchport port-security
    switchport port-security maximum 2
    switchport port-security mac-address sticky
    switchport port-security violation restrict


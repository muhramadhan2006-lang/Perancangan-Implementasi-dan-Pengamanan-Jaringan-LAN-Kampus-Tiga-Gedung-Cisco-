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

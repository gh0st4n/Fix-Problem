# Panduan Troubleshooting: Mengatasi Wi-Fi Soft Blocked (`wlp0s20f3`) pada Linux

## 1. Deskripsi Masalah

Antarmuka jaringan nirkabel (`wlp0s20f3`) berada dalam kondisi **DOWN** dan tidak dapat terhubung ke jaringan. Hal ini terjadi karena modul sistem Lenovo (`ideapad_laptop`) secara otomatis mengaktifkan *soft block* pada radio Wi-Fi.

## 2. Diagnosa

Periksa status pemblokiran perangkat dengan menjalankan perintah berikut:

```bash
rfkill list
```

> **Indikasi Masalah:** Output menampilkan `Soft blocked: yes` pada baris `ideapad_wlan` atau `phy0`.

## 3. Langkah-Langkah Penanganan

1. **Membuka Blokir Radio Wi-Fi**
Buka *soft block* secara keseluruhan:
```bash
sudo rfkill unblock all
```


2. **Melepas Modul Driver Lenovo yang Konflik** *(Opsional / Direkomendasikan)*
Lepas modul `ideapad_laptop` dari kernel untuk mengatasi indikasi pemblokiran palsu:
```bash
sudo modprobe -r ideapad_laptop
```


3. **Menambahkan Modul ke Blacklist (Permanen)** *(Opsional / Direkomendasikan)*
Buat berkas konfigurasi agar modul `ideapad_laptop` tidak dimuat ulang secara otomatis saat booting:
```bash
echo "blacklist ideapad_laptop" | sudo tee /etc/modprobe.d/ideapad.conf
```


4. **Mengaktifkan Kembali Perangkat Jaringan**
Nyalakan antarmuka kartu Wi-Fi dan aktifkan fitur nirkabel pada NetworkManager:
```bash
sudo ip link set wlp0s20f3 up
nmcli radio wifi on
```

## 4. Verifikasi Hasil

* Jalankan `rfkill list` untuk memastikan seluruh baris **Soft blocked** bernilai `no`.
* Jalankan `ip a` atau `nmcli device` untuk memastikan antarmuka `wlp0s20f3` berstatus aktif (**UP** atau **disconnected**).

---

<div align="center">

[@T4n-Labs](https://t4nlabs.web.id/) • [@Gh0sT4n](https://gh0st4n.my.id)

</div>

# Dokumentasi Perbaikan GRUB Rescue (UEFI Mode)

## 1. Ringkasan Masalah

* **Kendala**: Laptop gagal booting dan masuk ke prompt `grub rescue>` dengan pesan `error: no such partition`.
* **Penyebab**: Perubahan skema atau nomor partisi pada drive NVMe/SSD, sehingga variabel `prefix` bawaan GRUB merujuk ke partisi lama yang sudah tidak ditemukan (`hd1,gpt7`).

## 2. Informasi Spesifikasi Sistem

* **Sistem Operasi**: T4n OS (Linux x86_64 / Void Linux)
* **Mode Boot**: UEFI
* **Struktur Disk (`lsblk`)**:
  * **Drive Utama**: `/dev/nvme0n1`
  * **Partisi EFI (ESP)**: `/dev/nvme0n1p5` (Dimuat pada `/boot/efi`)
  * **Partisi Root (`/`)**: `/dev/nvme0n1p6` (Terbaca sebagai `hd1,gpt6` di GRUB)

## 3. Langkah-Langkah Perbaikan

### Tahap 1: Menemukan Partisi Boot & Root

Gunakan perintah `ls` di `grub rescue>` untuk mendaftar seluruh partisi, lalu periksa isi direktori partisi satu per satu untuk menemukan letak folder sistem utama:

```bash
grub rescue> ls
(hd0) (hd0,gpt1) ... (hd1) (hd1,gpt6) ...

grub rescue> ls (hd1,gpt6)/
./ ../ lost+found/ boot/ home/ sys/ mnt/ root/ bin/ tmp/ var/ media/ etc/ sbin/ lib/ lib32/ opt/ proc/ usr/ lib64/ run/ dev/

```

*(Partisi root ditemukan pada `(hd1,gpt6)` karena memiliki folder sistem lengkap seperti `boot/`, `etc/`, `usr/`, dll).*

### Tahap 2: Booting Sementara dari Prompt `grub rescue>`

Ketik perintah berikut secara berurutan di layar `grub rescue>` untuk memuat modul bootloader dan masuk ke OS secara sementara:

```bash
set root=(hd1,gpt6)
set prefix=(hd1,gpt6)/boot/grub
insmod normal
normal

```

### Tahap 3: Pemulihan dan Pembaruan GRUB Permanen

Setelah berhasil masuk ke desktop Linux, perbaikan permanen dilakukan melalui Terminal dengan akses `sudo`:

1. **Memeriksa Pemetaan Partisi**:
```bash
lsblk

```

*Memastikan partisi root (`/`) berada di `nvme0n1p6` dan partisi EFI di `nvme0n1p5` (`/boot/efi`).*
2. **Menginstal Ulang GRUB ke Partisi EFI**:
```bash
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

```

*Hasil*: `Installation finished. No error reported.`
3. **Memperbarui File Konfigurasi GRUB (`grub.cfg`)**:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg

```

*Proses ini memindai ulang seluruh kernel Linux yang terpasang (`vmlinuz-6.18.x`) serta Windows Boot Manager pada drive NVMe, lalu memperbarui pointer partisi root.*
4. **Verifikasi**:
```bash
sudo reboot

```

## 4. Status Akhir

Proses perbaikan selesai. Bootloader UEFI pada `/boot/efi` telah diperbarui dengan lokasi partisi root yang valid (`/dev/nvme0n1p6`). Laptop kini dapat booting langsung ke menu pilihan sistem operasi (GRUB Theme: Sleek) tanpa masuk ke mode *rescue*.


<div align="center">

[@T4n-Labs](https://t4n-labs.github.io/site) · [@Gh0sT4n](https://gh0st4n.github.io/site)

</div>
```

```

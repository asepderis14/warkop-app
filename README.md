# Kasir Warkop (Offline-First PWA)

Aplikasi Point of Sale (POS) berbasis web modern yang dirancang khusus untuk operasional **Warung Kopi (Warkop)**. Aplikasi dioptimalkan untuk perangkat **tablet dalam orientasi landscape**, mengutamakan kecepatan transaksi oleh *solo operator* (kasir merangkap pembuat pesanan), tombol besar ramah sentuhan jari, serta berjalan **100% offline-first (tanpa server backend)**.

---

## 🚀 Fitur Utama

1. **POS / Kasir Cepat (Split-Screen Tablet Landscape):**
   - **Kolom Kiri (65%):** Filter kategori (Kopi, Minuman Dingin, Mie, Gorengan, Lainnya), pencarian cepat, kartu menu sentuh besar dengan badge status dan jumlah pesanan.
   - **Kolom Kanan (35%):** Keranjang pesanan aktif dengan tombol `-` dan `+` besar, subtotal per item, total tagihan besar, tombol simpan Open Bill, dan tombol Bayar.

2. **Open Bill (Pesanan Aktif / Nongkrong):**
   - Simpan pesanan pelanggan yang masih nongkrong (misal: "Meja 2", "Mas Budi").
   - Badge jumlah Open Bill aktif secara *real-time* di sidebar.
   - Buka kembali pesanan ke keranjang kasir untuk menambah pesanan atau memproses pelunasan.

3. **Modal Pembayaran Cepat:**
   - **Tunai (Cash):** Tombol cepat pecahan uang (`Uang Pas`, `Rp 5.000`, `Rp 10.000`, `Rp 20.000`, `Rp 50.000`, `Rp 100.000`, serta penambahan cepat nominal), kalkulasi kembalian otomatis *real-time*.
   - **QRIS:** Otomatis uang pas, dapat menampilkan barcode QRIS toko yang diunggah dari menu Pengaturan.
   - Tombol penyelesaian: **"Selesai & Cetak Struk"** atau **"Selesai (Tanpa Cetak)"**.

4. **Cetak Struk Kasir (Printer Thermal Native):**
   - Kompatibel dengan printer thermal Bluetooth/USB ukuran kertas `58mm` dan `80mm`.
   - Menggunakan `@media print` native `window.print()` hemat kertas dengan font monospaced yang rapi.
   - Tampilan struk mencakup nama warkop, alamat, no telepon, no struk, waktu, nama pelanggan, daftar item, total, metode bayar, dan pesan ucapan terima kasih.

5. **Manajemen Menu:**
   - Tambah, edit, dan hapus menu warkop.
   - Sakelar 1-klik untuk status stok (Tersedia / Habis) agar kasir tidak menjual menu yang habis saat jam sibuk.

6. **Laporan & Rekap Harian:**
   - Pemilih tanggal harian.
   - 4 kartu ringkasan: Total Omzet, Total Tunai, Total QRIS, dan Jumlah Transaksi.
   - Daftar Menu Terlaris (*ranking* penjualan terbanyak).
   - Riwayat transaksi harian dengan tombol **"Cetak Ulang Struk"**.

7. **Pengaturan & Cadangan Data (Backup & Restore JSON):**
   - Kustomisasi nama warkop, alamat, nomor telepon, footer struk, dan ukuran kertas printer.
   - Pengelolaan kategori menu warkop.
   - Unggah gambar barcode QRIS warkop.
   - **Export Data (.JSON):** Unduh seluruh database lokal ke file JSON.
   - **Import & Pulihkan Data:** Muat ulang data dari file backup JSON.

8. **PWA & 100% Offline-First:**
   - Ditenagai oleh Service Worker (`vite-plugin-pwa`) dengan *precaching* lengkap seluruh aset.
   - Dapat diinstal di tablet (Add to Home Screen / Install App) dan tetap berjalan penuh tanpa koneksi internet.
   - Penyimpanan data lokal menggunakan browser IndexedDB melalui `dexie`.

---

## 🛠️ Tech Stack

- **Framework:** React 18 + Vite (TypeScript)
- **Styling:** Tailwind CSS v3
- **Database Lokal:** Dexie.js (IndexedDB) + `dexie-react-hooks`
- **Ikon:** Lucide React
- **PWA:** `vite-plugin-pwa`

---

## 🏃 Cara Menjalankan

### Mode Pengembangan (Development)
```bash
npm run dev
```
Buka browser di `http://localhost:5173`.

### Mode Produksi & Build PWA
```bash
npm run build
npm run preview
```
Aset siap pakai akan berada di folder `dist/`.


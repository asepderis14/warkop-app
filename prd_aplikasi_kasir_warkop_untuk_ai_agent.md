# PRD & Spesifikasi Teknis: Aplikasi Kasir Warkop (Offline-First PWA)

Dokumen ini adalah panduan lengkap bagi AI Agent untuk membangun aplikasi Point of Sale (POS) berbasis web untuk Warung Kopi (Warkop).

## 1. Konteks & Tujuan Utama
* **Perangkat Target:** Tablet dalam orientasi **Landscape** (layar lebar, layar sentuh).
* **Alur Pengguna:** Kasir merangkap pembuat pesanan (*solo operator*). Aplikasi harus sangat cepat, menggunakan tombol besar yang ramah jari, dan membutuhkan jumlah ketukan (*tap*) seminimal mungkin.
* **Konektivitas:** **100% Offline-First (Local-First)**. Aplikasi wajib tetap berjalan penuh, menyimpan data, dan mencetak struk meskipun tablet tidak terhubung ke internet sama sekali.
* **Bahasa Antarmuka (UI):** Bahasa Indonesia. Format mata uang: Rupiah (`Rp 10.000`, tanpa desimal `,00`).

---

## 2. Tech Stack (Wajib)
Gunakan *stack* berikut tanpa menambahkan *backend* atau *database server*:
* **Framework:** React (Vite)
* **Styling:** Tailwind CSS
* **Ikon:** `lucide-react`
* **Database Lokal:** IndexedDB menggunakan `dexie` dan `dexie-react-hooks`
* **PWA / Offline Mode:** `vite-plugin-pwa`
* **Cetak Struk:** Native `window.print()` dengan CSS khusus `@media print`

---

## 3. Skema Database Lokal (Dexie.js)
Buat file `src/db/database.js` (atau `.ts`) dengan tabel berikut:

1. **`menus`**
   * `++id`: Primary key (auto-increment)
   * `name`: String (Contoh: "Kopi Hitam", "Indomie Goreng Telur")
   * `price`: Number (Contoh: `5000`, `12000`)
   * `category`: String (Contoh: "Kopi", "Minuman Dingin", "Mie", "Gorengan", "Lainnya")
   * `isAvailable`: Boolean (default: `true`)

2. **`openBills`** (Pesanan yang belum dibayar / masih nongkrong)
   * `++id`: Primary key
   * `customerName`: String (Nama pelanggan atau nomor meja, contoh: "Meja 2", "Mas Budi")
   * `items`: Array of objects `[{ menuId, name, price, qty }]`
   * `total`: Number
   * `createdAt`: Timestamp / ISO String
   * `updatedAt`: Timestamp / ISO String

3. **`transactions`** (Riwayat transaksi lunas)
   * `++id`: Primary key
   * `receiptNumber`: String (Format unik, contoh: `WKP-YYYYMMDD-001`)
   * `customerName`: String (Default: "Pelanggan Umum")
   * `items`: Array of objects `[{ menuId, name, price, qty }]`
   * `total`: Number
   * `paymentMethod`: String (`"Tunai"` | `"QRIS"`)
   * `amountPaid`: Number
   * `change`: Number
   * `createdAt`: Timestamp / ISO String

4. **`settings`**
   * `key`: Primary key (String)
   * `value`: Any
   * *Data default:* `shopName` ("Warkop Nusantara"), `shopAddress` ("Jl. Contoh No. 123"), `receiptFooter` ("Terima kasih sudah mampir!").

### Data Awal (Seeding)
Saat database pertama kali kosong, otomatis isi tabel `menus` dengan 10–12 menu khas warkop (contoh: Kopi Hitam Rp 5.000, Kopi Susu Rp 7.000, Es Teh Manis Rp 4.000, Nutrisari Es Rp 5.000, Indomie Goreng Rp 8.000, Indomie Goreng + Telur Rp 12.000, Indomie Rebus + Telur Rp 12.000, Gorengan Rp 2.000, Roti Bakar Rp 10.000, Air Mineral Rp 4.000).

---

## 4. Desain UI/UX (Tablet Landscape First)
* **Navigasi Utama:** Sidebar kiri tipis (lebar ~80px) berisi ikon besar dan label singkat untuk 5 menu utama:
  1. **Kasir** (Halaman default)
  2. **Open Bill** (Dengan badge jumlah bill yang sedang aktif)
  3. **Menu** (Kelola daftar menu)
  4. **Laporan** (Rekap penjualan harian)
  5. **Pengaturan** (Profil warkop & Backup data)
* **Prinsip Sentuh:** Semua tombol menu, tombol `+` / `-`, dan tombol aksi minimal berukuran `48px` agar tidak salah pencet saat kasir sedang terburu-buru.
* **Tema Visual:** Bersih, kontras tinggi, nyaman dilihat dalam waktu lama (latar abu-abu sangat terang `bg-slate-50`, kartu putih bersih, warna aksen hangat seperti *amber/emerald* untuk tombol aksi utama).

---

## 5. Rincian Fitur Utama

### A. Halaman Kasir (POS) - Split Screen
* **Kolom Kiri (65% lebar layar) - Daftar Menu:**
  * Baris atas: Tab filter kategori (*Semua, Kopi, Minuman Dingin, Mie, Gorengan, Lainnya*) + kolom pencarian cepat.
  * Area utama: Grid kartu menu (3–4 kolom). Setiap kartu menampilkan Nama Menu dan Harga dengan huruf jelas.
  * Menu dengan `isAvailable: false` tampil redup dengan label "Habis" dan tidak bisa diklik.
  * **Interaksi:** 1x klik kartu menu langsung menambahkan item ke keranjang (atau menambah `qty` jika sudah ada).
* **Kolom Kanan (35% lebar layar) - Keranjang Aktif:**
  * Header keranjang: Menampilkan nama pelanggan jika sedang memuat dari *Open Bill*, plus tombol "Reset/Kosongkan".
  * Daftar item: Nama, harga satuan, tombol `-` / `+` besar, dan subtotal item.
  * Footer keranjang: Total Harga besar & mencolok, serta dua tombol utama:
    * **Simpan Bill (Open Bill):** Membuka popup singkat untuk input Nama Pelanggan/Meja (jika bill baru), lalu menyimpan ke tabel `openBills` dan mengosongkan keranjang. Jika sedang mengedit *Open Bill* lama, langsung perbarui bill tersebut.
    * **Bayar:** Membuka modal pembayaran.

### B. Modal Pembayaran Cepat
* Menampilkan Total Tagihan secara besar.
* Pilihan metode bayar: **Tunai** atau **QRIS**.
* Jika **Tunai**:
  * Input nominal uang diterima.
  * **Tombol Nominal Cepat:** `Uang Pas`, `Rp 5.000`, `Rp 10.000`, `Rp 20.000`, `Rp 50.000`, `Rp 100.000` (klik tombol langsung mengisi/menambah nominal agar kasir tidak perlu mengetik).
  * Menampilkan **Kembalian** secara *real-time* (hijau jika cukup, merah jika kurang).
* Jika **QRIS**: Nominal otomatis dianggap uang pas.
* Tombol aksi akhir: **"Selesai & Cetak Struk"** dan **"Selesai (Tanpa Cetak)"**.
* Setelah transaksi selesai: Simpan ke `transactions`, hapus dari `openBills` (jika berasal dari Open Bill), dan kosongkan keranjang.

### C. Halaman Open Bill (Pesanan Aktif)
* Menampilkan daftar kartu *Open Bill* yang belum dibayar (Nama Pelanggan/Meja, waktu pesan, ringkasan item, total sementara).
* **Interaksi:** Klik tombol **"Buka / Tambah Pesanan"** pada kartu akan memuat isi bill tersebut ke Halaman Kasir agar kasir bisa menambah pesanan baru atau langsung memproses pembayaran.
* Sediakan tombol hapus/batal bill dengan konfirmasi.

### D. Fitur Cetak Struk (Kompatibel Printer Biasa)
* Buat komponen khusus `ReceiptPrint.jsx` yang tersembunyi di tampilan layar biasa (`hidden print:block`), sementara seluruh elemen aplikasi lainnya disembunyikan saat mencetak (`print:hidden`).
* Gunakan CSS `@media print` agar hasil cetak rapi, hemat kertas (lebar konten maksimal `80mm` rata kiri atau tengah), menggunakan font monospace yang jelas, dan tidak memotong halaman secara aneh.
* Isi struk: Nama Warkop, Alamat, Nomor Struk, Tanggal & Jam, Nama Pelanggan, Daftar Item (`Nama`, `Qty x Harga`, `Subtotal`), Garis pemisah, Total, Metode Bayar, Bayar, Kembalian, dan Pesan Footer.

### E. Manajemen Menu
* Tabel atau daftar menu dengan tombol **Tambah Menu**, **Edit**, **Hapus**, dan **Toggle Tersedia/Habis** (agar kasir bisa menonaktifkan menu yang habis dalam 1 klik).

### F. Laporan Harian
* Pemilih tanggal (*Date Picker*, default: Hari Ini).
* **4 Kartu Ringkasan:** Total Omzet, Total Tunai, Total QRIS, dan Jumlah Transaksi.
* **Daftar Menu Terlaris:** Urutan menu yang paling banyak terjual pada tanggal tersebut.
* **Riwayat Transaksi:** Daftar transaksi pada tanggal tersebut dengan tombol **"Cetak Ulang Struk"** untuk setiap transaksi.

### G. Pengaturan & Backup Data (Penting untuk Offline App)
* Form untuk mengubah Nama Warkop, Alamat, dan Footer Struk.
* **Backup & Restore:**
  * Tombol **Export Data (JSON):** Mengunduh seluruh isi database (`menus`, `openBills`, `transactions`, `settings`) ke dalam satu file `.json` dengan nama `backup-warkop-YYYY-MM-DD.json`.
  * Tombol **Import Data (JSON):** Membaca file `.json` hasil backup dan memulihkan data ke IndexedDB (dengan dialog konfirmasi sebelum menimpa data).

---

## 6. Konfigurasi PWA (Offline-First)
Konfigurasikan `vite-plugin-pwa` di `vite.config.js`:
* `registerType: 'autoUpdate'`
* `workbox: { globPatterns: ['**/*.{js,css,html,ico,png,svg}'] }`
* Manifest lengkap: `name: 'Kasir Warkop'`, `short_name: 'Kasir Warkop'`, `display: 'standalone'`, `orientation: 'landscape'`, serta warna tema yang sesuai.

---

## 7. Instruksi Eksekusi untuk AI Agent
Kerjakan proyek ini secara terstruktur dan pastikan tidak ada *error* saat *build*:
1. **Inisialisasi:** Buat proyek Vite + React + Tailwind CSS, lalu install `lucide-react`, `dexie`, `dexie-react-hooks`, dan `vite-plugin-pwa`.
2. **Database:** Implementasikan skema Dexie.js dan fungsi *seeding* menu awal.
3. **Core POS & Open Bill:** Bangun layout tablet *split-screen*, manajemen keranjang, alur *Open Bill*, dan Modal Pembayaran Cepat.
4. **Cetak Struk:** Implementasikan komponen `ReceiptPrint` dan aturan `@media print` di `index.css`.
5. **Manajemen & Laporan:** Bangun halaman Manajemen Menu, Laporan Harian, serta halaman Pengaturan lengkap dengan fitur Export/Import JSON.
6. **PWA & Verifikasi:** Konfigurasi `vite-plugin-pwa`, lalu jalankan `npm run build` untuk memastikan tidak ada *error* kompilasi.
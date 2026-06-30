# 11 — Fokus: Integritas Keuangan & Akurasi Scan Area

Dua prioritas yang ditegaskan pemilik (2026-06-30):
1. **Data keuangan harus benar** — paling takut salah perhitungan.
2. **Scan Area se-akurat mungkin** — fitur vital untuk cari prospek warung.

Batasan mutlak: **semua tetap gratis** — Vercel free, Firebase Spark, peta OpenStreetMap,
tanpa layanan berbayar. Semua usulan di bawah dirancang agar tetap di tier gratis.

> Ini dokumen arah pengembangan (belum dikerjakan). Bug konkret yang sudah ditemukan ada di
> [07 — Bug Register](07-bug-register.md) & [10 — Task Tracker](10-task-tracker.md). Dokumen
> ini menambah perbaikan **struktural** agar dua area ini lebih tahan-bug ke depan.

---

## A. Integritas Keuangan

### Kenapa rawan sekarang
- Perhitungan uang **terduplikasi** di backoffice, mobile, dan kadang di komponen UI —
  bila satu beda rumus, hasilnya menyimpang diam-diam.
- Ada nilai uang yang disimpan sebagai **float hasil pembagian** (mis. `sellingPrice =
  sellingUnitPrice / packSize`, [T-30](10-task-tracker.md)) → drift pembulatan menumpuk.
- Sebagian agregasi (laba rugi, modal) disusun ulang dari beberapa koleksi dengan logika
  dedup yang rapuh ([T-26](10-task-tracker.md), [BUG-1](07-bug-register.md)).
- Tidak ada pemeriksaan **invarian** yang memastikan angka saling konsisten.

### Arah perbaikan (gratis, tanpa Cloud Functions)

**A1 — Satu sumber rumus uang.** [Kemungkinan Besar dampak besar]
Pindahkan semua perhitungan uang/stok ke satu modul murni (mis. `src/lib/money.ts`) yang
dipakai backoffice, dan **disalin-terjaga** ke mobile (lihat [T-08](10-task-tracker.md)
shared-schema). UI hanya menampilkan hasil server, tidak menghitung ulang. Tujuan: satu
rumus, satu tempat diuji.

**A2 — Uang sebagai bilangan bulat rupiah.** [Pasti benar prinsipnya]
Aturan: simpan semua nominal sebagai **integer rupiah**. Jangan pernah menyimpan hasil
bagi (`x / packSize`) sebagai harga. Simpan `sellingUnitPrice` (per pack) + `packSize`,
lalu turunkan harga per-pcs **saat baca** dengan pembulatan eksplisit. Hilangkan float dari
jalur uang. Ini menutup kelas bug pembulatan secara permanen, bukan satu per satu.

**A3 — Pemeriksa invarian (laporan rekonsiliasi).** [Kemungkinan Besar paling berharga]
Buat satu halaman/aksi read-only "Audit Data" yang memverifikasi:
- `warehouseStock + consignedStock + soldStock` konsisten dengan jumlah `stockMovements`.
- `Σ invoicePayments` per invoice == `paidAmount` tersimpan; `paidAmount ≤ totalAmount`.
- `consignedValue` pelanggan == `Σ (sisa titipan × harga)`.
- Total laba-rugi dari dua jalur (manual vs turunan movement) cocok.
Jalankan manual saat tutup buku. Karena read-only, aman & gratis. Ini "alarm" yang membuat
salah-hitung **ketahuan**, bukan diam-diam.

**A4 — Idempotensi & anti dobel-submit.** [Kemungkinan Besar]
Counter invoice/surat sudah transaksional, tapi tambahkan kunci idempoten (mis. tolak
pembuatan invoice kedua untuk kombinasi pelanggan+periode yang sama dalam X detik, atau
client token sekali-pakai) agar klik ganda tidak membuat dokumen uang dobel.

**A5 — Jejak audit untuk setiap mutasi uang.** [Menebak prioritas]
Koleksi `auditLogs` sudah ada di skema — pastikan setiap aksi yang mengubah uang/stok
(invoice, pembayaran, void, adjustment) menulis entri audit (siapa, kapan, sebelum→sesudah).
Memudahkan menelusuri ketika angka terlihat aneh.

**A6 — Kasus uji emas (golden cases).** [Kemungkinan Besar]
Dokumentasikan dan (saat fase coding) uji skenario yang paling sering salah:
titipan pack vs pcs, jual sebagian lalu void, pembayaran sebagian/berlebih, retur,
restock dengan unitCost berubah, transaksi di batas tengah malam WIB. Lihat juga
[T-23](10-task-tracker.md) (timezone WIB).

---

## B. Akurasi Scan Area (tetap gratis)

### Sumber data saat ini (semua gratis)
Dari `venovian-toys/docs/scan-area-strategy.md`:
- **OpenStreetMap tiles** (peta dasar), **Nominatim** (geocode/reverse), **Overpass API**
  (cari POI warung/sekolah/anchor), **Geoapify** (free tier, key di server).

### Risiko akurasi & kuota (gratis = ada batas)
- **Nominatim** punya kebijakan pemakaian ketat: **maks ~1 request/detik**, wajib
  User-Agent jelas, dilarang bulk — bila dilanggar, IP bisa diblokir. [Pasti]
- **Overpass** publik sering **rate-limit / timeout** saat ramai; hasil bisa kosong tanpa
  fallback. [Pasti]
- **Geoapify free tier** terbatas (dokumentasi mereka menyebut kuota harian, mis. ~3.000
  req/hari) — dan endpoint `/scan-prospects` saat ini **tanpa auth** ([T-29](10-task-tracker.md)),
  jadi kuota bisa terkuras pihak luar. [Kemungkinan Besar]
- **Kualitas data OSM bervariasi per wilayah** — warung kecil sering tak terpetakan; skor
  bisa meleset bila hanya mengandalkan POI OSM. [Pasti]

### Arah perbaikan (gratis)

**B1 — Lapisan cache hasil di Firestore.** [Kemungkinan Besar dampak besar]
Simpan hasil Overpass/geocode per-area (mis. per kelurahan/grid koordinat) ke koleksi cache
di Firestore dengan TTL (mis. 30–90 hari). Scan berulang area yang sama membaca cache, bukan
memanggil API lagi. Menghemat kuota, mempercepat, dan mengurangi risiko diblokir — semua gratis.

**B2 — Patuhi kebijakan & tahan-gangguan.** [Pasti perlu]
- Nominatim/Overpass: throttle ≤1 req/detik, set User-Agent identitas app, jeda antar-permintaan.
- Overpass: daftar **beberapa mirror endpoint** + retry dengan backoff; bila semua gagal,
  tampilkan status jelas ke user (bukan hasil kosong yang tampak seperti "tidak ada prospek").
- Amankan `/scan-prospects` dengan sesi + rate-limit ([T-29](10-task-tracker.md)) agar kuota
  Geoapify tak dikuras pihak luar.

**B3 — Umpan balik lapangan menyetel skor.** [Kemungkinan Besar paling menaikkan akurasi]
Graph konsep sudah menyinggung "Field Survey Feedback / Score Potensi / Confidence". Tutup
loop-nya: catat hasil nyata tiap prospek (dikunjungi → deal sukses/gagal) dan gunakan untuk
**menyetel bobot skor** (`classifyProspect`). Skor jadi belajar dari konversi nyata, bukan
hanya heuristik POI. Tidak butuh layanan berbayar — cukup data sendiri di Firestore.

**B4 — Transparansi confidence, bukan akurasi palsu.** [Kemungkinan Besar]
Karena data OSM tak seragam, tampilkan **tingkat keyakinan** tiap prospek + alasan skor
(anchor terdekat, dekat sekolah, kata-kunci) agar sales tahu mana yang layak disurvei
duluan. Lebih jujur daripada satu angka skor yang terkesan pasti.

**B5 — Dedup prospek vs mitra & antar-prospek.** [Kemungkinan Besar]
Pastikan prospek yang sudah jadi pelanggan, atau duplikat antar-scan, tidak muncul/dihitung
ganda (sebagian sudah ada via `existingPartners`/subdistrict). Verifikasi dan perkuat.

**B6 — Reverse-geocode hemat untuk kelurahan.** [Menebak]
Ekstraksi subdistrict kini heuristik (`KNOWN_SUBDISTRICTS`). Akurasi bisa naik dengan
reverse-geocode (Nominatim) **yang di-cache** (B1), bukan memanggil tiap render.

---

## Ringkasan jadi task

Ditambahkan ke [10 — Task Tracker](10-task-tracker.md): T-33…T-40. Semua dirancang gratis,
tanpa Cloud Functions, di atas stack yang ada.

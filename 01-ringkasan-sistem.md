# 01 — Ringkasan Sistem

## Model bisnis

Venovian Toys menjalankan bisnis **titipan (konsinyasi) mainan**: produk dikirim untuk
dititipkan ke warung/toko mitra, lalu performanya dipantau. Mitra menjual; Venovian
menagih hasil penjualan dan/atau menarik sisa stok. Istilah baku di UI adalah **"titipan"**
(bukan "konsinyasi").

Alur operasional inti:

```
Cari prospek warung  →  Survei & deal  →  Kirim stok titipan  →  Pantau penjualan
   (Scan Area)            (lapangan)        (surat titipan)         (kunjungan)
        │                                                               │
        └──────────────  Tagih / setor (invoice)  ←────────────────────┘
                                  │
                          Catat keuangan (laba rugi)
```

## Dua aplikasi, satu database

```
┌─────────────────────────┐         ┌─────────────────────────┐
│   BACKOFFICE (web)       │         │   MOBILE (lapangan)      │
│   Next.js 16             │         │   Expo / React Native    │
│   Operator kantor        │         │   Sales di lapangan      │
│                          │         │                          │
│  Server Actions +        │         │  Firebase JS SDK         │
│  Firebase Admin SDK      │         │  (akses langsung)        │
└───────────┬─────────────┘         └────────────┬─────────────┘
            │                                     │
            │         ┌───────────────────┐       │
            └────────►│   FIREBASE         │◄──────┘
                      │   - Firestore      │
                      │   - Authentication │
                      └───────────────────┘
```

**Poin kunci:** kedua app menulis ke **Firestore yang sama**. Tidak ada API backend
khusus yang memediasi keduanya — mobile mengakses Firestore **langsung** dari device untuk
semua operasi (termasuk update kunjungan prospek). Konsekuensi penting:
**Firestore Security Rules adalah satu-satunya batas keamanan untuk mobile** — lihat
[06 — Kontrak Integrasi](06-kontrak-integrasi.md).

## Pembagian peran fitur

| Domain | Backoffice | Mobile |
|--------|:----------:|:------:|
| Login (email/password Firebase) | ✓ | ✓ |
| Dashboard ringkasan | ✓ (lengkap) | ✓ (ringkas) |
| Pelanggan / mitra warung | ✓ CRUD penuh | ✓ lihat + kunjungan |
| Stok titipan per pelanggan | ✓ kelola penuh | ✓ tambah titipan, lihat |
| Inventori produk/pemasok/kategori | ✓ | ✓ katalog (baca) |
| Invoice & pembayaran | ✓ kelola penuh | ✓ lihat tagihan |
| Keuangan (transaksi, kategori, tren) | ✓ | — |
| Scan Area / prospek | ✓ (peta, scan, simpan) | ✓ daftar prospek + update kunjungan |
| Cetak dokumen | ✓ (cetak web) | ✓ (target printer termal portable) |

Mobile adalah **companion lapangan** — subset dari backoffice yang dioptimalkan untuk
kerja di lokasi warung (kunjungan, titip stok, cetak struk). Backoffice adalah sumber
kebenaran untuk konfigurasi (status, kategori, tipe keuangan) dan operasi administratif.

## Hosting & biaya

Seluruh stack memakai **tier gratis**:

- **Backoffice:** di-deploy ke **Vercel** (free / hobby), domain gratis.
- **Database & Auth:** **Firebase Spark plan** (gratis, tanpa Cloud Functions). Strategi
  query hemat dirancang khusus untuk batas Spark — lihat
  `venovian-toys-mobile/docs/mobile-firebase-free-plan.md`.
- **Mobile:** dibangun via **EAS Build** (Expo), distribusi internal (APK).

Implikasi desain dari Spark plan: **tidak boleh ada ketergantungan Cloud Functions**;
penomoran invoice, counter, dan agregasi dilakukan di sisi klien/server action dengan
pola counter document. Baca data dirancang hemat (hindari full-collection scan di mobile).

## Bahasa & konvensi UI

- Semua teks user-facing wajib **Bahasa Indonesia**; istilah bisnis **"titipan"**.
- Tanggal user-facing format pendek Indonesia: `2 Feb 2026` (bukan ISO).
- Identifier kode tetap Inggris.

Lihat [02 — Arsitektur](02-arsitektur.md) untuk detail teknis tiap app.

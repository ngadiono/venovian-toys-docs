# 03 — Backoffice (Web)

Repo: `venovian-toys` · Next.js 16 App Router · di-deploy ke Vercel.

## Peta route

Route memakai Route Group `(auth)` dan `(dashboard)`. URL **flat & feature-based**
(`/customers`, bukan `/dashboard/customers`).

```
(auth)
└── /login

(dashboard)
├── /dashboard                  ← ringkasan operasional (metrik, chart, antrian invoice)
├── /customers                  ← pelanggan/mitra warung (CRUD, stok titipan)
│   └── /customers/status       ← konfigurasi status pelanggan
├── /inventory                  ← produk
│   ├── /inventory/suppliers     ← pemasok
│   ├── /inventory/categories    ← kategori
│   └── /inventory/status        ← status produk
├── /scan-area                  ← peta scan prospek
│   └── /scan-area/saved         ← prospek tersimpan
├── /distribusi
│   └── /distribusi/surat        ← surat titipan
├── /invoices                   ← invoice & pembayaran
├── /finance                    ← transaksi keuangan
│   ├── /finance/categories      ← kategori keuangan
│   ├── /finance/types           ← tipe transaksi
│   ├── /finance/statuses        ← status transaksi
│   └── /finance/alokasi         ← alokasi
├── /archive                    ← arsip (soft-deleted)
│   ├── /archive/customers
│   ├── /archive/suppliers
│   └── /archive/inventory
├── /settings
└── /profile

API routes
├── /api/auth/session  (login → set session cookie)
├── /api/auth/logout
├── /api/scan-prospects               ← scan prospek (Overpass/Geoapify)
└── /api/geocode/subdistrict          ← reverse geocode kelurahan
```

## Domain fitur

### Pelanggan (Customers)
CRUD penuh: tambah, edit, detail, **soft delete** → arsip → pulihkan / hapus permanen.
Stok titipan per pelanggan: tambah titipan, catat terjual, tarik sisa, riwayat (`stockMovements`).
Status pelanggan dikonfigurasi di `/customers/status`.

### Inventori
Produk, kategori, pemasok, status produk; penyesuaian stok; arsip produk & pemasok.
Search server-side dengan field bantu keyword (`buildProductSearchKeywords`).

### Invoice
Buat invoice dari stok pelanggan, pratinjau, dokumen cetak, catat pembayaran (penuh/sebagian).
Penomoran via `invoiceCounters/{yyyyMMdd}`. Status turunan dihitung (`getDisplayStatus`).
Strategi dokumen: `venovian-toys/docs/invoice-strategy.md`.

### Keuangan
Transaksi, kategori, tipe, status; ringkasan & grafik tren bulanan. Sumber transaksi bisa
manual atau turunan dari pergerakan stok (`warehouse_in`) — perhatikan dedup agar tidak
dobel hitung (lihat `dashboard.ts`/finance service).

### Scan Area
Peta OSM/Leaflet; scan radius untuk cari warung prospek via Overpass/Geoapify; klasifikasi
skor potensi; simpan prospek; konversi prospek → pelanggan. Strategi:
`venovian-toys/docs/scan-area-strategy.md`.

## Server Actions — gerbang mutasi

Semua mutasi penting lewat `src/actions/*.ts` dengan pola konsisten:

```ts
export async function createXAction(input) {
  const session = await requireSession();   // wajib login
  assertCanWrite(session);                  // wajib role yang boleh tulis
  const data = schema.parse(input);         // validasi Zod
  return createX(data);                     // service Firestore (Admin SDK)
}
```

`requireSession()` adalah god node (70 edge) — hampir semua action lewat sini. Saat
menambah action baru, ikuti pola ini; jangan akses Firestore langsung dari komponen client.

## Konvensi UI yang wajib (dari AGENTS.md)

- Tabel: TanStack v8 + shadcn. Paginasi `[First] [1][2][3] [Last]` + "Showing X of Y".
  Baris aksi = dropdown icon "more" di kolom terakhir, header `Aksi` rata kanan terlihat.
- Sel teks panjang: truncate 1 baris + tooltip.
- Dialog operasional: **tidak** menutup dari klik backdrop; hanya via tombol/ikon.
- Dialog bertumpuk: latar di-blur & non-interaktif.
- Header sticky: `sticky top-0 z-50 backdrop-blur-md`.
- Detail dialog: label uppercase muted, value plain di bawah, grid 2 kolom.

## Skrip data

- `npm run seed` → `scripts/seed-data.js` (satu file kanonik, idempoten).
- `npm run clear:db` → `scripts/clear-data.js`.

Jangan tambah skrip seed per-fitur. Jangan jalankan seed otomatis saat startup.

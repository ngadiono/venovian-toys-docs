# 12 — Peta Sinkronisasi (Kontrak Field Denormalized)

> **Tujuan:** dokumen ini adalah **checklist rujukan** saat mereview/mengubah kode apa pun
> yang menyentuh stok, titipan, penjualan, invoice, atau keuangan. Aplikasi ini menyimpan
> banyak **angka ringkasan (denormalized)** di dokumen `customers` dan `products`, yang
> dijaga oleh transaksi tersebar di banyak service dan **dua app**. Bug paling mahal ada di
> **sambungan (seam)** ini, bukan di dalam satu menu.
>
> **Aturan emas:** setiap kali sebuah aksi mengubah kuantitas/nilai, ia **wajib** memperbarui
> **semua** field hub yang relevan (lihat tabel), dan implementasi backoffice ↔ mobile harus
> menghasilkan bentuk dokumen yang **identik**.
>
> Status: disusun 2026-07-02 dari pembacaan kode. Jalur rujukan paling lengkap =
> **`invoices.ts` `createInvoice`** (satu-satunya yang menulis bucket customer **dan** status
> produk). Bandingkan jalur lain terhadapnya.

---

## 1. Dokumen hub & field turunannya

### `customers/{id}` — hub agregat pelanggan
| Field | Arti | Diturunkan dari |
|---|---|---|
| `consignedStock` | total pcs titipan sedang di warung | Σ `customerStocks.quantityRemaining` |
| `consignedValue` | nilai sisa titipan (Rp) | Σ `quantityRemaining × unitPrice` |
| `totalSales` | akumulasi penjualan (Rp) | Σ transaksi jual |
| `consignedStockBucket` | `rendah/sedang/tinggi` | `getStockBucket(consignedStock)` |
| `totalSalesBucket` | `rendah/sedang/tinggi` | `getSalesBucket(totalSales)` |
| `lastVisitAt` / `nextVisitAt` | jadwal kunjungan | aksi invoice / titipan |
| `searchKeywords` | array prefix untuk search | `buildSearchKeywords` (saat create/update) |

### `products/{id}` — hub agregat produk
| Field | Arti | Diturunkan dari |
|---|---|---|
| `warehouseStock` | stok gudang | titipan − / retur + |
| `consignedStock` | stok tersebar di warung | titipan + / jual − / tarik − |
| `soldStock` | akumulasi terjual | jual + |
| `status` | `critical/low/healthy/overstock` | `getProductInventoryStatus(warehouseStock)` |

### `stockMovements/{id}` — ledger (append-only)
Dibaca oleh **Keuangan** dan daftar riwayat stok. Tipe: `consign_out`, `sold`,
`return_in`, `consign_void`, `warehouse_in`. Bentuk dokumen harus sama antar penulis.

---

## 2. Matriks: jalur tulis × field yang dipelihara

Legenda: ✅ ditulis · ➖ tidak relevan · ❌ **seharusnya ditulis tapi TIDAK** (drift).

### Customer hub
| Jalur tulis | App | consignedStock | consignedValue | totalSales | stockBucket | salesBucket |
|---|---|:--:|:--:|:--:|:--:|:--:|
| `createInvoice` (rujukan) | BO | ✅ | ✅ | ✅ | ✅ | ✅ |
| `voidInvoice` (batal) | BO | ✅ | ✅ | ✅ | ✅ | ✅ |
| `recordCustomerStockSale` (jual) | BO | ✅ | ✅ | ✅ | ✅ | ✅ |
| `deleteCustomerStock` (tarik) | BO | ✅ | ✅ | ➖ | ✅ | ➖ |
| `addCustomerConsignment` (quick) | BO | ✅ | ✅ | ➖ | ✅ | ➖ |
| `createConsignmentNote` (surat) | BO | ✅ | ✅ | ➖ | ❌ | ➖ |
| `voidConsignmentNote` | BO | ✅ | ✅ | ➖ | ❌ | ➖ |
| `createInvoiceSettlement` (setelmen) | **MOB** | ✅ | ✅ | ✅ | ✅ | ✅ |
| `createConsignmentNote` (restok) | **MOB** | ✅ | ✅ | ➖ | ❌ | ➖ |
| `recordCustomerSale` (jual) | **MOB** ⚠️ mau dipensiun | ✅ | ✅ | ✅ | ❌ | ❌ |
| `withdrawCustomerStock` (tarik) | **MOB** | ✅ | ✅ | ➖ | ❌ | ➖ |

### Product hub
| Jalur tulis | App | warehouseStock | consignedStock | soldStock | status |
|---|---|:--:|:--:|:--:|:--:|
| `createInvoice` (rujukan) | BO | ✅ | ✅ | ✅ | ✅ |
| `voidInvoice` (batal) | BO | ✅ | ✅ | ✅ | ✅ |
| `addCustomerConsignment` | BO | ✅ | ✅ | ➖ | ✅ |
| `deleteCustomerStock` (tarik) | BO | ✅ | ✅ | ➖ | ✅ |
| `recordCustomerStockSale` (jual) | BO | ➖ | ✅ | ✅ | ❌ |
| `createConsignmentNote` (surat) | BO | ✅ | ✅ | ➖ | ❌ |
| `createInvoiceSettlement` (setelmen) | **MOB** | ✅ | ✅ | ✅ | ✅ |
| `createConsignmentNote` (restok) | **MOB** | ✅ | ✅ | ➖ | ❌ |
| `recordCustomerSale` (jual) | **MOB** ⚠️ mau dipensiun | ➖ | ✅ | ✅ | ❌ |
| `withdrawCustomerStock` (tarik) | **MOB** | ✅ | ✅ | ➖ | ❌ |

> **Baca peta di atas begini:** kolom `stockBucket`/`salesBucket` dan `status` penuh dengan ❌.
> Artinya field-field ini **rapuh** — banyak jalur (di **kedua** app) lupa memperbaruinya.
> Karena itu rekomendasi arsitektur: **hitung `bucket` & `status` saat baca (in-memory)**,
> jangan andalkan field tersimpan untuk filter. Fix di sisi baca menutup semua ❌ sekaligus.

---

## 3. Siapa yang MEMBACA field turunan

| Field | Pembaca | Catatan |
|---|---|---|
| `consignedStockBucket` / `totalSalesBucket` | `customers.ts` `buildCustomersQuery` (filter daftar) | Satu-satunya konsumen → aman dipindah ke in-memory |
| `consignedValue` | daftar pelanggan **meng-hydrate ulang** dari `customerStocks`; kartu statistik & mobile pakai nilai tersimpan | 3 jalur → bisa beda angka |
| `products.status` | filter/badge di Inventory | |
| `stockMovements` | **Keuangan** (turunkan biaya/`warehouse_in`) + riwayat stok | seam invoice↔finance **verified bersih**: movement `sold` ber-`invoiceId` diabaikan finance, revenue lewat txn `collection` |

> **Keputusan akuntansi (2026-07-02): CASH-BASIS.** Pendapatan diakui saat uang diterima
> (invoice/setelmen), bukan saat barang terjual. Ringkasan laba backoffice (`getProfitSummary`,
> `getMonthlyFinance`) kini **hanya menghitung transaksi status `paid`** — selaras dengan mobile
> `dashboard.ts`. Konsekuensi: penjualan "belum ditagih" tetap tampil sebagai info, tidak masuk laba.
>
> **Setelmen kini bisa di mobile.** `createInvoiceSettlement` (MOB) = port setia `createInvoice`
> (BO): catat terjual/retur + terima uang + tulis `invoices`/`items`/`stockMovements`/
> `financeTransactions`/`invoicePayments` dengan bentuk **identik**. `recordCustomerSale` (catat-jual
> mobile) akan **dipensiun** karena hanya menandai terjual tanpa menangkap uang (bikin revenue pending yatim).

---

## 4. Guard yang harus konsisten backoffice ↔ mobile

Backoffice menolak; pastikan mobile menolak dengan cara yang sama:

| Guard | Backoffice | Mobile | Status |
|---|---|---|---|
| Pelanggan diarsip (`deletedAt`) tidak boleh ditransaksikan | ✅ | ❌ (semua transaksi) | **drift** |
| Produk diarsip (`deletedAt`) tidak boleh dititipkan | ✅ | ❌ (`createConsignmentNote`) | **drift** |
| Stok gudang cukup sebelum titip (tak boleh minus) | ⚠️ hanya `addCustomerConsignment`; `createConsignmentNote` **tidak** | ❌ | **laten (kedua app)** |
| Otorisasi role (owner/admin) | ✅ server action | ❌ hanya Security Rules | lihat [07 SEC-1](07-bug-register.md#keamanan) |

---

## 5. Catatan bentuk dokumen `stockMovements`

- Referensi dokumen sumber tidak seragam: sebagian movement pakai `invoiceId`, sebagian
  `noteId`, sebagian `pickupDueAt`. Saat menambah tipe/penulis baru, samakan set field.
- `sellingUnitQuantity` kadang ada kadang tidak. Finance/riwayat harus tahan terhadap absennya.

---

## 6. Cara pakai dokumen ini saat review

Untuk **setiap** area yang kita review, jawab 3 pertanyaan:

1. **Benar internal?** — math & validasi di dalam aksi.
2. **Benar update hub?** — cek matriks §2: apakah aksi ini menulis **semua** field ✅ yang seharusnya?
3. **Backoffice ↔ mobile identik?** — bentuk dokumen & guard (§4) sama?

Setiap ❌ / drift yang dikonfirmasi → catat di [07 — Bug Register](07-bug-register.md) dengan ID,
dan tautkan balik ke baris matriks di sini.

# 05 — Model Data (Firestore)

> **Sumber kebenaran lengkap:** `venovian-toys/docs/database-design.md` (930 baris — ERD,
> field types, nullable, enum, index, security model, acceptance criteria). Dokumen ini
> hanya **peta navigasi** koleksi agar tidak ada dua sumber yang menyimpang.

## Daftar koleksi

| Koleksi | Isi | Ditulis oleh |
|---------|-----|--------------|
| `userProfiles/{uid}` | Profil & role user app (bukan password/token) | Backoffice |
| `customers/{customerId}` | Mitra warung/toko | Backoffice (+ mobile baca) |
| `customerStatuses/{statusId}` | Konfigurasi status pelanggan | Backoffice |
| `customerStocks/{stockId}` | Stok titipan per pelanggan | Backoffice + **mobile** |
| `stockMovements/{movementId}` | Ledger pergerakan stok (in/out/sold/return) | Backoffice + **mobile** |
| `products/{productId}` | Produk inventori | Backoffice (mobile baca katalog) |
| `productSuppliers/{supplierId}` | Pemasok | Backoffice |
| `productCategories/{categoryId}` | Kategori produk | Backoffice |
| `productStatuses/{statusId}` | Status produk | Backoffice |
| `invoices/{invoiceId}` | Invoice tagihan | Backoffice (mobile baca) |
| `invoices/{invoiceId}/items/{itemId}` | Baris item invoice | Backoffice |
| `invoiceCounters/{yyyyMMdd}` | Counter penomoran invoice | Backoffice |
| `invoicePayments/{paymentId}` | Pembayaran invoice | Backoffice |
| `financeTransactions/{transactionId}` | Transaksi keuangan | Backoffice |
| `financeCategories/{categoryId}` | Kategori keuangan | Backoffice |
| `scanProspects/{prospectId}` | Prospek hasil scan area | Backoffice + **mobile** (visit) |
| `auditLogs/{logId}` | Jejak audit | Backoffice |

> Catatan: backoffice memetakan surat titipan ke koleksi `consignmentNotes` /
> `consignmentNoteCounters` (lihat seed & graphify community "Consignment Note Actions").
> Mobile `consignment-notes.ts` menulis ke koleksi yang sama lewat `runTransaction`.
> **[VERIFIKASI]** pastikan nama koleksi & bentuk dokumen surat titipan identik antara
> seed backoffice dan service mobile — ini titik rawan drift.

## Pola desain yang dipakai

- **Soft delete**: dokumen tidak dihapus, ditandai field arsip (mis. `deletedAt`/status)
  lalu muncul di menu Arsip. Hapus permanen tersedia terpisah.
- **Counter document**: penomoran (invoice, surat titipan) pakai dokumen counter harian/
  periodik, bukan auto-increment server — karena Spark tanpa Cloud Functions.
- **Search keywords**: field bantu array keyword (mis. `buildProductSearchKeywords`,
  `buildCustomerSearchKeywords`) untuk query prefix di Firestore.
- **Denormalisasi snapshot**: invoice menyimpan snapshot pelanggan (`buildCustomerSnapshot`)
  agar tahan terhadap perubahan data pelanggan kemudian.
- **Kuantitas dalam pcs**: stok/titipan disimpan dalam pcs; `pricingUnit` (`pcs`/`pack`) +
  `packSize` + `sellingUnitPrice` dipakai untuk tampilan & harga per unit jual.

## Index & Security

Index komposit yang dibutuhkan dan model Security Rules ada di
`venovian-toys/docs/database-design.md` (bagian "Index Yang Mungkin Dibutuhkan" dan
"Security Model"). **Wajib dibaca sebelum menambah query list baru** — Firestore akan
menolak query tanpa index komposit yang sesuai.

> ⚠️ Karena mobile mengakses Firestore langsung, **Security Rules harus benar-benar
> mengunci** apa yang boleh dibaca/ditulis tiap user. Lihat
> [06 — Kontrak Integrasi](06-kontrak-integrasi.md) dan [07 — Bug Register](07-bug-register.md#keamanan).

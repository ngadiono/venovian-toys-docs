# 07 — Bug & Issue Register

> **Status kejujuran:** daftar ini adalah hasil **pembacaan sepintas** saat menyusun
> dokumentasi (2026-06-30), **bukan** audit kode/keamanan menyeluruh. Item bertanda
> `[VERIFIKASI]` adalah dugaan yang perlu dikonfirmasi terhadap kode sebelum diperbaiki.
> Audit mendalam dijadwalkan — lihat [Rencana audit](#rencana-audit).

## Cara pakai

- Tiap item punya ID stabil (`DOC-n`, `INT-n`, `SEC-n`, `BUG-n`) supaya bisa dirujuk dari PR.
- Status: 🔴 Terbuka · 🟡 Perlu verifikasi · 🟢 Selesai.
- Jangan hapus item selesai — tandai 🟢 + tanggal + tautan PR.

---

## Dokumentasi

### DOC-1 🔴 README mobile basi (menyebut "web/Next.js")
`venovian-toys-mobile/README.md` masih menyalin teks backoffice: "web application",
"Next.js (App Router)", "TanStack Table", "Leaflet". Faktanya mobile = Expo/React Native.
**Dampak:** menyesatkan kontributor & agent AI. **Perbaikan:** tulis ulang README mobile
sesuai stack Expo sebenarnya.

### DOC-2 🔴 AGENTS.md mobile menyatakan fase "UI-first / mock only"
`venovian-toys-mobile/AGENTS.md` (dan backoffice) menyatakan: "strictly UI-first phase",
"Do NOT implement actual Firebase integration until explicitly requested". Faktanya service
Firestore nyata sudah ada & dipakai (`src/services/firebase/*`), termasuk `runTransaction`.
**Dampak:** agent/AI berikutnya bisa menolak menyentuh integrasi yang sudah live, atau
membangun ulang dengan mock. **Perbaikan:** update AGENTS.md agar mencerminkan fase
"sudah terintegrasi Firebase".

---

## Integrasi

### INT-1 🟢 Dua jalur update kunjungan prospek — ditutup (bukan bug) {#int-1}
**Audit 2026-07-02:** kedua jalur menulis field **identik** (`visitStatus`, `visitNotes`,
`visitedAt`, `visitUpdatedAt`, `updatedAt`, `updatedBy`), dan mobile **hanya** memakai jalur
client SDK langsung (`updateProspectVisit`) — 0 referensi ke `EXPO_PUBLIC_API_URL`. Jadi tak ada
inkonsistensi data. Endpoint `PATCH /api/mobile/prospects/[id]/visit` yang jadi **dead code sudah
dihapus** (SA-1). Keamanan `scanProspects` kini murni via Security Rules — lihat [SEC-1](#keamanan).

### INT-2 🟡 Logika transaksi stok titipan terduplikasi {#int-2}
`runTransaction` untuk titipan ada di backoffice **dan** di mobile
(`consignment-notes.ts`, `customer-stocks.ts`). **Risiko:** aturan hitung
`quantityRemaining`, penulisan `stockMovements`, dan nama/bentuk koleksi surat titipan bisa
berbeda. **[VERIFIKASI]** bandingkan field demi field kedua implementasi. Satukan logika
atau buat checklist sinkronisasi.

**Audit 2026-07-02 (field-demi-field selesai):** matriks lengkap ada di
[12 — Peta Sinkronisasi](12-peta-sinkronisasi.md#2-matriks-jalur-tulis--field-yang-dipelihara).
Hasil: **math inti titipan/jual/tarik SUDAH identik** backoffice ↔ mobile (kabar baik). Drift
nyata yang ditemukan & statusnya:
- 🟢 **Guard `deletedAt`** (mobile bisa transaksi ke pelanggan/produk terarsip) — **diperbaiki**
  di mobile `consignment-notes.ts` + `customer-stocks.ts` (4 titik guard).
- 🟢 **Bucket customer basi** (`consignedStockBucket`/`totalSalesBucket` dilupakan banyak jalur
  di kedua app) — **diperbaiki di sisi baca**: backoffice `customers.ts` kini menghitung bucket
  in-memory (`filterDocsByBucket`), jadi jalur tulis tak wajib lagi memeliharanya.
- 🔴 **`products.status` basi** — jalur jual (BO+MOB), surat titipan (BO+MOB) tidak menghitung
  ulang `getProductInventoryStatus`. Belum diperbaiki; ditangani di review area Inventory.
- 🔴 **Tak ada guard stok gudang minus** saat `createConsignmentNote` (kedua app) — bisa
  membuat `warehouseStock` negatif. Belum diperbaiki.

### INT-3 🟡 Tidak ada tipe bersama (risiko drift skema)
Tipe `Customer`, `Product`, dll. didefinisikan ulang terpisah di tiap app. **Risiko:**
perubahan skema di satu app tidak terdeteksi di app lain sampai runtime. **Perbaikan
(roadmap):** paket tipe bersama atau file `shared-schema` + checklist.

---

## Keamanan {#keamanan}

### SEC-1 🟢 Security Rules adalah satu-satunya penjaga akses mobile
Mobile mengakses Firestore langsung dari device (tak tepercaya). **Rules aktual didapat dari
pemilik (2026-07-02)** — ternyata **sudah bagus**: per-koleksi, gate `isActiveUser()` (auth +
`userProfiles.status=='active'`), default-deny `if false`. **Bukan** test-mode kebuka.
Disimpan di repo sebagai source of truth: [`venovian-toys/firestore.rules`](../venovian-toys/firestore.rules).
Tidak butuh penulisan ulang per-role — altitude yang ada sudah pas untuk 2 user.

### SEC-3 🔴 Rules read-only memblokir fitur setelmen mobil {#sec-3}
Rules berjalan dibuat **sebelum** `createInvoiceSettlement` (KEU-2). `invoices`,
`invoicePayments`, `financeTransactions` **read-only** dari client, tapi setelmen mobil
menulisnya → transaksi Firestore all-or-nothing → **setelmen selalu gagal `permission-denied`**.
Kemungkinan sebab tes setelmen di HP belum tembus. **Perbaikan** (di `firestore.rules`, tandai
`DELTA`): izin **create** (bukan update/delete) untuk 3 koleksi itu + `invoiceCounters` +
subcollection `invoices/{id}/items`. Terap manual → [13 — FB-1](13-firebase-perubahan-manual.md).

### SEC-2 🟢 Secret di environment
Backoffice `.env.local` memuat `FIREBASE_PRIVATE_KEY` (Admin SDK). **Terverifikasi bersih
(2026-07-02):** `.gitignore` mengabaikan `.env*` (kecuali `.env.example`); `git log` atas
`.env.local` **kosong** (tak pernah ter-commit); pencarian history atas `BEGIN PRIVATE KEY`
**nol hit** (nilai kunci asli tak pernah masuk git). Nama variabel `FIREBASE_PRIVATE_KEY`
hanya muncul sebagai referensi di `.env.example`/docs/scripts/`admin.ts`, bukan nilainya.
Sisa (cek pemilik, tak bisa dari repo): pastikan di **Vercel** tersimpan sebagai Encrypted
Environment Variable, bukan hardcode.

---

## Kode / Logika (perlu audit)

### BUG-1 🟡 Potensi dobel-hitung keuangan dari pergerakan stok
`dashboard.ts` (mobile) dan finance service backoffice menggabungkan `financeTransactions`
manual dengan turunan `stockMovements` (`warehouse_in`) dan punya logika dedup
(`postedMovementIds`, `postedProductIds`). **[VERIFIKASI]** uji kasus di mana satu
pergerakan stok dicatat manual **dan** otomatis — pastikan tidak terhitung dua kali, dan
kedua app menghitung dengan cara sama.

### BUG-2 🟡 Konsistensi format & timezone tanggal
Aturan UI: tanggal user-facing format pendek Indonesia (`2 Feb 2026`), ISO hanya internal.
**[VERIFIKASI]** sapu UI kedua app untuk ISO yang bocor ke layar; cek perhitungan
`nextVisitAtMs`/`dueDate` terhadap timezone (WIB vs UTC) agar "jatuh tempo hari ini" tidak
meleset sehari.

---

## Invoice (audit 2026-07-02)

### INV-1 🟢 Invoice tidak bisa dibatalkan → ditambah fitur void
Sebelumnya `createInvoice` memotong stok titipan, agregat pelanggan/produk, ledger, dan
finance tanpa cara membalik di app. **Perbaikan:** `voidInvoice(invoiceId, actorUid, reason)`
di backoffice `invoices.ts` membalik semua efek (customerStocks, products+status, customer+bucket),
menulis movement `invoice_void`, soft-delete `financeTransactions`/`invoicePayments` terkait,
set status `void`. Ada `voidInvoiceAction` + tombol "Batalkan Invoice" di `InvoiceTable`.
Diverifikasi field-demi-field = kebalikan persis `createInvoice`. Read-before-write aman.

### INV-2 🟢 Txn finance asal invoice bisa diutak-atik manual → dikunci
`updateFinanceTransaction`/`deleteFinanceTransaction` kini menolak `source === "invoice"`
(sebelumnya hanya `stock_movement`) dan `mapFinanceTransactionDocument` menandainya `readOnly`.
Satu-satunya jalur sah membalik efek finansial invoice = tombol Batalkan Invoice.

### INV-3 🟡 Seam invoice↔finance: verified BERSIH (bukan dobel-hitung)
Terkait [BUG-1](#bug-1-🟡-potensi-dobel-hitung-keuangan-dari-pergerakan-stok): movement `sold`
ber-`invoiceId` sengaja diabaikan finance (`mapSoldStockMovement` → `if (data.invoiceId) return null`);
revenue masuk lewat txn `collection`. **Tidak ada dobel-hitung di jalur invoice.** Sisa perhatian:
penjualan via "catat jual" (non-invoice) jadi revenue "pending" yang tak pernah direkonsiliasi —
ditangani di review area **Keuangan**.

### INV-4 🟢 Counter invoice aman dari race
`invoiceCounters` di-increment **di dalam** `runTransaction` → Firestore retry saat contention.
Kekhawatiran race pada penomoran invoice tidak berlaku.

### INV-5 🟢 Label movement `invoice_void`/`consign_void` di riwayat (ternyata mislabel, bukan kosmetik)
Kedua tipe pembatalan bawa `customerId`+`productId` → muncul di riwayat stok pelanggan. **Bug:**
mobile `getCustomerMovements` (customer-stocks.ts) meng-*coerce* tipe tak dikenal → `'sold'`,
jadi pembatalan **tampil "Terjual" (hijau)** — salah arti. Backoffice `CustomerTable` menampilkan
string mentah `invoice_void`. **Perbaikan (4 file):** perluas union `StockMovement.type`
(mobile customer-stocks.ts + backoffice customer-types.ts), hentikan coercion agar 2 tipe void
dipertahankan, tambah label: **"Invoice dibatalkan"** & **"Titipan dibatalkan"** (amber) di
`CustomerStockDetailModal` (mobile) & `CustomerTable` (backoffice). `tsc --noEmit` kedua repo exit 0.

## Keuangan (audit 2026-07-02)

### KEU-1 🟢 Web & mobile beda cara hitung laba → disamakan cash-basis
Backoffice memasukkan penjualan belum ditagih (pending) sebagai laba; mobile hanya `paid`.
**Keputusan: cash-basis** (akui saat uang diterima). **Perbaikan:** `getProfitSummary` &
`getMonthlyFinance` (backoffice `finance.ts`) kini melewati transaksi status !== "paid".
Daftar transaksi tak diubah — baris pending tetap tampil, hanya tidak masuk laba. Selaras
dengan mobile `dashboard.ts`.

### KEU-2 🟢 Mobile kini bisa setelmen invoice saat visit (uang tertangkap)
Sebelumnya mobile hanya bisa "catat jual" (`recordCustomerSale`) yang menandai terjual **tanpa
uang** → melahirkan revenue pending yatim. **Perbaikan:** `createInvoiceSettlement` di mobile
`invoices.ts` — port setia `createInvoice` backoffice (bentuk dokumen identik, `searchKeywords`
array agar ketemu di search web, guard minimal 1 terjual). Diverifikasi field-demi-field.

### KEU-3 🟢 "catat jual" mobile (`recordCustomerSale`) dipensiun
Sudah dihapus dari `CustomerStockDetailModal` (tombol/ view 'sale') dan fungsi service
`recordCustomerSale` dihapus. `ModalView` jadi `'list' | 'withdraw'`; alur "Tarik Sisa"
(retur murni ke gudang) tetap. Verified nol referensi tersisa. Kini penjualan **hanya** lewat
setelmen invoice → tidak ada lagi revenue pending yatim.

### KEU-4 🟢 (kode) / 🔵 (tes device) B2-UI layar setelmen mobile
`InvoiceSettlementModal.tsx` selesai + terwiring di `customers.tsx` (aksi "Buat Invoice /
Setelmen"): input sisa fisik → hitung terjual otomatis → adminFee/diskon → uang + metode →
`createInvoiceSettlement` → layar sukses + cetak struk PDF (expo-print/expo-sharing). Perhitungan
terjual, `canSubmit`, mapping item, status bayar — verified benar. **Sisa: tes di device**
(alur + sinkron ke web + cetak). Printer termal Bluetooth = iterasi lain (baru ada expo-print PDF).

### KEU-5 🟢 Timezone bucket bulan (BUG-2) & cap `.limit(200)`
Dulu: backoffice bucket per tanggal **UTC**, mobile per bulan **jam lokal device** — setelmen
00:00–07:00 WIB bisa masuk bulan berbeda antar app; dan `getFinanceTransactions` dibatasi 200 terbaru
(mobile baca semua). **Perbaikan (hardcode WIB +7, tanpa library):**
- Backoffice `finance.ts`: `toDateInput` kini turunkan tanggal **WIB** (helper `toWibDateParts`,
  `WIB_OFFSET_MS`), jadi `.date` + `getMonthlyFinance` mengelompokkan per bulan WIB. `.limit(200)`
  dihapus dari query `financeTransactions` (orderBy dipertahankan) → bulan lama tak terpotong.
- Mobile `dashboard.ts` `getMonthlyFinanceTrend`: rentang tahun WIB-eksplisit
  (`Date.UTC(...) - WIB_OFFSET_MS`) + ketiga `monthIndex` (transactionAt/movementAt/createdAt) pakai
  `getWibMonth` (bukan `getMonth()` lokal).
Hasil: kedua app menurunkan bulan WIB dari `transactionAt` yang sama → **konsisten**. Diverifikasi
diff-demi-diff; `tsc --noEmit` exit 0 kedua repo. Nilai (revenue/expense/sales/modal), cash-basis,
dedup tak diubah. Nol perubahan Firebase.

## Lifecycle Pelanggan (audit 2026-07-02)

Lifecycle = Aktif → Arsip (soft delete, retensi 30 hari) → Pulihkan / Hapus Permanen. Murni
backoffice (mobile hanya baca daftar pelanggan). Aturan inti yang ditegakkan: **pelanggan tidak
boleh dihilangkan selama masih memegang titipan** (sumber kebenaran = `customerStocks`, bukan
agregat `consignedStock`).

### LC-1 🟢 Arsip pelanggan bertitipan → deadlock (diperbaiki)
`deleteCustomer` dulu tak cek sisa titipan; padahal `deleteCustomerStock` menolak pelanggan
terarsip → barang nyangkut tak bisa ditarik. **Perbaikan:** `deleteCustomer` menjumlahkan
`quantityRemaining` dari `customerStocks`; jika > 0 tolak arsip dengan pesan minta tarik/setel dulu.

### LC-2 🟢 Hapus permanen tinggalkan orphan (diperbaiki)
`permanentlyDeleteCustomer` dulu hanya hapus dokumen pelanggan → `customerStocks` yatim &
`products.consignedStock` meleset. **Perbaikan:** guard tolak jika masih ada sisa > 0, lalu
`batch()` hapus semua `customerStocks` pelanggan + dokumen pelanggan sekaligus. `stockMovements`
sengaja dipertahankan (buku besar/audit + dibaca Keuangan). Karena LC-1, pelanggan yang bisa
diarsip pasti sisa 0 → stok produk sudah benar, tak perlu rekonsiliasi.

### LC-3 🟢 `restoreCustomer` selalu balik ke "Aktif" → diperbaiki
Status asli (mis. `follow_up`) sebelumnya hilang: `deleteCustomer` menimpa `status:"inactive"`
lalu `restoreCustomer` menimpa `status:"active"`. **Perbaikan** (backoffice `customers.ts`, mobile
read-only jadi tak tersentuh): field baru opsional `statusBeforeArchive` di `CustomerDocument`;
`deleteCustomer` menyimpan `currentData.status` ke situ saat mengarsip (tetap set `"inactive"` untuk
tampilan arsip); `restoreCustomer` mengembalikan `statusBeforeArchive ?? "active"` lalu menghapus
field temp (`FieldValue.delete()`). Fallback `"active"` menjaga kompat pelanggan lama tanpa field.
`tsc --noEmit` exit 0. Nol perubahan Firebase (tak butuh index/rule; Admin SDK).

## Inventory (audit 2026-07-02)

### INV-STK-1 🟢 `products.status` basi setelah surat titipan / tarik (diperbaiki)
Label stok gudang (critical/low/healthy/overstock) tidak dihitung ulang oleh 4 jalur yang
mengubah `warehouseStock`: `createConsignmentNote` (BO+MOB), `voidConsignmentNote` (BO),
`withdrawCustomerStock` (MOB). **Perbaikan:** semua kini set `status`, dengan menjaga `"inactive"`.
`status` produk overloaded (level stok **atau** "inactive" tanpa field terpisah) → tidak dipakai
hitung-saat-baca; ditambal per-jalur. Lihat [12 §2 Product hub](12-peta-sinkronisasi.md#product-hub).

### INV-STK-2 🟢 Threshold status produk tersalin di 4 tempat → disatukan
Dulu logika `warehouseStock <=20 critical / <=50 low / >=400 overstock / else healthy` diketik ulang
di 4 tempat backoffice (`getProductInventoryStatus` di customers.ts & invoices.ts, `getWarehouseStatus`
di consignment-notes.ts, `getInitialStatus` di inventory.ts). **Perbaikan:** sumber tunggal
`src/lib/inventory-status.ts` (fungsi `getProductInventoryStatus` + konstanta `WAREHOUSE_CRITICAL_MAX`
/`WAREHOUSE_LOW_MAX`/`WAREHOUSE_OVERSTOCK_MIN`); 4 definisi lokal dihapus, diganti impor (beralias agar
call site tak berubah). Diverifikasi grep: hanya 1 definisi tersisa, nol literal ambang nyasar; perilaku
byte-identik; `tsc` exit 0. Mobile `derived-fields.ts` tetap punya salinan identik (repo terpisah, sudah
sumber tunggal di repo-nya sendiri — berbagi lintas-repo butuh package bersama = over-engineering di skala ini).
Nol perubahan Firebase.

### INV-STK-3 🟢 Error `tsc` lama di mobile `mock-data.ts` (diperbaiki)
`dashboardTrend` kehilangan properti `modal` yang diwajibkan tipe `DashboardTrend`. Ditambahkan
`modal` pada 6 entri mock. `npx tsc --noEmit` mobile kini bersih total (exit 0) — verifikasi ke
depan bebas noise.

## Scan-Area / Prospek (audit 2026-07-02)

### SA-1 🟢 Endpoint kunjungan prospek = dead code (dihapus)
`PATCH /api/mobile/prospects/[id]/visit` tak dipanggil siapa pun (mobile update langsung via
client SDK, field identik). Dihapus. Docs 01/03/06 disinkronkan. Detail: [INT-1](#int-1).

### SA-2 🟢 Backoffice `updateSavedProspectVisitInDb` — audit + guard auth
Update kunjungan dari web tak mencatat pelaku (`updatedBy`/`updatedAt`), mobile mencatatnya.
**Temuan tambahan saat perbaikan:** server action `updateSavedProspectVisitAction` **tak punya
guard sesi/role** (beda dari saudaranya `convertSavedProspectToCustomerAction` yang pakai
`requireSession()` + `assertCanWrite()`). **Perbaikan:** service terima `actorUid` → tulis
`updatedAt` + `updatedBy`; action tambah `requireSession()` + `assertCanWrite(user.role)` lalu
teruskan `user.uid`. Kini paritas dgn mobile `updateProspectVisit` (yang juga patuh Security
Rules `scanProspects` — hanya menulis field yang di-allow). `tsc --noEmit` exit 0.

### SA-3 🟢 Konversi prospek→pelanggan — verified sehat
`convertSavedProspectToCustomerInDb`: cek duplikat telepon, guard (sudah dikonversi / proposal
harus sukses), bentuk dokumen pelanggan konsisten dengan `createCustomer`. Catatan minor: cek
duplikat telepon di luar transaksi (race kecil, diabaikan di skala 2 user).

## Rencana audit

Belum dilakukan; urutan yang disarankan saat masuk fase "review bug":

1. **Keamanan (prioritas 1):** temukan & review `firestore.rules`. Tanpa ini, semua data
   live berisiko. (SEC-1, SEC-2)
2. **Integrasi:** selesaikan INT-1, INT-2, INT-3 — sumber bug paling halus & mahal.
3. **Korektif per-domain:** stok/titipan (konsistensi transaksi), invoice (penomoran
   counter race condition), keuangan (dobel-hitung).
4. **Lint/type sweep:** jalankan `npm run lint` + `tsc --noEmit` di kedua app, catat hasil.

Saat audit dijalankan, ubah item terkait dari 🟡 → 🔴 (terkonfirmasi bug) atau tutup
(🟢 / bukan bug) dengan bukti.

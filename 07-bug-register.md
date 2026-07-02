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

### INT-1 🟡 Dua jalur update kunjungan prospek {#int-1}
Update kunjungan prospek bisa lewat **dua** jalur:
1. `PATCH /api/mobile/prospects/[id]/visit` (backoffice, Admin SDK) — mengisi `updatedBy`,
   `visitedAt`, cek token revoked. Mobile punya `EXPO_PUBLIC_API_URL` → indikasi jalur ini dipakai.
2. `src/services/firebase/prospects.ts` `updateDoc(...)` langsung via client SDK.
**Risiko:** field seperti `updatedBy`/`visitedAt` hanya terisi di jalur (1); bila mobile
kadang pakai (2), data jadi tidak konsisten. **[VERIFIKASI]** lacak pemanggil
`updateDoc` di `prospects.tsx`/`prospects.ts` vs pemanggil `EXPO_PUBLIC_API_URL`. Pilih
satu jalur. Detail: [06 — Kontrak Integrasi](06-kontrak-integrasi.md#int-1).

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

### SEC-1 🟡 Security Rules adalah satu-satunya penjaga akses mobile
Mobile mengakses Firestore langsung dari device (tak tepercaya). Bila Security Rules
longgar, siapa pun dengan kredensial bisa membaca/menulis koleksi apa pun. **[VERIFIKASI]**
audit `firestore.rules` aktual: pastikan per-koleksi membatasi read/write sesuai role
(`userProfiles.role`) dan kepemilikan. **Catatan:** file rules belum ditemukan di repo
mana pun saat dokumentasi ini disusun — **konfirmasi rules disimpan di mana** (Firebase
Console saja, atau ada di repo). Rules yang hanya di Console = tidak ter-version-control =
risiko sendiri.

### SEC-2 🟡 Secret di environment
Backoffice `.env.local` (4.8 KB) memuat `FIREBASE_PRIVATE_KEY` (Admin SDK). **[VERIFIKASI]**
pastikan `.env.local` ada di `.gitignore` dan tidak pernah ter-commit; pastikan di Vercel
disimpan sebagai Encrypted Environment Variable, bukan di kode.

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

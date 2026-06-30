# 10 — Task Tracker (Backlog Pengembangan)

Sumber tunggal untuk melacak pekerjaan pengembangan Venovian Toys. File ini
**version-controlled** — update statusnya lewat commit/PR, bukan di kepala.

## Cara pakai

- Tiap task punya **ID stabil** (`T-01`, …). Rujuk di commit/PR: `T-03: lock firestore rules`.
- **Status:** `[ ]` Todo · `[~]` Sedang dikerjakan · `[x]` Selesai · `[!]` Terblokir.
- **Prioritas:** `P0` (kerjakan lebih dulu, risiko data/keamanan) · `P1` · `P2` · `P3`.
- Jangan hapus task selesai — tandai `[x]` + tanggal + tautan PR di kolom Catatan.
- Task yang berasal dari bug → tautkan ke [07 — Bug Register](07-bug-register.md).
  Task fitur → selaras dengan [08 — Roadmap](08-roadmap.md).

## Papan ringkas

| ID | Prioritas | Status | Area | Task | Dep |
|----|:--------:|:------:|------|------|-----|
| T-01 | P0 | [ ] | Keamanan | Temukan & version-control `firestore.rules` | — |
| T-02 | P0 | [ ] | Keamanan | Audit & perketat Security Rules per-koleksi | T-01 |
| T-03 | P0 | [ ] | Keamanan | Pastikan `.env.local` tak ter-commit; secret di Vercel terenkripsi | — |
| T-04 | P1 | [ ] | Docs | Tulis ulang README mobile (Expo, bukan Next.js) | — |
| T-05 | P1 | [ ] | Docs | Update AGENTS.md kedua app: fase "sudah terintegrasi Firebase" | — |
| T-06 | P1 | [ ] | Integrasi | Satukan jalur update kunjungan prospek (HTTP vs client SDK) | — |
| T-07 | P1 | [ ] | Integrasi | Bandingkan & satukan logika transaksi stok titipan 2 app | — |
| T-08 | P2 | [ ] | Integrasi | Tipe/skema bersama + checklist perubahan skema | — |
| T-09 | P1 | [ ] | Kualitas | Jalankan lint + `tsc --noEmit` kedua app, catat & bersihkan | — |
| T-10 | P2 | [ ] | Keuangan | Uji & perbaiki potensi dobel-hitung finance dari stockMovements | T-09 |
| T-11 | P2 | [ ] | Data | Audit format tanggal & timezone (WIB) di UI kedua app | — |
| T-12 | P2 | [ ] | Data | Verifikasi semua query list punya index komposit Firestore | T-02 |
| T-13 | P2 | [ ] | Mobile | Cetak termal struk (dev build + abstraksi printing) | — |
| T-14 | P3 | [ ] | Mobile | Draft offline + finalisasi invoice saat online | T-13 |
| T-15 | P3 | [ ] | Mobile | Hardening kuota Spark (audit `onSnapshot` mahal) | — |
| T-16 | P2 | [ ] | Platform | Backup terjadwal export Firestore | T-01 |
| T-17 | P3 | [ ] | Platform | Logging error client mobile (observability murah) | — |

### Temuan review backoffice (2026-06-30) — lihat [detail](#detail-temuan-review-backoffice)

| ID | Prioritas | Status | Area | Task | Verifikasi |
|----|:--------:|:------:|------|------|:----------:|
| T-18 | P0 | [ ] | Keamanan | 3 Server Action scan-area tanpa `requireSession`/`assertCanWrite` | ✅ verified |
| T-19 | P0 | [ ] | Titipan | `voidConsignmentNote` kembalikan stok terjual ke gudang (data korup) | ✅ verified |
| T-20 | P0 | [ ] | Inventori | `updateInventoryProduct` timpa `warehouseStock` langsung & non-atomik | 🔎 agent |
| T-21 | P1 | [ ] | Titipan | `createConsignmentNote` tak cek `warehouseStock ≥ qty` → stok minus | ✅ verified |
| T-22 | P1 | [ ] | Titipan | Duplikat `productId` 1 surat → stok doc vs agregat divergen | ✅ verified |
| T-23 | P1 | [ ] | Invoice | Nomor invoice pakai `new Date()` UTC, bukan WIB → salah bucket harian | ✅ verified |
| T-24 | P1 | [ ] | Keamanan | API `/mobile/.../visit` verifikasi token tapi tanpa cek profil/role | ✅ verified |
| T-25 | P1 | [ ] | Titipan | `deleteCustomerStock` no-op diam saat sisa 0 tapi lapor sukses | 🔎 agent |
| T-26 | P1 | [ ] | Keuangan | Dedup finance pakai docs ter-filter → dobel-hitung setelah delete | 🔎 verify |
| T-27 | P1 | [ ] | Keuangan | `updateFinanceTransaction` tanpa validasi Zod (amount NaN/negatif) | 🔎 agent |
| T-28 | P2 | [ ] | Integritas | Hapus permanen pelanggan/produk orphan stok titipan & movement | ✅ verified |
| T-29 | P2 | [ ] | Keamanan | API `/scan-prospects` tanpa auth → kuras kuota Geoapify | 🔎 agent |
| T-30 | P2 | [ ] | Inventori | `sellingPrice` pack = float tanpa pembulatan → drift uang | 🔎 agent |
| T-31 | P2 | [ ] | Titipan | Semantik draft: surat draft sudah memotong stok saat dibuat | ❓ decision |
| T-32 | P2 | [ ] | Validasi | Kumpulan celah validasi (status param, packSize≥2, batch off-by-one) | 🔎 agent |

> Legenda verifikasi: ✅ saya cek langsung ke kode · 🔎 temuan agent, spesifik (file:line), perlu konfirmasi saat dikerjakan · ❓ keputusan produk, bukan bug murni.

---

## Detail task

### T-01 · P0 · Temukan & version-control `firestore.rules`
**Kenapa:** mobile menulis Firestore langsung dari device; rules = satu-satunya penjaga.
File rules belum ketemu di repo mana pun saat dokumentasi disusun.
**DoD:** lokasi rules dipastikan; rules ada di repo (mis. `venovian-toys/firestore.rules`)
dan ter-deploy via `firebase deploy --only firestore:rules` (atau dicatat prosesnya).
**Ref:** [SEC-1](07-bug-register.md#keamanan).

### T-02 · P0 · Audit & perketat Security Rules per-koleksi
**Kenapa:** rules longgar = siapa pun dengan kredensial bisa baca/tulis apa saja.
**DoD:** tiap koleksi (doc 05) punya aturan read/write eksplisit berbasis auth + role
(`userProfiles.role`) + kepemilikan. Uji: user non-authorized ditolak; mobile hanya bisa
operasi yang seharusnya. **Ref:** [SEC-1](07-bug-register.md#keamanan), [06](06-kontrak-integrasi.md).

### T-03 · P0 · Amankan secret
**DoD:** `.env.local` ada di `.gitignore` & tak pernah ter-commit (cek `git log`);
`FIREBASE_PRIVATE_KEY` & kunci lain hanya sebagai Encrypted Env Var di Vercel.
**Ref:** [SEC-2](07-bug-register.md#keamanan).

### T-04 · P1 · README mobile akurat
**DoD:** README mobile menjelaskan stack Expo/React Native sebenarnya; tak ada sisa teks
"web application / Next.js / Leaflet". **Ref:** [DOC-1](07-bug-register.md).

### T-05 · P1 · AGENTS.md cerminkan fase terintegrasi
**DoD:** bagian "Current Phase: strictly UI-first / mock only" diganti agar agent berikutnya
tahu Firebase sudah live. **Ref:** [DOC-2](07-bug-register.md).

### T-06 · P1 · Satukan jalur update prospek
**Kenapa:** ada jalur HTTP (Admin, isi `updatedBy`/`visitedAt`) **dan** `updateDoc` client
SDK langsung — hasil bisa beda. **DoD:** satu jalur dipilih; pemanggil lain dihapus;
field hasil konsisten. **Ref:** [INT-1](07-bug-register.md#int-1).

### T-07 · P1 · Satukan logika transaksi titipan
**DoD:** implementasi `runTransaction` titipan di backoffice vs mobile dibandingkan
field-demi-field (nama koleksi, `quantityRemaining`, tulis `stockMovements`); disatukan
atau kontraknya didokumentasikan + diuji identik. **Ref:** [INT-2](07-bug-register.md#int-2).

### T-08 · P2 · Tipe/skema bersama
**DoD:** ada `shared-schema` (paket atau file disalin-terjaga) untuk tipe dokumen Firestore
inti; checklist "ubah skema → update kedua app + database-design + rules + seed".
**Ref:** [INT-3](07-bug-register.md).

### T-09 · P1 · Lint + typecheck sweep
**DoD:** `npm run lint` & `tsc --noEmit` jalan bersih (atau temuan dicatat jadi task) di
kedua app. *(Catatan: AGENTS.md melarang agent menjalankan ini otomatis — task ini
dijalankan saat sesi audit yang disengaja.)*

### T-10 · P2 · Dobel-hitung keuangan
**DoD:** kasus pergerakan stok yang dicatat manual **dan** otomatis diuji; tak terhitung
dua kali; backoffice & mobile menghitung sama. **Ref:** [BUG-1](07-bug-register.md).

### T-11 · P2 · Tanggal & timezone
**DoD:** tak ada ISO bocor ke layar; "jatuh tempo hari ini"/`nextVisitAtMs`/`dueDate`
benar di WIB. **Ref:** [BUG-2](07-bug-register.md).

### T-12 · P2 · Index komposit
**DoD:** tiap query list (filter+sort+paginate) punya index komposit; daftar index
disinkronkan dgn `database-design.md`. **Ref:** [05](05-model-data.md).

### T-13–T-17
Fitur & platform — detail di [08 — Roadmap](08-roadmap.md) Fase 3–4 dan
`venovian-toys-mobile/docs/mobile-roadmap.md`.

---

## Detail temuan review backoffice

> Hasil review kode 2026-06-30 (4 agent paralel + verifikasi langsung). Repo:
> `venovian-toys`. Path semua relatif ke repo itu. Tanda ✅ = saya baca kodenya sendiri.

### T-18 · P0 · 3 Server Action scan-area tanpa guard auth ✅
**File:** `src/actions/scan-area.ts:23,35,76`
**Bukti:** `saveProspectAction`, `removeSavedProspectAction`, `updateSavedProspectVisitAction`
langsung memanggil DB tanpa `requireSession()`/`assertCanWrite()` — padahal file yang sama
mengimpor `requireSession` dan mendefinisikan `assertCanWrite`, dan `convertSavedProspectToCustomerAction`
memakainya. Server Action = endpoint POST yang bisa dipanggil siapa saja.
**Dampak:** klien tak terautentikasi bisa hapus/timpa/buat dokumen `scanProspects` apa pun by id.
**Fix:** tambahkan `const user = await requireSession(); assertCanWrite(user.role);` di awal
ketiga action + validasi `prospect` dengan Zod sebelum `saveProspectToDb`.

### T-19 · P0 · `voidConsignmentNote` korup stok untuk unit yang sudah terjual ✅
**File:** `src/services/firebase/consignment-notes.ts:618-635`
**Bukti:** pembatalan menambah **seluruh** `quantitySent` balik ke `warehouseStock`
(`currentWarehouse + quantitySent`) dan mengurangi `quantityRemaining` sebesar `quantitySent`,
tanpa memperhitungkan `quantitySold`.
**Skenario:** surat kirim 10 pcs → warung jual 4 → sisa 6. Saat void: gudang +10 (padahal
fisik cuma 6 yang kembali), `quantityRemaining = max(0, 6-10) = 0`, sementara `quantitySold`
tetap 4 → muncul `quantitySold(4) > quantitySent(0)`. Stok gudang & agregat korup.
**Fix:** balik hanya porsi yang masih ada: `min(quantitySent, currentRemaining)` untuk
gudang/consigned, jangan sentuh unit terjual; ATAU blokir void bila ada item yang sudah terjual.

### T-20 · P0/P1 · `updateInventoryProduct` timpa stok langsung & non-atomik 🔎
**File:** `src/services/firebase/inventory.ts:~968,998,1019-1063`
**Bukti (agent):** form edit produk menulis `warehouseStock: data.warehouseStock` langsung
(timpa akumulasi ledger), lalu sync finance/movement di batch terpisah (non-transaksional).
**Skenario:** produk di-restock 100→500, admin buka form (masih tampil 100), simpan →
stok jadi 100 tanpa movement kompensasi. Plus, gagal di tengah = produk/movement/finance
tak konsisten.
**Fix:** jangan tulis `warehouseStock` dari form edit (mutasi stok hanya via restock/reduce),
atau bungkus update + sync dalam satu `runTransaction`. **Cek dulu** apakah benar form edit
menulis stok.

### T-21 · P1 · `createConsignmentNote` tanpa cek stok cukup → gudang minus ✅
**File:** `src/services/firebase/consignment-notes.ts:499`
**Bukti:** `nextWarehouseStock = Number(warehouseStock||0) - qtySent` ditulis tanpa cek
`warehouseStock >= qtySent` — beda dengan `addCustomerConsignment` (customers.ts:~783) yang
menjaga ini. Concurrent note untuk produk sama juga bisa oversell.
**Fix:** lempar error bila `warehouseStock < qtySent` sebelum menulis nilai dekremen.

### T-22 · P1 · Duplikat `productId` dalam satu surat → divergensi ✅
**File:** `src/services/firebase/consignment-notes.ts:424-467,496-500` · schema `src/validations/consignment-note.ts`
**Bukti:** dua item dengan `productId` sama menulis ke `stockRef` yang sama 2x dalam satu
transaksi — `nextQuantitySent` tiap dihitung dari snapshot stale yang sama, **last write
wins** (kuantitas item pertama hilang di stock doc), sementara dekremen `warehouseStock`
produk menjumlahkan keduanya (`reduce`). Stok doc ≠ agregat produk.
**Fix:** tolak `productId` duplikat di schema (`superRefine`) atau merge item duplikat sebelum proses.

### T-23 · P1 · Nomor invoice pakai UTC, bukan WIB ✅
**File:** `src/services/firebase/invoices.ts:~581-583,704-720` (bandingkan consignment-notes
yang pakai `getIndonesianDateInput()`)
**Bukti:** `invoiceDateKey`/`buildInvoiceNumber` berbasis `new Date()` (UTC server), sedang
counter surat titipan WIB-aware. Invoice jam 22:00 WIB (15:00 UTC) bisa masuk bucket counter
hari sebelumnya → nomor & tanggal tampil beda hari, sekuens bisa tabrakan di batas tengah malam.
**Fix:** hitung tanggal invoice dari wall-clock Asia/Jakarta, samakan dgn pola consignment.
**Terkait:** T-11 (timezone), dan status `overdue`/`startOfToday()` juga server-local (perlu WIB).

### T-24 · P1 · API visit verifikasi token tapi tanpa cek profil/role ✅
**File:** `src/app/api/mobile/prospects/[id]/visit/route.ts:23-46`
**Bukti:** route hanya `verifyIdToken(idToken, true)` lalu `update` — tidak memuat
`getUserProfile(uid)` / cek role/status seperti jalur web. Siapa pun yang bisa mint ID token
valid project ini (mis. user `inactive`) bisa ubah visit prospek mana pun by id (IDOR).
**Fix:** setelah verifikasi token, muat profil, tolak bila tak ada/`inactive`/role tak berhak;
cek dokumen ada sebelum update; batasi panjang `visitNotes` (mis. Zod max 500).

### T-25 · P1 · `deleteCustomerStock` no-op diam tapi lapor sukses 🔎
**File:** `src/services/firebase/customers.ts:917-924`
**Bukti (agent):** saat `quantityRemaining <= 0`, fungsi set 0 lalu `return` sukses tanpa
mencatat movement retur — operator kira sisa stok ditarik ke gudang, padahal tak ada yang bergerak.
**Fix:** lempar error "tidak ada sisa stok untuk ditarik".

### T-26 · P1 · Dedup finance hilang setelah soft-delete 🔎
**File:** `src/services/firebase/finance.ts:438-445`
**Bukti (agent):** `postedMovementIds` dibangun dari `manualTransactions` yang sudah membuang
doc ber-`deletedAt`. Finance entry hasil posting movement yang di-soft-delete → `stockMovementId`-nya
hilang dari set → movement disintesis ulang jadi transaksi baru = dobel hitung.
**Verify:** apakah finance ber-source `stock_movement` pernah di-soft-delete dalam praktik.
**Fix:** bangun `postedMovementIds` dari `financeSnapshot.docs` mentah (termasuk yang terhapus).

### T-27 · P1 · `updateFinanceTransaction` tanpa validasi Zod 🔎
**File:** `src/services/firebase/finance.ts:~924-927`
**Bukti (agent):** `amount: number` inline tanpa schema — NaN/negatif/0 bisa menimpa transaksi
langsung (create pakai `min(1000)`, update tidak).
**Fix:** validasi input update dengan Zod (amount finite & positif).

### T-28 · P2 · Hapus permanen meng-orphan stok & movement ✅ (pola)
**File:** `src/services/firebase/customers.ts:1242-1256` & `inventory.ts:~1267-1276`
**Bukti:** `permanentlyDeleteCustomer`/`permanentlyDeleteInventoryProduct` hanya `.delete()`
dokumen utama; `customerStocks`, `stockMovements`, dan `consignedStock` produk tak direverse/
dibersihkan. Pelanggan dgn stok titipan berjalan bisa dihapus → produk tetap catat stok
sebagai consigned selamanya.
**Fix:** sebelum hapus permanen, reverse stok consigned berjalan + cascade/cek; atau blokir
hapus bila masih ada stok berjalan.

### T-29 · P2 · API `/scan-prospects` tanpa auth → kuras kuota Geoapify 🔎
**File:** `src/app/api/scan-prospects/route.ts:~553`
**Bukti (agent):** handler GET tanpa cek sesi (proxy mengecualikan `/api`); pemanggil anonim
bisa picu scan Overpass/Geoapify berulang, membakar kuota `GEOAPIFY_API_KEY` (amplifikasi biaya/DoS).
**Fix:** wajibkan `getSessionUser()` di awal handler + rate-limit.

### T-30 · P2 · `sellingPrice` pack = float tanpa pembulatan 🔎
**File:** `src/services/firebase/inventory.ts:~225-229,1009`
**Bukti (agent):** `sellingPrice = sellingUnitPrice / packSize` disimpan sbg float mentah
(mis. 10000/3 = 3333.333…) → drift pembulatan saat dikali balik.
**Fix:** bulatkan ke presisi tetap, atau simpan hanya `sellingUnitPrice`+`packSize` dan turunkan
per-pcs saat baca.

### T-31 · P2 · Semantik draft surat titipan ❓ (keputusan)
**File:** `src/services/firebase/consignment-notes.ts:316,373-508,573`
**Bukti:** `createConsignmentNote` dgn `status:"draft"` **sudah** memotong stok & menulis
movement; `publishConsignmentNote` hanya flip status. Apakah draft seharusnya sudah mengirim stok?
**Fix (bila tidak):** gate tulis stok/movement pada `status === "issued"`, terapkan stok saat publish.

### T-32 · P2 · Kumpulan celah validasi 🔎
- `createConsignmentNoteAction(input, status)` — argumen `status` tak divalidasi Zod
  (`src/actions/consignment-notes.ts:27`); pakai `z.enum(["draft","issued"])`.
- `consignmentNoteItemSchema` izinkan `pricingUnit:"pack"` dgn `packSize:1`
  (`src/validations/consignment-note.ts:11`) — beda dgn `customerConsignmentSchema` (≥2).
- `updateProductSupplier` batch off-by-one: slice `0..449` lalu loop mulai `449` → produk
  index 449 ditulis 2x (`inventory.ts:~1404-1442`); mulai dari 450.

### ❌ Diselidiki, BUKAN bug (jangan diangkat ulang)
- **Line-total pack `quantitySold × unitPrice`** (sempat dicurigai ×packSize): ✅ saya cek —
  `unitPrice` adalah **per-pcs** secara konsisten (ada `sellingUnitPrice` terpisah per selling
  unit). `invoices.ts:656,693` mengambil `unitPrice` dari `customerStock.unitPrice` per-pcs.
  Jadi `quantitySold(pcs) × unitPrice(per pcs)` benar. Tidak ada inflasi.
- **Authz Server Action lain**: ✅ seluruh action mutasi di customers/inventory/invoices/finance/
  consignment-notes punya `requireSession`+`assertCanWrite`. Hanya 3 scan-area (T-18) yang bolong.
- **Cookie sesi**: `httpOnly`+`sameSite:lax`+`secure` (prod), `verifyIdToken(...,true)`,
  `verifySessionCookie(...,true)` — bersih.
- **SSRF scan-area**: host tetap, hanya `lat/lng/radius` numerik ter-clamp; API key server-only. Bersih.

---

## Changelog task

Catat perubahan status penting di sini (tanggal — ID — aksi):

- 2026-06-30 — Backlog dibuat (T-01…T-17).
- 2026-06-30 — Review backoffice: T-18…T-32 ditambah (2 P0 verified, 5 P1, dst). 1 false-positive (pack line-total) ditolak setelah verifikasi.

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

### 🚧 EPIC AKTIF — Migrasi Monorepo (2026-07-05) — rancangan: [14](14-migrasi-monorepo.md)

> ⚠ **FREEZE:** selama T-51…T-55 berjalan, JANGAN kerjakan task kode lain di kedua app
> (repo lama read-only; lihat aturan keras di doc 14). Task docs tetap boleh.
> Protokol kerja Opus↔Sonnet: [workflow/AGENT-WORKFLOW.md](workflow/AGENT-WORKFLOW.md).

| ID | Prioritas | Status | Pelaksana | Task | Dep |
|----|:--------:|:------:|-----------|------|-----|
| T-51 | P0 | [ ] | Sonnet | Rakit monorepo: subtree import 3 repo (histori utuh) + npm workspaces + push | — |
| T-52 | P0 | [ ] | Sonnet | Backoffice hijau di monorepo (`next build`+lint+tsc) + pindah `firestore.rules` → `firebase/` | T-51 |
| T-53 | P0 | [ ] | Sonnet | Mobile hijau di monorepo (tsc+lint+`expo start` di device) + `.easignore` | T-51 |
| T-54 | P0 | [ ] | Sonnet+owner | EAS build preview APK dari monorepo + uji di HP | T-53 |
| T-55 | P0 | [ ] | Owner (dipandu Opus) | Cutover Vercel: connected repo + Root Directory `apps/backoffice` | T-52 |
| T-56 | P1 | [ ] | Sonnet | CI GitHub Actions (path filter per app) + branch protection `main` | T-55 |
| T-57 | P1 | [ ] | Opus | CLAUDE.md root + port skills octobot (`seam-audit` dkk.) + `.claude/settings.json` | T-51 |
| T-58 | P1 | [ ] | Sonnet | Kerangka kosong `packages/shared` + `transpilePackages` (TANPA pindah logika) | T-52, T-53 |
| T-59 | P1 | [ ] | Owner (dipandu Opus) | Archive 3 repo lama + swap folder lokal + perbaiki link docs | T-54, T-55 |
| T-60 | P2 | [ ] | Sonnet | Revisi AGENTS.md kedua app (izinkan verifikasi lint/tsc; rujuk workflow) | T-57 |
| T-61 | P2 | [ ] | Owner (dipandu Opus) | Env Preview Vercel → Firebase dev (`venovian-toys-dev`); catat doc 09+13 | T-55 |

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

### Penguatan struktural — keuangan & scan area (2026-06-30) — lihat [11](11-fokus-keuangan-scan-area.md)

Prioritas pemilik: **data keuangan benar** + **scan area akurat**, tetap **gratis**.

| ID | Prioritas | Status | Area | Task |
|----|:--------:|:------:|------|------|
| T-33 | P1 | [ ] | Keuangan | Satu modul rumus uang/stok dipakai bersama (A1) |
| T-34 | P1 | [ ] | Keuangan | Uang = integer rupiah; jangan simpan hasil bagi (A2) |
| T-35 | P1 | [ ] | Keuangan | Halaman "Audit Data" pemeriksa invarian/rekonsiliasi (A3) |
| T-36 | P2 | [ ] | Keuangan | Idempotensi anti dobel-submit invoice/pembayaran (A4) |
| T-37 | P2 | [ ] | Keuangan | Jejak audit tiap mutasi uang + golden cases (A5,A6) |
| T-38 | P1 | [ ] | Scan Area | Cache hasil Overpass/geocode di Firestore + TTL (B1) |
| T-39 | P1 | [ ] | Scan Area | Patuhi rate-limit Nominatim/Overpass + mirror+retry; amankan endpoint (B2) |
| T-40 | P2 | [ ] | Scan Area | Loop umpan balik lapangan untuk menyetel skor + confidence (B3,B4,B5) |

### Temuan review mobile (2026-06-30) — lihat [detail](#detail-temuan-review-mobile)

> Mobile menulis Firestore **langsung dari device**. Beberapa bug **mengulang** bug backoffice
> (logika titipan terduplikasi, [INT-2](07-bug-register.md#int-2)) → makin menguatkan T-33/T-08.

| ID | Prioritas | Status | Area | Task | Verifikasi |
|----|:--------:|:------:|------|------|:----------:|
| T-41 | P1 | [ ] | Keamanan | Visit prospek pakai `updateDoc` langsung + `actorUid` dari client; endpoint HTTP backoffice = dead code | ✅ verified |
| T-42 | P0 | [ ] | Titipan | Duplikat `productId` 1 surat → delta stok hilang (cermin T-22) | 🔎 agent |
| T-43 | P1 | [ ] | Titipan | `createConsignmentNote` mobile tanpa guard `warehouseStock ≥ qty` (cermin T-21) | 🔎 agent |
| T-44 | P1 | [ ] | Keuangan | Modal titip simpan `unitPrice` per-pcs pecahan (÷packSize tanpa bulat); server percaya mentah | ✅ verified |
| T-45 | P1 | [ ] | Keuangan | `dashboard.totalIn` jumlah `collection`+`revenue` → potensi dobel pemasukan | ✅ verified |
| T-46 | P1 | [ ] | Keuangan | Tulis tanpa validasi (qty/price NaN/negatif); `Math.max(0,…)` tutupi drift; fallback `sellingUnitPrice` pack salah | 🔎 agent |
| T-47 | P1 | [ ] | Keamanan | Tak ada cek `userProfiles` role/status sesudah login → akun nonaktif tetap akses penuh | 🔎 agent |
| T-48 | P2 | [ ] | Kuota | `onSnapshot` full-collection tanpa limit (customers/prospects) + prospek tanpa filter `deletedAt` | 🔎 agent |
| T-49 | P2 | [ ] | UI | Input qty: `packSize` kosong → baris 0-pcs; guard anti dobel-tap; fallback pakai packSize editable | 🔎 agent |
| T-50 | P3 | [ ] | UI | ISO tanggal bocor (nextVisitAt/joinDate); `packSize=0` → label NaN; race set-nama; error profil ditelan | 🔎 agent |

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

## Detail temuan review mobile

> Review kode mobile 2026-06-30 (3 agent paralel + verifikasi langsung). Repo:
> `venovian-toys-mobile`. ✅ = saya baca/verifikasi sendiri. **Konvensi terbukti benar:**
> `lineTotal = quantitySold(pcs) × unitPrice(per-pcs)` — sama dgn backoffice, BUKAN bug.

### T-41 · P1 · Visit prospek lewat client SDK; endpoint HTTP backoffice dead code ✅
**File:** `src/services/firebase/prospects.ts:76-90`
**Bukti:** `updateProspectVisit` memanggil `updateDoc(doc(db,'scanProspects',id), {... updatedBy: actorUid})`
langsung. Grep seluruh `src` mobile: **0 referensi** ke `EXPO_PUBLIC_API_URL`/`/api/mobile/.../visit`.
Jadi endpoint backoffice `PATCH /api/mobile/prospects/[id]/visit` **tidak dipakai (dead code)**,
dan `actorUid` adalah string dari client (bisa diisi uid siapa pun).
**Dampak:** menulis langsung ke koleksi `scanProspects` yang dipakai bersama backoffice;
keamanan 100% bergantung Firestore Rules (lihat T-01/T-02). `updatedBy` tak terjamin = pelaku asli.
**Fix:** pilih SATU jalur — entah pakai endpoint HTTP (Bearer token, `updatedBy=uid` dari server)
lalu hapus `updateProspectVisit` langsung, ATAU kunci Rules `scanProspects`: hanya field visit
yang boleh ditulis + `request.auth.uid == request.resource.data.updatedBy`. **Menyelesaikan
[INT-1](07-bug-register.md#int-1)**; jika endpoint dipertahankan, [T-24](#t-24--p1--api-visit-verifikasi-token-tapi-tanpa-cek-profilrole-) tetap perlu diperbaiki.

### T-42 · P0 · Duplikat productId → delta stok hilang (cermin backoffice T-22) 🔎
**File:** `src/services/firebase/consignment-notes.ts:58-66,~197`
**Bukti (agent):** dua item ber-`productId` sama membaca snapshot produk yang sama lalu menulis
`currentWarehouse - item.quantitySent` independen → **last write wins**, delta item pertama hilang.
**Fix:** pra-agregasi item per `productId` (Map delta), satu `transaction.update` per produk.

### T-43 · P1 · Tanpa guard stok cukup → gudang minus (cermin T-21) 🔎
**File:** `src/services/firebase/consignment-notes.ts:201-205`
**Bukti (agent):** `warehouseStock: currentWarehouse - item.quantitySent` tanpa cek `>=`. Oversell
mendorong stok negatif. Server adalah penjaga terakhir (UI hanya cek stok dari subscription stale).
**Fix:** dalam transaksi, lempar bila `currentWarehouse < item.quantitySent`.

### T-44 · P1 · `unitPrice` per-pcs pecahan tersimpan permanen ✅
**File:** `src/components/AddConsignmentModal.tsx:61-63` → `consignment-notes.ts:~101,156`
**Bukti:** `calcUnitPricePerPcs = displayPrice / Math.max(1, packSize)` **tanpa pembulatan**;
server menyimpan `unitPrice` & `lineTotal = quantitySent × unitPrice` mentah.
**Skenario:** 5 pack × Rp10.000, packSize 3 → per-pcs 3.333,33; tersimpan 15 × 3.333,33 =
Rp49.999,95, bukan Rp50.000. Rupiah pecahan permanen + UI "Nilai" bisa beda dari yang tersimpan.
**Fix:** kirim `sellingUnitPrice`+`packSize`, hitung lineTotal dari harga pack di server; atau
bulatkan ke integer sebelum simpan. **Ini contoh konkret [T-34](#t-34--p1--uang--integer-rupiah-jangan-simpan-hasil-bagi-a2).**

### T-45 · P1 · `dashboard.totalIn` potensi dobel pemasukan ✅
**File:** `src/services/firebase/dashboard.ts:36,113`
**Bukti:** `if (type === 'collection' || type === 'revenue') totalIn += amount;`. Dedup yang ada
(`postedMovementIds`/`postedProductIds`) hanya melindungi sisi modal/expense, **bukan** sisi
pemasukan. Bila backoffice mencatat `revenue` (akrual saat invoice) **dan** `collection` (kas saat
bayar) untuk satu penjualan → terhitung dua kali. Plus seluruh `financeTransactions`+`products`
di-scan penuh tiap buka dashboard (kuota Spark) dan modal "legacy" diturunkan dari stok hidup
(bisa drift). **Verify** semantik tipe finance backoffice. **Fix:** jumlahkan hanya SATU dari
revenue/collection; jadikan dashboard turunan dari sumber tunggal ([T-33](#t-33--p1--satu-modul-rumus-uangstok-dipakai-bersama-a1)/[T-35](#t-35--p1--halaman-audit-data-pemeriksa-invarianrekonsiliasi-a3)).

### T-46 · P1 · Validasi tulis lemah + clamp tutupi drift 🔎
**File:** `consignment-notes.ts:95-102`, `customer-stocks.ts:150-156,69`
**Bukti (agent):** tak ada cek `quantitySent>0 & integer`, `unitPrice finite & ≥0` sebelum tulis
(NaN/negatif bisa masuk). `Math.max(0, consignedValue - lineTotal)` meng-clamp diam — korupsi tak
terlihat. Fallback `sellingUnitPrice = ... || unitPrice` salah untuk produk pack (mestinya
`unitPrice × packSize`). **Fix:** validasi item sebelum tulis; ganti clamp dgn alert/log atau
hitung ulang agregat dari `customerStocks`; perbaiki fallback pack.

### T-47 · P1 · Tak ada cek role/status sesudah login 🔎
**File:** `src/store/auth-store.ts:69-83`
**Bukti (agent):** sesudah auth hanya ambil nama tampilan; tak muat `userProfiles` untuk cek
role/status. Akun yang ada di Firebase Auth tapi dinonaktifkan di data bisnis tetap login penuh &
menulis Firestore langsung. **Fix:** muat `userProfiles`, tolak (sign out) bila tak ada/role tak
berhak/`inactive` sebelum sesi aktif. (Cermin kekhawatiran [T-24](#t-24--p1--api-visit-verifikasi-token-tapi-tanpa-cek-profilrole-).)

### T-48 · P2 · Listener full-collection tanpa batas 🔎
**File:** `customers.ts:86`, `prospects.ts:96`
**Bukti (agent):** `subscribeToCustomers`/`subscribeToProspects` = `onSnapshot` seluruh koleksi
tanpa `limit`; `scanProspects` juga tanpa `where('deletedAt','==',null)` (arsip ikut terbaca,
hanya `rejected` difilter di klien). Banyak device → kuras kuota baca Spark. **Fix:** tambah
`limit`/scope per-area + filter `deletedAt` server-side. (Terkait [T-15](10-task-tracker.md), [T-38](#).)

### T-49 · P2 · Edge input kuantitas & dobel-submit 🔎
**File:** `AddConsignmentModal.tsx:35-52,146,184-188`, `CustomerStockDetailModal.tsx:166-222`
**Bukti (agent):** `packSize` free-text — kosong → `calcPcsSent = qty×0 = 0` lolos cek `quantity<=0`
(yang cek jumlah pack, bukan pcs) → baris 0-pcs ber-nilai non-0. Handler sale/withdraw tak ada
`if (isSubmitting) return;` di awal → dobel-tap cepat bisa picu 2 penjualan. Fallback harga pack
pakai `packSize` editable (bukan milik produk) → harga bisa berubah-ubah. **Fix:** validasi
`calcPcsSent>0` sebelum submit; guard re-entry; pakai `product.packSize` untuk fallback.

### T-50 · P3 · Kumpulan minor UI/robustness 🔎
- ISO tanggal mungkin bocor ke layar: `customers.tsx:520,522` (`nextVisitAt`/`joinDate` di-render
  mentah) — verifikasi tipe, format ke `2 Feb 2026`.
- `Math.floor(qtyRemaining / packSize)` saat `packSize=0` → `NaN`/`Infinity` di label
  (`CustomerStockDetailModal.tsx:47-53`, `CustomerStockReceiptModal.tsx:31-33`); guard `packSize>0`.
- Race set-nama di `auth-store.ts:80` (async profil resolve sesudah logout/relogin) — cek
  `state.user?.uid === user.uid` sebelum apply.
- `getUserProfileName` (`auth.ts:18-27`) telan semua error → permission-denied tak terlihat;
  minimal `console.error`.

### ❌ Diselidiki, BUKAN bug (mobile)
- **`lineTotal = quantity × unitPrice`** di `customer-stocks.ts`: ✅ benar — `unitPrice` per-pcs
  konsisten dgn backoffice. Tidak ada inflasi (beda dari isu pembulatan T-44 yang soal *pembagian*).
- **`config.ts` secret**: hanya config Firebase web publik (`apiKey` dll) lewat `EXPO_PUBLIC_*` —
  aman dikirim ke device. Tidak ada private key/admin secret. Bersih.
- **`client.ts` persistence**: AsyncStorage terpasang benar dgn fallback. Bersih.
- **`catalog.tsx`, `invoices.ts`, `products.ts`**: read-only display, tak ada mutasi uang. Bersih.

---

## Changelog task

Catat perubahan status penting di sini (tanggal — ID — aksi):

- 2026-06-30 — Backlog dibuat (T-01…T-17).
- 2026-06-30 — Review backoffice: T-18…T-32 ditambah (2 P0 verified, 5 P1, dst). 1 false-positive (pack line-total) ditolak setelah verifikasi.
- 2026-06-30 — Penguatan struktural keuangan & scan area: T-33…T-40 (lihat [doc 11](11-fokus-keuangan-scan-area.md)).
- 2026-06-30 — Review mobile: T-41…T-50 (1 P0, 6 P1). Jalur visit prospek = DIRECT (INT-1 terjawab; endpoint backoffice dead code). Mobile mengulang bug stok backoffice (T-42=T-22, T-43=T-21).
- 2026-07-05 — Keputusan migrasi monorepo (owner + Fable): T-51…T-61 ditambah, FREEZE task kode selama T-51…T-55. Rancangan: [doc 14](14-migrasi-monorepo.md); protokol kerja: [workflow/AGENT-WORKFLOW.md](workflow/AGENT-WORKFLOW.md).
- 2026-07-05 — T-16 dicicil: skrip `npm run backup` (read-only, per-project) di repo backoffice; backup prod pertama sukses (166 dok, 15 koleksi). Backup terjadwal/otomatis tetap terbuka di T-16.
- 2026-07-05 — Repo `venovian-toys` di-rename → `venovian-toys-web` (Vercel tetap nempel, diverifikasi); repo kosong `ngadiono/venovian-toys` dibuat untuk monorepo.

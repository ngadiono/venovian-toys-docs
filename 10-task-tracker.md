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

## Changelog task

Catat perubahan status penting di sini (tanggal — ID — aksi):

- 2026-06-30 — Backlog dibuat (T-01…T-17).

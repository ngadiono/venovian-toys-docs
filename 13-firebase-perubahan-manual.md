# 13 — Perubahan Firebase Manual (Checklist)

> **Kepemilikan:** perubahan sisi Firebase (composite index, Security Rules, seed data,
> config) **dikerjakan manual oleh pemilik** (ngadionocr@gmail.com), bukan oleh agent coding.
> Dokumen ini = daftar tugas Firebase yang muncul dari perubahan kode, lengkap langkahnya.
>
> **Dua project (keduanya free/Spark):**
> - `dev` — sedang dipakai untuk pengembangan sekarang.
> - `prod` — dipakai live oleh pemilik + istri.
>
> **Aturan:** setiap perubahan diterapkan ke **dev dulu**, diuji, baru ke **prod**. Centang
> per-project agar tak ada yang kececer.

## Pembagian tugas

| Hal | Pemilik | Cara |
|---|---|---|
| Security Rules | **Pemilik** | Firebase Console / CLI |
| Composite index | **Pemilik** | Console (link dari error) / `firestore.indexes.json` |
| Kode seed (`scripts/seed-data.js`) | Agent/Sonnet | edit kode biasa |
| Jalankan `npm run seed` | Agent boleh, **konfirmasi dulu** | guard hanya izinkan project di `FIREBASE_SEED_ALLOWED_PROJECT_IDS` (default `venovian-toys-dev`) — **dev-only** |
| Jalankan `npm run clear:db` | Agent boleh, **destruktif → konfirmasi** | dev-only |
| Seed/hapus **prod** | **Pemilik saja** | tidak pernah oleh agent |

> Agent tidak pernah auto-run seed/clear: selalu tampilkan `FIREBASE_PROJECT_ID` aktif dan
> tunggu persetujuan. Per 2026-07-02 project aktif = `venovian-toys-dev`.

## Cara pakai

- Tiap item punya status per project: `[ ] dev` · `[ ] prod`.
- Agent menambah item ke sini saat sebuah PR/prompt butuh langkah Firebase.
- Jangan hapus item selesai — centang + tanggal.

---

## Terbuka

### FB-1 🔴 Selaraskan Firestore Security Rules dgn fitur setelmen mobile (SEC-1, SEC-3)
Rules yang **sudah berjalan** ternyata bagus (per-koleksi, gate `status=='active'`,
default-deny) — **bukan** test-mode kebuka. Jadi bukan "tulis dari nol", tapi **tambah delta**.

**Masalah (SEC-3):** rules berjalan dibuat **sebelum** fitur setelmen mobil
(`createInvoiceSettlement`, KEU-2). Koleksi `invoices` / `invoicePayments` /
`financeTransactions` di rules itu **read-only** dari client, sedangkan setelmen mobil
**menulis** ke sana. Karena transaksi Firestore all-or-nothing → **setelmen mobil selalu
gagal `permission-denied`**. (Inilah kemungkinan sebab tes setelmen di HP belum tembus.)

**Perbaikan:** file [`venovian-toys/firestore.rules`](../venovian-toys/firestore.rules) =
rules lama **+ delta** (ditandai `DELTA` di komentar): izin **CREATE** (bukan update/delete —
catatan uang tetap tak bisa diubah/dihapus dari HP) untuk `invoices`, `invoicePayments`,
`financeTransactions`; plus koleksi `invoiceCounters` & subcollection `invoices/{id}/items`.
Void/koreksi invoice tetap lewat backoffice (Admin SDK bypass rules). Tidak menyentuh
koleksi/gate lain → **nol regresi**.

**Langkah terap (dev dulu, uji, baru prod):**
1. **Publish rules.** Firebase Console → project → Firestore Database → tab **Rules** →
   tempel isi `firestore.rules` → **Publish**. (Reversibel instan, tidak menyentuh data.)
2. **Uji di dev:** buka app mobil → **buat 1 setelmen** → harus **berhasil** + tersinkron ke
   web. Fitur lama (lihat stok, restok/tarik, consignment note, kunjungan prospek) tetap jalan
   karena path-nya tak berubah. Baru terapkan ke prod.
- [ ] dev · [ ] prod
- (Opsional, kapan saja) Pengetatan auth: matikan self-signup di Authentication → Settings,
  agar hanya akun yang dibuat manual yang bisa login. Bukan bagian rules, tak mendesak.
- Rujukan: [07 — Bug Register SEC-1/SEC-3](07-bug-register.md#keamanan)

---

## Tidak ada perubahan Firebase (dicatat agar jelas)

| Pekerjaan | Butuh Firebase? | Alasan |
|---|---|---|
| Area B — guard `deletedAt` mobile | ❌ | hanya logika transaksi |
| Area B — bucket difilter in-memory | ❌ | justru menghapus klausa query (kurangi kebutuhan index) |
| Invoice **F1** — void invoice | ❌ | query `where invoiceId ==` single-field (index otomatis); movement `invoice_void` schemaless |
| Invoice **F3** — guard finance invoice-source | ❌ | hanya logika |

---

## Selesai

_(belum ada)_

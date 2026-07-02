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

### FB-1 🔴 Firestore Security Rules belum ada di repo (SEC-1)
Mobile akses Firestore **langsung** dari device; Security Rules adalah satu-satunya penjaga.
File `firestore.rules` belum ditemukan di repo mana pun — kemungkinan hanya ada di Console
(atau default test-mode). **Belum dibutuhkan untuk kerja invoice/titipan sekarang**, tapi
**wajib** sebelum dianggap aman di prod.
- Langkah (nanti, saat masuk area Keamanan): tulis `firestore.rules` per-koleksi (read/write
  per role `userProfiles.role` + kepemilikan), simpan di repo backoffice, terapkan ke dev → prod.
- [ ] dev · [ ] prod
- Rujukan: [07 — Bug Register SEC-1](07-bug-register.md#keamanan)

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

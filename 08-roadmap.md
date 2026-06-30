# 08 — Roadmap

Status: app **sudah live dan dipakai**. Roadmap ini fokus ke (a) stabilkan & dokumentasi,
(b) tutup risiko integrasi, (c) lanjut fitur lapangan. Urutan = prioritas.

## Fase 0 — Dokumentasi & kebenaran dasar (sedang berjalan)

- [x] Dokumentasi payung lintas-app (repo ini).
- [ ] Betulkan README & AGENTS.md mobile yang basi ([DOC-1, DOC-2](07-bug-register.md)).
- [ ] Temukan & version-control `firestore.rules` ([SEC-1](07-bug-register.md#keamanan)).
- [ ] Konfirmasi `.env.local` tidak ter-commit & secret aman di Vercel ([SEC-2](07-bug-register.md#keamanan)).

## Fase 1 — Audit bug (langkah "review bug" yang diminta)

Jalankan [rencana audit](07-bug-register.md#rencana-audit). Output: bug register terupdate
dengan status terkonfirmasi + PR perbaikan untuk temuan prioritas tinggi. Fokus utama:
keamanan rules, lalu integrasi (INT-1/2/3).

## Fase 2 — Kurangi risiko integrasi

- **Satukan jalur update prospek** (pilih HTTP **atau** client SDK, bukan keduanya).
- **Tipe/skema bersama**: paket atau file `shared-schema` dijaga sinkron antar app, plus
  checklist "ubah skema → update kedua app + database-design + rules + seed".
- **Satukan logika transaksi titipan** atau dokumentasikan kontraknya secara eksplisit.

## Fase 3 — Fitur lapangan (mobile)

Mengikuti `venovian-toys-mobile/docs/mobile-roadmap.md`:

- **Cetak termal** struk titipan/tagihan (printer Bluetooth portable) — butuh Expo dev build.
  Strategi: `mobile-printing-strategy.md`.
- **Draft offline** + finalisasi invoice saat online (hemat baca Spark).
- **Hardening** kuota Spark: audit pola baca, batasi `onSnapshot` yang mahal.

## Fase 4 — Penguatan platform

- **Firestore index**: pastikan semua query list punya index komposit (doc 05).
- **Observability murah**: logging error client mobile (mis. ke koleksi `auditLogs` atau
  layanan gratis) — saat ini error transaksi di device sulit terlihat.
- **Backup data**: jadwalkan export Firestore (manual/gcloud) — Spark tak punya backup otomatis.

## Yang sengaja TIDAK dikerjakan (agar fokus)

Per scope yang ditetapkan AGENTS.md, **jangan** tambah tanpa permintaan eksplisit:
self-registration, social login, password reset, MFA, role-management UI. Tetap di
email/password Firebase. Jangan migrasi UI framework (tetap Tailwind+shadcn / RN custom).

## Cara mengusulkan perubahan roadmap

Tambah item ke fase yang sesuai dengan satu baris alasan. Jika item adalah bug, daftarkan
dulu di [07 — Bug Register](07-bug-register.md) lalu tautkan dari sini.

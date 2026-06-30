# 06 — Kontrak Integrasi

Bagaimana backoffice dan mobile saling terhubung. Ini bagian paling rawan dan paling
kurang terdokumentasi dari sistem — baca sebelum mengubah skema atau menambah fitur
lintas-app.

## Model integrasi: Firestore bersama

Tidak ada API REST/GraphQL umum antara kedua app. Integrasi terjadi melalui **Firestore
yang sama**:

```
Backoffice  ──tulis/baca──►  ┌───────────┐  ◄──tulis/baca──  Mobile
(Admin SDK)                  │ Firestore │                   (Client SDK)
                             └───────────┘
                                   ▲
                                   │ satu pengecualian:
                          PATCH /api/mobile/prospects/[id]/visit
                          (mobile → backoffice → Firestore via Admin)
```

Konsekuensi: **kontrak antar app = bentuk dokumen Firestore + Security Rules**. Tidak ada
versi API, tidak ada tipe bersama. Jika satu app mengubah bentuk dokumen, app lain bisa
rusak diam-diam.

## Satu-satunya endpoint HTTP: update kunjungan prospek

`PATCH /api/mobile/prospects/[id]/visit` (backoffice, App Router route handler).

**Auth:** header `Authorization: Bearer <Firebase ID token>`. Backoffice memverifikasi
token via `adminAuth.verifyIdToken(idToken, true)` (cek revoked). Tanpa Bearer → 401.

**Request body:**
```json
{
  "visitStatus": "belum_dikunjungi" | "sudah_dikunjungi",
  "visitNotes": "string (opsional)"
}
```

**Efek:** update `scanProspects/{id}`:
```
visitStatus, visitNotes,
visitedAt      = serverTimestamp() bila "sudah_dikunjungi", else null
visitUpdatedAt = serverTimestamp()
updatedAt      = serverTimestamp()
updatedBy      = uid (dari token)
```

**Response:** `{ "success": true }` (200) · error `{ "message": "..." }` dengan 400/401/500.

### ⚠️ Inkonsistensi yang harus diklarifikasi

Endpoint di atas memakai **Admin SDK** untuk update `scanProspects`. Namun service mobile
`src/services/firebase/prospects.ts` juga punya `updateDoc(...)` **langsung ke Firestore
via client SDK** untuk update prospek. Artinya ada **dua jalur** untuk operasi yang mirip:

1. Jalur HTTP (Admin SDK, ada `updatedBy`, cek token revoked).
2. Jalur langsung client SDK (tanpa server, bergantung Security Rules).

**TERJAWAB (2026-06-30):** mobile memakai **jalur (2) — `updateDoc` client SDK langsung**
(`src/services/firebase/prospects.ts:76`). Grep seluruh kode mobile: **0 referensi** ke
`EXPO_PUBLIC_API_URL`/`/api/mobile/.../visit`. Jadi endpoint backoffice (1) adalah **dead code**,
dan `actorUid` dikirim dari client (tak terjamin = pelaku asli). Lihat
[T-41](10-task-tracker.md#t-41--p1--visit-prospek-lewat-client-sdk-endpoint-http-backoffice-dead-code-)
untuk keputusan & perbaikan. Tercatat semula sebagai [BUG #INT-1](07-bug-register.md#int-1).

## Risiko drift skema (tidak ada tipe bersama)

Tipe didefinisikan **terpisah** di tiap app. Contoh `Customer`:

| Field | Backoffice | Mobile (`customers.ts`) |
|-------|-----------|--------------------------|
| status | (lihat database-design) | `'active' \| 'follow_up' \| 'inactive'` |

Tidak ada paket/sumber tipe bersama. Bila backoffice menambah field wajib atau mengubah
enum, mobile tidak tahu sampai runtime. **Rekomendasi** (lihat [08 — Roadmap](08-roadmap.md)):
buat satu paket tipe bersama atau minimal satu file `shared-schema` yang disalin & dijaga
sinkron, plus checklist "ubah skema → update kedua app".

## Logika bisnis terduplikasi

Mutasi stok titipan (`runTransaction`) ada **di kedua app**:
- Backoffice: server actions + service Firestore (Admin).
- Mobile: `consignment-notes.ts`, `customer-stocks.ts` (client SDK, di device).

Risiko: aturan konsistensi stok (mis. cara hitung `quantityRemaining`, update
`stockMovements`) bisa berbeda antar implementasi. **[VERIFIKASI]** bandingkan kedua
implementasi transaksi titipan agar identik. Tercatat [BUG #INT-2](07-bug-register.md#int-2).

## Aturan emas saat mengubah sistem

1. **Ubah bentuk dokumen Firestore?** Update **kedua** app + `database-design.md` +
   Security Rules + seed.
2. **Tambah query list?** Pastikan index komposit ada (lihat doc 05).
3. **Tambah operasi tulis di mobile?** Pastikan Security Rules mengizinkan **hanya** yang
   seharusnya — mobile berjalan di device yang tak tepercaya.
4. **Tambah field hasil mutasi server (mis. `updatedBy`)?** Pastikan jalur yang dipakai
   benar-benar mengisinya; jangan tinggalkan dua jalur dengan hasil berbeda.

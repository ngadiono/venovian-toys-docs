# 09 — Deployment & Environment

## Ikhtisar

| Komponen | Platform | Tier |
|----------|----------|------|
| Backoffice (web) | Vercel | Free / Hobby |
| Firestore + Auth | Firebase | Spark (gratis, tanpa Cloud Functions) |
| Mobile build | EAS Build (Expo) | distribusi internal (APK) |

## Backoffice (Vercel)

- Node: lihat `.nvmrc` → **Node 22** (package.json izinkan `^20.19 || ^22.13 || >=24`).
- Build: `next build`. Dev lokal: `next dev` (jalankan sendiri; agent tidak menjalankan).
- Proteksi route via `proxy.ts` (Next 16) + verifikasi sesi server-side.

### Environment variables (backoffice)

Client (publik, prefix `NEXT_PUBLIC_`):
```
NEXT_PUBLIC_FIREBASE_API_KEY
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN
NEXT_PUBLIC_FIREBASE_PROJECT_ID
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID
NEXT_PUBLIC_FIREBASE_APP_ID
```
Server (RAHASIA — Admin SDK & lainnya):
```
FIREBASE_PROJECT_ID
FIREBASE_CLIENT_EMAIL
FIREBASE_PRIVATE_KEY                       # jaga newline; jangan commit
FIREBASE_AUTH_SESSION_COOKIE_NAME=vt_session
FIREBASE_AUTH_SESSION_EXPIRES_IN_DAYS=5
FIREBASE_SEED_ALLOWED_PROJECT_IDS=venovian-toys-dev   # pengaman: seed/clear hanya untuk project ini
GEOAPIFY_API_KEY                           # untuk scan area
```

> `FIREBASE_SEED_ALLOWED_PROJECT_IDS` adalah pengaman penting: `clear-data.js` menolak
> menghapus bila project id tidak terdaftar (`assertProjectCanBeCleared`). **Jangan**
> masukkan project produksi ke daftar ini.
>
> ⚠️ Di Vercel, simpan semua var server sebagai **Encrypted Environment Variable**.
> Pastikan `.env.local` ada di `.gitignore` ([SEC-2](07-bug-register.md#keamanan)).

## Mobile (Expo / EAS)

- `app.json`: name "Venovian Toys", slug `venovian-toys-mobile`, bundle
  `com.venovian.toysmobile`, scheme `venoviantoys`. EAS projectId
  `3166a4d7-3242-4af7-b5b8-69b9d8d68849`, owner `ngadiono7s-organization`.
- `eas.json` profil: `development` (dev client, internal), `preview` (APK Android, internal),
  `production` (autoIncrement, appVersionSource remote).
- Eksperimen aktif: `typedRoutes`, `reactCompiler`.

### Environment variables (mobile)

```
EXPO_PUBLIC_FIREBASE_API_KEY
EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN
EXPO_PUBLIC_FIREBASE_PROJECT_ID
EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET
EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID
EXPO_PUBLIC_FIREBASE_APP_ID
EXPO_PUBLIC_API_URL                        # base URL backoffice (untuk endpoint /api/mobile/...)
```

> Semua var mobile berprefix `EXPO_PUBLIC_` → **ter-bundle ke aplikasi & dapat dibaca user**.
> Jangan pernah taruh secret server (private key) di sini. Itu sebabnya operasi yang butuh
> hak istimewa (mis. update prospek dgn `updatedBy`) lewat endpoint backoffice, bukan
> langsung. Untuk cetak termal butuh **development build** (bukan Expo Go).

## Build & rilis (ringkas)

| Aksi | Perintah |
|------|----------|
| Dev mobile | `npm start` (`expo start`) |
| Build preview APK | `eas build --profile preview --platform android` |
| Build produksi | `eas build --profile production` |
| Seed Firestore (dev) | `npm run seed` (di backoffice; hanya project di allowlist) |
| Clear Firestore (dev) | `npm run clear:db` (di backoffice; diproteksi allowlist) |

## Daftar setup Firebase Console (acuan)

Untuk fitur baru yang menyentuh Firebase, AGENTS.md mewajibkan menyertakan bagian "Setup
Firebase": collection id, strategi document id, field + tipe + contoh nilai, field nullable,
catatan soft-delete/enum, dan index komposit yang diperlukan. Acuan lengkap koleksi:
`venovian-toys/docs/database-design.md`.

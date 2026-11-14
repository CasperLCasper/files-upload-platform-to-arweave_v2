# PermRepo Restore — Arweave upload platform

Platforma augšupielādē Restore vietnes failus Turbo/Arweave un izveido vienu `arweave/paths` manifestu. Atbalstīta gan atsevišķu failu, gan visas būves mapes augšupielāde, saglabājot apakšmapju struktūru.

## Prasības

- Node.js 20 vai jaunāks (ieteicams LTS)
- npm
- Turbo kontam pieejama Ethereum maka privātā atslēga un finansējums

## Instalēšana

```bash
npm install
```

## Vides mainīgie

Iestatiet tos hostinga platformas Environment sadaļā. Privāto atslēgu nekad neievietojiet `public/` mapē vai Git repozitorijā.

| Mainīgais | Obligāts | Noklusējums | Nozīme |
|---|---:|---|---|
| `RESTORE_PAGE_PRIVATE_KEY` | Jā | — | Maka privātā atslēga, ko izmanto Turbo autentifikācijai |
| `TURBO_TOKEN` | Nē | `base-eth` | Turbo maksājuma tokens |
| `TURBO_UPLOAD_URL` | Nē | `https://upload.services.ar-io.dev` | Turbo augšupielādes serviss |
| `TURBO_PAYMENT_URL` | Nē | `https://payment.services.ar-io.dev` | Turbo maksājumu serviss |
| `GATEWAY_URL` | Nē | `https://sepolia.base.org` | SDK konfigurācijai nodotais gateway URL (saglabāts no sākotnējās platformas) |
| `PORT` | Nē | `3001` | HTTP porta numurs |

`GATEWAY_URL` noklusējums ir saglabāts no sākotnējā projekta. Pirms konfigurācijas maiņas pārbaudiet sava Turbo/Arweave devnet pakalpojuma prasības.

## Palaišana

```bash
npm start
```

Atveriet `/upload-restore.html`. `/` automātiski pāradresē uz augšupielādes lapu. Veselības pārbaude pieejama `/api/health`.

## Mapju augšupielāde

1. Restore projekta datorā palaidiet tā būvēšanu, lai izveidotos `dist/` mape.
2. Augšupielādes lapā nospiediet **Izvēlēties mapi** un izvēlieties `dist` mapi. Alternatīvi ievelciet `dist` mapi lapā.
3. Platforma parāda katra faila relatīvo ceļu, piemēram, `restore.html` un `vendor/ethers.umd.min.js`.
4. Nospiediet **Augšupielādēt uz Arweave**.
5. Serveris katru failu augšupielādē atsevišķā DataItem un izveido kopīgu `arweave/paths` manifestu ar ceļu karti.

Izvēlētās `dist` mapes nosaukums netiek pievienots manifestā priekšā: `dist/vendor/a.js` kļūst par `vendor/a.js`. Apakšmapes tiek saglabātas. Ja vienlaikus izvēlas atsevišķus failus, to ceļi ir paši failu nosaukumi.

Manifestā `restore.html` tiek izvēlēts kā sākumlapa, ja tas atrodas saknē; citādi prioritāte ir `index.html`, pēc tam pirmais atrastais HTML fails.

## Limiti

- Maksimums 500 faili vienā manifestā.
- Maksimums 10 MiB katram failam.
- Maksimums 50 MiB visu failu kopējam dekodētajam izmēram.
- HTTP JSON pieprasījuma limits 100 MiB.

Augšupielāde notiek secīgi. Ja kāds fails neizdodas, API atgriež kļūdu un jau augšupielādēto failu TX ID sarakstu (`partial`); šie DataItem netiek automātiski dzēsti.

## API

- `POST /api/upload-bundle` — `{ "files": [{ "name": "vendor/library.js", "content": "<base64>" }] }`; atgriež manifest TX ID un failu TX ID karti.
- `POST /api/upload-restore` — sākotnējā viena faila augšupielādes saderības API.
- `GET /api/health` — servera konfigurācijas stāvoklis un limiti (privātā atslēga netiek atgriezta).

## Drošība

- Privātā atslēga tiek nolasīta tikai no servera vides mainīgā.
- Failu ceļi tiek validēti kā relatīvi POSIX ceļi; absolūtie ceļi, `..`, `.` segmenti, atpakaļsvītras un vadības simboli tiek noraidīti.
- Failu nosaukumi tiek parādīti ar `textContent`, nevis ievietoti kā HTML.
- Manifestā katram ceļam jābūt unikālam.

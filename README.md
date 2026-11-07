# Google Ads reporting

Návody a skripty na získavanie dát z Google Ads, cez **Google Ads API** aj cez **Google Ads Scripts**, a ich reportovanie (Google Sheets, Looker Studio).

## Obsah

| # | Dokument | O čom |
|---|---|---|
| 1 | [Pripojenie cez Google Ads API](docs/01-pripojenie-cez-api.md) | Developer token, OAuth, refresh token, prvý dotaz v Pythone aj cez REST |
| 2 | [Alternatívy k Whatagraph](docs/02-alternativy-whatagraph.md) | Bezplatné a platené nástroje na reporting, odporúčania |
| 3 | [Google Ads Script: export do Sheets](docs/03-google-ads-script.md) | Hotový skript s denným exportom kampaní a vyhľadávacích dopytov a postup nasadenia |
| 4 | [Kde si používateľ nastavuje reporty](docs/04-nastavenie-reportov.md) | Konfigurácia v kóde, v Google Sheete, v Looker Studiu alebo v Google Ads |

Samotný skript na skopírovanie: [`scripts/google-ads-export.js`](scripts/google-ads-export.js)

## Rýchly štart (bez programovania)

1. Vytvorte prázdny Google Sheet a skopírujte jeho URL.
2. V Google Ads otvorte **Nástroje → Hromadné akcie → Skripty → +**.
3. Vložte obsah [`scripts/google-ads-export.js`](scripts/google-ads-export.js) a URL tabuľky doplňte do `CONFIG.SPREADSHEET_URL`.
4. Kliknite na **Autorizovať**, potom na **Náhľad** a nakoniec na **Spustiť**. Frekvenciu nastavte na **Denne**.
5. Tabuľku pripojte ako zdroj dát v [Looker Studiu](https://lookerstudio.google.com) a postavte nad ňou report.

## Štruktúra repozitára

```
.
├── README.md
├── docs/
│   ├── 01-pripojenie-cez-api.md
│   ├── 02-alternativy-whatagraph.md
│   ├── 03-google-ads-script.md
│   └── 04-nastavenie-reportov.md
└── scripts/
    └── google-ads-export.js
```

## Upozornenie

Do repozitára nikdy necommitujte `google-ads.yaml`, developer token, client secret ani refresh token. Súbor `.gitignore` ich už vylučuje.

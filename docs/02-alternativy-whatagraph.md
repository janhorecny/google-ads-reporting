# Alternatívy k Whatagraph

## Zadarmo

- **[Looker Studio](https://lookerstudio.google.com)** je najlepšia bezplatná voľba. Má natívny konektor na Google Ads, GA4, Search Console a YouTube bez ďalších poplatkov. Viete v ňom urobiť šablóny reportov pre klientov, zdieľanie cez link a automatické posielanie PDF mailom. Slabinou sú Meta, LinkedIn a TikTok, na ktoré treba platený konektor (Supermetrics, Windsor.ai, Porter Metrics) alebo vlastný export do Google Sheets či BigQuery.
- **Google Ads Scripts + Google Sheets** sú zadarmo. Skript priamo v Google Ads každý deň zapíše dáta do Sheetu a nad ním môže bežať Looker Studio. Hotový skript je v [03-google-ads-script.md](03-google-ads-script.md).
- **Vlastný dashboard cez API** (pozri [01-pripojenie-cez-api.md](01-pripojenie-cez-api.md)). Dáta sa ťahajú do DB (napr. Supabase alebo Postgres) a zobrazujú v Metabase alebo Grafane (open-source, self-hosted) alebo vo vlastnej webovej aplikácii. Je to viac práce, ale bez mesačných poplatkov a s plnou kontrolou.

## Platené, lacnejšie alebo podobné ako Whatagraph

| Nástroj | Pre koho |
|---|---|
| **Databox** | Má obmedzený free plán, dobrý na KPI dashboardy |
| **DashThis** | Jednoduché agentúrne reporty, platí sa podľa počtu dashboardov |
| **AgencyAnalytics** | Agentúry, white-label, klientsky portál |
| **Swydo** | Lacnejší agentúrny reporting, PPC zameranie |
| **Reporting Ninja** | Lacná alternatíva, white-label |
| **Windsor.ai / Porter Metrics** | Nie sú to dashboardy, ale lacné konektory (Meta, LinkedIn…) do Looker Studia |

Ceny a free plány sa často menia, preto si ich pred rozhodnutím overte na webe.

## Odporúčanie

- **Len Google Ads (+ GA4):** Looker Studio, 0 €.
- **Viac platforiem (Meta, LinkedIn…):** Looker Studio s lacným konektorom (Windsor.ai alebo Porter). Vyjde to výrazne lacnejšie ako Whatagraph.
- **Veľa klientov a potreba white-label portálu:** AgencyAnalytics alebo DashThis.

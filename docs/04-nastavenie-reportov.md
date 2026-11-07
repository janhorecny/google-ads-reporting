# Kde si používateľ nastavuje, aké dáta potrebuje

V aktuálnom skripte ([03-google-ads-script.md](03-google-ads-script.md)) sa to nastavuje **priamo v kóde**, v poli `reports` (GAQL dotaz, stĺpce, názov hárku) a v `CONFIG.DAYS`. Na bežného používateľa je to nepraktické. Možnosti sú tri:

## 1. Nastavenia v Google Sheete (pre netechnických používateľov)

Skript si konfiguráciu prečíta z hárku **„Nastavenia“** v tej istej tabuľke. Používateľ len zaškrtáva a vyberá:

| Report | Zapnutý | Obdobie (dni) | Úroveň | Metriky |
|---|---|---|---|---|
| Kampane | ✅ | 90 | deň | Impresie, Kliky, Náklady, Konverzie, ROAS |
| Vyhľadávacie dopyty | ✅ | 30 | spolu | Kliky, Náklady, Konverzie |
| Kľúčové slová | ☐ | 30 | spolu | … |
| Reklamné skupiny | ☐ | 90 | deň | … |

Výber metrík je z rozbaľovacieho zoznamu (data validation). Skript si podľa toho poskladá GAQL dotaz sám a do kódu už nikto nesiahne.

> Táto verzia skriptu zatiaľ nie je v repozitári.

## 2. Skript exportuje všetko a výber sa robí v Looker Studiu

Skript ťahá širokú sadu dát a používateľ si v **Looker Studiu** sám skladá tabuľky, grafy, filtre a obdobia. Na reporty pre klientov je to najflexibilnejšie a v skripte netreba nič meniť.

## 3. Bez skriptu, cez report editor v Google Ads

Google Ads má vlastný report editor (**Kampane → Prehľady → Prehľady**). Používateľ si tam naklikne stĺpce a vie nastaviť **pravidelné posielanie** CSV alebo Sheetu mailom. Je to najjednoduchšie, ale obmedzené na jeden účet a bez prepočtov a spájania dát.

## Odporúčanie

Kombinácia 1 a 2: hárok „Nastavenia“ určí, *čo* sa sťahuje (reporty a obdobie), a Looker Studio určí, *ako* sa to zobrazí.

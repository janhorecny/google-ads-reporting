# Google Ads Script: denný export do Google Sheets

Skript zapíše do Google Sheetu dva hárky. Samostatný súbor na skopírovanie: [`../scripts/google-ads-export.js`](../scripts/google-ads-export.js)

Hárky:

- **Kampane**: denné dáta za posledných 90 dní (impresie, kliky, CTR, CPC, náklady, konverzie, CPA, ROAS)
- **Vyhľadávacie dopyty**: top 5000 dopytov za rovnaké obdobie, zoradené podľa nákladov

Pri každom spustení prepíše celé obdobie, takže nevznikajú duplicity a dopočítajú sa aj oneskorené konverzie. Nad tabuľkou môže bežať Looker Studio.

## Skript

```javascript
/**
 * Google Ads → Google Sheets export
 * Google Ads → Nástroje → Hromadné akcie → Skripty → +
 */

var CONFIG = {
  // Vytvorte prázdny Google Sheet a vložte sem jeho URL.
  // Ak necháte prázdne, skript vytvorí novú tabuľku a URL vypíše do logu.
  SPREADSHEET_URL: '',
  DAYS: 90 // koľko dní dozadu exportovať
};

function main() {
  var ss = getSpreadsheet_();
  var tz = AdsApp.currentAccount().getTimeZone();

  var end = new Date();
  end.setDate(end.getDate() - 1); // do včera (dnešok nie je kompletný)
  var start = new Date();
  start.setDate(start.getDate() - CONFIG.DAYS);

  var from = Utilities.formatDate(start, tz, 'yyyy-MM-dd');
  var to = Utilities.formatDate(end, tz, 'yyyy-MM-dd');
  var dateFilter = "segments.date BETWEEN '" + from + "' AND '" + to + "'";

  var METRIC_HEADERS = ['Impresie', 'Kliky', 'CTR', 'Priem. CPC', 'Náklady',
                        'Konverzie', 'Hodnota konv.', 'CPA', 'ROAS'];
  var METRIC_FIELDS = 'metrics.impressions, metrics.clicks, metrics.cost_micros, ' +
                      'metrics.conversions, metrics.conversions_value';

  var reports = [
    {
      sheet: 'Kampane',
      headers: ['Dátum', 'Kampaň', 'ID kampane', 'Typ', 'Stav'].concat(METRIC_HEADERS),
      query:
        'SELECT segments.date, campaign.name, campaign.id, ' +
        'campaign.advertising_channel_type, campaign.status, ' + METRIC_FIELDS +
        ' FROM campaign WHERE ' + dateFilter + ' AND metrics.impressions > 0' +
        ' ORDER BY segments.date DESC',
      map: function (r) {
        return [r.segments.date, r.campaign.name, r.campaign.id,
                r.campaign.advertisingChannelType, r.campaign.status]
               .concat(metrics_(r.metrics));
      }
    },
    {
      sheet: 'Vyhľadávacie dopyty',
      headers: ['Dopyt', 'Kampaň', 'Reklamná skupina'].concat(METRIC_HEADERS),
      query:
        'SELECT search_term_view.search_term, campaign.name, ad_group.name, ' + METRIC_FIELDS +
        ' FROM search_term_view WHERE ' + dateFilter +
        ' ORDER BY metrics.cost_micros DESC LIMIT 5000',
      map: function (r) {
        return [r.searchTermView.searchTerm, r.campaign.name, r.adGroup.name]
               .concat(metrics_(r.metrics));
      }
    }
  ];

  reports.forEach(function (report) {
    writeSheet_(ss, report);
  });

  removeDefaultSheet_(ss);
  Logger.log('Hotovo (' + from + ' – ' + to + '): ' + ss.getUrl());
}

/** Prepočet metrík; chýbajúce polia (nulové hodnoty) → 0 */
function metrics_(m) {
  var impr = Number(m.impressions) || 0;
  var clicks = Number(m.clicks) || 0;
  var cost = (Number(m.costMicros) || 0) / 1e6;
  var conv = Number(m.conversions) || 0;
  var value = Number(m.conversionsValue) || 0;

  return [
    impr,
    clicks,
    impr ? clicks / impr : 0,            // CTR (formátované ako %)
    clicks ? round_(cost / clicks) : 0,  // priem. CPC
    round_(cost),
    round_(conv),
    round_(value),
    conv ? round_(cost / conv) : '',     // CPA
    cost ? round_(value / cost) : ''     // ROAS
  ];
}

function writeSheet_(ss, report) {
  var rows = [];
  var it = AdsApp.search(report.query);
  while (it.hasNext()) {
    rows.push(report.map(it.next()));
  }

  var sh = ss.getSheetByName(report.sheet) || ss.insertSheet(report.sheet);
  sh.clearContents();

  var cols = report.headers.length;
  var data = [report.headers].concat(rows);
  sh.getRange(1, 1, data.length, cols).setValues(data);
  sh.getRange(1, 1, 1, cols).setFontWeight('bold');
  sh.setFrozenRows(1);

  var ctrCol = report.headers.indexOf('CTR') + 1;
  if (rows.length && ctrCol > 0) {
    sh.getRange(2, ctrCol, rows.length, 1).setNumberFormat('0.00%');
  }

  Logger.log(report.sheet + ': ' + rows.length + ' riadkov');
}

function getSpreadsheet_() {
  if (CONFIG.SPREADSHEET_URL) {
    return SpreadsheetApp.openByUrl(CONFIG.SPREADSHEET_URL);
  }
  var ss = SpreadsheetApp.create('Google Ads export – ' + AdsApp.currentAccount().getName());
  Logger.log('Vytvorená nová tabuľka: ' + ss.getUrl() +
             ' → vložte túto URL do CONFIG.SPREADSHEET_URL!');
  return ss;
}

function removeDefaultSheet_(ss) {
  ['Sheet1', 'Hárok1', 'List1'].forEach(function (name) {
    var sh = ss.getSheetByName(name);
    if (sh && ss.getSheets().length > 1) ss.deleteSheet(sh);
  });
}

function round_(n) {
  return Math.round(n * 100) / 100;
}
```

## Postup nasadenia

1. Vytvorte prázdny **Google Sheet** a skopírujte jeho URL do `SPREADSHEET_URL`. Ak URL nevyplníte, každé spustenie založí novú tabuľku.
2. V Google Ads otvorte **Nástroje → Hromadné akcie → Skripty → modré +**.
3. Vložte skript, kliknite na **Autorizovať** a potom na **Náhľad**. V logu uvidíte počet riadkov za každý hárok.
4. Ak náhľad prebehne v poriadku, kliknite na **Spustiť**.
5. Pri skripte nastavte **Frekvenciu: Denne**, ideálne skoro ráno, napr. o 6:00.

## Úpravy

- **Iné obdobie:** zmeňte `DAYS`.
- **Ďalší report** (napr. kľúčové slová alebo reklamné skupiny): pridajte ďalší objekt do poľa `reports` s vlastným GAQL dotazom.
- **Viac klientskych účtov naraz:** treba MCC verziu (`AdsManagerApp.accounts()`), ktorá pridá stĺpec s názvom účtu.

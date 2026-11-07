# Pripojenie na Google Ads cez API

## 1. Developer token

1. Potrebujete **manažérsky účet (MCC)**. Ak ho nemáte, založíte ho na <https://ads.google.com/home/tools/manager-accounts/>.
2. V MCC otvorte **Admin → API Center**, vyplňte formulár a dostanete **developer token**.
3. Nový token má **Test access** a funguje len s testovacími účtami. Pre reálne účty treba v API Center požiadať o vyšší prístup (Basic). Schválenie trvá zvyčajne pár dní.

## 2. Google Cloud projekt a OAuth

1. Na <https://console.cloud.google.com> vytvorte projekt a zapnite **Google Ads API**.
2. Nastavte **OAuth consent screen**. Pre interné použitie stačí typ External so sebou ako test userom.
3. V Credentials vytvorte **OAuth client ID** typu *Desktop app* a uložte si `client_id` a `client_secret`.

## 3. Refresh token

Najjednoduchšie je použiť skript z oficiálnej knižnice:

```bash
pip install google-ads
# stiahnite z GitHubu google-ads-python: examples/authentication/generate_user_credentials.py
python generate_user_credentials.py --client_secrets_path=client_secret.json
```

Prihláste sa Google účtom, ktorý má prístup k Ads účtu, a dostanete `refresh_token`. Scope je `https://www.googleapis.com/auth/adwords`.

Na serverovú automatizáciu bez prihlasovania sa dá použiť aj **service account**. Jeho e-mail treba pridať ako používateľa v Google Ads účte.

## 4. Konfigurácia `google-ads.yaml`

```yaml
developer_token: XXXXXXXXXXXX
client_id: xxxx.apps.googleusercontent.com
client_secret: xxxx
refresh_token: 1//xxxx
login_customer_id: 1234567890   # ID MCC bez pomlčiek
use_proto_plus: True
```

> Tento súbor obsahuje tajné údaje. Nikdy ho necommitujte do gitu.

## 5. Test

```python
from google.ads.googleads.client import GoogleAdsClient

client = GoogleAdsClient.load_from_storage("google-ads.yaml")
ga = client.get_service("GoogleAdsService")

query = """
  SELECT campaign.id, campaign.name, metrics.clicks, metrics.cost_micros
  FROM campaign
  WHERE segments.date DURING LAST_30_DAYS
"""
for batch in ga.search_stream(customer_id="9876543210", query=query):
    for row in batch.results:
        print(row.campaign.name, row.metrics.clicks, row.metrics.cost_micros / 1e6)
```

## Na čo si dať pozor

- **ID účtov** píšte bez pomlčiek (`123-456-7890` sa zadá ako `1234567890`).
- **`login_customer_id`** je ID MCC, cez ktorý pristupujete. `customer_id` v dotaze je konkrétny klientsky účet.
- **Test token** na reálnom účte vráti chybu `DEVELOPER_TOKEN_NOT_APPROVED`.
- Na dotazy sa používa **GAQL**. Polia a reporty si viete poskladať v Query Builderi: <https://developers.google.com/google-ads/api/fields>.
- Ak nechcete Python, existujú oficiálne knižnice aj pre PHP, Java, .NET, Ruby a Perl a komunitná knižnica pre **Node.js** (`google-ads-api`). Dá sa použiť aj priamy **REST**:
  `POST https://googleads.googleapis.com/vXX/customers/{id}/googleAds:search` s hlavičkami `Authorization`, `developer-token` a `login-customer-id`. Za `vXX` doplňte aktuálnu verziu API.

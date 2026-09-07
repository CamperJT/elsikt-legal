# elsikt-legal

Statisk GitHub Pages-sajt för elsikt.com: personvernerklæring (norsk bokmål) och en kort
landningssida. Byggd som förlaga från `CamperJT/wattitutka-legal` (struktur), anpassad till
Elsikts namn, färger och (verifierade mot koden i `WattNordic/`) faktiska databehandling.

## Filer

- `CNAME` — `elsikt.com` (custom domain för GitHub Pages)
- `index.html` — landningssida
- `personvern.html` — personvernerklæring, nb-NO
- `integritetspolicy.html` — integritetspolicy, sv-SE

## DNS-poster hos registraren

Källa för GitHub Pages IP:n och CNAME-mönster: GitHub Docs, "Managing a custom domain for your
GitHub Pages site":
<https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site>
(hämtad 7.9.2026).

| Typ | Namn/host | Värde | Kommentar |
|---|---|---|---|
| A | `@` (elsikt.com) | `185.199.108.153` | GitHub Pages apex-IP 1/4 |
| A | `@` (elsikt.com) | `185.199.109.153` | GitHub Pages apex-IP 2/4 |
| A | `@` (elsikt.com) | `185.199.110.153` | GitHub Pages apex-IP 3/4 |
| A | `@` (elsikt.com) | `185.199.111.153` | GitHub Pages apex-IP 4/4 |
| AAAA | `@` (elsikt.com) | `2606:50c0:8000::153` | GitHub Pages apex-IPv6 1/4 |
| AAAA | `@` (elsikt.com) | `2606:50c0:8001::153` | GitHub Pages apex-IPv6 2/4 |
| AAAA | `@` (elsikt.com) | `2606:50c0:8002::153` | GitHub Pages apex-IPv6 3/4 |
| AAAA | `@` (elsikt.com) | `2606:50c0:8003::153` | GitHub Pages apex-IPv6 4/4 |
| CNAME | `www` | `camperjt.github.io` | Pekar mot GitHub Pages-defaultdomänen för detta repo |
| A | `api` (api.elsikt.com) | `PLACEHOLDER — Hetzner-serverns publika IP` | Backend (FastAPI), sätts när servern är provisionerad — se `docs/lanseringsplan-2026-09-03.md` i WattNordic |

Lägg alla fyra A-poster för apex (`@`/`elsikt.com`) — inte bara en. Ta bort ev. befintlig
parkerings-CNAME/A-post på `@` och `www` innan dessa läggs in; en apex-domän kan inte ha en CNAME
samtidigt som andra poster (ALIAS/ANAME-poster hos vissa registrarer är ett alternativ till de
fyra A-posterna om registraren stödjer det).

## Skapa repot och slå på Pages

```bash
# Från denna mapp, efter granskning och godkännande:
gh repo create CamperJT/elsikt-legal --public --source=. --remote=origin --push

# Slå på GitHub Pages med custom domain (branch = main, root):
gh api -X POST repos/CamperJT/elsikt-legal/pages \
  -f "source[branch]=main" -f "source[path]=/"

# Sätt custom domain (skriver/verifierar CNAME-filen, GitHub validerar DNS):
gh api -X PUT repos/CamperJT/elsikt-legal/pages \
  -f "cname=elsikt.com"

# När DNS har propagerat och GitHub har verifierat domänen, slå på enforce HTTPS:
gh api -X PUT repos/CamperJT/elsikt-legal/pages \
  -F "https_enforced=true"
```

Om `gh api ... /pages` PUT för `https_enforced` inte accepteras direkt (GitHub kräver ibland att
domänverifieringen är klar och certifikatet utfärdat först — kan ta upp till 24 h efter DNS): kör
om samma kommando senare, eller slå på "Enforce HTTPS" manuellt under
**Settings → Pages** i repot.

## Innan lansering: databehandleravtal med Hetzner

`personvern.html`/`integritetspolicy.html` beskriver Hetzner Online GmbH som databehandlare
(personuppgiftsbiträde) för API-serverns drift. Det formella databehandleravtalet (DPA) tecknas
inte av detta repo — ägaren måste själv acceptera/underteckna det i Hetzner-konsolen (Robot/Cloud
Console → Data Processing Agreement) innan produktionsdrift med riktiga användardata påbörjas.

## Verifiering efter deploy

- `nslookup elsikt.com` (Windows) → ska ge de fyra GitHub-IP:erna ovan.
- `curl -sI https://elsikt.com/` → 200, och `https://elsikt.com/personvern.html` → 200.
- Kontrollera i repots **Settings → Pages** att domänen visas som verifierad (grön bock), inte
  "improperly configured domain".

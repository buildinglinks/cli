# Buildinglinks CLI

`bl` draait de Buildinglinks-dataspace als complete simulatie op je eigen machine. Je krijgt een
Trust Authority en zeven organisaties, elk met een eigen connector, data plane, inlogomgeving
(Keycloak-realm) en waar dat past een gesimuleerd gebouwbeheersysteem of een gesimuleerde app.
De organisaties worden lid, bieden de data van negentien gebouwen aan, sluiten overeenkomsten
en wisselen data uit, zoals dat in de echte dataspace gaat. Je kunt in elke connector inloggen
en zelf verder spelen.

Meer over de simulatie en wat je ermee kunt: https://docs.buildinglinks.nl/proberen/lokaal/.

Buildinglinks is een dataspace voor kantoorgebouwen, gebouwd op de Eclipse
Dataspace-standaarden: Dataspace Protocol (DSP) 2025-1, Decentralized Claims Protocol (DCP)
1.0 en Data Plane Signaling (DPS) 1.0.

## Wat je nodig hebt

* Docker met compose: Docker Desktop op macOS en Windows, Docker Engine op Linux.
* Ongeveer 2 GB vrij geheugen voor de simulatie. Geef Docker Desktop minstens 4 GB
  (*Settings* → *Resources*).
* Ongeveer 3 GB schijfruimte voor de images.
* Poort 80 vrij op je machine (of kies een andere poort, zie [Problemen](#problemen)).

Macs met Apple Silicon worden ondersteund: de images zijn er voor amd64 en arm64. Op Windows
is `bl` nog niet getest.

## Installeren

macOS en Linux:

```bash
curl -fsSL https://github.com/buildinglinks/cli/releases/latest/download/bl-installer.sh | sh
```

Windows (PowerShell):

```powershell
powershell -c "irm https://github.com/buildinglinks/cli/releases/latest/download/bl-installer.ps1 | iex"
```

De installer zet `bl` in `~/.local/bin` en voegt die map toe aan je `PATH` (via `~/.profile`,
op Windows in je gebruikersinstellingen).
Open daarna een nieuwe terminal en controleer:

```bash
bl --version
```

Gebruik je zsh (standaard op macOS) en vindt de terminal `bl` niet, zet dan deze regel in
`~/.zshrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Een nieuwere versie installeer je met hetzelfde commando. Je kunt de binaries ook zelf
downloaden onder [Releases](https://github.com/buildinglinks/cli/releases).

## Starten

```bash
bl sim start
```

De eerste keer haalt Docker de images op; daarna start `bl` alles en richt het de
simulatie in. Reken op vijf tot tien minuten. `bl` toont onderweg wat er gebeurt en sluit af
met een overzicht van alle adressen.

Open dan **http://localhost**. Op die startpagina kies je een rol (gebouweigenaar,
beheerder van een gebouwbeheersysteem, app-bouwer, beheerder van de dataspace of nieuwkomer)
en ga je met één klik naar de juiste plek. Bovenaan staat of alles werkt.

Inloggen kan in elke connector met *Inloggen met SSO*:

| Gebruiker | Wachtwoord | Mag |
|---|---|---|
| `admin` | `admin` | alles |
| `operator` | `operator` | aanbieden, aanvragen, goedkeuren en het request-log instellen |
| `viewer` | `viewer` | alleen kijken |

Elke deelnemer heeft een eigen adres, dus je kunt in één browser bij meerdere deelnemers
tegelijk ingelogd zijn.

## Wie er meedoet

| Organisatie | Rol | Connector |
|---|---|---|
| Buildinglinks Trust Authority | beheert de dataspace: aanmelding, register, monitoring, notaris | http://authority.localhost |
| RealEstator | rechthebbende van dertien panden, zonder eigen BMS | http://realestator.localhost |
| Sensortrust | datahouder: het BMS van die dertien panden; levert namens RealEstator | http://sensortrust.localhost |
| Gebouwbeheer Noord | rechthebbende en datahouder van zes eigen gebouwen; keurt sommige aanvragen met de hand goed | http://noord.localhost |
| Installatiebedrijf Van de Ven | datahouder van de indeling van Delftechpark 26, uit een BIM-model, namens RealEstator | http://vandeven.localhost |
| Optimaforma | dataontvanger: klimaatoptimalisatie, leest en stuurt setpoints | http://optimaforma.localhost |
| Facilitair Inzicht | dataontvanger: comfortmonitoring, alleen lezen | http://facilitair.localhost |
| Voltaflex | dataontvanger: energieflexibiliteit, stuurt batterijen | http://voltaflex.localhost |
| Nieuwe connector | vers geïnstalleerd, nog geen lid | http://nieuwkomer.localhost |

De gesimuleerde systemen achter de deelnemers hebben een eigen dashboard op
`http://<deelnemer>-backend.localhost`, bijvoorbeeld http://optimaforma-backend.localhost. Verder zijn
er Mailpit voor de e-mail van de Trust Authority (http://mail.localhost), een gesimuleerde
vertrouwensdienst voor elektronisch ondertekenen (http://qtsp.localhost) en de Digital
Signature Service van de Europese Commissie (http://dss.localhost/validation).

De logopslag staat op **http://storage.localhost**. Log in met gebruiker en wachtwoord
`admin`; op een afgeschermde demo is het wachtwoord hetzelfde als het demowachtwoord.
Open de bucket `logs`: onder `runtime/<deelnemer>/<onderdeel>/<jaar>/<maand>/<dag>/` staan
gzip-bestanden met één JSON-object per regel. Download een bestand en lees het bijvoorbeeld
met `gzip -dc bestand.jsonl.gz | jq .`. Nieuwe bestanden verschijnen ongeveer elke 10 seconden.
Voor live meekijken gebruik je `bl sim logs -f`.

De datarequests van de data planes staan apart, onder `requests/<deelnemer>/dt=…/hour=…/level=…/`.
Die hoef je niet zelf te downloaden:

```bash
bl sim requests --from 08:00 --to 12:00 --level warn
```

toont ze op tijd gesorteerd, met `--participant`, `--counterparty` en `--process` om te
filteren en `--json` voor `jq`. Het vraagt ze aan de connectors, zoals `bl logs requests` na
`bl login`; `--direct` leest de logopslag zelf.

Onder **Instellingen → Request-log** kiezen operators en beheerders welke datarequests de
data plane logt: `off`, `errors`, `warnings` of `all`. Buiten de simulatie is `warnings` de
standaard; in de simulatie zet de seed hem op `all`, zodat je ook geslaagde requests ziet.
Kies je zelf iets anders, dan laat de seed dat bij een volgende start staan. Je kunt daar ook de grens voor trage requests
instellen. Een wijziging werkt binnen 10 seconden, zonder herstart.

Logs blijven bij stoppen, starten en resetten bewaard. De bucket laat ze na zeven dagen
verlopen; alleen `bl sim stop --purge` wist ook de logopslag. Vector buffert bij een
opslagstoring maximaal 512 MiB op schijf. De Docker-logs roteren per container op drie
bestanden van 10 MiB. Bij een langdurige storing kunnen daardoor logs verloren gaan.

## Wat je kunt proberen

De startpagina (http://localhost) heeft de hele looproute, met links naar de juiste schermen.
Een paar stappen:

* **Kijk mee met de apps.** Het dashboard van Optimaforma (http://optimaforma-backend.localhost)
  leest via de dataspace de zones van zijn gebouwen en stuurt setpoints terug. Het dashboard van
  het BMS van Sensortrust (http://sensortrust-backend.localhost) toont wat er binnenkomt.
* **Vraag zelf data aan.** Log bij Facilitair Inzicht in als `operator` en open
  http://facilitair.localhost/find/realestator/realestator-rtm-01 (*Data zoeken* → *Alle data ·
  Weena 505, Rotterdam*). Kies *Alle leden van buildinglinks* en *Toegang aanvragen*. Binnen enkele
  seconden heb je een abonnement, en binnen een minuut gebruikt de app van Facilitair het gebouw.
* **Beslis wat openstaat.** Bij RealEstator → *Aanvragen* → *Te beoordelen* wachten een aanvraag van
  Facilitair Inzicht voor verdieping 2 van Delftechpark 26 en een aanbod van Sensortrust om
  Kennemerplein 8 in Haarlem te leveren. De simulatie handelt ze niet zelf af; na een afwijzing
  staan ze bij de volgende start weer klaar.
* **Zie het beleid werken.** Vraag bij een ander gebouw het product aan onder de voorwaarde voor
  klimaatoptimalisatoren. RealEstator weigert: Facilitair Inzicht heeft die rol niet. De reden
  staat onder *Aanvragen* → *Afgehandeld*.
* **Laat een datahouder leveren.** Bij RealEstator zie je onder *Gebouwen* per pand wie de data
  levert: Sensortrust, na een aanwijzing door RealEstator.
* **Bekijk het bewijs.** Elke connector legt onder *Protocol en bewijs* → *Bewijs* elke stap vast,
  met handtekeningen van beide partijen. Download een bewijspakket en controleer het zelf:
  `bl evidence verify bewijspakket.json --online` (haalt de did:webvh van de authority op).
* **Meld een nieuwe deelnemer aan.** Open http://nieuwkomer.localhost en log in met setup-code
  `SETUP-NIEUWKOMER`. Vul de aanmelding in, onderteken de e-mail in Mailpit bij de gesimuleerde
  vertrouwensdienst en keur de aanvraag goed in de Trust Authority onder *Onboarding*. Wat je
  nodig hebt om de Keycloak van de nieuwkomer te koppelen, toont `bl sim status`. Opnieuw
  beginnen: `bl sim reset --participant nieuwkomer`.
* **Schors een deelnemer.** In de Trust Authority → *Deelnemers* → Facilitair Inzicht →
  *Schorsen*. Binnen een minuut stoppen de leveringen aan Facilitair Inzicht.

## Commando's

| Commando | Wat het doet |
|---|---|
| `bl sim start` | start de simulatie en richt haar in |
| `bl sim status` | wat er draait en gezond is, de adressen en de inloggegevens |
| `bl sim stop` | stopt; de data blijft bewaard en `start` gaat verder waar je was |
| `bl sim stop --purge` | stopt en wist alle data, inclusief logs |
| `bl sim reset` | deelnemersdata wissen en opnieuw beginnen; logs blijven |
| `bl sim reset --participant nieuwkomer` | alleen de nieuwe connector terug naar een verse installatie |
| `bl sim logs [-f] [service …]` | logs, bijvoorbeeld `bl sim logs -f sensortrust-cp` |
| `bl sim requests [--from] [--to] [--level]` | de datarequests uit de logopslag, bijvoorbeeld `--from 2h --level warn` |
| `bl logs runtime --direct` | de eigen logs van de diensten uit de logopslag, als een connector niet draait |
| `bl login <url>` / `bl whoami` / `bl logout` | inloggen bij een connector via zijn UI en je IdP; de sleutel blijft op je machine |
| `bl evidence verify <pakket.json> --authority <did:webvh>` | een bewijspakket offline controleren |
| `bl connector init --target k8s\|compose …` | sleutels, geheimen en configuratie voor een eigen connector, zie [Een eigen connector](#een-eigen-connector) |
| `bl connector check <url>` | een draaiende connector controleren, van buitenaf en (na `bl login`) van binnen; elke mislukte controle noemt de volgende stap |
| `bl connector join <url> --org … --kvk … --role … --contact …` | je organisatie aanmelden bij de Trust Authority en wachten tot het lidmaatschap actief is |
| `bl api <METHODE> <pad> [--data …]` | één aanroep van de beheer-API, met de sessie van `bl login` |

Opties van `bl sim start`:

| Optie | Standaard | Betekenis |
|---|---|---|
| `--domain` | `localhost` | domein voor alle adressen: `sensortrust.<domein>`, `authority.<domein>`, … |
| `--port` | `80` | poort waarop alles bereikbaar is |
| `--project` | `bl-sim` | naam van de simulatie; zo draai je er meerdere naast elkaar |
| `--tag` | de versie van `bl` | versie van de images |
| `--https` | uit | HTTPS met certificaten van Let's Encrypt, voor een publiek domein |
| `--password` | geen | één wachtwoord voor de hele simulatie (gebruiker `demo`) |
| `--view-only` | uit | kijkdemo: alleen `viewer`/`viewer` is openbaar, `admin` en `operator` krijgen een gegenereerd wachtwoord (`bl sim status`) |
| `--static-site` | geen | `HOST=MAP`: nog een statische site op een eigen hostnaam, uit `MAP/current`, zonder wachtwoord |
| `--cast` | geen | een bezetting: andere namen en gebouwen, zie hieronder; `""` zet de gewone terug |
| `--onboarding` | `qualified` | `admin`: de Trust Authority keurt aanmeldingen goed zonder gekwalificeerde handtekening; de tekenbevoegde aanvaardt het rulebook in de connector |
| `--ui` | `buildinglinks` | het profiel van de beheeromgeving van elke connector: `buildinglinks`, `standaard` (alleen de algemene schermen) of `flexlab` |

`bl` onthoudt de opties van de vorige keer: een tweede `bl sim start` gebruikt hetzelfde
domein en dezelfde poort. Alle opties staan in `bl --help` en `bl sim <commando> --help`.

### Een andere bezetting

Met `--cast bezetting.json` spelen andere organisaties de vier hoofdrollen, en staan andere
gebouwen bij de rechthebbende. Het verhaal blijft hetzelfde; namen, adressen, realms en
dataset-id's volgen de bezetting. Zo vindt iemand zijn eigen organisatie en gebouwen terug:

```json
{
  "parties": {
    "optimaforma": {"name": "AIMZ", "signatory": "Piet-Jan de Vries"},
    "realestator": {"name": "Vastgoedfonds Noorderlicht", "slug": "noorderlicht"}
  },
  "buildings": [
    {"object": "nl.bag.pand.0344100000012848", "name": "Moreelsepark 3",
     "address": {"street": "Moreelsepark", "number": "3", "postcode": "3511EP", "city": "Utrecht"},
     "location": {"lat": 52.0886, "lon": 5.1156}, "functions": ["kantoorfunctie"]}
  ],
  "own": "optimaforma"
}
```

- `parties`: per rol (`realestator`, `sensortrust`, `optimaforma`, `vandeven`) een naam, en
  eventueel het adres (`slug`, anders uit de naam) en de ondertekenaar.
- `buildings`: gebouwen van de rechthebbende, zoals PDOK ze beschrijft. Het eerste neemt de
  plaats in van Weena 505, waar de looproute langs gaat, dan Gustav Mahlerplein 2,
  Delftechpark 26, Promenade 154, Kennemerplein 8 en de rest.
- `own`: de rol waarmee de startpagina opent.

Een andere bezetting past niet bij wat al draait: begin daarna opnieuw met `bl sim reset`. Wie
welke rol speelt, schrijft `bl sim` naar `roster.json` in de map van de simulatie; de smoketest
(`deploy/smoke.sh`) leest het daar en werkt zo met elke bezetting.

## De simulatie aan iemand anders laten zien

Met [Tailscale](https://tailscale.com) kun je de simulatie op je machine delen met iemand in
je tailnet. Zoek je Tailscale-adres op (`tailscale ip -4`, bijvoorbeeld `100.64.1.2`) en
start met dat adres in een sslip.io-naam:

```bash
bl sim start --domain 100-64-1-2.sslip.io
```

De andere opent dan http://100-64-1-2.sslip.io, en jijzelf ook. Geef in Tailscale alleen poort
80 van je machine vrij. Terug naar lokaal: `bl sim start --domain localhost`.

De simulatie gebruikt vaste wachtwoorden (`admin`/`admin`). Zet haar daarom niet zonder
eigen wachtwoord open op internet. Op een server met een eigen domein (DNS `demo.example.nl`
en `*.demo.example.nl` naar de server, poort 80 en 443 open):

```bash
bl sim start --domain demo.example.nl --https --password 'lang-wachtwoord'
```

De browser vraagt één keer om gebruiker `demo` en het wachtwoord; daarna zijn alle adressen
open. Met `--password` op `localhost` vraagt de browser het wachtwoord per adres.

Wil je dat iedereen mag rondkijken zonder iets te kunnen veranderen, start dan een kijkdemo:

```bash
bl sim start --domain demo.example.nl --https --view-only
```

Alleen `viewer`/`viewer` is dan openbaar. `admin` en `operator` krijgen een sterk wachtwoord,
dat `bl sim status` toont. De nieuwe connector, Mailpit, de vertrouwensdienst, DSS, de
logopslag en de beheerconsoles van Keycloak zijn niet bereikbaar, of alleen achter `--password`
als je dat erbij zet.

## Een eigen connector

Met `bl connector init` installeer je een connector in Kubernetes (met de Helm-chart
`connector`) of met Docker Compose. Hij stelt geen vragen, zodat ook een AI-agent hem kan
draaien:

```bash
bl connector init --target k8s \
  --authority https://authority.acc.buildinglinks.nl \
  --control-plane-host connector.example.nl \
  --data-plane-host data.connector.example.nl \
  --oidc-issuer https://keycloak.example.nl/realms/organisatie \
  --bundled-db \
  --namespace buildinglinks --ingress-class traefik \
  --out ./connector
```

| Optie | Betekenis |
|---|---|
| `--target` | `k8s` (Helm) of `compose` |
| `--authority` | het adres van de Trust Authority: haar DID, de dataspace en de omgeving komen daarvandaan (`/api/v1/self`) |
| `--environment` | `production`, `acceptance` of `development`; met `--authority` standaard die van de Trust Authority |
| `--control-plane-host`, `--data-plane-host` | de twee publieke hosts; de DID volgt uit de eerste. Ook een volledig adres mag; `http://` alleen in `development` |
| `--authority-did` | de DID van de Trust Authority, als je `--authority` niet geeft; haar adres volgt eruit |
| `--dataspace` | `BL_DATASPACE`, de dataspace zoals de Trust Authority hem noemt; met `--authority` komt hij daarvandaan, anders standaard `buildinglinks` |
| `--oidc-issuer` | de issuer-URL van je identity provider: `BL_OIDC_ISSUER` in de configuratie, en `keycloak-client.json` om in Keycloak te importeren. Dan heeft niemand de setup-code nodig. Met `--oidc-client-id` (standaard `connector`), `--oidc-client-secret` (standaard maakt `bl` er een) en `--oidc-internal-url` |
| `--database-url` en `--redis-url`, of `--bundled-db` | een eigen Postgres en Redis, of die van de chart of compose (voor staging en tests) |
| `--out` | de map voor sleutels en configuratie |
| `--namespace`, `--release`, `--ingress-class` | alleen bij `k8s`: namespace, naam van de Helm-release (standaard `connector`) en `ingress.className` |
| `--force` | maak alle sleutels opnieuw; een draaiende connector verliest dan zijn DID |

In `--out` komen:

- `geheimen/connector/`: `identity.jwk`, `dataplane.pub.jwk`, `did-update.jwk`,
  `did-update-next.jwk`, `data.key`, `session.key` en `did-next-key.hash`, de sleutelmap van de
  control plane;
- `geheimen/dataplane/`: `dataplane.jwk` en dezelfde `data.key`;
- `geheimen/database/`: de wachtwoorden van Postgres en Redis, met `--bundled-db`;
- `offline/did-update-after.jwk`: de update-sleutel na de volgende. **Bewaar hem offline, niet
  op de server**; alleen zijn hash staat in `did-next-key.hash`;
- bij `k8s`: `secrets.yaml` (de Secrets `connector-keys`, `dataplane-keys` en `connector-env`)
  en `values.yaml` met de minimale configuratie;
- bij `compose`: `compose.yaml`, `Caddyfile` en `.env`. In `.env` staan ook `BL_UID` en
  `BL_GID`, de eigenaar van `geheimen/`: connector en data plane draaien als die gebruiker en
  lezen de sleutels zonder `chown`;
- met `--oidc-issuer`: `geheimen/connector/oidc-client.secret` en `keycloak-client.json`, de
  client voor Keycloak (*Clients*, *Import client*) met de redirect-URI's van de connector en dat
  secret. De identity provider staat dan in `values.yaml` (`oidc`) en `secrets.yaml`, of in `.env`.

Bestaande sleutels laat hij staan; de configuratie schrijft hij elke keer opnieuw. De uitvoer
noemt elk bestand, de id van de datasleutel en de volgende stappen, zoals
`kubectl apply -f connector/secrets.yaml`,
`helm install connector oci://ghcr.io/buildinglinks/charts/connector --version <versie van bl> -f connector/values.yaml`,
de eerste login en `bl connector join`.

Daarna controleert `bl connector check https://connector.example.nl` de installatie: DNS en TLS
van beide hosts, de DID-documenten (de DID moet bij de host horen), DSP, de publieke data plane,
dat `/api/v1` en `/gateway` niet publiek zijn, de Trust Authority (met `--authority-did`) en
`/healthz`. Met `--dataspace` en `--environment` controleert hij ook dat de Trust Authority
lidmaatschap geeft van de dataspace die de connector eist: de naam, en buiten `production` met
de omgeving erachter (`buildinglinks-pilot-acceptance`). Ben je ingelogd met `bl login`, dan ook de effectieve configuratie, het lidmaatschap
en de koppeling met je identity provider. Elke controle die niet in orde is, noemt de volgende
stap (een commando waar dat kan), en het rapport eindigt met de eerste. Draai je de controle van
buiten je netwerk, geef dan `--extern`: een bereikbare `/api/v1` of `/gateway` is dan een fout in
plaats van een waarschuwing.

Lid worden doe je met `bl connector join`, na `bl login`:

```bash
bl connector join https://connector.example.nl \
  --org "Voorbeeld B.V." --kvk 12345678 \
  --role bms_provider \
  --contact "Naam <naam@example.nl>" \
  --accept-rulebook
```

Het stuurt de aanmelding naar de Trust Authority (zoals de wizard onder *Aanmelden*) en wacht
tot haar beheerder heeft goedgekeurd en het lidmaatschapsbewijs binnen is (standaard tien
minuten, `--wait`). `--contact` is de contactpersoon en de tekenbevoegde, als `"Naam <e-mail>"`
(een andere tekenbevoegde: `--signatory`); `--accept-rulebook` is diens aanvaarding van het
rulebook dat bij de Trust Authority geldt, nodig als zij toelaat zonder handtekening. `--role`
mag vaker; de rollen staan op `/api/v1/onboarding/roles` van de Trust Authority. Nog eens
aanroepen, ook zonder gegevens, wacht verder; is de connector al lid, dan zegt het dat.

## Scripts en AI-agents

`bl` stelt nooit vragen; alles gaat via opties. `bl sim status --json` geeft de toestand als
JSON: de adressen per deelnemer, de inloggegevens en `seed` (`not_started`, `running`, `done`
of `failed`). Exitcode 0 is gelukt, 1 een fout, 2 een ongeldige aanroep.

Elke connector heeft een management-API op `http://<deelnemer>.localhost/api/v1/...`. In de
simulatie kun je die aanroepen met `Authorization: Bearer dev-admin-<deelnemer>` (op een
kijkdemo staan de tokens in `bl sim status --json`), bijvoorbeeld:

```bash
curl -s -H "Authorization: Bearer dev-admin-optimaforma" http://optimaforma.localhost/api/v1/connections
```

Bij een echte connector log je eerst in met `bl login <url>`; daarna doet `bl api` elke
aanroep van de beheer-API met die sessie:

```bash
bl connector join https://connector.example.nl --org "Voorbeeld B.V." --kvk 12345678 --role bms_provider --contact "Naam <naam@example.nl>" --accept-rulebook
bl api GET /api/v1/self
bl api POST /api/v1/sources --data @systeem.json
bl api PUT /api/v1/settings/idp --data - < idp.json
```

`bl connector` en `bl api` volgen de afspraken voor agents: buiten een terminal is de uitvoer
JSON, en een fout staat als JSON op stderr (`code`, `message`, `hint`). De exitcodes:

| Exitcode | Betekenis |
|---|---|
| 0 | gelukt |
| 1 | overig |
| 2 | ongeldige aanroep |
| 3 | niet ingelogd of sessie verlopen |
| 4 | niet gevonden |
| 5 | conflict |
| 6 | geen rechten, of de Trust Authority wees de aanmelding af |
| 7 | connector onbereikbaar |
| 8 | een controle van `bl connector check` mislukt |

Zet `BL_AGENT=<naam>` (bijvoorbeeld `claude-code`), dan stuurt `bl` die naam mee in de kop
`BL-Agent`, zodat de audit-log een agent onderscheidt van een mens met hetzelfde profiel.

## Problemen

**Poort 80 is bezet.** Start op een andere poort: `bl sim start --port 8080`. De adressen
worden dan `http://sensortrust.localhost:8080` enzovoort.

**`sensortrust.localhost` opent niet.** Chrome, Firefox en Edge sturen `*.localhost` zelf naar je
eigen machine; niet elke browser doet dat. Gebruik een van die browsers, of start met een naam
die via DNS naar je machine wijst: `bl sim start --domain 127-0-0-1.sslip.io`.

**Werkt alles?** Bovenaan de startpagina (http://localhost) staat de status van elk onderdeel,
elke 15 seconden bijgewerkt: antwoordt het, en doet het zijn werk (vertrouwt DSS de
gesimuleerde vertrouwensdienst, zijn alle deelnemers aangemeld). `bl sim status` noemt de
services die volgens Docker niet gezond zijn.

**De start duurt lang of loopt vast.** Kijk wat de inrichting doet met `bl sim logs -f seed`.
Heeft Docker te weinig geheugen, dan stoppen containers onverwacht; zie `bl sim status` en
verhoog het geheugen in Docker Desktop. Mislukt de start, dan toont `bl` van elke service die
niet opkwam de laatste healthcheck en logregels.

**De images passen niet bij `bl`.** `bl` schrijft de configuratie, de images voeren haar uit.
Komen ze uit verschillende commits, dan kan een service wachten op iets wat zijn image nog
niet kan. `bl sim start` waarschuwt daarvoor. Bouw de images opnieuw (`bl sim start --build`,
in de repository), of kies de passende `--tag`.

**Images niet gevonden.** Elke versie van `bl` start de images van dezelfde versie. Kort na
een nieuwe release kunnen die er nog niet zijn; probeer het na een half uur opnieuw.

**De eerste minuut na een herstart mislukken dataverzoeken.** `bl sim start` meldt "klaar" zodra
de inrichting klaar is, maar de apps van de dataontvangers moeten hun verbindingen dan nog
opnieuw opzetten. Dat duurt ongeveer een minuut.

**Het sslip.io-adres opent niet.** Sommige routers blokkeren DNS-antwoorden met een privé-adres,
zoals een Tailscale-IP. Gebruik dan een eigen domein met een wildcard-record naar dat adres.

**De schijf loopt vol.** De database groeit met het verkeer: met `--traffic quiet` ongeveer
1 GB per dag, `normal` 3 à 4 GB en `busy` 15 à 20 GB. `bl sim reset` begint weer leeg.

**Opnieuw beginnen.** `bl sim reset` wist alles en richt de simulatie opnieuw in.

**Verbindingen worden geweigerd na een update van `bl`.** Versies na 0.1.1 geven de simulatie
een eigen dataspace (`buildinglinks-development`). Lidmaatschappen uit een simulatie van een
oudere versie gelden daar niet. Begin opnieuw met `bl sim reset`.

## Verwijderen

```bash
bl sim stop --purge                 # containers en data weg
rm ~/.local/bin/bl                  # de CLI
rm -rf ~/.local/share/bl            # de bestanden van de simulaties
```

De images verwijder je met `docker image prune -a` of in Docker Desktop.

## Licentie

`bl` en de images zijn © TNO, alle rechten voorbehouden. Dat ze openbaar te downloaden zijn,
geeft geen recht om ze te gebruiken, te kopiëren, te wijzigen of te verspreiden: daarvoor is
schriftelijke toestemming van TNO nodig, bijvoorbeeld in een pilotovereenkomst. De voorwaarden
staan in `LICENSE` in het archief van `bl` en in `/licenses/` in elk image, met de licenties van
onderdelen van derden in `THIRD_PARTY_NOTICES`.

# Buildinglinks CLI

`bl` draait de Buildinglinks-dataspace als complete simulatie op je eigen machine. Je krijgt een
Trust Authority en zes deelnemers, elk met een eigen connector, data plane, inlogomgeving (Keycloak-realm) en
waar dat past een gesimuleerd gebouwbeheersysteem of een gesimuleerde applicatie. De
deelnemers melden zich aan, publiceren gebouwdata, sluiten overeenkomsten en wisselen data
uit, zoals dat in de echte dataspace gaat. Je kunt in elke connector inloggen en zelf verder
spelen.

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

Open dan **http://localhost**. Je komt op de overzichtspagina van de Trust Authority, met
links naar elke deelnemer.

Inloggen kan in elke connector met *Inloggen met SSO*:

| Gebruiker | Wachtwoord | Mag |
|---|---|---|
| `admin` | `admin` | alles |
| `operator` | `operator` | alles behalve de instellingen |
| `viewer` | `viewer` | alleen kijken |

Elke deelnemer heeft een eigen adres, dus je kunt in één browser bij meerdere deelnemers
tegelijk ingelogd zijn.

## Wie er meedoet

| Deelnemer | Rol | Connector |
|---|---|---|
| Buildinglinks Trust Authority | aanmelding, register, monitoring, notaris | http://authority.localhost |
| Sensortrust | levert data van twee gebouwen (BMS) | http://sensortrust.localhost |
| Gebouwbeheer Noord | levert data van één gebouw (BMS) | http://noord.localhost |
| Optimaforma | klimaatoptimalisatie, schrijft setpoints | http://optimaforma.localhost |
| Facilitair Inzicht | comfortmonitoring, alleen lezen | http://facilitair.localhost |
| RealEstator | eigenaar van een gebouw waarvan Sensortrust het BMS beheert; Sensortrust levert namens RealEstator | http://realestator.localhost |
| Nieuwe connector | vers geïnstalleerd, nog niet aangemeld | http://nieuwkomer.localhost |

De gesimuleerde systemen achter de deelnemers hebben een eigen dashboard op
`http://<deelnemer>-backend.localhost`, bijvoorbeeld http://optimaforma-backend.localhost. Verder zijn
er Mailpit voor de e-mail van de Trust Authority (http://mail.localhost), een gesimuleerde
vertrouwensdienst voor elektronisch ondertekenen (http://qtsp.localhost) en de Digital
Signature Service van de Europese Commissie (http://dss.localhost/validation).

## Wat je kunt proberen

* **Kijk mee met de applicaties.** Het dashboard van Optimaforma (http://optimaforma-backend.localhost) leest
  via de dataspace meetwaarden uit de gebouwen van Sensortrust en stuurt setpoints terug. Het
  BMS-dashboard van Sensortrust (http://sensortrust-backend.localhost) toont wat er binnenkomt.
* **Verbind zelf een dataset.** Log in bij Facilitair Inzicht → *Dataspace verkennen* → Sensortrust →
  *Kantoor Weena Rotterdam* → *Verbinden*, met het aanbod "Alle leden mogen lezen". Binnen
  enkele seconden is de verbinding actief en gebruikt de app van Facilitair de nieuwe bron.
* **Zie het beleid werken.** Probeer vanuit Facilitair Inzicht het aanbod "Klimaatregeling" te
  nemen. Sensortrust weigert: dat aanbod is alleen voor klimaatoptimalisatoren. De reden staat bij
  de onderhandeling.
* **Bekijk het bewijs.** Elke connector legt onder *Bewijsvoering* elke stap vast, met
  handtekeningen van beide partijen. Download een bewijspakket en controleer het zelf:
  `bl evidence verify bewijspakket.json`.
* **Meld een nieuwe deelnemer aan.** Open http://nieuwkomer.localhost en log in met setup-code
  `SETUP-NIEUWKOMER`. Vul de aanmelding in, onderteken de e-mail in Mailpit bij de gesimuleerde
  vertrouwensdienst en keur de aanvraag goed in de Trust Authority onder *Onboarding*. Wat je
  nodig hebt om de Keycloak van de nieuwkomer te koppelen, toont `bl sim status`. Opnieuw
  beginnen: `bl sim reset --participant nieuwkomer`.
* **Schors een deelnemer.** In de Trust Authority → *Deelnemers* → Facilitair Inzicht →
  *Schorsen*. Binnen een minuut stoppen de providers hun leveringen aan Facilitair.

## Commando's

| Commando | Wat het doet |
|---|---|
| `bl sim start` | start de simulatie en richt haar in |
| `bl sim status` | wat er draait, de adressen en de inloggegevens |
| `bl sim stop` | stopt; de data blijft bewaard en `start` gaat verder waar je was |
| `bl sim stop --purge` | stopt en wist alle data |
| `bl sim reset` | alles wissen en opnieuw beginnen |
| `bl sim reset --participant nieuwkomer` | alleen de nieuwe connector terug naar een verse installatie |
| `bl sim logs [-f] [service …]` | logs, bijvoorbeeld `bl sim logs -f sensortrust-cp` |
| `bl evidence verify <pakket.json>` | een bewijspakket offline controleren |

Opties van `bl sim start`:

| Optie | Standaard | Betekenis |
|---|---|---|
| `--domain` | `localhost` | domein voor alle adressen: `sensortrust.<domein>`, `authority.<domein>`, … |
| `--port` | `80` | poort waarop alles bereikbaar is |
| `--project` | `bl-sim` | naam van de simulatie; zo draai je er meerdere naast elkaar |
| `--tag` | de versie van `bl` | versie van de images |
| `--password` | geen | één wachtwoord voor de hele simulatie (gebruiker `demo`) |

`bl` onthoudt de opties van de vorige keer: een tweede `bl sim start` gebruikt hetzelfde
domein en dezelfde poort. Alle opties staan in `bl --help` en `bl sim <commando> --help`.

## De simulatie aan iemand anders laten zien

Met [Tailscale](https://tailscale.com) kun je de simulatie op je machine delen met iemand in
je tailnet. Zoek je Tailscale-adres op (`tailscale ip -4`, bijvoorbeeld `100.88.15.58`) en
start met dat adres in een sslip.io-naam:

```bash
bl sim start --domain 100-88-15-58.sslip.io
```

De andere opent dan http://100-88-15-58.sslip.io, en jijzelf ook. Geef in Tailscale alleen poort
80 van je machine vrij. Terug naar lokaal: `bl sim start --domain localhost`.

De simulatie gebruikt vaste wachtwoorden (`admin`/`admin`). Zet haar daarom niet zonder
eigen wachtwoord open op internet. Op een server met een eigen domein (DNS `demo.example.nl`
en `*.demo.example.nl` naar de server, poort 80 en 443 open):

```bash
bl sim start --domain demo.example.nl --https --password 'lang-wachtwoord'
```

De browser vraagt één keer om gebruiker `demo` en het wachtwoord; daarna zijn alle adressen
open.

## Scripts en AI-agents

`bl` stelt nooit vragen; alles gaat via opties. `bl sim status --json` geeft de toestand als
JSON: de adressen per deelnemer, de inloggegevens en `seed` (`not_started`, `running`, `done`
of `failed`). Exitcode 0 is gelukt, 1 een fout, 2 een ongeldige aanroep.

Elke connector heeft een management-API op `http://<deelnemer>.localhost/api/v1/...`. In de
simulatie kun je die aanroepen met `Authorization: Bearer dev-admin-<deelnemer>`, bijvoorbeeld:

```bash
curl -s -H "Authorization: Bearer dev-admin-optimaforma" http://optimaforma.localhost/api/v1/connections
```

## Problemen

**Poort 80 is bezet.** Start op een andere poort: `bl sim start --port 8080`. De adressen
worden dan `http://sensortrust.localhost:8080` enzovoort.

**`sensortrust.localhost` opent niet.** Chrome, Firefox en Edge sturen `*.localhost` zelf naar je
eigen machine; niet elke browser doet dat. Gebruik een van die browsers, of start met een naam
die via DNS naar je machine wijst: `bl sim start --domain 127-0-0-1.sslip.io`.

**De start duurt lang of loopt vast.** Kijk wat de inrichting doet met `bl sim logs -f seed`.
Heeft Docker te weinig geheugen, dan stoppen containers onverwacht; zie `bl sim status` en
verhoog het geheugen in Docker Desktop.

**Images niet gevonden.** Elke versie van `bl` start de images van dezelfde versie. Kort na
een nieuwe release kunnen die er nog niet zijn; probeer het na een half uur opnieuw.

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

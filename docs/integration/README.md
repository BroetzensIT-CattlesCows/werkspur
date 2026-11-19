<!--
SPDX-FileCopyrightText: 2026 Werkspur-Mitwirkende
SPDX-License-Identifier: CC-BY-4.0
-->

# Schnittstellen

Werkspur hat drei Schnittstellen. Alle laufen über dieselbe Adresse wie die Weboberfläche
(`https://<host>`, im Folgenden `ihre-adresse.de`) und gelten mit den Rechten der Person bzw. des
Dienstzugangs, der sich anmeldet. Mandantengrenzen gelten überall gleich: Daten fremder
Organisationen beantwortet Werkspur so, als gäbe es sie nicht.

| Schnittstelle | Pfad | Anmeldung | Zweck |
| --- | --- | --- | --- |
| [REST-API](#rest-api) | `/api/v1/` | persönliches API-Token (`Bearer`) | lesen und schreiben, Skripte, n8n |
| [MCP-Server](#mcp-server-ki-assistenten) | `/mcp` | persönliches API-Token (`Bearer`) | KI-Assistenten (Claude Code, Claude Desktop) |
| [OGC API – Features](#kartendienst-ogc-api--features) | `/ogc/<kennung>/` | Dienstzugang (HTTP Basic) oder Open Data | QGIS, ArcGIS, nur lesend |
| [Austauschprofile](#austauschprofile) | Import/Export, `/api/v1/austauschprofile` | wie REST | fremde Kataster (CSV, GeoJSON, GeoPackage) |

Hinter einem Reverse Proxy mit Basic Auth brauchen `/mcp` und `/ogc/` eigene Locations ohne
Basic Auth: siehe README im Betriebspaket (`werkspur.tar.gz`), Abschnitt „Hinter Nginx Proxy Manager“.

## API-Token

Weboberfläche → „Verwaltung“ → Reiter „API-Tokens“ → Token anlegen (Name z. B. „n8n“ oder
„KI-Assistent“). Der Wert (`ws_…`) erscheint genau einmal. Widerruf im selben Reiter; die nächste
Anfrage endet dann mit 401. Das Token gehört nur in die Konfiguration des Clients, nie in ein
Repository. `werkspur_einrichten` gibt für das erste Konto ebenfalls ein Token aus.

## REST-API

- Beschreibung: OpenAPI 3 unter `https://ihre-adresse.de/api/v1/openapi.json`, lesbar unter
  `https://ihre-adresse.de/api/v1/docs`. Die Weboberfläche nutzt selbst nur diese API.
- Versioniert unter `/api/v1/`. Geodaten als GeoJSON mit `bbox` und Seitenlinks, Fehler als
  Problem Details nach RFC 9457 (`application/problem+json`).
- Gesundheitsprüfung ohne Anmeldung: `GET /api/v1/gesundheit`.

```bash
curl -H "Authorization: Bearer $TOKEN" https://ihre-adresse.de/api/v1/auftraege
```

Import und Export laufen als Jobs (Upload, Prüfung, Zuordnung, Übernahme). Formate: GeoJSON,
GeoPackage, CSV, Shapefile (ZIP), KML, GML, FlatGeobuf; Importe bis 200 MB, Rückgaben aus QField
bis 2 GB.

## MCP-Server (KI-Assistenten)

Ein KI-Assistent fragt Werkspur-Daten mit den Rechten der angemeldeten Person ab und kann
Auftragsentwürfe anlegen. **Freigeben kann nur ein Mensch im Portal.** Transport: Streamable HTTP
unter `/mcp`, Anmeldung mit dem persönlichen API-Token (kein OAuth; Clients, die nur OAuth
sprechen, z. B. Connectoren in Claude Web, gehen nicht).

Einrichtung: In der Verwaltung zeigt der Block „KI-Assistent verbinden“ Adresse und Befehl.

```bash
# Claude Code
claude mcp add --transport http werkspur https://ihre-adresse.de/mcp \
  --header "Authorization: Bearer $TOKEN"

# Claude Desktop und andere Clients ohne Header-Unterstützung
npx mcp-remote https://ihre-adresse.de/mcp --header "Authorization: Bearer ${WERKSPUR_TOKEN}"
```

| Tool | liefert |
| --- | --- |
| `auftraege_suchen` | Aufträge mit Status, Frist und Portal-Link (Filter: Status, Fachschale, Gebiet, fällig bis) |
| `objekte_im_gebiet` | Zusammenfassung und Objektliste eines Gebiets oder einer bbox |
| `ueberfaellige_kontrollen` | überfällige Objekte, sofort fällige zuerst |
| `nachweis_abrufen` | Rückmeldungen und Stand zu Auftrag oder Objekt, Link auf den PDF-Nachweis |
| `maengel_auswerten` | offene, beauftragte und erledigt gemeldete Mängel nach Art, Frist und Objekt |
| `auftrag_entwerfen` | nur einen Entwurf (Kontrolle oder Maßnahme aus Mängeln), Label „KI-Entwurf“ im Portal |

Resources: je Fachschalenversion JSON Schema und Beschreibung
(`werkspur://fachschalen/<kennung>@<version>/schema`, `…/beschreibung`), Kurzdokus
`werkspur://doku/datenmodell` und `werkspur://doku/status`, Liste `werkspur://austauschprofile`.

Spielregeln:

- Höchstens 200 Treffer je Antwort; die Zusammenfassung zählt alle.
- Dienstleister sehen nur Objekte aktiver Aufträge des eigenen Betriebs und dürfen nichts
  entwerfen.
- Nicht über MCP: freigeben, verwerfen, Pakete, Rückgaben, Konflikte, Mängel schließen, Abnahme,
  importieren, löschen.
- `nachweis_abrufen` höchstens 6-mal pro Minute und Nutzer; Fehlversuche bei der Anmeldung
  30 je Minute und IP (danach 429).
- Jeder Aufruf steht im Protokoll (Betreiber: Admin → „MCP-Aufrufe“), Parameter nur bereinigt,
  nie Freitexte oder Namen.

Betrieb: Dienst `mcp` (ein Prozess, Port 8001 nur intern). Neue Fachschalen-Versionen erscheinen
nach `docker compose restart mcp`. Fällt `mcp` aus, antwortet nur `/mcp` mit 502. Hinter Nginx
Proxy Manager braucht `/mcp` `proxy_buffering off`, sonst kommen Antworten verspätet an.

## Kartendienst (OGC API – Features)

Das GIS der Kommune liest den aktuellen Kontrollstand live, nur lesend. OGC API – Features Teil 1
(Core, GeoJSON, OpenAPI 3.0) und Teil 2 (Koordinatensysteme); QGIS ab 3.16, ArcGIS Pro, Online und
Enterprise ab 10.9. Der Dienst hat den OGC-Konformitätstest (Team Engine, „ogcapi-features-1.0“)
bestanden.

**Adresse:** `https://ihre-adresse.de/ogc/<kennung>/`, `<kennung>` ist die Kennung der FM (z. B.
`stadt-musterhausen`); die fertige Adresse steht in der Verwaltung unter „Kartendienst“. Eine
geänderte Kennung macht gespeicherte Adressen ungültig (danach 401).

**Anmeldung:** HTTP Basic mit einem **Dienstzugang** (Verwaltung → „Kartendienst“ → „Zugang
anlegen“; Benutzername = Name des Zugangs, Passwort = angezeigtes Token, erscheint einmal). Ein
Dienstzugang ist kein Nutzerkonto, gilt nur für den Kartendienst seiner FM und lässt sich sofort
widerrufen. Ohne Anmeldung sind nur Collections lesbar, für die **Open Data** eingeschaltet ist
(Standard aus; nur Stammdaten, Mängel nie).

| Pfad | Inhalt |
| --- | --- |
| `/ogc/<kennung>/` | Landing Page |
| `…/conformance`, `…/api` | Konformitätsklassen, OpenAPI |
| `…/collections` | je Fachschale und Objekttyp eine Collection (z. B. `baumkontrolle-fll.baum`), dazu `gebiete` und `maengel` |
| `…/collections/<id>/items` | Features (GeoJSON), blätterbar mit `limit` (Standard 100, höchstens 1000) und dem `next`-Link |

Parameter: `bbox`, `crs` (CRS84, EPSG 4326, 25832, 25833, 3857), `datetime` (letzte Kontrolle),
Filterfelder wie `gesperrt`, `faelligkeit_<prüfart>`, `ampel`, `status`. Fehler im OGC-Format
`{"code": …, "description": …}`. Nie enthalten: Personen, Fotos, Freitexte aus Rückmeldungen.
CORS ist für `/ogc/` freigeschaltet (Browser-Clients wie ArcGIS Online). Ratenbegrenzung: 600
Anfragen pro Minute je Dienstzugang, 120 je IP ohne Anmeldung.

**QGIS:** Datenquellenverwaltung (Strg+L) → „WFS / OGC API – Features“ → „Neu“ → URL wie oben,
Version „OGC API – Features“ → Authentifizierung „Basic“ mit Name und Token des Dienstzugangs →
„Verbinden“, Collections hinzufügen.

**ArcGIS Pro:** „Einfügen“ → „Verbindungen“ → „Server“ → „Neuer OGC API Server“, URL,
Benutzername und Token des Dienstzugangs.

## Austauschprofile

Ein Austauschprofil (YAML) ordnet die Spalten eines fremden Katasters (z. B. Open-Data-
Baumkataster) den Feldern einer Fachschale zu. Beim Import schlägt Werkspur passende Profile vor
(ab 60 % passender Spalten) und belegt Fachschale, Koordinatensystem und Zuordnung vor; beim
Export heißen Spalten und Werte wie im fremden Format („Profil (optional)“, „Nur Profilfelder“).
Was mit einem Profil exportiert wird, lässt sich mit demselben Profil verlustfrei wieder
importieren.

Mitgeliefert: `opendata-berlin-baumbestand@1` (Geoportal Berlin, EPSG:25833) und
`opendata-dortmund-baumkataster@1` (Open-Data-Portal Dortmund, EPSG:4326). Eigene Profile:
Verwaltung → „Austauschprofile“ → „Eigenes Profil bereitstellen“, oder per API
`POST /api/v1/austauschprofile`. Profile sind nach dem Hochladen unveränderbar; Änderungen als
neue `version`.

```yaml
kennung: opendata-baumkataster-beispiel
version: 1
titel: Open-Data-Baumkataster (Beispielstadt)
fachschale: baumkontrolle-fll
objekttyp: baum
quelle:
  koordinatensystem: "EPSG:25832"
  csv_trenner: ";"
  kennung_feld: BAUMNR
felder:
  baumnummer:        { quelle: BAUMNR }
  baumart_botanisch: { quelle: ART_BOT }
  entwicklungsphase:
    quelle: ALTERSPHASE
    werte: { J: jugendphase, R: reifephase, A: alterungsphase }
  verkehrserwartung:
    standard: mittel
  letzte_kontrolle_import:
    quelle: LETZTE_KONTR
    datumsformat: "%d.%m.%Y"
```

Je Feld: `quelle` (Spalte der Fremddatei) oder `standard` (fester Wert), dazu `werte` (nur
Auswahlfelder) und `datumsformat` (nur Datumsfelder). Fehler beim Hochladen nennen den Pfad, z. B.
`felder.pflanzjahr.quelle: …`.

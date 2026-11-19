# Werkspur – Website, Releases, Doku und Issues

Öffentliches Repository von Werkspur, der offenen Plattform für Verkehrssicherungskontrollen
(Bäume nach FLL, Spielplätze nach DIN EN 1176). Der Code liegt in einem privaten Repository; hier
liegt alles, was Betreiber und Interessierte brauchen:

| Was | Wo |
| --- | --- |
| Produkt-Website | `index.html` (GitHub Pages) |
| Betriebspaket zum Selbstbetrieb | [Releases](https://github.com/BroetzensIT-CattlesCows/werkspur/releases), Asset `werkspur.tar.gz` |
| Doku | [`docs/`](docs/): [Betrieb und Update](docs/betrieb-und-update.md), [Schnittstellen](docs/integration/README.md) |
| Fehler und Wünsche | [Issues](https://github.com/BroetzensIT-CattlesCows/werkspur/issues) |

Das Betriebspaket (`compose.yaml`, `.env.example`, `db/init/`, Paket-README) liegt bewusst nicht lose
im Repository, sondern nur als Release-Asset.

## Selbst betreiben (Kurzanleitung)

Voraussetzung: ein Server mit Docker und Docker Compose, etwa 6 GB freier Platz.

```bash
curl -LO https://github.com/BroetzensIT-CattlesCows/werkspur/releases/latest/download/werkspur.tar.gz
tar xzf werkspur.tar.gz && cd werkspur-*
cp .env.example .env
openssl rand -base64 48   # als WERKSPUR_SECRET_KEY eintragen
openssl rand -hex 24      # je einmal für die drei Datenbank-Passwörter
docker network create docker-apps_proxy
docker compose up -d
docker compose exec app python manage.py werkspur_einrichten \
  --fm-name "Stadt Musterhausen" --email it@musterhausen.de
```

Alles Weitere (Proxy, Update, Sicherung, Schnittstellen) steht im Paket-README und in
[docs/](docs/). Update: neue Version in `.env` eintragen (`WERKSPUR_VERSION`), dann
`docker compose pull && docker compose up -d`.

## Website

```
index.html            Seite (CSS eingebettet, Logo als Inline-SVG)
fonts/                IBM Plex Sans 400/600, IBM Plex Mono 400 (WOFF2, OFL, siehe fonts/OFL.txt)
bilder/               Bildschirmfotos aus der Testumgebung mit Demodaten (WebP, je 1440 und 720 px)
favicon.svg, favicon-32.png, apple-touch-icon.png   aus dem Werkspur-Logo-Set
.nojekyll             GitHub Pages liefert die Dateien unverändert aus, kein Jekyll
```

Keine Fremd-CDNs, keine Cookies, kein Tracking. Alle Pfade in `index.html` sind relativ.

### Veröffentlichen (GitHub Pages)

*Settings → Pages*: Quelle „Deploy from a branch“, Branch `main`, Ordner `/ (root)`. Pages
liefert `index.html` unter `https://broetzensit-cattlescows.github.io/werkspur/` aus; `docs/*.md`
stören nicht (`.nojekyll` schaltet Jekyll ab, die Markdown-Dateien bleiben nur im Repo lesbar).
Kurzlink `l.broetzens.de/werkspur` im Kurzlink-Dienst darauf zeigen lassen. Vor dem Veröffentlichen
sollte es mindestens einen Release geben, sonst laufen „Releases“ und der `curl`-Befehl ins Leere.

- **og:image** ist relativ angegeben; manche Dienste verlangen eine absolute URL. Nach dem
  Veröffentlichen ggf. auf die volle Pages-Adresse setzen.

## Bildschirmfotos erneuern

Die Fotos entstehen im Ende-zu-Ende-Stack von Werkspur (Compose-Projekt `werkspur-e2e`, Demodaten
aus `manage.py demodaten_laden`, Playwright mit `locale de-DE`, `timezoneId Europe/Berlin`).
Nur feste Ausschnitte des Inhaltsbereichs, ohne Kopfzeile mit Kontonamen. Danach den Stack mit
`docker compose -p werkspur-e2e down -v` entfernen.

## Lizenzen

- Website (Texte, Bildschirmfotos, Seitencode) und Dokumentation in `docs/`: CC BY 4.0, siehe `LICENSE`.
- Werkspur-Logo, Wortmarke und App-Icon (Inline-SVG in `index.html`, `favicon.svg`,
  `favicon-32.png`, `apple-touch-icon.png`): Marke von Broetzens IT, alle Rechte vorbehalten;
  siehe `TRADEMARKS.md` im Code-Repository von Werkspur.
- Schriften: IBM Plex unter SIL Open Font License 1.1 (`fonts/OFL.txt`).
- Betriebspaket (Release-Asset `werkspur.tar.gz`): EUPL-1.2, Lizenztext im Archiv (`LICENSE`).

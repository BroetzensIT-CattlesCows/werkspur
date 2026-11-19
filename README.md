# Werkspur – Produkt-Website

Statische Seite für Werkspur, die offene
Plattform für Verkehrssicherungskontrollen (Bäume nach FLL, Spielplätze nach DIN EN 1176).
Eine HTML-Datei mit eingebettetem CSS, Schriften lokal, keine Fremd-CDNs, keine Cookies, kein Tracking.

```
index.html            Seite (CSS eingebettet, Logo als Inline-SVG)
fonts/                IBM Plex Sans 400/600, IBM Plex Mono 400 (WOFF2, OFL, siehe fonts/OFL.txt)
bilder/               Bildschirmfotos aus der Testumgebung mit Demodaten (WebP, je 1440 und 720 px)
favicon.svg, favicon-32.png, apple-touch-icon.png   aus dem Werkspur-Logo-Set (design/assets/app-icon)
```

## Repo-Struktur

| Repository | sichtbar | Inhalt |
| --- | --- | --- |
| `BroetzensIT-CattlesCows/werkspur` (dieses) | öffentlich | nur diese Website, ausgeliefert über GitHub Pages |
| [`BroetzensIT-CattlesCows/werkspur-release`](https://github.com/BroetzensIT-CattlesCows/werkspur-release) | öffentlich | Betriebspaket (`compose.yaml`, `.env.example`, `db/init/`), Betriebs- und Schnittstellen-Doku, Releases, Issues |
| Code-Repository | privat | Code; baut bei einem Tag `v*` die Images und legt den Release in `werkspur-release` an |

Die Images liegen öffentlich unter `ghcr.io/broetzensit-cattlescows/` (`werkspur-app`,
`werkspur-qgis`, `werkspur-caddy`). Jeder Release in `werkspur-release` trägt das Paket unter dem
festen Namen `werkspur.tar.gz`; der Block „Selbst betreiben“ lädt es über
`releases/latest/download/werkspur.tar.gz`. Alle Links der Seite (Releases, Issues, Anleitung,
Schnittstellen-Doku) zeigen auf `werkspur-release`.

## Veröffentlichen (GitHub Pages)

1. Auf GitHub unter der Organisation `BroetzensIT-CattlesCows` das öffentliche Repository
   `werkspur` anlegen (ohne README, damit der erste Push sauber durchgeht).
2. Lokal den Remote setzen und pushen:
   ```bash
   git remote add origin git@github.com:BroetzensIT-CattlesCows/werkspur.git
   git push -u origin main
   ```
3. Im Repository unter *Settings → Pages* als Quelle „Deploy from a branch“, Branch `main`,
   Ordner `/ (root)` wählen. Die Seite erscheint danach unter
   `https://broetzensit-cattlescows.github.io/werkspur/`.
4. Kurzlink `l.broetzens.de/werkspur` im Kurzlink-Dienst (Kutt) auf diese Adresse zeigen lassen.

Alle Pfade in `index.html` sind relativ; die Seite läuft also auch unter einem Unterpfad oder
einer eigenen Domain (dann `CNAME`-Datei ergänzen).

Vor dem Veröffentlichen sollte `werkspur-release` mindestens einen Release haben, sonst laufen
„Releases“ und der `curl`-Befehl ins Leere.

## Offene Punkte

- **og:image** ist relativ angegeben; manche Dienste verlangen eine absolute URL. Nach dem
  Veröffentlichen ggf. auf die volle Pages-Adresse setzen.

## Bildschirmfotos erneuern

Die Fotos entstehen im Ende-zu-Ende-Stack von Werkspur (Compose-Projekt `werkspur-e2e`, Demodaten
aus `manage.py demodaten_laden`, Playwright mit `locale de-DE`, `timezoneId Europe/Berlin`).
Nur feste Ausschnitte des Inhaltsbereichs, ohne Kopfzeile mit Kontonamen. Danach den Stack mit
`docker compose -p werkspur-e2e down -v` entfernen.

## Lizenz

- Texte, Bildschirmfotos und Seitencode: CC BY 4.0 (wie die Werkspur-Dokumentation), siehe `LICENSE`.
- Werkspur-Logo, Wortmarke und App-Icon (Inline-SVG in `index.html`, `favicon.svg`,
  `favicon-32.png`, `apple-touch-icon.png`): Marke von Broetzens IT, alle Rechte vorbehalten;
  siehe `TRADEMARKS.md` im Code-Repository von Werkspur.
- Schriften: IBM Plex unter SIL Open Font License 1.1 (`fonts/OFL.txt`).

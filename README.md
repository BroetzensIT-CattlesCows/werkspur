# Werkspur – Produkt-Website

Statische Seite für [Werkspur](https://github.com/BroetzensIT-CattlesCows/werkspur), die offene
Plattform für Verkehrssicherungskontrollen (Bäume nach FLL, Spielplätze nach DIN EN 1176).
Eine HTML-Datei mit eingebettetem CSS, Schriften lokal, keine Fremd-CDNs, keine Cookies, kein Tracking.

```
index.html            Seite (CSS eingebettet, Logo als Inline-SVG)
fonts/                IBM Plex Sans 400/600, IBM Plex Mono 400 (WOFF2, OFL, siehe fonts/OFL.txt)
bilder/               Bildschirmfotos aus der Testumgebung mit Demodaten (WebP, je 1440 und 720 px)
favicon.svg, favicon-32.png, apple-touch-icon.png   aus dem Werkspur-Logo-Set (design/assets/app-icon)
```

## Veröffentlichen (GitHub Pages)

1. Auf GitHub unter der Organisation `BroetzensIT-CattlesCows` ein öffentliches Repository anlegen,
   z. B. `werkspur-website` (ohne README, damit der erste Push sauber durchgeht).
2. Lokal den Remote setzen und pushen:
   ```bash
   git remote add origin git@github.com:BroetzensIT-CattlesCows/werkspur-website.git
   git push -u origin main
   ```
3. Im Repository unter *Settings → Pages* als Quelle „Deploy from a branch“, Branch `main`,
   Ordner `/ (root)` wählen. Die Seite erscheint danach unter
   `https://broetzensit-cattlescows.github.io/werkspur-website/`.
4. Kurzlink `l.broetzens.de/werkspur` im Kurzlink-Dienst (Kutt) auf diese Adresse zeigen lassen.

Alle Pfade in `index.html` sind relativ; die Seite läuft also auch unter einem Unterpfad oder
einer eigenen Domain (dann `CNAME`-Datei ergänzen).

## Platzhalter und offene Punkte

- **Repo-Links:** Code, Releases und Issues zeigen vorläufig auf
  `https://github.com/BroetzensIT-CattlesCows/werkspur`. Dieses Repository gibt es noch nicht;
  vor dem Veröffentlichen anlegen oder die Links in `index.html` anpassen (Suchen nach
  `BroetzensIT-CattlesCows/werkspur`). Der Installationsblock unter „Selbst betreiben“ nutzt
  `git clone` dieses Repositorys.
- **„In Kürze“:** Austauschprofile und OGC API Features (Teilprojekt 9) sind als „In Kürze“
  gekennzeichnet. Nach Fertigstellung die Marken in „Neu“ ändern.
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
  siehe `TRADEMARKS.md` im Werkspur-Repository.
- Schriften: IBM Plex unter SIL Open Font License 1.1 (`fonts/OFL.txt`).

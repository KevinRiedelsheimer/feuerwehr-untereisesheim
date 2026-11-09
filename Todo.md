# Todo

## Vor dem Go-Live — muss erledigt sein

- [ ] **E-Mail-Adresse prüfen** — In [src/impressum.html](src/impressum.html) (Zeile 13) und [src/datenschutz.html](src/datenschutz.html) (Zeile 52) ist `info@feuerwehr-untereisesheim.de` als Platzhalter — bitte echte Adresse eintragen.

- [ ] **Placeholder-Blogposts löschen** — Die 4 englischsprachigen Testbeiträge aus dem Starter-Template erscheinen aktuell auf der Live-Seite:
  - [src/posts/my-first-post.md](src/posts/my-first-post.md)
  - [src/posts/second-post.md](src/posts/second-post.md)
  - [src/posts/my-third-big-post.md](src/posts/my-third-big-post.md)
  - [src/posts/this-is-latest-post.md](src/posts/this-is-latest-post.md)

- [ ] **Mannschaft eintragen** — [src/_data/mannschaft.yaml](src/_data/mannschaft.yaml) enthält nur „Vorname Nachname" Platzhalter — echte Namen und Funktionen eintragen.

- [ ] **VHUG Zugangsdaten einholen** — SSH-Benutzername, Server-Hostname, Webroot-Pfad in `.env` eintragen (Vorlage: `.env.example`). Danach: `npm run deploy`.

## Nach dem Go-Live

- [ ] **Fahrzeugdaten prüfen** — [src/_data/vehicles.yaml](src/_data/vehicles.yaml) enthält Fahrzeugdaten — bitte auf Korrektheit prüfen (Baujahre, Bezeichnungen).

- [ ] **Erster echter Einsatzbericht** — Placeholder-Posts löschen und ersten echten Beitrag unter [src/posts/](src/posts/) anlegen.
